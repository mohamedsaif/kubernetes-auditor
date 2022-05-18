# Kubernetes-Auditor
Utilizing Azure Monitor capabilities to raise alerts based on specific Kubernetes audit logs

## Overview 

In highly regulated deployments of AKS clusters, you might want to keep close eye on audit logs.

AKS diagnostic logs have many categories which can help in answering questions like:

- What happened?
- When did it happen?
- Who initiated it?
- On what did it happen?
- Where was it observed?
- From where was it initiated?
- To where was it going?

In this repo, I explore how we can leverage AKS diagnostic settings along with Azure Monitor to setup alerts that are pushed to different channels to keep close eye on Kubernetes API calls.

## Understand AKS logs

In order to optimize your diagnostics logs configuration, these are the available sources:

| Category                | Description |
|:---|:---|
| cluster-autoscaler       | Understand why the AKS cluster is scaling up or down, which may not be expected. This information is also useful to correlate time intervals where something interesting may have happened in the cluster. |
| guard                   | Managed Azure Active Directory and Azure RBAC audits. For managed Azure AD, this includes token in and user info out. For Azure RBAC, this includes access reviews in and out. |
| kube-apiserver          | Logs from the API server. |
| kube-audit              | Audit log data for every audit event including get, list, create, update, delete, patch, and post. |
| kube-audit-admin        | Subset of the kube-audit log category. Significantly reduces the number of logs by excluding the get and list audit events from the log. |
| kube-controller-manager | Gain deeper visibility of issues that may arise between Kubernetes and the Azure control plane. A typical example is the AKS cluster having a lack of permissions to interact with Azure. |
| kube-scheduler          | Logs from the scheduler. |
| AllMetrics              | Includes all platform metrics. Sends these values to Log Analytics workspace where it can be evaluated with other data using log queries. |

You can check further documentation in [AKS Monitoring Reference](https://docs.microsoft.com/en-us/azure/aks/monitor-aks-reference)

## AKS diagnostic settings

Using Azure Portal under AKS -> Monitoring -> Diagnostic settings you will find the configuration you can setup to allow the collection of many diagnostic data.

![aks-diagnostics](/res/aks-diagnostics.jpg)

Here I will be interested in **kube-audit-admin** to monitor specific actions on particular deployment which let's say is critical to my AKS cluster operational health.

### View the diagnostic logs

In order to create alerts, we need a good understanding of the logs.

Below are few queries that you can leverage to get started:

```kusto

// View all captured kube-audit-admin logs
AzureDiagnostics 
| where Category == "kube-audit-admin"

//View logs specific to particular deployment and verbs:
AzureDiagnostics 
| where Resource == "AKS-WEU"
| where Category == "kube-audit-admin"
| where log_s has "node-resolver-ds"
| where log_s has_any ("\"verb\":\"patch\"",  "\"verb\":\"delete\"" )

```