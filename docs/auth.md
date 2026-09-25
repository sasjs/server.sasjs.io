---
layout: article
title: Authentication
description: Configuring Authentication Mechanisms in SASjs Server (Internal, LDAP and OpenID Connect)
og_image: /img/ldapconfig.png
---

# Authentication

SASjs Server supports three authentication methods - Internal, LDAP, and OpenID Connect (OIDC).  Would you like to see more?  [Sponsor](https://github.com/sponsors/sasjs) us!

These are selected with [AUTH_PROVIDERS](/settings/#auth_providers), which is a **list** - more than one may be enabled at once, for example LDAP for directory users together with OIDC for single sign-on.

Note that authentication is only available in **server mode** (not desktop).

## Internal Authentication

By default, users are created using the internal database with a password configured by an admin.  Groups can also be added, and permissions set against those groups.

## LDAP Authentication

SASjs Server can connect to an LDAP server (internally, we use the [LDAPjs](http://ldapjs.org/client.html) library).  Any users / groups that are imported will be in _addition_ to any internal users / groups.  If there are conflicts, those particular users/groups will not be imported - to fix this, just delete the relevant (SASjs internal) users/groups and re-import.

Note that at least one internal admin user is necessary, to be able to log in and do the import.  After this, the internal user may nominate other (LDAP) users as SASjs admins.

Configuration is made in the following `.env` settings:

```
AUTH_PROVIDERS=ldap
LDAP_URL= ldaps://LDAP_SERVER_URL:PORT
LDAP_BIND_DN= cn=admin,ou=system,dc=companyname
LDAP_BIND_PASSWORD = <password>
LDAP_USERS_BASE_DN = ou=users,dc=companyname
LDAP_GROUPS_BASE_DN = ou=groups,dc=companyname
```

Next, restart the server and log in with the admin user. Navigate to the settings tab.  You should see a screen like the below.  Import the users & groups by clicking the 'synchronise' button.

![LDAP in SASjs](img/ldapconfig.png)

## OpenID Connect Authentication

SASjs Server can act as an OpenID Connect relying party, so users can sign in with any compliant provider (Okta, Entra ID, Keycloak, Google, Auth0, and so on).  It is driven entirely by the provider's own discovery document, so no provider-specific code is involved - only configuration.

When OIDC is configured, a **"Sign in with ..." button** appears on the login screen automatically.  The password form stays below it, so internal and LDAP users are unaffected - and a local admin can still sign in if the provider is unavailable.

### Configuration

```env
AUTH_PROVIDERS=oidc
OIDC_ISSUER_URL=https://id.example.com
OIDC_CLIENT_ID=sasjs-server
OIDC_CLIENT_SECRET=<client secret>
OIDC_REDIRECT_URI=https://sas.example.com/SASLogon/openid/callback
```

The endpoints are then read from `<OIDC_ISSUER_URL>/.well-known/openid-configuration`.  If your provider does not serve discovery at that path, supply it directly with [OIDC_DISCOVERY_URL](/settings/#oidc_discovery_url) instead.

Register `OIDC_REDIRECT_URI` with your provider as the callback URL - the full URL, including the `/SASLogon/openid/callback` path.  It must match exactly.

The remaining settings are optional and are listed under [Settings](/settings/#oidc_issuer_url).

### How users are matched

The provider's `sub` claim is the durable identity.  On each sign-in:

1. The `sub` is looked up in the SASjs user table.  A match signs that user in, regardless of what the username claim says.
2. Otherwise, the username claim (`preferred_username` by default) is normalised to a SASjs username - lowercase, alphanumeric, up to 16 characters - and a user with that name is created.  The **first** user to sign in becomes a SASjs admin; after that, new users are created as normal users.
3. If a user with that normalised name **already exists** and is not already linked to that provider account, the sign-in is refused rather than adopting the account.  This is deliberate: the existing account may hold a local password and be an administrator.  An admin must resolve the clash (rename one of the two) before that person can sign in.

Note that OIDC provides authentication only.  Groups and permissions are managed inside SASjs Server - see [Authorisation](/permissions).  If you need directory groups, use LDAP alongside OIDC.

### Logout

The `/SASLogon/openid/logout` route clears the SASjs session and, when the provider advertises an end-session endpoint, redirects there so the single sign-on session is closed too.

### Example: Cloudron

Cloudron's user directory is an OIDC provider, and its addon exports the values you need.  Map them onto the generic settings - no Cloudron-specific code is involved:

```env
AUTH_PROVIDERS=oidc
OIDC_ISSUER_URL=$CLOUDRON_OIDC_ISSUER
OIDC_DISCOVERY_URL=$CLOUDRON_OIDC_DISCOVERY_URL
OIDC_CLIENT_ID=$CLOUDRON_OIDC_CLIENT_ID
OIDC_CLIENT_SECRET=$CLOUDRON_OIDC_CLIENT_SECRET
OIDC_PROVIDER_NAME=$CLOUDRON_OIDC_PROVIDER_NAME
OIDC_REDIRECT_URI=https://$CLOUDRON_APP_DOMAIN/SASLogon/openid/callback
```

and declare the callback in the app manifest:

```json
"addons": {
  "oidc": {
    "loginRedirectUri": "/SASLogon/openid/callback",
    "logoutRedirectUri": "/",
    "tokenSignatureAlgorithm": "RS256"
  }
}
```

Cloudron asserts the username as the `sub` claim and does not send group claims, so groups are managed in SASjs Server.

## Brute Force Protection

SASjs Server now protects authentication endpoints from DDoS and brute force attacks at any scale. We adopted a simple and powerful technique to block authorization attempts using two metrics:

1. The first is number of consecutive failed attempts by the same user name and IP address.
2. The second is number of failed attempts from an IP address over some long period of time. For example, block an IP address if it makes 100 failed attempts in one day.

To achieve above metrics we used an npm package [rate-limiter-flexible](https://www.npmjs.com/package/rate-limiter-flexible).

This technique has following configurable env variables:

```
# After this, access is blocked for 1 day
# default: 100
MAX_WRONG_ATTEMPTS_BY_IP_PER_DAY = <number>

# After this, access is blocked for an hour
# Store number for 90 days since first fail
# Once a successful login is attempted, it resets
# Default: 10
MAX_CONSECUTIVE_FAILS_BY_USERNAME_AND_IP = <number>
```

## Admin Account

The default credentials for login are `secretuser` and `secretpassword`.  These can be adjusted using the [ADMIN_USERNAME](/settings/#admin_username) and [ADMIN_PASSWORD_INITIAL](/settings/#admin_password_initial) options on server startup.  

If the admin password is misplaced, it can be reset by restarting the server with [ADMIN_PASSWORD_RESET](/settings/#admin_password_reset) set to `YES`.  Be sure to set it back to `NO` (or remove the option) to prevent the password being reset on any subsequent server restart.
