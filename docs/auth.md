# Authorisation

SASjs Server supports 3 authentication methods - Viya auth, Oauth0, and username/password.

## Viya Auth
This approach is recommended whenever Viya authorisation is available. The configuration steps are as follows:

1. Configure SASjs Server for Viya Auth
2. Register a client and secret (administrator task) using the **authorization_code** grant type.  Here are some [helpful resources](https://cli.sasjs.io/faq/#how-can-i-obtain-a-viya-client-and-secret).
3. Enter the client / secret into the SASjs database (tool to be provided)

The user flow is as follows:

1. Navigate to {SASjsServerUrl}/SASjsLogon
2. Click "LOGON"

Behind the scenes, the following flow will occur:

1. SASjs Server Frontend will direct the user to `/SASLogon/oauth/authorize` with the configured CLIENT_ID
2. Following authentication, the Backend will send the CLIENT_ID, CLIENT_SECRET and AUTH_CODE (invisible to the user / dev tools) to the `/SASLogon/oauth/token` endpoint, receiving the ACCESS_TOKEN and REFRESH_TOKEN in response
3. The ACCESS_TOKEN is sent to the Frontend for local storage, and the REFRESH_TOKEN is discarded.  No user tokens are stored at Backend.
4. Subsequent requests by the Frontend, or API clients, will make use of the ACCESS_TOKEN (until expiry).

The following diagram illustrates:

![viya flow](/img/viyaflow.svg)

## Oauth0

Oauth0 is a third party authentication provider, which supports all major authentication methods - such as SAML, SSO, LDAP, Social Media, etc etc.

## Username / Password
This authentication mode is recommended for Desktop Apps (running SAS locally), or - for server deploys - where there is no Viya, no internet acecss, or no appetite for Oauth0.



