# Stored Programs

A Stored Program is a ".sas" program or ".js" script that has been saved to SASjs Drive.  When the `/SASjsApi/stp/execute` endpoint is called with a `_PROGRAM` URL parameter (pointing to the logical program location), a "Stored Program" is prepared and executed in a SAS or JS [session](/sessions) according to the file extension.

For the usual case where the _program variable does NOT contain an extension (eg `/some/stored/program`) then by default, SASjs Server will attempt to find "program.sas" in the `/some/stored` folder.  To change this default, see the [RUN_TIMES](/settings/#RUN_TIMES) settings.

When Stored Programs are executed, some pre-code is injected to make available various inputs, such as:

* Input Macro Variables
* Input Files
* Utility Macros / Functions

The implementation varies depending on whether it is running as a SAS or JS session:

## SAS Programs

### Input Variables

Any URL parameters (eg `&name1=value1&something=else`) are attached as macro variables eg as follows:

```sas
%let name1=value1;
%let something=else;
```

A number of "fixed" variables are also added at the start of the program - you can see the code that generates these in the repo, [here](https://github.com/sasjs/server/blob/main/api/src/controllers/internal/Execution.ts#L93).

The following variables are "special":

#### _PROGRAM
This is a required input, and will point to the Stored Program location in SASjs Drive.  The extension may be omitted, subject to the behaviour in the [RUN_TIMES](/settings/#RUN_TIMES) setting.

#### _DEBUG
Setting this to 131 or greater will result in the log being returned, and the following line added to the start of the program:  `options mprint;`.  The resulting output behaviour of SASjs server will depend on the http method:

* GET request:  Setting debug will result in a Content-Type header of 'text/plain', and the log will be streamed in the response body.
* POST request: The log will be returned in the `log` attribute of the response JSON object.


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
%let _WEBIN_NAME1=F2;
```

If there are no files uploaded, only the following code will be generated:

```sas
/* request files */
%let _WEBIN_FILE_COUNT=0;
```

### Output

Any response data should be written to the `_webout` fileref (which is pre-assigned in the session).  The corresponding header records (content type etc) can be written using the [mfs_httpheader](https://core.sasjs.io/mfs__httpheader_8sas.html) macro. Additional [SASjs Server specific macros](https://core.sasjs.io/dir_41e1742e44e2de38b3bc91f993fed282.html) are also available.

Note that the response will be served differently depending on which http method is used:

* GET request: The response will be streamed according to the headers, and the _debug setting
* POST request: The response is always JSON, with _webout contents stringified into the `webout` attribute of the response.


## JS Programs

The use of JS "programs" as your backend is highly beneficial for the following use cases:

* You would like to test your application in a non SAS environment (such as a CI/CD pipeline or developer machine)
* You would like to make use of JS packages in your backend SAS applications
* You would like to provide backup services in the case that SAS is down (eg licence expiry)

Any `console.log()` statements written by your JavaScript will be returned in the response "log" (identically to SAS). 

### Input Variables

Any URL parameters (eg `&name1=value1&something=else`) are attached as macro variables eg as follows:

```js
const name1='value1'
const something='else'
```

A number of "fixed" variables are also added at the start of the program - you can see the code that generates these in the repo, [here](https://github.com/sasjs/server/blob/main/api/src/controllers/internal/Execution.ts).

The following variables are "special":

#### `_webout`
Any content added to this variable will be stringified and returned to the browser by SASjs Server.   

#### headersPath
This variable points to a text file where header records (such as `Content-type: application/zip`) can be written


### Input Files

Any files attached to a request will be saved in the [session folder](/sessions) and the following variables will be dynamically added to the JS program:

* `_WEBIN_FILE_COUNT` - contains an integer, from 0 to the number of files to being provided. This variable is always created.
* `_WEBIN_FILENAME1` - the value that was specified in the FILENAME attribute by the frontend
* `_WEBIN_FILEREF1` - This will contain the actual file object
* `_WEBIN_NAME1` - the value that was specified in the NAME attribute by the frontend

To illustrate with an example - we are uploading two files, F1 and F2. The following SAS code will be generated, and inserted at the beginning of the executed program:

```js
const _WEBIN_FILEREF1 = fs.readFileSync('/path/to/file1')
const _WEBIN_FILEREF2 = fs.readFileSync('/path/to/file2')
const _WEBIN_FILENAME1 = 'F1'
const _WEBIN_FILENAME2 = 'F2'
const _WEBIN_NAME1 = 'FormRef1'
const _WEBIN_NAME1 = 'FormRef2'

const _WEBIN_FILE_COUNT = 2

```

If there are no files uploaded, only the following code will be generated:

```js
const _WEBIN_FILE_COUNT = 0
```
