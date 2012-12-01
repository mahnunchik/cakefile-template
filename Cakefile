# ** Cakefile Template ** is a Template for a common Cakefile that you may use in a coffeescript nodejs project.
# 
# It comes baked in with 5 tasks:
#
# * build - compiles your src directory to your lib directory
# * watch - watches any changes in your src directory and automatically compiles to the lib directory
# * test  - runs mocha test framework, you can edit this task to use your favorite test framework
# * docs  - generates annotated documentation using docco
# * clean - clean generated .js files

lib_dir = 'lib'

fs = require 'fs'
{print} = require 'util'
{spawn, exec} = require 'child_process'

try
  which = require('which').sync
catch err
  if process.platform.match(/^win/)?
    console.log 'WARNING: the which module is required for windows\ntry: npm install which'
  which = null

try
  glob = require 'glob'  
catch err
  console.log 'WARNING: the glob module is required\ntry: npm install glob'

# ANSI Terminal Colors
bold = '\x1b[0;1m'
green = '\x1b[0;32m'
reset = '\x1b[0m'
red = '\x1b[0;31m'

# Cakefile Tasks
#
# ## *docs*
#
# Generate Annotated Documentation
#
# <small>Usage</small>
#
# ```
# cake docs
# ```
task 'docs', 'generate documentation', -> docco()

# ## *build*
#
# Builds Source
#
# <small>Usage</small>
#
# ```
# cake build
# ```
task 'build', 'compile source', -> build -> log ":)", green

# ## *watch*
#
# Builds your source whenever it changes
#
# <small>Usage</small>
#
# ```
# cake watch
# ```
task 'watch', 'compile and watch', -> build true, -> log ":-)", green

# ## *test*
#
# Runs your test suite.
#
# <small>Usage</small>
#
# ```
# cake test
# ```
task 'test', 'run tests', -> build -> mocha -> log ":)", green

# ## *clean*
#
# Cleans up generated js files
#
# <small>Usage</small>
#
# ```
# cake clean
# ```
task 'clean', 'clean generated files', -> clean -> log ";)", green

# ## *lint*
#
# Lint coffee files
#
# <small>Usage</small>
#
# ```
# cake lint
# ```
task 'lint', 'clean generated files', -> lint -> log ";)", green


# Internal Functions

# ## *log* 
# 
# **given** string as a message
# **and** string as a color
# **and** optional string as an explanation
# **then** builds a statement and logs to console.
# 
log = (message, color=reset, explanation) -> console.log color + message + reset + ' ' + (explanation or '')

# ## *launch*
#
# **given** string as a cmd
# **and** optional array and option flags
# **and** optional callback
# **then** spawn cmd with options
# **and** pipe to process stdout and stderr respectively
# **and** on child process exit emit callback if set and status is 0
launch = (cmd, options=[], callback) ->
  cmd = which(cmd) if which
  app = spawn cmd, options
  app.stdout.pipe(process.stdout)
  app.stderr.pipe(process.stderr)
  app.on 'exit', (status) -> callback?() if status is 0

# ## *_build*
#
# **given** directory
# **and** optional boolean as watch
# **and** optional function as callback
# **then** invoke launch passing coffee command
# **and** defaulted options to compile src to lib
_build = (dir, watch, callback) ->
  if typeof watch is 'function'
    callback = watch
    watch = false

  options = ['-c', '-b', dir]
  options.unshift '-w' if watch
  launch 'coffee', options, callback


_browserify = (in_file, out_file, watch, callback)->
  if typeof watch is 'function'
    callback = watch
    watch = false
  options = ['-o', out_file, in_file] 
  options.unshift '-w' if watch
  launch 'browserify', options, callback


# ## *unlinkIfCoffeeFile*
#
# **given** string as file
# **and** file ends in '.coffee'
# **then** convert '.coffee' to '.js'
# **and** remove the result
unlinkIfCoffeeFile = (file) ->
  if file.match /\.coffee$/
    fs.unlink file.replace(/\.coffee$/, '.js')
    true
  else false


# ## *_clean*
#
# **given** directory to clean generated files 
# **and** optional function as callback
# **then** loop through files variable
# **and** call unlinkIfCoffeeFile on each
_clean = (dir, callback) ->
  dir = [dir] unless dir instanceof Array
  try
    for file in dir
      unless unlinkIfCoffeeFile file
        glob "#{file}/**/*.coffee", (err, results) ->
          for f in results
            unlinkIfCoffeeFile f
    callback?()
  catch err

_lint = (dir, callback)->
  if moduleExists('coffeelint')
    options = ['-r', '-f', 'coffeelint.json', dir] 
    launch 'coffeelint', options, callback


# ## *moduleExists*
#
# **given** name for module
# **when** trying to require module
# **and** not found
# **then* print not found message with install helper in red
# **and* return false if not found
moduleExists = (name) ->
  try 
    require name 
  catch err 
    log "nodejs module '#{name}' is required", red
    log "try: npm install #{name}", green
    false


# ## *mocha*
#
# **given** optional array of option flags
# **and** optional function as callback
# **then** invoke launch passing mocha command
mocha = (options, callback) ->
  if moduleExists('mocha')
    if typeof options is 'function'
      callback = options
      options = []
    # add coffee directive
    options.push '--compilers'
    options.push 'coffee:coffee-script'
    
    launch 'mocha', options, callback

# ## *_docco*
#
# **given** source directory
# **and** optional function as callback
# **then** invoke launch passing docco command
_docco = (dir, callback) ->
  if moduleExists('docco')
    glob "#{file}/**/*.coffee", (err, files) -> launch 'docco', files, callback


build = (watch, callback)->
  _build lib_dir, watch, callback

clean = (callback)->
  _clean lib_dir, callback

docco = (callback)->
  _docco lib_dir, callback

lint = (callback)->
  _lint(lib_dir, callback)


