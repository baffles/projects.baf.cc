# My Project Write-Ups

This is the source to my project write-ups. The writeups are written in Markdown, while the webpage components are
written with Jade, and Stylus. Generation is scripted using CoffeeScript and Cakefiles. The content is designed to be
statically compiled and hosted via Amazon S3 and/or CloudFront.

## Third Party Code

I'm making use of [CoffeeScript](http://coffeescript.org/) (running on [node.js](http://nodejs.org/)) for building the
site.

* CoffeeScript must be installed and in the path to build the site.
* pandoc and latex must be installed to generate output from the project writeup source.
* The [jade](http://jade-lang.com/) `v0.28.x`, [stylus](http://learnboost.github.com/stylus/) `v0.32.x`,
  [fs-extra](https://github.com/jprichardson/node-fs-extra) `v0.4.x`, and
  [marked](https://github.com/chjj/marked) `v0.2.x` node packages must be installed to build the site. You can install
  these locally by running `npm install jade@0.28.x stylus@0.32.x fs-extra@0.4.x marked@0.2.x`.
* The [knox](https://github.com/LearnBoost/knox) `v0.4.x` node package must be installed to deploy the site to S3. You
  can install this locally by running `npm install knox@0.4.x`.

## Building

To generate PDF files, run `cake generate:pdf`. To build the web content itself, run `cake generate:web`. Run `cake` to
view other options.

To remove the output files, run `cake clean`.

## Deploying

To deploy, you'll need to specify S3 information. This can either be done via `config.json` (see `config.json.sample`)
or via command line (run `cake` to view the options).

The actual deployment process is accomplished by running `cake deploy`.
