fs      = require('fs')
http    = require('http')
path    = require('path')
$       = require('jQuery')
_       = require('underscore')
jsdom   = require("jsdom")
mkdirp  = require('mkdirp')
request = require('request')

global.base_path = "/Users/micho/code/4cheese/output"

# Get all threads from the given board
getThreads = (board, callback) ->
  request { uri: "http://4chan.org/#{board}" }, (error, response, body) ->
    if !error && response.statusCode == 200
      matches = _(body.match(/\"res\/\d+\"/g)).uniq()
      threads = _(matches).map (m) ->
        m.match(/\d+/)[0]
      callback(board, threads)

# Get images for a given thread
getImages = (board, thread_id, terms, callback) ->
  request { uri: "http://4chan.org/#{board}/res/#{thread_id}" }, (error, response, body) ->
    if !error && response.statusCode == 200
      # Search string found in the body
      if body.match terms
        imgs = _(body.match /http:\/\/images.4chan.org(.*?)(\d+)\..../g).uniq()
        # Save thread file
        mkdirp.sync("#{global.base_path}/#{board}/#{thread_id}", 0755)
        fs.writeFile(
          "#{global.base_path}/#{board}/#{thread_id}/_.html",
          body.replace(/http(.*)src\//, "")
        )
        # Run callback to save images
        callback(imgs)

saveImage = (url, board, thread) ->
  options =
    host: "images.4chan.org"
    port: 80
    path: url.match(/\.org(.*)/)[1]

  filename = url.match(/src\/(.*)/)[1]
  local_path = "output/#{board}/#{thread}/#{filename}"

  path.exists local_path, (exists) ->
    if !exists
      console.log "Writing #{local_path}"
      file = fs.createWriteStream local_path
      http.get options, (res) ->
        res.on('data', (data) ->
          file.write(data)
        ).on('end', () ->
          file.end()
        )

# Download new threads from /board_id
downloadNew = (board_id) ->
  console.log "Checking for changes in /#{board_id}"
  getThreads board_id, (board, threads) ->
    _(threads).each (thread) ->
      getImages board, thread, /\Wmoar\W/gi, (images) ->
        _(images).each (i) ->
          saveImage(i, board, thread)

# Start checking the board every **interval** seconds, with an offset of **start_offset** seconds
startPeriodicDownloader = (board_id, interval, start_offset) ->
  setTimeout( ->
    downloadNew(board_id)
    setInterval( ->
      downloadNew(board_id)
    , interval * 1000)
  , start_offset * 1000)

startPeriodicDownloader("b", 180, 0)
startPeriodicDownloader("s", 180, 60)
startPeriodicDownloader("r", 180, 120)

