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

## Environment Variables

### MODE

Whether to launch the server in `desktop` (single user / workstation) or `server` mode (multi-user).  For server mode, a Mongo DB connection string must be provided in the [DB_CONNECT](#db_connect) variable.

Default: `desktop`

### RUN_TIMES
A comma separated string that defines the available runtimes.

**Priority is given to the runtime that comes first in string**.

So given a `RUNTIME=js,sas` if `_program=/some/program` then SASjs Server will look for `program.js` in the `/some` folder before `program.sas`.  If `_program=/some/program.sas` then a SAS runtime will always be used.

In the future we plan to support Python and R runtimes in addition to SAS and JS.

Default: `sas,js`

Example:

`RUN_TIMES=js,sas`

### SAS_PATH
The full path to the SAS executable (sas.exe / sas.sh).  It is highly recommended to provide an instance with UTF-8 encoding, for the following reasons:

* SASjs Adapter compatibility
* Broad language support
* Viya compatibility

To force UTF-8 encoding, update the appropriate `sasv9.cfg` file with the following option:

```
-ENCODING UTF-8
```

Example:

`SAS_PATH=/path/to/sas/executable.exe`

### SASJS_ROOT

If omitted, this will be the SASjs Server installation directory.  The location is used for SAS WORK, staged files, DRIVE, configuration etc

Example:

`SASJS_ROOT=./sasjs_root`

### PROTOCOL

Whether to use http or https protocol. Default: http

### PORT
The port on which to serve.  Default: 5000

Binding processes to ports in the lower ranges (eg 80, 443) requires elevated privileges.  To avoid running SASjs Server under a privileged account, you can bind the port to an executable - eg:  `setcap 'cap_net_bind_service=+ep' /home/sasjssrv/api-linux`

If the executable is updated (eg downloading a new version) you will need to run this command again.


### SAS_OPTIONS

Windows only. See: [https://documentation.sas.com/doc/en/pgmsascdc/9.4_3.5/hostwin/p0drw76qo0gig2n1kcoliekh605k.htm#p09y7hx0grw1gin1giuvrjyx61m6](https://documentation.sas.com/doc/en/pgmsascdc/9.4_3.5/hostwin/p0drw76qo0gig2n1kcoliekh605k.htm#p09y7hx0grw1gin1giuvrjyx61m6)

Example:  `SAS_OPTIONS= -NOXCMD`

### SASV9_OPTIONS
Unix only.  See: [https://documentation.sas.com/doc/en/pgmsascdc/9.4_3.5/hostunx/p0wrdmqp8k0oyyn1xbx3bp3qy2wl.htm](https://documentation.sas.com/doc/en/pgmsascdc/9.4_3.5/hostunx/p0wrdmqp8k0oyyn1xbx3bp3qy2wl.htm)

Example: `SASV9_OPTIONS= -NOXCMD`

### PRIVATE_KEY
Only relevant on https / server connections.  This value is passed as the `key:` option to NodeJS as described here:  [https://nodejs.org/api/tls.html#tlscreatesecurecontextoptions](https://nodejs.org/api/tls.html#tlscreatesecurecontextoptions)

Example: `PRIVATE_KEY=localhost.key`

### CERT_CHAIN
Only relevant on https / server connections.  This value is pased as the `cert:` option to NodeJS as described here: [https://nodejs.org/api/tls.html#tlscreatesecurecontextoptions](https://nodejs.org/api/tls.html#tlscreatesecurecontextoptions)

Example: `CERT_CHAIN=localhost.crt`

### CA_ROOT

Only relevant on https / server connections.  This value is pased as the `ca:` option to NodeJS as described here: [https://nodejs.org/api/tls.html#tlscreatesecurecontextoptions](https://nodejs.org/api/tls.html#tlscreatesecurecontextoptions)

Example: `CA_ROOT=fullchain.pem`


### DB_CONNECT
Necessary for server mode.  Connection string for Mongo DB instance.  Example:
```
DB_CONNECT=mongodb+srv://<DB_USERNAME>:<DB_PASSWORD>@<CLUSTER>/<DB_NAME>?retryWrites=true&w=majority
```

### CORS

Options: [disable|enable]
Default: `disable` for `server` & `enable` for `desktop`

If enabled, be sure to also configure the WHITELIST of any third party servers.

### WHITELIST
Space separated urls, eg: `WHITELIST=http://localhost:3000 https://abc.com`

### HELMET_COEP

HELMET Cross Origin Embedder Policy.  Sets the Cross-Origin-Embedder-Policy header to require-corp when `true`

Options: [true|false]

Default: true

Docs: [https://helmetjs.github.io/#reference](https://helmetjs.github.io/#reference) (`crossOriginEmbedderPolicy`)

###  HELMET_CSP_CONFIG_PATH

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

### LOG_FORMAT_MORGAN

These setting determines the level of logging produced by SASjs server.
More details on this can be found in the Morgan documentation here:[https://www.npmjs.com/package/morgan#predefined-formats](https://www.npmjs.com/package/morgan#predefined-formats)

Options: [`combined`|`common`|`dev`|`short`|`tiny`]

Default: `common`

### LOG_LOCATION

Location in which to write server logs (one file per day).  If not provided, logs are written in a `/logs` subfolder of the `SASJS_ROOT` location.  Can be a full path, else relative to the directory in which the server instance was launched.

Example: `LOG_LOCATION=./sasjs_root/logs`

![](img/log_location.png)
