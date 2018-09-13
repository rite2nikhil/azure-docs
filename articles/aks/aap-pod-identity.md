---
title: AAD Pod Identity on Azure Kubernetes Service (AKS)
description: Use AAD Pod Identity on Azure Kubernetes Service (AKS).
services: container-service
author: rite2nikhil
manager: 

ms.service: container-service
ms.topic: article
ms.date: 09/12/2018
ms.author: rite2nikhil
---

# AAD Pod Identity

Applications running on the POD on AKS cluster require access to identities in Azure Active Directory (AAD) to access resources that use an identity provider. AAD provides a construct called a Service Principal that allows applications to assume identities with limited permissions, and Managed Service Identity (MSI) - automatically generated and rotated credentials that easily retrieved by an application at run-time to authenticate as a service principal. An cluster admin configures the Azure Identity Binding to the Pod. Without any change of auth code the application running on the pod works on the cluster

For more information about Managed Identities for Azure Resources, see [managed-identities-azure-resources][managed-identities-azure-resources].


## AAD Pod Identity scenario overview 

Some of the AAD Pod Identity scenarios for applications innclude:

- **Service Principal**:Application need to access Azure resources based on Service Principal
- **Managed Assigned Identities**:Application need to access Azure resources based on Managed Service Identity
- **Mount Azure Key Vault atrifacts as volumes for Pod**:Applicmation seamlessly has access to key-vault secrets mouted as volumes to pods.

## AAD Pod Identity solution overview

The detailed design of the project can be found in the following docs:
[Concept][aad-pod-id-concept]
[Block Diagram][aad-pod-id-block-diagram]

Deploys two components: a Managed Identity Controller (MIC) and an Node Managed Identity (NMI).

- **Managed Identity Controller (MIC)**: This controller watches for pod changes through the api server and caches pod to admin configured azure identity map.
- **Node Managed Identity (NMI)**: The authorization request of fetching Service Principal Token from MSI endpoint is sent to a standard Instance Metadata endpoint which is redirected to the NMI pod by adding ruled to redirect POD CIDR traffic with metadata endpoint IP on port 80 to be sent to the NMI endpoint. The NMI server identifies the pod based on the remote address of the request and then queries the k8s (through MIC) for a matching azure identity. It then make a adal request to get the token for the client id and returns as a reponse to the request. If the request had client id as part of the query it validates it againsts the admin configured client id.

Similarly a host can make an authorization request to fetch Service Principal Token for a resource directly from the NMI host endpoint (http://127.0.0.1:2579/host/token/). The request must include the pod namespace `podns` and the pod name `podname` in the request header and the resource endpoint of the resource requesting the token. The NMI server identifies the pod based on the `podns` and `podname` in the request header and then queries k8s (through MIC) for a matching azure identity.  Then nmi makes a adal request to get a token for the resource in the request, returns the `token` and the `clientid` as a reponse to the request

## Deploy 

- **AAD Pod Indentity Infra**: Deploy pod identity components to your cluster [deploy-pod-identity][deploy-pod-identity].
- **Application code with Service Pricipal/ Managed Identity**: Follow this tutorial to use the aad-pod-identity demo app which shows service principal / Managed identiy in action [tutorial][aad-pod-id-tutorial].
- **Mount KeyVault atrifacts as volume**: Follow this example to use aad-pod-identity to mount keyvault secrets as volume [aad-pod-id-keyvault][aad-pod-id-keyvault]

<!-- LINKS - external -->
[managed-identities-azure-resources]: https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview
[aad-pod-id-concept]
https://github.com/Azure/aad-pod-identity/blob/master/docs/design/concept.md
[aad-pod-id-block-diagram]
https://github.com/Azure/aad-pod-identity/blob/master/docs/design/concept.png
[deploy-pod-identity]
https://github.com/Azure/aad-pod-identity#deploy-the-azure-aad-identity-infra
[aad-pod-id-keyvault]
https://github.com/Azure/kubernetes-keyvault-flexvol#option-2---pod-identity
[aad-pod-id-turorial]
https://github.com/Azure/aad-pod-identity/tree/master/docs/tutorial
