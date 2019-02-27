# Step CLI

`step` is a zero trust swiss army knife. It's an easy-to-use and hard-to-misuse
utility for building, operating, and automating systems that use zero trust
technologies like authenticated encryption (X.509, TLS), single sign-on (OAuth
OIDC, SAML), multi-factor authentication (OATH OTP, FIDO U2F),
encryption mechanisms (JSON Web Encryption, NaCl), and verifiable
claims (JWT, SAML assertions).

[Website](https://smallstep.com) |
[Documentation](https://smallstep.com/docs/cli) |
[Installation Guide](#installation-guide) |
[Examples](#examples) |
[Contribution Guide](./docs/CONTRIBUTING.md)

[![GitHub release](https://img.shields.io/github/release/smallstep/cli.svg)](https://github.com/smallstep/cli/releases)
[![CA Image](https://images.microbadger.com/badges/image/smallstep/step-cli.svg)](https://microbadger.com/images/smallstep/step-cli)
[![Go Report Card](https://goreportcard.com/badge/github.com/smallstep/cli)](https://goreportcard.com/report/github.com/smallstep/cli)
[![Build Status](https://travis-ci.com/smallstep/cli.svg?branch=master)](https://travis-ci.com/smallstep/cli)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![CLA assistant](https://cla-assistant.io/readme/badge/smallstep/cli)](https://cla-assistant.io/smallstep/cli)

[![GitHub stars](https://img.shields.io/github/stars/smallstep/cli.svg?style=social)](https://github.com/smallstep/cli/stargazers)
[![Twitter followers](https://img.shields.io/twitter/follow/smallsteplabs.svg?label=Follow&style=social)](https://twitter.com/intent/follow?screen_name=smallsteplabs)


For more information see [the step website](https://smallstep.com/cli/)
and the [blog post](https://smallstep.com/blog/zero-trust-swiss-army-knife.html)
announcing step.

![Animated terminal showing step in practice](https://smallstep.com/images/blog/2018-08-07-unfurl.gif)

## Installation Guide

These instructions will install an OS specific version of the `step` binary on
your local machine. To build from source see [getting started with
development](#getting-started-with-development) below.


### Mac OS

Install `step` via [Homebrew](https://brew.sh/):

```
brew install step

# test ...
step certificate inspect https://smallstep.com
```

> Note: If you have installed `step` previously through the `smallstep/smallstep`
> tap you will need to run the following commands before installing:
```
brew untap smallstep/smallstep
brew uninstall step
```

### Linux

Download the latest Debian package from [releases](https://github.com/smallstep/cli/releases):

```
wget https://github.com/smallstep/cli/releases/download/X.Y.Z/step_X.Y.Z_amd64.deb
```

Install the Debian package:

```
sudo dpkg -i step_X.Y.Z_amd64.deb
```

Test:

```
step certificate inspect https://smallstep.com
```

## Examples

### X.509 Certificates

Create a root CA, an intermediate, and a leaf X.509 certificate. Bundle the
leaf with the intermediate for use with TLS:

```
$ step certificate create --profile root-ca \
    "Example Root CA" root-ca.crt root-ca.key
$ step certificate create \
    "Example Intermediate CA 1" intermediate-ca.crt intermediate-ca.key \
    --profile intermediate-ca --ca ./root-ca.crt --ca-key ./root-ca.key
$ step certificate create \
    example.com example.com.crt example.com.key \
    --profile leaf --ca ./intermediate-ca.crt --ca-key ./intermediate-ca.key
$ step certificate bundle \
    example.com.crt intermediate-ca.crt example.com-bundle.crt
```

Extract the expiration date from a certificate (requires
[`jq`](https://stedolan.github.io/jq/)):

```
$ step certificate inspect example.com.crt --format json | jq -r .validity.end
$ step certificate inspect https://smallstep.com --format json | jq -r .validity.end
```

### JSON Object Signing & Encryption (JOSE)

Create a [JSON Web Key](https://tools.ietf.org/html/rfc7517) (JWK), add the
public key to a keyset, and sign a [JSON Web Token](https://tools.ietf.org/html/rfc7519) (JWT):

```
$ step crypto jwk create pub.json key.json
$ cat pub.json | step crypto jwk keyset add keys.json
$ JWT=$(step crypto jwt sign \
    --key key.json \
    --iss "issuer@example.com" \
    --aud "audience@example.com" \
    --sub "subject@example.com" \
    --exp $(date -v+15M +"%s"))
```

Verify your JWT and return the payload:

```
$ echo $JWT | step crypto jwt verify \
    --jwks keys.json --iss "issuer@example.com" --aud "audience@example.com"
```

### Single Sign-On

Login with Google, get an access token, and use it to make a request to
Google's APIs:

```
curl -H"$(step oauth --header)" https://www.googleapis.com/oauth2/v3/userinfo
```

Login with Google and obtain an OAuth OIDC identity token for single sign-on:

```
$ step oauth \
    --provider https://accounts.google.com \
    --client-id 1087160488420-8qt7bavg3qesdhs6it824mhnfgcfe8il.apps.googleusercontent.com \
    --client-secret udTrOT3gzrO7W9fDPgZQLfYJ \
    --bare --oidc
```

Obtain and verify a Google-issued OAuth OIDC identity token:

```
$ step oauth \
    --provider https://accounts.google.com \
    --client-id 1087160488420-8qt7bavg3qesdhs6it824mhnfgcfe8il.apps.googleusercontent.com \
    --client-secret udTrOT3gzrO7W9fDPgZQLfYJ \
    --bare --oidc \
 | step crypto jwt verify \
   --jwks https://www.googleapis.com/oauth2/v3/certs \
   --iss https://accounts.google.com \
   --aud 1087160488420-8qt7bavg3qesdhs6it824mhnfgcfe8il.apps.googleusercontent.com
```

### Multi-factor Authentication

Generate a [TOTP](https://en.wikipedia.org/wiki/Time-based_One-time_Password_algorithm)
token and a QR code:

```
$ step crypto otp generate \
    --issuer smallstep.com --account name@smallstep.com \
    --qr smallstep.png > smallstep.totp
```

Scan the QR Code using Google Authenticator, Authy or similar software and use
it to verify the TOTP token:

```
$ step crypto otp verify --secret smallstep.totp
```

## Documentation

Documentation can be found in three places:

1. On the command line with `step help xxx` where `xxx` is the subcommand you
   are interested in. Ex: `step help crypto jwk`

2. On the web at https://smallstep.com/docs/cli

3. On your browser by running `step help --http :8080` and visiting
   http://localhost:8080

## The Future

We plan to build more tools that facilitate the use and management of zero trust
networks.

* Tell us what you like and don't like about managing identity in your
network - we're eager to help solve problems in this space.
* Tell us what features you'd like to see - open issues or hit us on
[Twitter](https://twitter.com/smallsteplabs).

## Further Reading

* [Blog post](https://smallstep.com/blog/zero-trust-swiss-army-knife.html)
announcing `step`.
* Check out [`step certificates`](https://github.com/smallstep/certificates) -
an online certificate authority and related tools for secure automated
certificate management, so you can use TLS everywhere.
