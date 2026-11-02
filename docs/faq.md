---
layout: article
title: Frequently Asked SASjs Server Questions
description: Common questions and answers when working with SASjs Server
---

# FAQ

## Where can I learn more?

The following resources may be helpful:

* [Deploying SASjs Server on a VPS](https://sasapps.io/sasjs-server-on-vps)
* [Load Balancing with SASjs](https://sasapps.io/load-balancing-sasjs-server)
* Youtube [playlist](https://www.youtube.com/playlist?list=PLkepSRl0qS449OqZ1rCCVxjokBJuLBXLI)
* Github: [https://github.com/sasjs/server](https://github.com/sasjs/server)
* RawSAS [blog site](https://rawsas.com/category/sasjs-server/)


## Do I need a server?

No!  You can run SASjs Server on your laptop or computer - so long as you have a local instance of SAS.

## Will this work on SAS Viya?

Yes - through the `sasviya` runtime, which submits each program to a compute session on the Viya server and relays the session log and the program's output back to the caller. Everything the server already does then applies to Viya: bearer-token auth, the group and permission model, SASjs Drive as the code store, `/SASjsApi/stp/execute` and `/trigger` with session-state polling, and the webout contract.

Enable it by listing `sasviya` in [`RUN_TIMES`](/settings/#run_times) and supplying the connection details:

```
RUN_TIMES=js,sasviya
VIYA_URL=https://sas.example.com
VIYA_USER=sasjs-service
VIYA_PASSWORD=...
VIYA_CONTEXT=Compute Reusable
```

All three of `VIYA_URL`, `VIYA_USER` and `VIYA_PASSWORD` are checked at startup, so a typo fails the boot rather than every request. The server mints an access token with an OAuth2 password grant against `sas.cli`, the public client every Viya deployment registers, so no administrator has to create an OAuth client - only a service account with access to the compute context. `VIYA_CONTEXT` defaults to `Compute Reusable`.

There is no local SAS executable in a Viya deployment to point `SAS_PATH` at; `SAS_PATH` and the `sas` runtime are for a local or SAS 9 binary.

A program runs on Viya when it is stored with the `.sasviya` extension - `_program=/Public/app/run.sasviya`, or `runTime: "sasviya"` on `/SASjsApi/code/execute`. A `.sas` file always uses the local runtime, which is why the extension matters rather than the position in `RUN_TIMES`.

Five things differ from a local SAS session, all consequences of running on the compute server:

* the program writes its response with `%mv_webout`, the Viya-native macro from @sasjs/core - and that macro must already exist on the estate, because the server injects no `SASAUTOS` or header paths;
* uploaded files are not relayed: the upload staging points at paths on the SASjs Server host, which the compute server cannot read, so a service that reads an uploaded table needs the `sas` runtime;
* there is no print output - the response carries the webout and the log only;
* the session reports no `pid`, because there is no process on the server host;
* every run executes as the one service account in `VIYA_USER`, so write access to SASjs Drive is the Viya privilege boundary in this release. Per-user Viya identities are not supported.

The program also sees Viya's filesystem and caslibs, not the SASjs Server's disk - Drive files are not visible to it unless they exist on the estate.

See [Settings](/settings/#viya_url) for each variable, and [Sessions](/sessions/) for how a session's log and output are read back.

## Will this work on SAS EBI?

Yes - just point to the sas executable in the SASHOME folder

## Will this work with SAS University Edition

No.  If you are running University Edition, or the free / online learning instance, you cannot use SASjs.  This is because there is no connectivity outside of these environments.

