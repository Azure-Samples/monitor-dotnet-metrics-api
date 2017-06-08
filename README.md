---
services: azure-monitor
platforms: dotnet
author: devigned
---

# Retrieve Azure Monitor metrics with .NET

This sample explains how to retrieve Monitor metrics and metric definitions using the Azure .NET SDK.

**On this page**

- [Run this sample](#run)
- [What is program.cs doing?](#example)
    - [List metric definitions for a resource](#list-metricdefinitions)
    - [List metrics for a resource](#list-metrics)

<a id="run"></a>
## Run this sample

1. If you don't have it, install the [.NET Core SDK](https://www.microsoft.com/net/core).

1. Clone the repository.

    ```
    git clone https://github.com/Azure-Samples/monitor-metrics-dotnet.git
    ```

1. Install the dependencies.

    ```
    dotnet restore
    ```

1. Create an Azure service principal either through
    [Azure CLI](https://azure.microsoft.com/documentation/articles/resource-group-authenticate-service-principal-cli/),
    [PowerShell](https://azure.microsoft.com/documentation/articles/resource-group-authenticate-service-principal/)
    or [the portal](https://azure.microsoft.com/documentation/articles/resource-group-create-service-principal-portal/).

1. Export these environment variables using your subscription id and the tenant id, client id and client secret from the service principle that you created. 

    ```
    export AZURE_TENANT_ID={your tenant id}
    export AZURE_CLIENT_ID={your client id}
    export AZURE_CLIENT_SECRET={your client secret}
    export AZURE_SUBSCRIPTION_ID={your subscription id}
    ```

1. Run the sample.

    ```
    dotnet run resourceId
    ```

<a id="example"></a>
## What is program.cs doing?

The sample retrieves metric definitions and metrics for a given resource.
It starts by setting up a MonitorClient object using your subscription and credentials.

```csharp
// Build the service credentials and Monitor client
var serviceCreds = await ApplicationTokenProvider.LoginSilentAsync(tenantId, clientId, secret);
var monitorClient = new MonitorClient(serviceCreds);
monitorClient.SubscriptionId = subscriptionId;
```

<a id="list-metricdefinitions"></a>
### List metric definitions for a resource

List the metric definitions for the given resource, which is defined in the current subscription.

```csharp
IEnumerable<MetricDefinition> metricDefinitions = await readOnlyClient.MetricDefinitions.ListAsync(resourceUri: resourceUri, cancellationToken: new CancellationToken());
```

or using a filter

```csharp
var odataFilterMetricDef = new ODataQuery<MetricDefinition>("names eq 'CpuPercentage'");
metricDefinitions = await readOnlyClient.MetricDefinitions.ListAsync(resourceUri: resourceUri, odataQuery: odataFilterMetricDef, cancellationToken: new CancellationToken());
```

<a id="list-metrics"></a>
### List metrics for a resource

```csharp
IEnumerable<Metric> metrics = await readOnlyClient.Metrics.ListAsync(resourceUri: resourceUri, cancellationToken: CancellationToken.None);
```

or with a filter

```csharp
// The comma-separated list of metric names must be present if a filter is used
var metricNames = "name.value eq 'CpuPercentage'"; // could be concatenated with " or name.value eq '<another name>'" ...

// Time grain is optional when metricNames is present
string timeGrain = " and timeGrain eq duration'PT5M'";

// Defaulting to 3 hours before the time of execution for these datetimes
string startDate = string.Format(" and startTime eq {0}", DateTime.Now.AddHours(-3).ToString("o"));
string endDate = string.Format(" and endTime eq {0}", DateTime.Now.ToString("o"));

var odataFilterMetrics = new ODataQuery<Metric>(
    string.Format(
        "{0}{1}{2}{3}",
        metricNames,
        timeGrain,
        startDate,
        endDate));

Write("Call with filter parameter (i.e. $filter = {0})", odataFilterMetrics);
metrics = await readOnlyClient.Metrics.ListAsync(resourceUri: resourceUri, odataQuery: odataFilterMetrics, cancellationToken: CancellationToken.None);
```
