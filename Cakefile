# cakefile to build project writeups and site
{exec} = require 'child_process'
fs = require 'fs-extra'
path = require 'path'
jade = require 'jade'
stylus = require 'stylus'

option '-c', '--config [FILE]', 'Configuration File, defaults to \'config.json\''

getConfig = (options) ->
	configFile = options.config ? 'config.json'
	
	if fs.existsSync configFile
		try
			config = fs.readJSONFileSync configFile
		catch err
			console.log "Error reading #{configFile}; skipping: #{err}"
			console.log ''
			config = null
	else
		config = null

	effectiveConfig = 
		projectSource: options.projectSource ? config?.projectSource ? './source'
		webSource: options.webSource ? config?.webSource ? './site'
		output: options.output ? config?.output ? './generated'
		key: options.key ? config?.key
		secret: options.secret ? config?.secret
		bucket: options.bucket ? config?.bucket
		path: options.path ? config?.path ? '/'

walkDirs = (sourceDir, destDir, action) ->
	for entry in fs.readdirSync sourceDir
		stat = fs.statSync path.join sourceDir, entry
		if stat.isDirectory()
			walkDirs path.join(sourceDir, entry), path.join(destDir, entry), action
		else if stat.isFile()
			fs.mkdirsSync destDir
			action sourceDir, destDir, entry

buildSiteDir = (sourceDir, destDir) ->
	fs.mkdirsSync destDir if not fs.existsSync destDir
	
	for entry in fs.readdirSync sourceDir
		stat = fs.statSync path.join sourceDir, entry
		if stat.isDirectory()
			buildSiteDir path.join(sourceDir, entry), path.join(destDir, entry)
		else if stat.isFile()
			buildSiteFile sourceDir, entry, destDir

buildSiteFile = (sourceDir, sourceFile, destDir) ->
	switch path.extname sourceFile
		when '.jade'
			outFile = path.join destDir, sourceFile.replace /\.jade$/, '.html'
			console.log "Building #{outFile} [jade]..."
			
			try
				html = jade.compile(fs.readFileSync(path.join(sourceDir, sourceFile), 'utf8'), { pretty: false, filename: path.join(sourceDir, sourceFile) })()
				fs.writeFileSync outFile, html, 'utf8'
			catch err
				console.log "Error compiling #{outFile}: #{err}"
		when '.styl'
			outFile = path.join destDir, sourceFile.replace /\.styl$/, '.css'
			console.log "Building #{outFile} [stylus]..."
			
			try
				css = stylus(fs.readFileSync(path.join(sourceDir, sourceFile), 'utf8'))
					.set('filename', sourceFile)
					.set('compress', true)
					#.use(nib())
					.render (err, css) ->
						throw err if err?
						fs.writeFileSync outFile, css, 'utf8'
			catch err
				console.log "Error compiling #{outFile}: #{err}"
		when '.coffee'
			outFile = path.join destDir, sourceFile.replace /\.coffee$/, '.js'
			console.log "Building #{outFile} [coffee]..."
			
			exec "coffee -cp #{path.join sourceDir, sourceFile}", (err, stdout, stderr) ->
				console.log "Error compiling #{outFile}: #{err}" if err?
				console.log "Error compiling #{outFile}: #{stderr}" if stderr?.trim().length > 0
				
				if stdout?
					# it worked
					fs.writeFileSync outFile, stdout, 'utf8'
		else
			console.log "Skipping #{sourceFile}"

deployDir = (knox, localDir, destDir) ->
	for entry in fs.readdirSync localDir
		stat = fs.statSync path.join localDir, entry
		if stat.isDirectory()
			deployDir knox, path.join(localDir, entry), path.join(destDir, entry)
		else if stat.isFile()
			do ->
				file = path.join(localDir, entry)
				remoteFile = path.join(destDir, entry)
				knox.putFile file, remoteFile, { 'x-amz-acl': 'public-read', 'Cache-Control': 'no-cache' }, (err, res) ->
					console.log "Error uploading #{remoteFile}: #{err}" if err?
					if res.statusCode is 200
						console.log "Uploaded #{remoteFile}."
					else
						console.log "Error uploading #{remoteFile}. [#{res.statusCode}]"

sources = [
	{ source: 'html', destination: '.' }
	{ source: 'css', destination: 'css' }
	{ source: 'js', destination: 'js' }
]

task 'generate:pdf', 'generate PDF files', (options) ->
	config = getConfig options

	walkDirs config.projectSource, "#{config.output}/pdf", (sourceDir, destDir, file) ->
		destFile = file.replace(/\.md$/, '.pdf')
		console.log "#{path.join sourceDir, file} -> #{path.join destDir, destFile}"
		exec "pandoc -f markdown -S --toc #{path.join sourceDir, file} -o #{path.join destDir, destFile}", (err, stdout, stderr) ->
				console.log "Error generating #{destFile}: #{err}" if err?
				console.log "Error generating #{destFile}: #{stderr}" if stderr?.trim().length > 0

task 'generate:web', 'generate site', (options) ->
	config = getConfig options
	
	for sourceDir in sources
		if fs.existsSync path.join config.webSource, sourceDir.source
			buildSiteDir path.join(config.webSource, sourceDir.source), path.join(config.output, sourceDir.destination)

task 'generate:all', 'generate site and PDF files', (options) ->
	invoke 'generate:pdf'
	invoke 'generate:web' 

task 'sbuild', 'sublime build', (options) ->
	# sublime coffee plugin uses sbuild task
	invoke 'generate:all'

task 'deploy', 'deploy to S3', (options) ->
	knox = require 'knox'
	
	gotOpts = true
	config = getConfig options
	
	if not config.key?
		console.log 'S3 API Key required'
		gotOpts = false
	if not config.secret?
		console.log 'S3 API Secret required'
		gotOpts = false
	if not config.bucket?
		console.log 'S3 Bucket required'
		gotOpts = false
	
	if not gotOpts
		console.log 'Run `cake` to view all available config.'
		process.exit 1
	
	console.log "Deploying to #{path.join "[#{config.bucket}]", config.path}"
	
	client = knox.createClient
		key: config.key
		secret: config.secret
		bucket: config.bucket
	
	deployDir client, config.output, config.path

task 'clean', 'cleans the output folder', (options) ->
	config = getConfig options

	console.log 'Cleaning output folder...'
	fs.remove config.output, (err) -> if err? then console.log "Error cleaning output folder: #{err}" else console.log 'Output folder clean.'
