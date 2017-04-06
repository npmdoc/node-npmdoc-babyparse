# api documentation for  [babyparse (v0.4.6)](https://github.com/Rich-Harris/BabyParse#readme)  [![npm package](https://img.shields.io/npm/v/npmdoc-babyparse.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-babyparse) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-babyparse.svg)](https://travis-ci.org/npmdoc/node-npmdoc-babyparse)
#### Fast and reliable CSV parser based on PapaParse

[![NPM](https://nodei.co/npm/babyparse.png?downloads=true)](https://www.npmjs.com/package/babyparse)

[![apidoc](https://npmdoc.github.io/node-npmdoc-babyparse/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-babyparse_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-babyparse/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-babyparse/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-babyparse/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Rich Harris",
        "url": "http://rich-harris.co.uk"
    },
    "bugs": {
        "url": "https://github.com/Rich-Harris/BabyParse/issues"
    },
    "dependencies": {},
    "description": "Fast and reliable CSV parser based on PapaParse",
    "devDependencies": {},
    "directories": {},
    "dist": {
        "shasum": "8ff29b62d1e600c0654afd63457f97fa2c36e9c1",
        "tarball": "https://registry.npmjs.org/babyparse/-/babyparse-0.4.6.tgz"
    },
    "gitHead": "7c0508681e07fa1af7f3eaf425658cf31f6ae195",
    "homepage": "https://github.com/Rich-Harris/BabyParse#readme",
    "keywords": [
        "csv",
        "parse",
        "parsing",
        "parser",
        "delimited",
        "text",
        "data",
        "auto-detect",
        "comma",
        "tab",
        "stream"
    ],
    "licenses": [
        {
            "type": "MIT",
            "url": "http://opensource.org/licenses/MIT"
        }
    ],
    "main": "babyparse.js",
    "maintainers": [
        {
            "name": "rich_harris",
            "email": "richard.a.harris@gmail.com"
        },
        {
            "name": "mholt",
            "email": "Matthew.Holt+npm@gmail.com"
        }
    ],
    "name": "babyparse",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+ssh://git@github.com/Rich-Harris/BabyParse.git"
    },
    "scripts": {},
    "title": "BabyParse",
    "version": "0.4.6"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module babyparse](#apidoc.module.babyparse)
1.  [function <span class="apidocSignatureSpan">babyparse.</span>Parser (config)](#apidoc.element.babyparse.Parser)
1.  [function <span class="apidocSignatureSpan">babyparse.</span>ParserHandle (_config)](#apidoc.element.babyparse.ParserHandle)
1.  [function <span class="apidocSignatureSpan">babyparse.</span>parse (_input, _config)](#apidoc.element.babyparse.parse)
1.  [function <span class="apidocSignatureSpan">babyparse.</span>parseFiles (_input, _config)](#apidoc.element.babyparse.parseFiles)
1.  [function <span class="apidocSignatureSpan">babyparse.</span>unparse (_input, _config)](#apidoc.element.babyparse.unparse)
1.  object <span class="apidocSignatureSpan">babyparse.</span>BAD_DELIMITERS
1.  string <span class="apidocSignatureSpan">babyparse.</span>BYTE_ORDER_MARK
1.  string <span class="apidocSignatureSpan">babyparse.</span>DefaultDelimiter
1.  string <span class="apidocSignatureSpan">babyparse.</span>RECORD_SEP
1.  string <span class="apidocSignatureSpan">babyparse.</span>UNIT_SEP



# <a name="apidoc.module.babyparse"></a>[module babyparse](#apidoc.module.babyparse)

#### <a name="apidoc.element.babyparse.Parser"></a>[function <span class="apidocSignatureSpan">babyparse.</span>Parser (config)](#apidoc.element.babyparse.Parser)
- description and source-code
```javascript
function Parser(config)
	{
		// Unpack the config object
		config = config || {};
		var delim = config.delimiter;
		var newline = config.newline;
		var comments = config.comments;
		var step = config.step;
		var preview = config.preview;
		var fastMode = config.fastMode;

		// Delimiter must be valid
		if (typeof delim !== 'string'
			|| delim.length != 1
			|| Baby.BAD_DELIMITERS.indexOf(delim) > -1)
			delim = ",";

		// Comment character must be valid
		if (comments === delim)
			throw "Comment character same as delimiter";
		else if (comments === true)
			comments = "#";
		else if (typeof comments !== 'string'
			|| Baby.BAD_DELIMITERS.indexOf(comments) > -1)
			comments = false;

		// Newline must be valid: \r, \n, or \r\n
		if (newline != '\n' && newline != '\r' && newline != '\r\n')
			newline = '\n';

		// We're gonna need these at the Parser scope
		var cursor = 0;
		var aborted = false;

		this.parse = function(input)
		{
			// For some reason, in Chrome, this speeds things up (!?)
			if (typeof input !== 'string')
				throw "Input must be a string";

			// We don't need to compute some of these every time parse() is called,
			// but having them in a more local scope seems to perform better
			var inputLen = input.length,
				delimLen = delim.length,
				newlineLen = newline.length,
				commentsLen = comments.length;
			var stepIsFunction = typeof step === 'function';

			// Establish starting state
			cursor = 0;
			var data = [], errors = [], row = [];

			if (!input)
				return returnable();

			if (fastMode)
			{
				// Fast mode assumes there are no quoted fields in the input
				var rows = input.split(newline);
				for (var i = 0; i < rows.length; i++)
				{
					if (comments && rows[i].substr(0, commentsLen) == comments)
						continue;
					if (stepIsFunction)
					{
						data = [ rows[i].split(delim) ];
						doStep();
						if (aborted)
							return returnable();
					}
					else
						data.push(rows[i].split(delim));
					if (preview && i >= preview)
					{
						data = data.slice(0, preview);
						return returnable(true);
					}
				}
				return returnable();
			}

			var nextDelim = input.indexOf(delim, cursor);
			var nextNewline = input.indexOf(newline, cursor);

			// Parser loop
			for (;;)
			{
				// Field has opening quote
				if (input[cursor] == '"')
				{
					// Start our search for the closing quote where the cursor is
					var quoteSearch = cursor;

					// Skip the opening quote
					cursor++;

					for (;;)
					{
						// Find closing quote
						var quoteSearch = input.indexOf('"', quoteSearch+1);

						if (quoteSearch === -1)
						{
							// No closing quote... what a pity
							errors.push({
								type: "Quotes",
								code: "MissingQuotes",
								message: "Quoted field unterminated",
								row: data.length,	// row has yet to be inserted
								index: cursor
							});
							return finish();
						}

						if (quoteSearch === inputLen-1)
						{
							// Closing quote at EOF
							row.push(input.substring(cursor, quoteSearch).replace(/""/g, '"'));
							data.push(row);
							if (stepIsFunction)
								doStep();
							return returnable();
						}

						// If this quote is escaped, it's part of the data; skip it
						if (input[quoteSearch+1] == '"')
						{
							quoteSearch++;
							continue;
						}

						if (input[quoteSearch+1] == delim)
						{
							// Closing quote followed by delimiter
							row.push(input.substring(cursor, quoteSearch).replace(/""/g, '"'));
							cursor = quoteSearch + 1 + delimLen;
							nextDelim = input.indexOf(delim, cursor);
							nextNewline = input.indexOf(newline, cursor);
							break;
						}

						if (input.substr(quoteSearch+1, newlineLen) === newline)
						{
							// Closing quote followed by newline
							row.push(input.substring(cursor, quoteSearch).replace(/""/g, '"'));
							saveRow(quoteSearch + 1 + newlineLen);
							nextDelim = input.indexOf(delim, cursor);	// because we may have skipped the nextDelim in the quoted field

							if (stepIsFunction)
							{
								doStep();
								if (aborted) ...
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.babyparse.ParserHandle"></a>[function <span class="apidocSignatureSpan">babyparse.</span>ParserHandle (_config)](#apidoc.element.babyparse.ParserHandle)
- description and source-code
```javascript
function ParserHandle(_config)
	{
		// One goal is to minimize the use of regular expressions...
		var FLOAT = /^\s*-?(\d*\.?\d+|\d+\.?\d*)(e[-+]?\d+)?\s*$/i;

		var self = this;
		var _stepCounter = 0;	// Number of times step was called (number of rows parsed)
		var _input;				// The input being parsed
		var _parser;			// The core parser being used
		var _paused = false;	// Whether we are paused or not
		var _delimiterError;	// Temporary state between delimiter detection and processing results
		var _fields = [];		// Fields are from the header row of the input, if there is one
		var _results = {		// The last results returned from the parser
			data: [],
			errors: [],
			meta: {}
		};

		if (isFunction(_config.step))
		{
			var userStep = _config.step;
			_config.step = function(results)
			{
				_results = results;

				if (needsHeaderRow())
					processResults();
				else	// only call user's step function after header row
				{
					processResults();

					// It's possbile that this line was empty and there's no row here after all
					if (_results.data.length == 0)
						return;

					_stepCounter += results.data.length;
					if (_config.preview && _stepCounter > _config.preview)
						_parser.abort();
					else
						userStep(_results, self);
				}
			};
		}

		this.parse = function(input)
		{
			if (!_config.newline)
				_config.newline = guessLineEndings(input);

			_delimiterError = false;
			if (!_config.delimiter)
			{
				var delimGuess = guessDelimiter(input);
				if (delimGuess.successful)
					_config.delimiter = delimGuess.bestDelimiter;
				else
				{
					_delimiterError = true;	// add error after parsing (otherwise it would be overwritten)
					_config.delimiter = Baby.DefaultDelimiter;
				}
				_results.meta.delimiter = _config.delimiter;
			}

			var parserConfig = copy(_config);
			if (_config.preview && _config.header)
				parserConfig.preview++;	// to compensate for header row

			_input = input;
			_parser = new Parser(parserConfig);
			_results = _parser.parse(_input);
			processResults();
			if (isFunction(_config.complete) && !_paused && (!self.streamer || self.streamer.finished()))
				_config.complete(_results);
			return _paused ? { meta: { paused: true } } : (_results || { meta: { paused: false } });
		};

		this.pause = function()
		{
			_paused = true;
			_parser.abort();
			_input = _input.substr(_parser.getCharIndex());
		};

		this.resume = function()
		{
			_paused = false;
			_parser = new Parser(_config);
			_parser.parse(_input);
			if (!_paused)
			{
				if (self.streamer && !self.streamer.finished())
					self.streamer.resume();		// more of the file yet to come
				else if (isFunction(_config.complete))
					_config.complete(_results);
			}
		};

		this.abort = function()
		{
			_parser.abort();
			if (isFunction(_config.complete))
				_config.complete(_results);
			_input = "";
		};

		function processResults()
		{
			if (_results && _delimiterError)
			{
				addError("Delimiter", "UndetectableDelimiter", "Unable to auto-detect delimiting character; defaulted to '"+Baby.DefaultDelimiter
+"'");
				_delimiterError = false;
			}

			if (_config.skipEmptyLines)
			{
				for (var i = 0; i < _results.data.length; i++)
					if (_results.data[i].length == 1 && _results.data[i][0] == "")
						_results.data.splice(i--, 1);
			}

			if (needsHeaderRow())
				fillHeaderFields();

			return applyHeaderAndDynamicTyping();
		}

		function needsHeaderRow()
		{
			return _config.header && _fields.length == 0;
		}

		function fillHeaderFields()
		{
			if (!_results)
				return;
			for (var i = 0; needsHeaderRow() && i < _results.data.length; i++)
				for (var j = 0; j < _results.data[i].length; j++)
					_fields.push(_results.data[i][j]);
			_results.data.splice(0, 1);
		}

		function applyHeaderAndDynamicTyping()
		{
			if (!_results || (!_config.header && !_config.dynamicTyping))
				return _results;

			for (var i = 0; i < _results.data.length; i++)
			{
				var row = {};

				for (var j = 0; j < _results.data[i].length; j++)
				{
					if (_config.dynamicTyping)
					{
						var value = _r ...
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.babyparse.parse"></a>[function <span class="apidocSignatureSpan">babyparse.</span>parse (_input, _config)](#apidoc.element.babyparse.parse)
- description and source-code
```javascript
function CsvToJson(_input, _config)
	{
		var config = copyAndValidateConfig(_config);
		var ph = new ParserHandle(config);
		var results = ph.parse(_input);
		return results;
	}
```
- example usage
```shell
...
'''

Basic Usage
-----

'''js
// pass in the contents of a csv file
parsed = Baby.parse(csv);

// voila
rows = parsed.data;
'''


Parse File(s)
...
```

#### <a name="apidoc.element.babyparse.parseFiles"></a>[function <span class="apidocSignatureSpan">babyparse.</span>parseFiles (_input, _config)](#apidoc.element.babyparse.parseFiles)
- description and source-code
```javascript
function ParseFiles(_input, _config)
	{
		if (Array.isArray(_input)) {
			var results = [];
			_input.forEach(function(input) {
				if(typeof input === 'object')
					results.push(ParseFiles(input.file, input.config));
				else
					results.push(ParseFiles(input, _config));
			});
			return results;
		} else {
			var results = {
				data: [],
				errors: []
			};
			if ((/(\.csv|\.txt)$/).test(_input)) {
				try {
					var contents = fs.readFileSync(_input).toString();
					return CsvToJson(contents, _config);
				} catch (err) {
					results.errors.push(err);
					return results;
				}
			} else {
				results.errors.push({
					type: '',
					code: '',
					message: 'Unsupported file type.',
					row: ''
				});
				return results;
			}
		}
	}
```
- example usage
```shell
...
Parse File(s)
-----

Baby Parse will assume the input is a filename if it ends in .csv or .txt.

'''js
// Parse single file
parsed = Baby.parseFiles(file[, config])

rows = parsed.data
'''

'''js
// Parse multiple files
// Files can be either an array of strings or objects { file: filename[, config: config] }
...
```

#### <a name="apidoc.element.babyparse.unparse"></a>[function <span class="apidocSignatureSpan">babyparse.</span>unparse (_input, _config)](#apidoc.element.babyparse.unparse)
- description and source-code
```javascript
function JsonToCsv(_input, _config)
	{
		var _output = "";
		var _fields = [];

		// Default configuration
		var _quotes = false;	// whether to surround every datum with quotes
		var _delimiter = ",";	// delimiting character
		var _newline = "\r\n";	// newline character(s)

		unpackConfig();

		if (typeof _input === 'string')
			_input = JSON.parse(_input);

		if (_input instanceof Array)
		{
			if (!_input.length || _input[0] instanceof Array)
				return serialize(null, _input);
			else if (typeof _input[0] === 'object')
				return serialize(objectKeys(_input[0]), _input);
		}
		else if (typeof _input === 'object')
		{
			if (typeof _input.data === 'string')
				_input.data = JSON.parse(_input.data);

			if (_input.data instanceof Array)
			{
				if (!_input.fields)
					_input.fields = _input.data[0] instanceof Array
									? _input.fields
									: objectKeys(_input.data[0]);

				if (!(_input.data[0] instanceof Array) && typeof _input.data[0] !== 'object')
					_input.data = [_input.data];	// handles input like [1,2,3] or ["asdf"]
			}

			return serialize(_input.fields || [], _input.data || []);
		}

		// Default (any valid paths should return before this)
		throw "exception: Unable to serialize unrecognized input";


		function unpackConfig()
		{
			if (typeof _config !== 'object')
				return;

			if (typeof _config.delimiter === 'string'
				&& _config.delimiter.length == 1
				&& Baby.BAD_DELIMITERS.indexOf(_config.delimiter) == -1)
			{
				_delimiter = _config.delimiter;
			}

			if (typeof _config.quotes === 'boolean'
				|| _config.quotes instanceof Array)
				_quotes = _config.quotes;

			if (typeof _config.newline === 'string')
				_newline = _config.newline;
		}


		// Turns an object's keys into an array
		function objectKeys(obj)
		{
			if (typeof obj !== 'object')
				return [];
			var keys = [];
			for (var key in obj)
				keys.push(key);
			return keys;
		}

		// The double for loop that iterates the data and writes out a CSV string including header row
		function serialize(fields, data)
		{
			var csv = "";

			if (typeof fields === 'string')
				fields = JSON.parse(fields);
			if (typeof data === 'string')
				data = JSON.parse(data);

			var hasHeader = fields instanceof Array && fields.length > 0;
			var dataKeyedByField = !(data[0] instanceof Array);

			// If there a header row, write it first
			if (hasHeader)
			{
				for (var i = 0; i < fields.length; i++)
				{
					if (i > 0)
						csv += _delimiter;
					csv += safe(fields[i], i);
				}
				if (data.length > 0)
					csv += _newline;
			}

			// Then write out the data
			for (var row = 0; row < data.length; row++)
			{
				var maxCol = hasHeader ? fields.length : data[row].length;

				for (var col = 0; col < maxCol; col++)
				{
					if (col > 0)
						csv += _delimiter;
					var colIdx = hasHeader && dataKeyedByField ? fields[col] : col;
					csv += safe(data[row][colIdx], col);
				}

				if (row < data.length - 1)
					csv += _newline;
			}

			return csv;
		}

		// Encloses a value around quotes if needed (makes a value safe for CSV insertion)
		function safe(str, col)
		{
			if (typeof str === "undefined" || str === null)
				return "";

			str = str.toString().replace(/"/g, '""');

			var needsQuotes = (typeof _quotes === 'boolean' && _quotes)
							|| (_quotes instanceof Array && _quotes[col])
							|| hasAny(str, Baby.BAD_DELIMITERS)
							|| str.indexOf(_delimiter) > -1
							|| str.charAt(0) == ' '
							|| str.charAt(str.length - 1) == ' ';

			return needsQuotes ? '"' + str + '"' : str;
		}

		function hasAny(str, substrings)
		{
			for (var i = 0; i < substrings.length; i++)
				if (str.indexOf(substrings[i]) > -1)
					return true;
			return false;
		}
	}
```
- example usage
```shell
n/a
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
