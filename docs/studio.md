---
layout: article
title: Studio
description: SASjs Studio is an executor and log/results manager for SAS and JS code
---

# SASjs Studio

Run SAS or JS code in a single interface!  Just select your preferred runtime from the dropdown next to the running man.

![Untitled_ Jun 19, 2022 8_31 PM](https://user-images.githubusercontent.com/4420615/174497164-fbe87231-a4e9-4de2-b5cc-d5328deff093.gif)

It is [possible to ctrl-click](https://github.com/sasjs/server/issues/119#issuecomment-1100668248) to run code, or to highlight and select the running man icon.

Code is saved in local storage, so you don't lose your work if you close the browser tab.

## SAS specific

The big difference between SASjs Studio and other SAS executors is that every execution is with a CLEAN session.

Other than that, everything works as you would expect.  The log is sent to the log window, and the WEBOUT page contains any content sent to the _webout fileref.

## JS specific

When running JS code, any `console.log()` statements are sent to the log window, and any content saved in the `_webout` variable is sent to the `webout` page.
