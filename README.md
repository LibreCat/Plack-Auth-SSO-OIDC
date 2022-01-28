# NAME

Plack::Auth::SSO::OIDC - implementation of OpenID Connect for Plack::Auth::SSO

# STATUS

[![Build Status](https://travis-ci.org/LibreCat/Plack-Auth-SSO-OIDC.svg?branch=master)](https://travis-ci.org/LibreCat/Plack-Auth-SSO-OIDC)
[![Coverage](https://coveralls.io/repos/LibreCat/Plack-Auth-SSO-OIDC/badge.png?branch=master)](https://coveralls.io/r/LibreCat/Plack-Auth-SSO-OIDC)
[![CPANTS kwalitee](http://cpants.cpanauthors.org/dist/Plack-Auth-SSO-OIDC.png)](http://cpants.cpanauthors.org/dist/Plack-Auth-SSO-OIDC)

# DESCRIPTION

This is an implementation of [Plack::Auth::SSO](https://metacpan.org/pod/Plack::Auth::SSO) to authenticate against a openid connect server.

It inherits all configuration options from its parent.

# SYNOPSIS

    # in your app.psi (Plack)

    use strict;
    use warnings;
    use Plack::Builder;
    use JSON;
    use Plack::Auth::SSO::OIDC;
    use Plack::Session::Store::File;

    my $uri_base = "http://localhost:5000";

    builder {

        # session middleware needed to store "auth_sso" and/or "auth_sso_error"
        # in memory session store for testing purposes
        enable "Session";

        # for authentication, redirect your users to this path
        mount "/auth/oidc" => Plack::Auth::SSO::OIDC->new(

            # plack application needs to know about the base url of this application
            uri_base => $uri_base,

            # after successfull authentication, user is redirected to this path (uri_base is used!)
            authorization_path => "/auth/callback",

            # when authentication fails at the identity provider
            # user is redirected to this path with session key "auth_sso_error" (hash)
            error_path => "/auth/error",

            # base url of openid connect server
            idp_url => "https://example.oidc.org/auth/oidc",
            client_id => "my-client-id",
            client_secret => "myclient-secret",
            uid_key => "email"

        )->to_app();

        # example psgi app that is called after successfull authentication at /auth/oidc (see above)
        # it expects session key "auth_sso" to be present
        # here you typically create a user session based on the uid in "auth_sso"
        mount "/auth/callback" => sub {

            my $env     = shift;
            my $session = Plack::Session->new($env);
            my $auth_sso= $session->get("auth_sso");
            my $user    = MyUsers->get( $auth_sso->{uid} );
            $session->set("user_id", $user->{id});
            [ 200, [ "Content-Type" => "text/plain" ], [
                "logged in! ", $user->{name}
            ]];

        };

        # example psgi app that is called after unsuccessfull authentication at /auth/oidc (see above)
        # it expects session key "auth_sso_error" to be present
        mount "/auth/error" => sub {

            my $env = shift;
            my $session = Plack::Session->new($env);
            my $auth_sso_error = $session->get("auth_sso_error");

            [ 200, [ "Content-Type" => "text/plain" ], [
                "something happened during single sign on authentication: ",
                $auth_sso_error->{content}
            ]];

        };
    };

# CONSTRUCTOR ARGUMENTS

- `uri_base`

    See ["uri\_base" in Plack::Auth::SSO](https://metacpan.org/pod/Plack::Auth::SSO#uri_base)

- `id`

    See ["id" in Plack::Auth::SSO](https://metacpan.org/pod/Plack::Auth::SSO#id)

- `session_key`

    See ["session\_key" in Plack::Auth::SSO](https://metacpan.org/pod/Plack::Auth::SSO#session_key)

- `authorization_path`

    See ["authorization\_path" in Plack::Auth::SSO](https://metacpan.org/pod/Plack::Auth::SSO#authorization_path)

- `error_path`

    See ["error\_path" in Plack::Auth::SSO](https://metacpan.org/pod/Plack::Auth::SSO#error_path)

- `idp_url`

    base url of the OIDC service.

    The openid configuration is expected at and retrieved from ${idp\_url}/.well-known/openid-configuration

- `client_id`

    client-id as given by the OIDC service

- `client_secret`

    client-secret as given by the OIDC service

- `scope`

    Scope requested from the OIDC service.

    Space separated string containing all scopes

    Default: `"openid profile email"`

    Please include scope `"openid"`

    cf. [https://openid.net/specs/openid-connect-basic-1\_0.html#Scopes](https://openid.net/specs/openid-connect-basic-1_0.html#Scopes)

- `uid_key`

    Attribute from claims to be used as uid

    Note that all claims are also stored in `$session->get("auth_sso")->{info}`

# HOW IT WORKS

- the openid configuration is retrieved from `{idp_url}/.well-known/openid-configuration`
    - key `authorization_endpoint` must be present in openid configuration
    - key `token_endpoint` must be present in openid configuration
    - key `jwks_uri` must be present in openid configuration
    - the user is redirected to the authorization endpoint with extra query parameters
- after authentication at the authorization endpoint, the user is redirected back to this url with query parameters `code` and `state`. When something happened at the authorization endpoint, query parameters `error` and `error_description` are returned, and no `code`.
- `code` is exchanged for a json string, using the token endpoint. This json string is a record that contains attributes like `id_token` and `access_token`. See [https://openid.net/specs/openid-connect-core-1\_0.html#TokenResponse](https://openid.net/specs/openid-connect-core-1_0.html#TokenResponse) for more information.
- key `id_token` in the token json string contains three parts:
    - jwt jose header. Can be decoded with base64 into a json string
    - jwt payload. Can be decoded with base64 into a json string
    - jwt signature
- the `id_token` is decoded into a json string and then to a perl hash. All this data is stored `$session->{auth_sso}->{info}`. One of these attributes will be the uid that will be stored at `$session->{auth_sso}->{uid}`. This is determined by configuration key `uid_key` (see above). e.g. "email"

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
