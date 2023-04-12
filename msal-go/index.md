---
title: Microsoft Authentication Library (MSAL) for Go
description: "An introduction to using Microsoft Authentication Library (MSAL) for Go."
---

# Microsoft Authentication Library (MSAL) for Go

>[!NOTE]
> Microsoft Authentication Library (MSAL) for Go is a new addition to the MSAL family of libraries. It has been made available in production-ready preview to gauge customer interest and to gather feedback from the community. We welcome all contributors (see [contribution guidelines in the library repository](https://github.com/AzureAD/microsoft-authentication-library-for-go/blob/dev/CONTRIBUTING.md)) to help us improve the library.

The Microsoft Authentication Library (MSAL) for Go is part of the [Microsoft identity platform for developers](https://aka.ms/aaddevv2). It allows you to sign in users or apps with Microsoft identities ([Azure AD](https://azure.microsoft.com/services/active-directory/) and [Microsoft Accounts](https://account.microsoft.com)) and obtain tokens to call APIs such as [Microsoft Graph](https://graph.microsoft.io/) or your own APIs registered with the Microsoft identity platform. It is built using industry standard OAuth2 and OpenID Connect protocols.

The latest code resides in the `dev` branch in the [library GitHub repository](https://github.com/AzureAD/microsoft-authentication-library-for-go).

## Installation

### Setting up Go

To install Go, visit [this link](https://golang.org/dl/).

### Installing MSAL Go

```bash
go get -u github.com/AzureAD/microsoft-authentication-library-for-go/
```

## Usage

Before using MSAL Go, you will need to [register your application with the Microsoft identity platform](/azure/active-directory/develop/quickstart-v2-register-an-app).

### Public surface

The Public API of the library can be found in the following directories under `apps`.

- `confidential` - The confidential application API
- `public` - The public application API
- `cache` - The cache interface that can be implemented to provide persistence cache storage of credentials

Acquiring tokens with MSAL Go follows this general three step pattern. There might be some slight differences for other token acquisition flows. Here is a basic example:

1. MSAL separates [public and confidential client applications](https://tools.ietf.org/html/rfc6749#section-2.1). So, you would create an instance of a `PublicClientApplication` and `ConfidentialClientApplication` and use this throughout the lifetime of your application.

   - Initializing a public client:

    ```go
    publicClientApp, err := public.New("client_id", public.WithAuthority("https://login.microsoftonline.com/Enter_The_Tenant_Name_Here"))
    ```

   - Initializing a confidential client:

    ```go
    // Initializing the client credential
    cred, err := confidential.NewCredFromSecret("client_secret")
    if err != nil {
        return nil, fmt.Errorf("could not create a cred from a secret: %w", err)
    }
    confidentialClientApp, err := confidential.New("client_id", cred, confidential.WithAuthority("https://login.microsoftonline.com/Enter_The_Tenant_Name_Here"))
    ```

1. MSAL comes packaged with an in-memory cache. Utilizing the cache is optional, but we would highly recommend it.

    ```go
    var userAccount public.Account
    accounts := publicClientApp.Accounts()
    if len(accounts) > 0 {
        // Assuming the user wanted the first account
        userAccount = accounts[0]
        // found a cached account, now see if an applicable token has been cached
        result, err := publicClientApp.AcquireTokenSilent(context.Background(), []string{"your_scope"}, public.WithSilentAccount(userAccount))
        accessToken := result.AccessToken
    }
    ```

1. If there is no suitable token in the cache, or you choose to skip this step, now we can send a request to AAD to obtain a token.

    ```go
    result, err := publicClientApp.AcquireToken"ByOneofTheActualMethods"([]string{"your_scope"}, ...(other parameters depending on the function))
    if err != nil {
        log.Fatal(err)
    }
    accessToken := result.AccessToken
    ```

You can view the [dev apps](https://github.com/AzureAD/microsoft-authentication-library-for-go/tree/dev/apps/tests/devapps) on how to use MSAL Go with various application types in various scenarios. For more detailed information, please refer to the [wiki](https://github.com/AzureAD/microsoft-authentication-library-for-go/wiki).

## Releases

For a full list of library releases, refer to the [Releases](https://github.com/AzureAD/microsoft-authentication-library-for-go/releases) section in the library source code repository.

## Community help and support

We use [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-ad-msal) to work with the community on supporting Azure Active Directory and its SDKs, including this one! We highly recommend you ask your questions on Stack Overflow. You can also browse existing questions to see if someone has encountered the problem before. Please use the `azure-ad-msal` tag when asking your questions.

If you find a bug or have a feature request, please open a new issue in the [Issues](https://github.com/AzureAD/microsoft-authentication-library-for-go/issues) section.

## Submit feedback

If you have any library feedback, make sure to submit your feature requests and bug reports [on GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-go/issues).

## Security library

This library controls how users sign-in and access services. We recommend you always take the latest version of our library in your app when possible. We use [semantic versioning](http://semver.org) so you can control the risk associated with updating your app. As an example, always downloading the latest minor version number (e.g. x.*y*.x) ensures you get the latest security and feature enhancements but our API surface remains the same. You can always see the latest version and release notes under the Releases tab of GitHub.

## Security reporting

If you find a security issue with our libraries or services please report it to [secure@microsoft.com](mailto:secure@microsoft.com) with as much detail as possible. Your submission may be eligible for a bounty through the [Microsoft Bounty](https://aka.ms/bugbounty) program. Please do not post security issues to GitHub Issues or any other public site. We will contact you shortly upon receiving the information. We encourage you to get notifications of when security incidents occur by visiting [this page](https://www.microsoft.com/msrc/technical-security-notifications) and subscribing to Security Advisory Alerts.
