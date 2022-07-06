---
layout: article
title: Permissions
description: Configuring SASjs Server permissions
og_image: /img/uri-access.png
---

# Permissions

Permissions in SASjs are enabled when `MODE=server`.   Rules do not apply to admin users.  The menu is available under SETTINGS, and is divided into "URI Access" and "Folder Access".

URI rules can be at GROUP or USER level.

Rules take precedence as follows:

* By default, deny all
* Apply group grants
* Apply group deny
* Apply user grants
* Apply user deny

![](./img/uri-access.png)
## URI Access

URI rules are applied to the following endpoints:

* /AppStream
* /SASjsApi/code/execute
* /SASjsApi/stp/execute
* /SASjsApi/drive/deploy
* /SASjsApi/drive/deploy/upload
* /SASjsApi/drive/file
* /SASjsApi/drive/folder
* /SASjsApi/drive/fileTree
* /SASjsApi/permission

They can also be applied to individual apps.  This part of the list is dynamically generated, for example:

* /AppStream/DataController
* /AppStream/Mario
* /AppStream/Sonic


## Folder Access

Coming soon.  Need it sooner?  [Sponsor us](https://github.com/sponsors/sasjs)!


