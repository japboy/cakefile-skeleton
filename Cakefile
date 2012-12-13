# cakefile-skeleton
#
# This Cakefile is a fork of [cakefile-template][1] by [@twilson63][2], and
# I modified the codes for my convention and added my basic functions for
# [CoffeeScript][3], [CSSO][4], [Jade][5], [RequireJS][6], [Sass][7],
# [Stylus][8], [UgilifyJS][9], [AssetGraph][10], etc.
#
# Written and modified by [@japboy][11] and distributed under
# the [Public Domain][12] in 2012.
#
# [1]: https://github.com/twilson63/cakefile-template
# [2]: https://github.com/twilson63
# [3]: http://coffeescript.org/
# [4]: http://css.github.com/csso/
# [5]: http://jade-lang.com/
# [6]: http://requirejs.org/
# [7]: http://sass-lang.com/
# [8]: http://learnboost.github.com/stylus/
# [9]: https://github.com/mishoo/UglifyJS
# [10]: https://github.com/One-com/assetgraph
# [11]: https://github.com/japboy
# [12]: http://creativecommons.org/licenses/publicdomain/


# Module dependencies
events = require 'events'
fs = require 'fs'
{print} = require 'util'
{spawn, exec} = require 'child_process'

try
  which = require('which').sync
catch err
  if process.platform.match(/^win/)?
    console.log 'WARNING: the which module is required for windows\ntry: npm install which'
  which = null


# Path specifications
ROOT = '.'
DOCCO_OUT_DIR = "#{ROOT}/docs"


# ANSI terminal styles
ANSI_BOLD = '\x1b[0;1m'
ANSI_GREEN = '\x1b[0;32m'
ANSI_RESET = '\x1b[0m'
ANSI_RED = '\x1b[0;31m'


# Namespace object
cake = new events.EventEmitter()


# Fundamental functions
# ---------------------

# ### *moduleExists*
#
# Check if specified module is loadable or not
#
# Example:
#
#     return false if not moduleExists 'coffee-script'
#
moduleExists = (name) ->
  try
    require name
  catch err
    log "#{name} required: npm install #{name}", ANSI_RED
    return false

# ### *launch*
#
# Run a command-line interface asynchronously.
#
# Example:
#
#     launch 'cd', ['/path/to'], ->
#         launch 'ls', ['-laF']
#
launch = (cmd, options=[], callback) ->
  cmd = which(cmd) if which
  app = spawn cmd, options

  app.on 'exit', (status) -> callback?() if 0 is status

  app.stdout.pipe process.stdout
  app.stderr.pipe process.stderr

# ### *walk*
#
# Walk through the path recursively and store the list of files.
#
# Example:
#
#     walk './path', (err, results) ->
#         return err if err
#         console.log results
#
walk = (dir, done) ->
  results = []

  fs.readdir dir, (err, list) ->
    return done(err, []) if err

    pending = list.length

    return done(null, results) unless pending

    for name in list
      file = "#{dir}/#{name}"

      try
        stat = fs.statSync file
      catch err
        stat = null

      if stat?.isDirectory()
        walk file, (err, res) ->
          results.push name for name in res
          done(null, results) unless --pending
      else
        results.push file
        done(null, results) unless --pending

# ### *log*
#
# Example:
#
#     log 'All green :)', ANSI_GREEN
#
log = (message, color, explanation) ->
  if moduleExists 'growl'
    _growl = require 'growl'

    _growl(message + ' ' + (explanation or ''))

  console.log(color + message + ANSI_RESET + ' ' + (explanation or ''))


# Functions for preprocessors
# ---------------------------

# ### *coffee*
#
# Read `.coffee` file and compile it to a new `.js` file.
#
# Example:
#
#     coffee data, file, (err, result, newFile) ->
#         return err if err
#         console.log result, newFile
#
coffee = (data, path, done) ->
  if not moduleExists 'coffee-script'
    return done 'No CoffeeScript module found', undefined, undefined

  _coffee = require 'coffee-script'
  result = _coffee.compile data

  path = path.replace /\.coffee$/, '.js'

  done undefined, result, path

# ### *jade*
#
# Read `.jade` file and compile it to a new `.html` file.
#
# Example:
#
#     jade data, file, (err, result, newFile) ->
#         return err if err
#         console.log result, newFile
#
jade = (data, path, done) ->
  if not moduleExists 'jade'
    return done 'No Jade module found', undefined, undefined

  _jade = require 'jade'
  html = _jade.compile data, { filename: path, pretty: true }
  result = html()

  path = path.replace /\.jade$/, '.html'

  done undefined, result, path

# ### *stylus*
#
# Read `.styl` file and compile it to a new `.css` file.
#
# Example:
#
#     stylus data, file, (err, result, newFile) ->
#         return err if err
#         console.log result, newFile
#
stylus = (data, path, done) ->
  if not moduleExists('stylus') or not moduleExists('nib')
    return done 'No Stylus module found', undefined, undefined

  _stylus = require 'stylus'
  _nib = require 'nib'

  _stylus(data)
  .set('filename', path)
  .set('compress', false)
  .use(_nib())
  .render (err, result) ->
    return done(err, undefined, undefined) if err

    path = path.replace /\.styl$/, '.css'

    done undefined, result, path


# Abstract functions for preprocessors
# ------------------------------------

# ### *type*
#
# Return the object of file type and token from the specified path.
#
# Example:
#
#     actual = type '/path/to/file.coffee'
#
type = (path) ->
  formats =
    'coffee': /\/?(.+\.coffee$)/
    'jade': /\/?(.+\.jade$)/
    'stylus': /\/?(.+\.styl$)/

  for key, val of formats
    if path.match val
      obj = {}
      obj[key] = val
      return obj

  return undefined

