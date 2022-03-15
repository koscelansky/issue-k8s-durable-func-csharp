# issue-k8s-durable-func-csharp

Example repo for issue in Azure Durable Functions deployed in Kubernetes.

## Detailed steps

1. Run `docker build --tag issue-k8s-durable-func-csharp:v1.0.0 .`
2. Install KEDA [https://keda.sh/docs/2.6/deploy/]
   * `helm repo add kedacore https://kedacore.github.io/charts`
   * `helm repo update`
   * `kubectl create namespace keda`
   * `helm install keda kedacore/keda --namespace keda`
3. Create namespace for functions `kubectl create namespace functions`
4. Create secret with Azure Storage Account connection string `kubectl create secret generic storage-conn-str --from-literal='AzureWebJobsStorage=YOUR CONNECTION STRING' -n functions`
5. Deploy to Kubernetes `func kubernetes deploy --name issue-k8s-durable-func-csharp --image-name "issue-k8s-durable-func-csharp:v1.0.0" --secret-name "storage-conn-str" --min-replicas 1 --namespace functions`
6. Set up hostname `kubectl --namespace functions set env deployment/issue-k8s-durable-func-csharp-http WEBSITE_HOSTNAME=YOURFUNCTIONHOSTNAME` (if you access it just from inside of cluster, you can use `issue-k8s-durable-func-csharp-http.functions.svc.cluster.local` as hostname)
7. Try to access orchestrator `GET /api/TestDurableFunctionsOrchestration_HttpStart`. It will succeed, but is in state Pending and the exceptions in logs is 

```plain
fail: Function.TestDurableFunctionsOrchestration[3]
      Executed 'TestDurableFunctionsOrchestration' (Failed, Id=71b08df3-deb2-4a52-8ff8-b7a215bbbfcd, Duration=4ms)
System.InvalidOperationException: Unable to load metadata for function 'TestDurableFunctionsOrchestration'.
   at Microsoft.Azure.WebJobs.Script.WebHost.Diagnostics.FunctionInstanceLogger.StartFunction(FunctionInstanceLogEntry item) in /src/azure-functions-host/src/WebJobs.Script.WebHost/Diagnostics/FunctionInstanceLogger.cs:line 125
   at Microsoft.Azure.WebJobs.Script.WebHost.Diagnostics.FunctionInstanceLogger.AddAsync(FunctionInstanceLogEntry item, CancellationToken cancellationToken) in /src/azure-functions-host/src/WebJobs.Script.WebHost/Diagnostics/FunctionInstanceLogger.cs:line 89
   at Microsoft.Azure.WebJobs.Host.Loggers.CompositeFunctionEventCollector.AddAsync(FunctionInstanceLogEntry item, CancellationToken cancellationToken) in C:\projects\azure-webjobs-sdk-rqm4t\src\Microsoft.Azure.WebJobs.Host\Loggers\CompositeFunctionEventCollector.cs:line 23
   at Microsoft.Azure.WebJobs.Host.Executors.FunctionExecutor.TryExecuteAsync(IFunctionInstance functionInstance, CancellationToken cancellationToken) in C:\projects\azure-webjobs-sdk-rqm4t\src\Microsoft.Azure.WebJobs.Host\Executors\FunctionExecutor.cs:line 106
```

## What have been tried

* Using mssql backend as described here [https://microsoft.github.io/durabletask-mssql/#/kubernetes]. No luck.
* Using `ExternalClient = true` in `[DurableClient] IDurableOrchestrationClient starter`. Still failed with *Unable to load metadata*.