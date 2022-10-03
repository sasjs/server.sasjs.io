---
layout: article
title: Authentication
description: Configuring Authentication Mechanisms in SASjs Server (LDAP and Internal)
og_image: /img/ldapconfig.png
---

# Auth

SASjs Server currently supports two authentication methods - LDAP, and Internal.  Would you like to see more?  [Sponsor](https://github.com/sponsors/sasjs) us!

Note that authentication is only available in **server mode** (not desktop).

## Internal Authentication

By default, users are created using the internal database with a password configured by an admin.  Groups can also be added, and permissions set against those groups.

## LDAP Authentication

SASjs Server can connect to an LDAP server (internally, we use the [LDAPjs](http://ldapjs.org/client.html) library).  Any users / groups that are imported will be in _addition_ to any internal users / groups.  If there are conflicts, those particular users/groups will not be imported - to fix this, just delete the relevant (SASjs internal) users/groups and re-import.

Note that at least one internal admin user is necessary, to be able to log in and do the import.  After this, the internal user may nominate other (LDAP) users as SASjs admins.

Configuration is made in the following `.env` settings:

```
AUTH_PROVIDERS=ldap
LDAP_URL= <LDAP_SERVER_URL>
LDAP_BIND_DN= <cn=admin,ou=system,dc=cloudron>
LDAP_BIND_PASSWORD = <password>
LDAP_USERS_BASE_DN = <ou=users,dc=cloudron>
LDAP_GROUPS_BASE_DN = <ou=groups,dc=cloudron>
```

Next, restart the server and log in with the admin user. Navigate to the settings tab.  You should see a screen like the below.  Import the users & groups by clicking the 'synchronise' button.

![LDAP in SASjs](img/ldapconfig.png)