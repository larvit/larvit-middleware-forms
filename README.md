[![Build Status](https://travis-ci.org/larvit/larvitreqparser.svg?branch=master)](https://travis-ci.org/larvit/larvitreqparser) [![Dependencies](https://david-dm.org/larvit/larvitreqparser.svg)](https://david-dm.org/larvit/larvitreqparser.svg)
[![Coverage Status](https://coveralls.io/repos/larvit/larvitreqparser/badge.svg)](https://coveralls.io/github/larvit/larvitreqparser)

# Request parser middleware

Middleware for [larvitbase](https://github.com/larvit/larvitbase) or [express](https://expressjs.com/) to handle parsing url, forms and file uploads. This is just a wrapper for the following libraries:

* [url.parse](https://nodejs.org/api/url.html#url_url_parse_urlstring_parsequerystring_slashesdenotehost)
* [qs](https://github.com/ljharb/qs)
* [busboy](https://github.com/mscdex/busboy)

## Installation

```shell
npm i --save larvitreqparser
```

## Usage

### Larvitbase

Usage with [larvitbase](https://github.com/larvit/larvitbase)

```javascript
const App       = require('larvitbase'),
      ReqParser = require('larvitreqparser'),
      reqParser = new ReqParser({
      	'storage': 'memory' // Default. Options: 'memory' or a file path, for example '/tmp'.
      });

new App({
	'httpOptions': 8001,
	'middleware': [
		reqParser.parse,
		function (req, res) {
			// Now the following properties is populated depending on the request type:

			// req.urlParsed - URL parsed by require('url').parse()
			// req.formFields
			// req.formFiles
			// req.formFiles.filename?
			// req.formFiles.mimetype?

			// If storage === 'memory'
			// req.rawBody
			// req.formFiles.buffer

			// If storage is path on disk
			// req.rawBodyPath
			// req.formFiles.path

			res.end('Hello world');
		}
	]
});
```

#### Cleanup when not using memory

When not using memory, files are stored on disk. They must be manually removed or they will just fill up infinitly!

```javascript
const App        = require('larvitbase'),
      ReqParser  = require('larvitreqparser'),
      reqParser  = new ReqParser({
      	'storage': '/tmp'
      }),
      fs         = require('fs');

new App({
	'httpOptions': 8001,
	'middleware': [
		reqParser.parse,
		function (req, res, cb) {
			res.end('Hello world');
			cb();
		},
		function (req, res, cb) {

			// Check if we have a form file, and if we do, remove it
			if (req.formFiles && req.formFiles.path) {
				return fs.unlink(req.formFiles.path, cb);
			}
			cb();
		}
	]
});
```
