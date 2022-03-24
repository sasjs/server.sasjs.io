# Stored Programs

A stored program is a file with a ".sas" extension that has been saved to the SASjs Drive.  When the `/SASjsApi/stp/execute` endpoint is called with a `_PROGRAM` URL parameter (pointing to the logical program location), a "Stored Program" is prepared and executed in a SASjs [session](/sessions).

"Preparation" involves adding some precode to provide the following:

* Input Macro Variables
* Input Files
* Input Macros

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

