---
layout: article
title: Permissions
description: Configuring SASjs Server permissions
---

# Permissions - COMING SOON

## URI Access

* By default, deny everything
* Apply group grants
* Apply group deny
* Apply user grants
* Apply user deny

Permissions applied at top level (eg /SASjsApi) cascade to lower levels (eg /SASjsApi/group)

Permissions at lower level (eg /SASjsApi/group) have higher priority than upper levels (eg /SASjsApi)