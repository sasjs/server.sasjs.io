---
layout: article
title: Stored Programs
description: Stored Programs may be written in SAS, Python, R, or JavaScript.  They are saved to SASjs Drive and can be executed with a GET or POST request.
og_image: /img/stps.png
---

# Stored Programs

Stored Programs may be written in SAS, Python, R, or JavaScript and are saved to SASjs Drive.  When the `/SASjsApi/stp/execute` endpoint is called with a `_PROGRAM` URL parameter (pointing to the logical program location), a "Stored Program" is prepared and executed in a SAS or JS [session](/sessions) according to the file extension.

For the usual case where the `_program` variable does NOT contain an extension (eg `/some/stored/program`) then by default, SASjs Server will attempt to find "program.sas" in the `/some/stored` folder.  To change this default, see the [RUN_TIMES](/settings/#RUN_TIMES) settings.

When Stored Programs are executed, some pre-code is injected to make available various inputs, such as:

* Input Macro Variables
* Input Files
* Utility Macros / Functions

The implementation varies depending on whether it is running as a SAS, JS, Python or R session.

## Special Variables

The following variables are "special":

### `_PROGRAM`
This is a required input, and will point to the Stored Program location in SASjs Drive.  The extension may be omitted, subject to the behaviour in the [RUN_TIMES](/settings/#RUN_TIMES) setting.

### `_DEBUG`
Setting this to 131 or greater will result in the log being returned.  The log will have the following UUID, which can be used as a seperator:  `SASJS_LOGS_SEPARATOR_163ee17b6ff24f028928972d80a26784`

Setting `_DEBUG` also results in the creation of a `Content-Type` header of `text/plain`.

## SAS Programs

### Input Variables

Any URL parameters (eg `&name1=value1&something=else`) are attached as macro variables eg as follows:

```sas
%let name1=value1;
%let something=else;
```

A number of "fixed" variables are also added at the start of the program - you can see the code that generates these in the repo, [here](https://github.com/sasjs/server/blob/main/api/src/controllers/internal/Execution.ts#L93).

If `_DEBUG` is set then `options mprint;` will also be automatically added to the start of the Stored Program.


### Input Files

Any files attached to a request will be saved in the [session folder](/sessions) and the following variables will be dynamically added to the SAS program:

* `_WEBIN_FILE_COUNT` - contains an integer, from 0 to the number of files to being provided. This variable is always created.
* `_WEBIN_FILENAME1` - the value that was specified in the FILENAME attribute by the frontend
* `_WEBIN_FILEREF1` - An 8 letter code, starting with an underscore and finishing with 7 random alphanumeric characters
* `_WEBIN_NAME1` - the value that was specified in the NAME attribute by the frontend

We will also create the fileref(s) using the filename statement.

To illustrate with an example - we are uploading two files, F1 and F2. The following SAS code will be generated, and inserted at the beginning of the executed program:

```sas
/* request files */
filename _SJS0001 "/some/temp/location/file1";
filename _SJS0002 "/some/temp/location/file2";
%let _WEBIN_FILE_COUNT=2;
%let _WEBIN_FILENAME1=myfile.csv;
%let _WEBIN_FILENAME2=myotherfile.txt;
%let _WEBIN_FILEREF1=_SJS0001;
%let _WEBIN_FILEREF2=_SJS0002;
%let _WEBIN_NAME1=F1;
%let _WEBIN_NAME2=F2;
```

If there are no files uploaded, only the following code will be generated:

```sas
/* request files */
%let _WEBIN_FILE_COUNT=0;
```

### Output

Any response data should be written to the `_webout` fileref (which is pre-assigned in the session).  The corresponding header records (content type etc) can be written using the [mfs_httpheader](https://core.sasjs.io/mfs__httpheader_8sas.html) macro. Additional [SASjs Server specific macros](https://core.sasjs.io/dir_41e1742e44e2de38b3bc91f993fed282.html) are also available.

The response (content and header) will vary depending on the [`_debug`](#_debug) setting.

## JS Programs

The use of JS "programs" as your backend is highly beneficial for the following use cases:

* You would like to test your application in a non SAS environment (such as a CI/CD pipeline or developer machine)
* You would like to make use of JS packages in your backend SAS applications
* You would like to provide backup services in the case that SAS is down (eg licence expiry)

Any `console.log()` statements written by your JavaScript will be returned in the response "log" (identically to SAS). 

If your Job / Service / Test depends on any third party libraries / packages, then you need to bundle that service before deploying it to sasjs server. You can choose any bundler for this process. One possible option could be `webpack`. If you choose webpack then you can use it like this:

`npx webpack --mode none --target node --entry <path-of-file-to-bundle/build> --output-path <path-of-output-folder> --output-filename <name-of-output-file>`

For instance if you have a mock service at path `sasjs/services/common/appinit.js` and you want to build and make it ready for deployment, you can run the following in the terminal:

`npx webpack --mode none --target node --entry ./sasjs/services/common/appinit.js --output-path sasjsbuild/services/common --output-filename appinit.js`

Repeat this for any other JS Stored Programs as needed.

Finally, run `sasjs build` to create the deployment pack, and `sasjs deploy` to deploy.


### Input Variables

Any URL parameters (eg `&name1=value1&something=else`) are attached as JS variables eg as follows:

```js
const name1='value1'
const something='else'
```

A number of "fixed" variables are also added at the start of the program, eg:

```js
let _webout = '';
weboutPath = '/home/sasjssrv/sasjs_root/sessions/20220808081136-29012-1659946296436/webout.txt'; 
const _sasjs_tokenfile = '/home/sasjssrv/sasjs_root/sessions/20220808081136-29012-1659946296436/reqHeaders.txt';
const _sasjs_username = 'allan';
const _sasjs_userid = '6';
const _sasjs_displayname = 'Allan Bowe';
const _metaperson = _sasjs_displayname;
const _metauser = _sasjs_username;
const sasjsprocessmode = 'Stored Program';
```


### Input Files

Any files attached to a request will be saved in the [session folder](/sessions) and the following variables will be dynamically added to the JS program:

* `_WEBIN_FILE_COUNT` - contains an integer, from 0 to the number of files to being provided. This variable is always created.
* `_WEBIN_FILENAME1` - the value that was specified in the FILENAME attribute by the frontend
* `_WEBIN_FILEREF1` - This will contain the actual file object
* `_WEBIN_NAME1` - the value that was specified in the NAME attribute by the frontend

To illustrate with an example - we are uploading two files, F1 and F2. The following JS code will be generated, and inserted at the beginning of the executed program:

```js
const _WEBIN_FILEREF1 = fs.readFileSync('/path/to/file1')
const _WEBIN_FILEREF2 = fs.readFileSync('/path/to/file2')
const _WEBIN_FILENAME1 = 'F1'
const _WEBIN_FILENAME2 = 'F2'
const _WEBIN_NAME1 = 'FormRef1'
const _WEBIN_NAME2 = 'FormRef2'

const _WEBIN_FILE_COUNT = 2

```

If there are no files uploaded, only the following code will be generated:

```js
const _WEBIN_FILE_COUNT = 0
```

### Output

The following variables are "special":

#### `_webout`
Any content added to this variable will be stringified and returned to the browser by SASjs Server.   

#### `_SASJS_WEBOUT_HEADERS`
This variable points to a text file where header records (such as `Content-type: application/zip`) can be written

Example:

```
fs.writeFile(_SASJS_WEBOUT_HEADERS, 'Content-type: application/zip', function (err) {
    if (err) throw err;
})
```

So now the response that will be returned will contain `content type` appended in the file.

## Python Programs

The use of Python "programs" as your backend is highly beneficial for the following use cases:

* You would like to test your application in a non SAS environment (such as a CI/CD pipeline or developer machine)
* You would like to make use of Python packages in your SAS applications
* You would like to provide backup services in case SAS is unavailable

Any content written to `sys.stdout` or `sys.stderr` will be returned in the response "log" (identically to SAS). 

### Input Variables

Any URL parameters (eg `&name1=value1&something=else`) are attached as Python variables in uppercase, eg as follows:

```python3
NAME1='value1'
SOMETHING='else'
```

A number of "fixed" variables are also added at the start of the program, eg as follows:

```python
_SASJS_TOKENFILE = '/home/sasjssrv/sasjs_root/sessions/20220808081136-29012-1659946296436/reqHeaders.txt';
_SASJS_USERNAME = 'allan';
_SASJS_USERID = '6';
_SASJS_DISPLAYNAME = 'Allan Bowe';
_METAPERSON = _SASJS_DISPLAYNAME;
_METAUSER = _SASJS_USERNAME;
SASJSPROCESSMODE = 'Stored Program';
```

### Input Files

Any files attached to a request will be saved in the [session folder](/sessions).  The session folder (with the input files) will also be the current folder, eg `os.chdir("/path/to/session/folder")`. 

When input files exist, the following variables will also be dynamically added to the Python program:

* `_WEBIN_FILE_COUNT` - contains an integer, from 0 to the number of files to being provided. This variable is always created.
* `_WEBIN_FILENAME1` - the value that was specified in the FILENAME attribute by the frontend
* `_WEBIN_NAME1` - the value that was specified in the NAME attribute by the frontend

To illustrate with an example - we are uploading two files to a Python Stored Program, namely f1.txt and f2.xls. The following Python code will be generated, and inserted at the beginning of the executed program:

```python
_WEBIN_FILENAME1 = 'f1.txt'
_WEBIN_FILENAME2 = 'f2.xls'
_WEBIN_NAME1 = 'FormRef1'
_WEBIN_NAME2 = 'FormRef2'

_WEBIN_FILE_COUNT = 2
```

If there are no files uploaded, only the following code will be generated:

```python
_WEBIN_FILE_COUNT = 0
```

Note that there are no `_WEBIN_FILEREF` variables created - in Python it is necessary to know the type of file (eg binary / text) before it can be ingested with the `open()` function, eg `open(_WEBIN_FILENAME1,"rt")` or `open(_WEBIN_FILENAME2,"rb")`.  Therefore it is left as an exercise for the developer to ingest as appropriate.


### Output

The following variables are "special":

#### `_WEBOUT`
This variable points to a text file in the session folder (eg `_WEBOUT="/path/to/session/folder/webout.txt"`).  All content written to `_WEBOUT` is streamed in the STP API call result.

Example code:

```python
f = open(_WEBOUT, 'w')
f.write('{"hello":"world"}')
```

#### `HEADERSPATH`
This variable points to a text file where header records (such as `Content-type: application/zip`) can be written, eg `HEADERSPATH="/path/to/session/folder/headers.txt"`).

To return data to the client, just write it to the `_WEBOUT` file.  It will then be returned to the browser / client application.  Be sure to set the `Content-Type` in the `HEADERSPATH` file if it is anything other than JSON.

## R Programs

Stored Programs may also run using the language of R.  Any content written to `.ROut` will be returned in the response "log" (identically to SAS).  As R does not support the underscore (`_`) as the first character in a variable name, the automatic variables are preceded with a period (`.`) instead.

### Input Variables

Any URL parameters (eg `&name1=value1&something=else`) are attached as R variables in uppercase, eg as follows:

```r
NAME1 <- 'value1'
SOMETHING <- 'else'
```

A number of "fixed" variables are also added at the start of the program, eg as follows:

```r
._SASJS_TOKENFILE <- '/home/sasjssrv/sasjs_root/sessions/20220808081136-29012-1659946296436/reqHeaders.txt';
._SASJS_USERNAME <- 'allan';
._SASJS_USERID <- '6';
._SASJS_DISPLAYNAME <- 'Allan Bowe';
._METAPERSON <- ._SASJS_DISPLAYNAME;
._METAUSER <- ._SASJS_USERNAME;
SASJSPROCESSMODE <- 'Stored Program';
```

### Input Files

Any files attached to a request will be saved in the [session folder](/sessions).  The session folder (with the input files) will also be the current folder, eg `setwd("/path/to/session/folder")`. 

When input files exist, the following variables will also be dynamically added to the R program:

* `._WEBIN_FILE_COUNT` - contains an integer, from 0 to the number of files to being provided. This variable is always created.
* `._WEBIN_FILENAME1` - the value that was specified in the FILENAME attribute by the frontend
* `._WEBIN_NAME1` - the value that was specified in the NAME attribute by the frontend

To illustrate with an example - we are uploading two files to an R Stored Program, namely f1.txt and f2.xls. The following R code will be generated, and inserted at the beginning of the executed program:

```r
._WEBIN_FILENAME1 <- 'f1.txt'
._WEBIN_FILENAME2 <- 'f2.xls'
._WEBIN_NAME1 <- 'FormRef1'
._WEBIN_NAME1 <- 'FormRef2'

._WEBIN_FILE_COUNT <- 2
```

If there are no files uploaded, only the following code will be generated:

```r
._WEBIN_FILE_COUNT <- 0
```

Note that there are no `._WEBIN_FILEREF` variables created - in R it is necessary to know the type of file (eg binary / text) before it can be ingested, eg `read.delim(._WEBIN_FILENAME1, header = FALSE)` or `read.delim2(._WEBIN_FILENAME2, header = FALSE)`.  Therefore it is left as an exercise for the developer to ingest as appropriate.


### Output

The following variables are "special":

#### `._WEBOUT`
This variable points to a text file in the session folder (eg `._WEBOUT <- "/path/to/session/folder/webout.txt"`).  All content written to `._WEBOUT` is streamed in the STP API call result.

Example code:

```r
cat('{"hello":"world"}',file=._WEBOUT)
```

#### `HEADERSPATH`
This variable points to a text file where header records (such as `Content-type: application/zip`) can be written, eg `HEADERSPATH <- "/path/to/session/folder/headers.txt"`).

To return data to the client, just write it to the `._WEBOUT` file.  It will then be returned to the browser / client application.  Be sure to set the `Content-Type` in the `HEADERSPATH` file if it is anything other than JSON.


