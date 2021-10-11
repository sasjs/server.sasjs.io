# Drive

SASjs Drive provides a virtual filesystem for stored programs and other content. It is available on the `/SASjsDrive` endpoint.

It provides the following functionality:

* Explorer window (folders & files)
* Ability to VIEW files (default  action when clicking is to VIEW)
* Ability to EDIT files (button action)
* Ability to EXECUTE files (button action)

## Explorer
The explorer view allows folders to be expanded and collapsed, displaying their content.

## Viewer
The viewer is used to display content, with additional buttons to EDIT and to EXECUTE.

## Editor
In edit mode, when edits are made, the EXECUTE button is greyed out.  After clicking SAVE, the EXECUTE button is re-enabled.

## Executor
When clicking EXECUTE, a NEW window (or tab) is opened with the file location in the `_program` url param.  For instance, if opening `/Public/subfolder/someprog` then the url will be: `/SASjsExecutor?_program=/Public/subfolder/someprog`.

