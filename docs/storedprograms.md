# Stored Programs

A stored program is a ".sas" program or ".js" script that has been saved to SASjs Drive.  When the `/SASjsApi/stp/execute` endpoint is called with a `_PROGRAM` URL parameter (pointing to the logical program location), a "Stored Program" is prepared and executed in a SAS or JS [session](/sessions) according to the file extension.

For the usual case where the _program variable does NOT contain an extension (eg `/some/stored/program`) then by default, SASjs Server will attempt to find "program.sas" in the `/some/stored` folder.  To change this default, see the [RUN_TIMES](/settings/#RUN_TIMES) settings.

When Stored Programs are executed, some pre-code is injected to make available various inputs, such as:

* Input Macro Variables
* Input Files
* Utility Macros / Functions

The implementation varies depending on whether it is running as a SAS or JS session:

## SAS Programs

## Input Variables

Any URL parameters passed in the request will be converted into macro variables.  The following variables are "special":

* `_PROGRAM` - this is a required input, and will point to the Stored Program location in SASjs Drive.
* `_DEBUG` - setting this to 131 or greater will result in the log being returned, and the following line added to the start of the program:  `options mprint;`


## Input Files

Any files attached to a request will be saved in the [session folder](/sessions) and the following variables will be added to the SAS program:

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


## JS Programs