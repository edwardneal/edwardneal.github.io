---
layout: post
title:  "Connecting to Application Insights with Managed Service Identities"
date:   2020-02-09 20:00:00 +0000
categories: application-insights managed-service-identity
---
A lot of services can drop data into [Application Insights][app-insights] - Azure VMs, Azure App Services, Azure Functions, custom web services and on-premise services. Getting access to this data typically requires an API key though, which can add complexity.
If you don't want to deal with managing a secret, you can also use Azure AD authentication. This will allow you to access the API using the identity your Azure VM, Function or App Service is running as.

### NuGet Packages

* `Microsoft.Azure.ApplicationInsights`
* `Microsoft.Azure.Services.AppAuthentication`
* `Microsoft.Rest.ClientRuntime`

### Method

Rather than generating an API key, we'll get an access token for the Application Insights resource (`https://api.applicationinsights.io/`) and pass those as TokenCredentials.

### Sample Code

{% highlight csharp %}
public static async Task<ApplicationInsightsDataClient>
    CreateDataClientAsync(string appId, string tenantId)
{
    var ApplicationInsightsResource = "https://api.applicationinsights.io/"
    var tokenProvider = new AzureServiceTokenProvider();
    var accessToken = await tokenProvider.GetAccessTokenAsync(ApplicationInsightsResource, tenantId);
    var wrappedTokenCredentials = new TokenCredentials(accessToken);
	
    return new ApplicationInsightsDataClient(wrappedTokenCredentials) { AppId = appId };
}
{% endhighlight %}

### Notes

1. I've found that `tenantId` needs to be specified - unlike other services, it can't be left blank.
2. `appId` is *not* the instrumentation key. There are two ways to find this:
    1. In the Azure portal, open your Application Insights resource, select the "API Access" section and use the "Application ID" field.
    2. In an Azure Resource Manager template, this is the `AppId` property of a component of type `Microsoft.Insights/components`. API version 2015-05-01 and higher should work.

### Wrapping Up

Use Azure's standard RBAC mechanisms to add your application's Managed Service Identity to the Application Insights resource's Reader role.

### Useful Links

[Application Insights overview][app-insights]

[Azure Resource Manager Application Insights schema](https://docs.microsoft.com/en-us/rest/api/application-insights/components/list)
	
[app-insights]:	https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview