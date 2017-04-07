# api documentation for  [through (v2.3.8)](https://github.com/dominictarr/through)  [![npm package](https://img.shields.io/npm/v/npmdoc-through.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-through) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-through.svg)](https://travis-ci.org/npmdoc/node-npmdoc-through)
#### simplified stream construction

[![NPM](https://nodei.co/npm/through.png?downloads=true)](https://www.npmjs.com/package/through)

[![apidoc](https://npmdoc.github.io/node-npmdoc-through/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-through_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-through/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-through/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-through/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Dominic Tarr",
        "email": "dominic.tarr@gmail.com",
        "url": "dominictarr.com"
    },
    "bugs": {
        "url": "https://github.com/dominictarr/through/issues"
    },
    "dependencies": {},
    "description": "simplified stream construction",
    "devDependencies": {
        "from": "~0.1.3",
        "stream-spec": "~0.3.5",
        "tape": "~2.3.2"
    },
    "directories": {},
    "dist": {
        "shasum": "0dd4c9ffaabc357960b1b724115d7e0e86a2e1f5",
        "tarball": "https://registry.npmjs.org/through/-/through-2.3.8.tgz"
    },
    "gitHead": "2c5a6f9a0cc54da759b6e10964f2081c358e49dc",
    "homepage": "https://github.com/dominictarr/through",
    "keywords": [
        "stream",
        "streams",
        "user-streams",
        "pipe"
    ],
    "license": "MIT",
    "main": "index.js",
    "maintainers": [
        {
            "name": "dominictarr",
            "email": "dominic.tarr@gmail.com"
        }
    ],
    "name": "through",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+https://github.com/dominictarr/through.git"
    },
    "scripts": {
        "test": "set -e; for t in test/*.js; do node $t; done"
    },
    "testling": {
        "browsers": [
            "ie/8..latest",
            "ff/15..latest",
            "chrome/20..latest",
            "safari/5.1..latest"
        ],
        "files": "test/*.js"
    },
    "version": "2.3.8"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module through](#apidoc.module.through)
1.  [function <span class="apidocSignatureSpan"></span>through (write, end, opts)](#apidoc.element.through.through)



# <a name="apidoc.module.through"></a>[module through](#apidoc.module.through)

#### <a name="apidoc.element.through.through"></a>[function <span class="apidocSignatureSpan"></span>through (write, end, opts)](#apidoc.element.through.through)
- description and source-code
```javascript
function through(write, end, opts) {
  write = write || function (data) { this.queue(data) }
  end = end || function () { this.queue(null) }

  var ended = false, destroyed = false, buffer = [], _ended = false
  var stream = new Stream()
  stream.readable = stream.writable = true
  stream.paused = false

//  stream.autoPause   = !(opts && opts.autoPause   === false)
  stream.autoDestroy = !(opts && opts.autoDestroy === false)

  stream.write = function (data) {
    write.call(this, data)
    return !stream.paused
  }

  function drain() {
    while(buffer.length && !stream.paused) {
      var data = buffer.shift()
      if(null === data)
        return stream.emit('end')
      else
        stream.emit('data', data)
    }
  }

  stream.queue = stream.push = function (data) {
//    console.error(ended)
    if(_ended) return stream
    if(data === null) _ended = true
    buffer.push(data)
    drain()
    return stream
  }

  //this will be registered as the first 'end' listener
  //must call destroy next tick, to make sure we're after any
  //stream piped from here.
  //this is only a problem if end is not emitted synchronously.
  //a nicer way to do this is to make sure this is the last listener for 'end'

  stream.on('end', function () {
    stream.readable = false
    if(!stream.writable && stream.autoDestroy)
      process.nextTick(function () {
        stream.destroy()
      })
  })

  function _end () {
    stream.writable = false
    end.call(stream)
    if(!stream.readable && stream.autoDestroy)
      stream.destroy()
  }

  stream.end = function (data) {
    if(ended) return
    ended = true
    if(arguments.length) stream.write(data)
    _end() // will emit or queue
    return stream
  }

  stream.destroy = function () {
    if(destroyed) return
    destroyed = true
    ended = true
    buffer.length = 0
    stream.writable = stream.readable = false
    stream.emit('close')
    return stream
  }

  stream.pause = function () {
    if(stream.paused) return
    stream.paused = true
    return stream
  }

  stream.resume = function () {
    if(stream.paused) {
      stream.paused = false
      stream.emit('resume')
    }
    drain()
    //may have become paused again,
    //as drain emits 'data'.
    if(!stream.paused)
      stream.emit('drain')
    return stream
  }
  return stream
}
```
- example usage
```shell
n/a
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
