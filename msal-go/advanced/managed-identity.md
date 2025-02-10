---
title: Managed identity with MSAL GO
description: "How to use Azure managed identities in MSAL GO applications."
---

# Managed Identity with MSAL GO

>[!NOTE]
>This feature is available starting with MSAL for Go version [`1.3.0-preview`](https://github.com/AzureAD/microsoft-authentication-library-for-go/releases/tag/v1.3.0-preview).

A common challenge for developers is the management of secrets, credentials, certificates, and keys used to secure communication between services. [Managed identities](/azure/active-directory/managed-identities-azure-resources/overview) in Azure eliminate the need for developers to handle these credentials manually. MSAL for Go supports acquiring tokens through the managed identity service when used with applications running inside Azure infrastructure, such as:

* [Azure VMs](https://azure.microsoft.com/free/virtual-machines/)
* [Azure Arc](/azure/azure-arc/overview)

For a complete list, refer to [Azure services that can use managed identities to access other services](/azure/active-directory/managed-identities-azure-resources/managed-identities-status).

## Which SDK to use - Azure SDK or MSAL?

MSAL libraries provide lower level APIs that are closer to the OAuth2 and OIDC protocols.

Both MSAL for Go and [Azure SDK for Go](/azure/developer/go/) allow you to acquire tokens via managed identity. Internally, Azure SDK uses MSAL for Go and it provides a higher-level API via its `DefaultAzureCredential` and `ManagedIdentityCredential` abstractions.

If your application already uses one of the aforementioned SDKs, continue using the same SDK. Use Azure SDK if you are writing a new application and plan to call other Azure resources. Azure SDK provides a simpler developer experience by allowing the app to use local-ready APIs, such as `DefaultAzureCredential`, enabling testing on machines where managed identity doesn't exist. Consider using MSAL if you need to call other downstream web APIs like Microsoft Graph or your own web API.

## How to use managed identities

There are two types of managed identities available to developers **system-assigned** and **user-assigned**. You can learn more about the differences in the [Managed identity types](/entra/identity/managed-identities-azure-resources/overview#managed-identity-types) article. MSAL for Go supports acquiring tokens for both.  
A quick overview of each is:

**System Assigned** - Created and managed by Azure and tied to the lifecycle of a resource. When resource is deleted, so is the system assigned identity  
**User Assigned** - Created as a standalone resource on Azure. It is not tied to any specific resource. Can be assigned to multiple resources and managed independently. Useful when you need multiple resources sharing same identity and permissions

Prior to using managed identities from MSAL for Go, developers must enable them for the resources they want to use through Azure CLI or the Azure Portal.

## Creating Azure Resources

You can create the resources needed for running the sample manually via the [Azure Portal](https://portal.azure.com/#home), or for a quick and
concise breakdown of how to do it using Azure CLI or Azure Powershell, follow the instructions in the [Authentication with the Azure SDK for Go using a managed identity](/azure/developer/go/azure-sdk-authentication-managed-identity?tabs=azure-cli) article.

## Quick start

To quickly get started and see managed identity in action, you can use one of the samples:

> [Use Managed Identity sample](https://github.com/Azure-Samples/msal-managed-identity/tree/main/src/go)

## Examples

For both user-assigned and system-assigned identities, developers can use the `New` function in [`managedidentity.go`](https://github.com/AzureAD/microsoft-authentication-library-for-go/blob/c5febcbae287a26a0cfedd45f4edeaf3c41ad7dc/apps/managedidentity/managedidentity.go#L107)

### System-assigned managed identities

For system-assigned managed identities, pass `SystemAssigned()` to the `New` function

```go
mi.New(mi.SystemAssigned())
```

[AcquireToken](https://github.com/AzureAD/microsoft-authentication-library-for-go/blob/c5febcbae287a26a0cfedd45f4edeaf3c41ad7dc/apps/managedidentity/managedidentity.go#L216) is called with the context, resource to acquire a token for, such as `https://management.azure.com`, along with any optionals.

```go
miClient, err := mi.New(mi.SystemAssigned())
if err != nil {
    log.Fatalf("failed to create a new managed identity client: %v", err)
    return
}

accessToken, err := miClient.AcquireToken(context.Background(), "https://vault.azure.net")
if err != nil {
    log.Fatalf("failed to acquire token: %v", err)
    return
}
```

### User-assigned managed identities

For user-assigned managed identities, the developer needs to pass either the client ID, full resource identifier, or the object ID of the managed identity when creating [`New`](https://github.com/AzureAD/microsoft-authentication-library-for-go/blob/c5febcbae287a26a0cfedd45f4edeaf3c41ad7dc/apps/managedidentity/managedidentity.go#L107).

Similar to system-assigned managed identities, [`AcquireToken`](https://github.com/AzureAD/microsoft-authentication-library-for-go/blob/c5febcbae287a26a0cfedd45f4edeaf3c41ad7dc/apps/managedidentity/managedidentity.go#L216) is called with the resource to acquire a token for, such as `https://management.azure.com`.

```go
miClient, err := mi.New(mi.UserAssignedClientID("my-client-id"))
miClient, err := mi.New(mi.UserAssignedObjectID("my-object-id"))
miClient, err := mi.New(mi.UserAssignedResourceID("my-resource-id"))

accessToken, err := miClient.AcquireToken(context.Background(), "https://vault.azure.net")
if err != nil {
    log.Fatalf("failed to acquire token: %v", err)
    return
}
```

## Caching

By default, MSAL for Go supports in-memory caching
MSAL does not support cache extensibility for managed identities due to security concerns when using a distributed cache.
Since a token acquired for a managed identity belongs to an Azure resource, using a distributed cache might expose it to other Azure resources sharing the cache.

## Troubleshooting and error handling

Errors in MSAL are intended for app developers to troubleshoot and not for displaying to end-users.  

For more information on how to handle errors from MSAL go see [error_design.md](https://learn.microsoft.com/en-us/entra/msal/go/error-design)  
Returned errors (originating from the managed identity service) contain actionable context that will help you take mitigation steps.

### Potential errors

For more information on potential errors returned from the managed identity service, refer to the [list of error codes](/entra/identity-platform/reference-error-codes)