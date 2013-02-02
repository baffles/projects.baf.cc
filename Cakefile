# cakefile to build project writeups
{exec} = require 'child_process'
path = require 'path'
fs = require 'fs-extra'

option '-s', '--source', 'Source folder, defaults to source'
option '-o', '--output', 'Output folder, defaults to generated'

getConfig = (options) ->
	effectiveConfig = 
		source: options.source ? 'source'
		output: options.output ? 'generated'

walkDirs = (sourceDir, destDir, action) ->
	for entry in fs.readdirSync sourceDir
		stat = fs.statSync path.join sourceDir, entry
		if stat.isDirectory()
			walkDirs path.join(sourceDir, entry), path.join(destDir, entry), action
		else if stat.isFile()
			fs.mkdirsSync destDir
			action sourceDir, destDir, entry

task 'generate:pdf', 'generate PDF files', (options) ->
	config = getConfig options

	walkDirs config.source, "#{config.output}/pdf", (sourceDir, destDir, file) ->
		destFile = file.replace(/\.md$/, '.pdf')
		console.log "#{path.join sourceDir, file} -> #{path.join destDir, destFile}"
		exec "pandoc -f markdown -S --toc #{path.join sourceDir, file} -o #{path.join destDir, destFile}", (err, stdout, stderr) ->
				console.log "Error generating #{destFile}: #{err}" if err?
				console.log "Error generating #{destFile}: #{stderr}" if stderr?.trim().length > 0

task 'sbuild', 'sublime build', (options) ->
	# sublime coffee plugin uses sbuild task
	invoke 'generate:pdf'

task 'clean', 'cleans the output folder', (options) ->
	config = getConfig options

	console.log 'Cleaning output folder...'
	fs.remove config.output, (err) -> if err? then console.log "Error cleaning output folder: #{err}" else console.log 'Output folder clean.'
