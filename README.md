# NAME

Plack::Auth::SSO::OIDC - implementation of OpenID Connect for Plack::Auth::SSO

# STATUS

[![Build Status](https://travis-ci.org/LibreCat/Plack-Auth-SSO-OIDC.svg?branch=master)](https://travis-ci.org/LibreCat/Plack-Auth-SSO-OIDC)
[![Coverage](https://coveralls.io/repos/LibreCat/Plack-Auth-SSO-OIDC/badge.png?branch=master)](https://coveralls.io/r/LibreCat/Plack-Auth-SSO-OIDC)
[![CPANTS kwalitee](http://cpants.cpanauthors.org/dist/Plack-Auth-SSO-OIDC.png)](http://cpants.cpanauthors.org/dist/Plack-Auth-SSO-OIDC)

# DESCRIPTION

This is an implementation of [Plack::Auth::SSO](https://metacpan.org/pod/Plack::Auth::SSO) to authenticate against a CAS server.

It inherits all configuration options from its parent.

# CONFIG

- idp\_url

    base url of the OIDC service.

    The openid configuration is expected at and retrieved from ${idp\_url}/.well-known/openid-configuration

- client\_id

    client-id as given by the OIDC service

- client\_secret

    client-secret as given by the OIDC service

- scope

    Scope requested from the OIDC service.

    Space separated string containing all scopes

    Default: `"openid profile email"`

    Please include scope `"openid"`

    cf. [https://openid.net/specs/openid-connect-basic-1\_0.html#Scopes](https://openid.net/specs/openid-connect-basic-1_0.html#Scopes)

- uid\_key

    Attribute from claims to be used as uid

    Note that all claims are also stored in $session->get("auth\_sso")->{info}

# HOW IT WORKS

\* the openid configuration is retrieved from `{idp_url}/.well-known/openid-configuration`

    * key C<< authorization_endpoint >> must be present in openid configuration

    * key C<< token_endpoint >> must be present in openid configuration

    * key C<< jwks_uri >> must be present in openid configuration

\* the user is redirected to the authorization endpoint with extra query parameters

\* after authentication at the authorization endpoint, the user is redirected back to this url with query parameters `code` and `state`. When something happened at the authorization endpoint, query parameters `error` and `error_description` are returned, and no `code`.

\* `code` is exchanged for a json string, using the token endpoint. This json string is a record that contains the following attributes:

    * C<< id_token >> : jwt token that contains the claims

    * C<< token_type >>: Bearer

    * C<< expires_in >>

\* key `id_token` in the token json string contains three parts:

    * jwt jose header. Can be decoded with base64 into a json string

    * jwt payload. Can be decoded with base64 into a json string

    * jwt signature

\* the `id_token` is decoded into a json string and then to a perl hash. All this data is stored `$session->{auth_sso}->{info}`. One of these attributes will be the uid that will be stored at `$session->{auth_sso}->{uid}`. This is determined by configuration key `uid_key` (see above). e.g. "email"

# LOGGING

All subclasses of [Plack::Auth::SSO](https://metacpan.org/pod/Plack::Auth::SSO) use [Log::Any](https://metacpan.org/pod/Log::Any)
to log messages to the category that equals the current
package name.

# AUTHOR

Nicolas Franck, `<nicolas.franck at ugent.be>`

# LICENSE AND COPYRIGHT

This program is free software; you can redistribute it and/or modify it
under the terms of either: the GNU General Public License as published
by the Free Software Foundation; or the Artistic License.

See [http://dev.perl.org/licenses/](http://dev.perl.org/licenses/) for more information.

# SEE ALSO

[Plack::Auth::SSO](https://metacpan.org/pod/Plack::Auth::SSO)
