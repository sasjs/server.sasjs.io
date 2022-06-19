---
layout: article
title: Settings
description: Configuring SASjs Server settings
---

# Settings

All settings in SASjs server are made by means of environment variables.  These can be set in the following places:

- Configured globally in `/etc/environment` file
- Export in terminal or shell script (export VAR=VALUE)
- Prepended in the command
- Enter in the `.env` file alongside the executable

Some variables (such as secrets) are removed following initialisation to ensure they aren't available to any child sessions.

# Environment Variables

## MODE

Whether to launch the server in desktop (single user / workstation) or server mode (multi-user).  For server mode, a Mongo DB is required.

Default: `desktop`

## RUN_TIMES
A comma separated string that defines the available runtimes.
**Priority is given to the runtime that comes first in string**.

So given a `RUNTIME=js,sas` if `_program=/some/program` then SASjs Server will look for `program.js` in the `/some` folder before `program.sas`.  If `_program=/some/program.sas` then a SAS runtime will always be used.

In the future we plan to support Python and R runtimes in addition to SAS and JS.

Default: `sas,js`

Example:

`RUN_TIMES=js,sas`

## SAS_PATH
The path to the SAS executable (sas.exe / sas.sh)

Example:

`SAS_PATH=/path/to/sas/executable.exe`

## SASJS_ROOT

If omitted, this will be the SASjs Server installation directory.  The location is used for SAS WORK, staged files, DRIVE, configuration etc

Example:

`SASJS_ROOT=./sasjs_root`

## PROTOCOL

Whether to use http or https protocol. Default: http

## PORT
The port on which to serve.  Default: 5000


## SAS_OPTIONS

Windows only. See: [https://documentation.sas.com/doc/en/pgmsascdc/9.4_3.5/hostwin/p0drw76qo0gig2n1kcoliekh605k.htm#p09y7hx0grw1gin1giuvrjyx61m6](https://documentation.sas.com/doc/en/pgmsascdc/9.4_3.5/hostwin/p0drw76qo0gig2n1kcoliekh605k.htm#p09y7hx0grw1gin1giuvrjyx61m6)

Example:  `SAS_OPTIONS= -NOXCMD`

## SASV9_OPTIONS
Unix only.  See: [https://documentation.sas.com/doc/en/pgmsascdc/9.4_3.5/hostunx/p0wrdmqp8k0oyyn1xbx3bp3qy2wl.htm](https://documentation.sas.com/doc/en/pgmsascdc/9.4_3.5/hostunx/p0wrdmqp8k0oyyn1xbx3bp3qy2wl.htm)

Example: `SASV9_OPTIONS= -NOXCMD`

## PRIVATE_KEY
Only relevant on https / server connections.  This value is passed as the `key:` option to NodeJS as described here:  https://nodejs.org/api/tls.html#tlscreatesecurecontextoptions

Example: `PRIVATE_KEY=privkey.pem`

## FULL_CHAIN
Only relevant on https / server connections.  This value is pased as the `cert:` option to NodeJS as described here: https://nodejs.org/api/tls.html#tlscreatesecurecontextoptions

Example: `FULL_CHAIN=fullchain.pem`

## ACCESS_TOKEN_SECRET
Only relevant in server mode.  Example: `ACCESS_TOKEN_SECRET=somesecret`.

## REFRESH_TOKEN_SECRET
Only relevant in server mode.  Example: `REFRESH_TOKEN_SECRET=somesecret`.

## AUTH_CODE_SECRET
Only relevant in server mode.  Example: `AUTH_CODE_SECRET=somesecret`.

## SESSION_SECRET
Only relevant in server mode.  Example: `SESSION_SECRET=somesecret`.

## SESSION_SECRET
Only relevant in server mode.  Example: `SESSION_SECRET=somesecret`.

## DB_CONNECT
Necessary for server mode.  Connection string for Mongo DB instance.  Example:
```
DB_CONNECT=mongodb+srv://<DB_USERNAME>:<DB_PASSWORD>@<CLUSTER>/<DB_NAME>?retryWrites=true&w=majority
```

## CORS

Options: [disable|enable] 
Default: `disable` for `server` & `enable` for `desktop`

If enabled, be sure to also configure the WHITELIST of any third party servers.

## WHITELIST
Space separated urls, eg: `WHITELIST=http://localhost:3000 https://abc.com`

## HELMET_COEP

HELMET Cross Origin Embedder Policy.  Sets the Cross-Origin-Embedder-Policy header to require-corp when `true`

Options: [true|false] 

Default: true

Docs: [https://helmetjs.github.io/#reference](https://helmetjs.github.io/#reference) (`crossOriginEmbedderPolicy`)

##  HELMET_CSP_CONFIG_PATH

HELMET Content Security Policy

Path to a json file containing HELMET `contentSecurityPolicy` directives

Docs: [https://helmetjs.github.io/#reference](https://helmetjs.github.io/#reference)

Example config:
```
{
  "img-src": ["'self'", "data:"],
  "script-src": ["'self'", "'unsafe-inline'"],
  "script-src-attr": ["'self'", "'unsafe-inline'"]
}
```

Example: `HELMET_CSP_CONFIG_PATH=./csp.config.json`

## LOG_FORMAT_MORGAN

These setting determines the level of logging produced by SASjs server.  
More details on this can be found in the Morgan documentation here:[https://www.npmjs.com/package/morgan#predefined-formats](https://www.npmjs.com/package/morgan#predefined-formats)

Options: [`combined`|`common`|`dev`|`short`|`tiny`]

Default: `common`