# ### *filter*
#
# Filter out non-targets in the files.
#
# Example:
#
#     filter ['a.txt', 'b.coffee', 'c.styl'], (err, results) ->
#         return err if err
#         console.log results
#
filter = (paths, done) ->
  results = []

  for path in paths
    results.push path if type path

  return done 'No targets found', undefined if 0 is results.length

  done undefined, results

# ### *compile*
#
# Read a file and compile it with specific mixin function, then
# write it to a new file at the same path of the original.
#
# TODO: Add header string to the compiled string  
# TODO: Skip if the content is not updated
#
# Example:
#
#     compile './a.coffee', coffee, (err, path) ->
#         return err if err
#         console.log path
#
compile = (file, mixin, done) ->
  encoding = 'utf8'

  fs.readFile file, encoding, (err, data) ->
    return done(err, file) if err

    mixin data, file, (err, result, newFile) ->
      return done(err, file) if err

      fs.writeFile newFile, result, encoding, (err) ->
        return done(err, newFile) if err

        done undefined, newFile

# ### *runCompiler*
#
# Call proper compiler for the specified file.
#
# Example:
#
#     runCompiler file, (err, path) ->
#         return err if err
#         console.log path
#
runCompiler = (file, done) ->
  _type = type file

  return done 'Unable to compile the invalid file', file if not _type

  if _type['coffee']
    compile file, coffee, (err, newFile) ->
      return done(err, file) if err
      return done undefined, newFile

  if _type['jade']
    compile file, jade, (err, newFile) ->
      return done(err, file) if err
      return done undefined, newFile

  if _type['stylus']
    compile file, stylus, (err, newFile) ->
      return done(err, file) if err
      return done undefined, newFile


# Functions for command-line interface
# ------------------------------------

# ### *docco*
#
# Run `docco` command to generate the documentation.
#
# Example:
#
#     docco()
#
docco = (callback) ->
  walk ROOT, (err, files) ->
    return log "Error: #{err}", ANSI_RED if err

    launch 'cp', ['-f', "#{ROOT}/Cakefile", "#{ROOT}/Cakefile.coffee"], ->
      launch 'docco', ['--output', DOCCO_OUT_DIR, "#{ROOT}/Cakefile.coffee"], ->
        launch 'rm', ['-f', "#{ROOT}/Cakefile.coffee"]

# ### *mocha*
#
# Run JSUnit test by `mocha`
#
# Example:
#
#     mocha()
#
mocha = (options, callback) ->
  if 'function' is typeof options
    callback = options
    options = []

  options.push '--compilers'
  options.push 'coffee:coffee-script'

  launch 'mocha', options, callback


# Internal functions
# ------------------

# ### *build*
#
# Compile and generate files if they are updated.
#
# Example:
#
#     build()
#
build = (watch, callback) ->
  if 'function' is typeof watch
    callback = watch
    watch = false

  walk ROOT, (err, files) ->
    return log("Error: #{err}", ANSI_RED) if err

    filter files, (err, results) ->
      return log("Error: #{err}", ANSI_RED) if err

      pending = results.length

      cake.once 'built', ->
        log 'All the build process done.', ANSI_BOLD
        callback undefined

      for result in results
        runCompiler result, (err, newFile) ->
          return log("Error: #{err}, #{newFile}", ANSI_RED) if err

          log "Compiled #{newFile}", ANSI_GREEN
          cake.emit 'built' if 0 is --pending

# ### *clean*
#
# Remove all the generated file.
#
# TODO: Prevent to unlink the original files
#
# Example:
#
#     clean()
#
clean = (callback) ->
  callback?()

# ### *concat*
#
# Concatenates source files into a destination file.
#
# TODO: Skip if files are not updated
#
# Example:
#
#     concat ['./a.js', './b/c.js'], './app.js'
#
concat = (sources, destination, done) ->
  encoding = 'utf8'
  pending = sources.length
  buf = ''

  cake.once 'concatenated', ->
    fs.writeFile destination, buf, encoding, (err) ->
      return log "Error: #{err}", ANSI_RED if err

      log "Concatenated to #{destination}", ANSI_GREEN
      log 'All the concatenation process done.', ANSI_BOLD
      done undefined

  for file in sources
    fs.readFile file, encoding, (err, data) ->
      return log "Error: #{err}", ANSI_RED if err

      buf += data

      cake.emit 'concatenated' if 0 is --pending


# Cakefile tasks
# --------------

# ### *build*
#
# Builds source
#
# Command:
#
#     cake build
#
task 'build', 'compile source', ->
  build ->
    log ':)', ANSI_GREEN

# ### *concat*
#
# Builds and concatenates
#
# Command:
#
#     cake concat
#
task 'concat', 'concatenate source', (args...) ->
  console.log args
  build ->
    concat ["#{ROOT}/test/template.html", "#{ROOT}/test/parallax.html"], "#{ROOT}/app.html", ->
      log ':)', ANSI_GREEN

# ### *watch*
#
# Builds source whenever it changes
#
# Command:
#
#     cake watch
#
task 'watch', 'compile and watch', ->
  build true, ->
    log ':-)', ANSI_GREEN

# ### *test*
#
# Runs test suite
#
# Command:
#
#     cake test
#
task 'test', 'run tests', ->
  build ->
    mocha ->
      log ':)', ANSI_GREEN

# ### *docs*
#
# Generate annotated documentation
#
# Command:
#
#     cake docs
#
task 'docs', 'generate documentation', ->
  docco()

# ### *clean*
#
# Cleans up generated JavaScript files
#
# Command:
#
#     cake clean
#
task 'clean', 'clean generated files', ->
  clean ->
    log ';)', ANSI_GREEN
