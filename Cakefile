fs      = require('fs')
http    = require('http')
path    = require('path')
$       = require('jQuery')
_       = require('underscore')
jsdom   = require("jsdom")
request = require('request')

# Get all threads from the given board
getThreads = (board, callback) ->
  request { uri: "http://4chan.org/#{board}" }, (error, response, body) ->
    if !error && response.statusCode == 200
      matches = _(body.match(/\"res\/\d+\"/g)).uniq()
      threads = _(matches).map (m) ->
        m.match(/\d+/)[0]
      callback(board, threads)

# Get images for a given thread
getImages = (board, thread_id, callback) ->
  request { uri: "http://4chan.org/#{board}/res/#{thread_id}" }, (error, response, body) ->
    if !error && response.statusCode == 200
      imgs = _(body.match /http:\/\/images.4chan.org(.*?)(\d+)\..../g).uniq()
      callback(imgs)

saveImage = (url, board, thread) ->
  options =
    host: "images.4chan.org"
    port: 80
    path: url.match(/\.org(.*)/)[1]

  filename = url.match(/src\/(.*)/)[1]
  local_path = "output/#{board}-#{thread}-#{filename}"

  path.exists local_path, (exists) ->
    if !exists
      file = fs.createWriteStream "output/#{filename}"
      http.get options, (res) ->
        res.on('data', (data) ->
          file.write(data)
        ).on('end', () ->
          file.end()
        )

getThreads "b", (board, threads) ->
  _(threads).each (thread) ->
    getImages board, thread, (images) ->
      _(images).each (i) ->
        saveImage(i, board, thread)


