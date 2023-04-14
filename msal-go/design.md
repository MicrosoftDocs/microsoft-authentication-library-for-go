---
title: Microsoft Authentication Library for Go Design Guide
description: "Design guidelines for the Microsoft Authentication Library for Go."
---

# Microsoft Authentication Library for Go Design Guide

## General Structure

Public Surface

```bash
apps/ - Contains all our code
  confidential/ - The confidential application API
  public/ - The public application API
  cache/ - The cache interface that can be implemented to provide persistence cache storage of credentials
```

Internals

```bash
apps/
  internal/
    client/ - Shared package for common calls that Public and Confidential apps share
    json/ - Our own json encoder/decoder for special needs
    shared/ - Holds types that need to be in multiple packages and can't be moved into a single one due to import cycles
    requests/ - The pacakge to communicate to services to get tokens
```

### Use of the Go special internal/ directory

In Go, a directory called `internal/` contains packages that should only be used by other packages rooted at the same location. This is covered in the [official Go documentation](https://golang.org/doc/go1.4#internalpackages).

For example, a package `.../a/b/c/internal/d/e/f` can be imported only by code in the directory tree rooted at `.../a/b/c`. It cannot be imported by code in `.../a/b/g` or in any other repository.

We use this feature quite liberally to make clear what is using an internal package.  For example:

```bash
apps/internal/base - Only can be used by packages defined at apps/
apps/internal/base/internal/storage - Only can be use by package client
```

## Public API

The public API is encapsulated in `apps/`.  `apps/` has 3 packages of interest to users:

- `public/` - Covers the Public Application Client. These are apps that run on the user's computer, such as command line interface apps. 
- `confidential/` - Covers the Confidential Application Client. These are apps that run on servers - web apps, web APIs which get tokens for users and web APIs which get tokens on behalf of themselves (service to service communication)
- `cache/` - This provides the interfaces that must be implemented to create persistent caches for any MSAL client

## Internals

In this section we will be talking about `internal/`.

### JSON Handling

JSON must be handled specially in our app. The basics are, if we receive fields that our structs do not contain, we cannot drop them.  We must send them back to the service.

To handle that, we use our own custom `json` package that handles this.

### Backend communication

Communication to the backends is done via the `requests/` package. `oauth.Token` is the client for all communication.

`oauth.Token` communicates via REST calls that are encapsulated in the `ops/` client.

## Adding A Feature

This is the general way to add a new feature to MSAL:

- Add the REST calls to `ops.REST`
- Add the higher level manipulations to `oauth.Token`
- Add your logic to the `app/\<client\>` and access the services via your `oauth.Token`

## Notable Differences To Other Clients

### TBD: Confidential applications needs to handle multiple users without one big cache

The MSAL caching design is rather simple. These design decisions and the fact that multiple applications in different languages can share a cache mean it cannot be easily changed.

The entire cache contents of a `confidential.Client` is read and written on almost any action to and from an external cache.

It is not clear to a user that a confidential client should be per user to prevent scaling problems.

We cannot change the MSAL cache design at this time, therefore it should be clear that `confidential.Client` should be done per user.

### Use of x509.Certificate and CertFromPEM() function

The original version of this package used an thumbprint and a private key to do authorizations
based on a certificate. But there wasn't a real way to get a thumbprint.

A thumbprint is defined in the OAuth spec, which we had to track down. It is an SHA-1 hash from the x509 certificate's DER-encoded ASN1 bytes.

Since the user was going to need the x509, we moved to having the user provide the `x509.Certificate` object.

We wrote the thumbprint creator for the internals. Since we also require the private key and it is not straightforward to get, we added a `CertFromPEM()` function that will extract the `x509.Certificate` and private key. We did support encrypted PEM.

>[!NOTE]
>It should be noted that Key Vault stores things in PKCS12 and PEM.

## Logging

For errors, see [error design](./error-design.md).

This library does not log personal identifiable information (PII). For a definition of PII, see [Customer Data Definitions](https://www.microsoft.com/trust-center/privacy/customer-data-definitions). MSAL Go does not log any of the three data categories listed on the website.

The library may log information related to your organization, such as tenant ID, authority, and client ID, as well as information that cannot be tied to a user, such as request correlation ID, HTTP status codes and other generalizable metadata.
