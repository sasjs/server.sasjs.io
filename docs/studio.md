---
layout: article
title: Studio
description: SASjs Studio is an executor and log/results manager for SAS and JS code
---

# SASjs Studio

Run SAS or JS code in a single interface!  Just select your preferred runtime from the dropdown next to the running man.

<video controls autoplay loop muted playsinline width="1000" height="646">
  <source src="/img/studio-demo.webm" type="video/webm">
  <source src="/img/studio-demo.mp4" type="video/mp4">
  Running code in SASjs Studio, and switching runtime.
</video>

It is [possible to ctrl-click](https://github.com/sasjs/server/issues/119#issuecomment-1100668248) to run code, or to highlight and select the running man icon.

Code is saved in local storage, so you don't lose your work if you close the browser tab.

## SAS specific

The big difference between SASjs Studio and other SAS executors is that every execution is with a CLEAN session.

Other than that, everything works as you would expect.  The log is sent to the log window, and the WEBOUT page contains any content sent to the _webout fileref.

## JS specific

When running JS code, any `console.log()` statements are sent to the log window, and any content saved in the `_webout` variable is sent to the `webout` page.

## Launch Mode

When clicking the LAUNCH icon, a NEW window (or tab) is opened with the file location in the `_program` url param.  For instance, if opening `/Public/subfolder/someprog.sas` then the url will be: `/SASjsExecutor?_program=/Public/subfolder/someprog.sas`.
