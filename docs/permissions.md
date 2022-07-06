---
layout: article
title: Permissions
description: Configuring SASjs Server permissions
---

# Permissions

Permissions in SASjs are enabled when `MODE=server`.   Rules do not apply to admin users.  The menu is available under SETTINGS, and is divided into "URI Access" and "Folder Access".

Rules take precedence as follows:

* By default, deny all
* Apply group grants
* Apply group deny
* Apply user grants
* Apply user deny

## URI Access

API rules are applied to the following:

0: "/AppStream"
1: "/SASjsApi/code/execute"
2: "/SASjsApi/stp/execute"
3: "/SASjsApi/drive/deploy"
4: "/SASjsApi/drive/deploy/upload"
5: "/SASjsApi/drive/file"
6: "/SASjsApi/drive/folder"
7: "/SASjsApi/drive/fileTree"
8: "/SASjsApi/permission"

They can also be applied to individual apps.  This part of the list is dynamically generated, for example:

9: "/AppStream/DataController"
1: "/AppStream/Mario"
1: "/AppStream/Sonic"



## Folder Access

Coming soon.  Need it sooner?  [Sponsor us](https://github.com/sponsors/sasjs)!


