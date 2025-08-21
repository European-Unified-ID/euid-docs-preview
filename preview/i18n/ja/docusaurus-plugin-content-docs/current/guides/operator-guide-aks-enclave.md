---
title: UID2 Private Operator for AKS Integration Guide
sidebar_label: Azure Kubernetes Service (AKS)
pagination_label: UID2 Private Operator for AKS Integration Guide
description: Integration information for Private Operator in AKS.
hide_table_of_contents: false
sidebar_position: 18
displayed_sidebar: docs
---

import Link from '@docusaurus/Link';
import UpgradePolicy from '../snippets/_private-operator-upgrade-policy.mdx';

# UID2 Private Operator for AKS Integration Guide

UID2 Operator は UID2 エコシステムの API サーバーです。詳細は、[The UID2 Operator](../ref-info/ref-operators-public-private.md) を参照してください。

:::note
AKS の Private Operator をセットアップしたい場合は、UID2 の連絡先にお問い合わせください。詳細は、[Contact Info](../getting-started/gs-account-setup.md#contact-info) を参照してください。
:::

This guide provides information for setting up the UID2 Operator Service as a <Link href="../ref-info/glossary-uid#gl-private-operator">Private Operator</Link> in an Azure Kubernetes Service (<Link href="../ref-info/glossary-uid#gl-aks">AKS</Link>) cluster, running on [virtual nodes on Azure Container Instances](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-virtual-nodes) (ACI). Virtual nodes on Azure Container Instances enable us to take advantage of confidential containers, which run in a hardware-backed Trusted Execution Environment (TEE) that provides intrinsic capabilities such as data integrity, data confidentiality, and code integrity. 

When the Docker container for the UID2 Operator confidential Azure Container instance starts up, it completes the attestation process that allows the UID2 Core Service to verify the authenticity of the Operator Service and of the enclave environment that the Operator Service is running in.

When the attestation is successful, the UID2 Core Service provides seed information such as salts and keys to bootstrap the UID2 Operator in the secure UID2 Operator confidential Azure Container Instance.

:::caution
UID2 Private Operator for AKS is not supported in these areas: Europe, China.
:::

<!-- ## Operator Version

The latest ZIP file is linked in the AKS Download column in the following table.

| AKS Version Name | Version&nbsp;#/Release&nbsp;Notes | AKS Download | Date | Deprecation Date |
| ------- | ------ | ------ | ------ | ------ |
| Q2 2025 | xxx | xxx | xxx | xxx |

:::note
For information about supported versions and deprecation dates, see [Private Operator Versions](../ref-info/deprecation-schedule.md#private-operator-versions).
::: -->

## Private Operator Upgrade Policy

<UpgradePolicy />

## Prerequisites

Before deploying the UID2 Private Operator for AKS, complete these prerequisite steps:

- [Set Up the UID2 Operator Account](#set-up-the-uid2-operator-account)
- [Install the Azure CLI](#install-the-azure-cli)
- [Get the Required Azure Permissions](#get-the-required-azure-permissions)
- [Install the kubectl CLI](#install-the-kubectl-cli)
- [Install the Helm CLI](#install-the-helm-cli)

### Set Up the UID2 Operator Account

Ask your UID2 contact to register your organization as a UID2 Operator. If you're not sure who to ask, see [Contact Info](../getting-started/gs-account-setup.md#contact-info).

When the registration process is complete, you'll receive an operator key, exclusive to you, that identifies you with the UID2 service as a Private Operator. During configuration, use this as the value for `OPERATOR_KEY`. This value is both your unique identifier and a password; store it securely and do not share it.

:::note
You'll receive a separate operator key for each deployment environment.
:::

### Install the Azure CLI

Azure コマンドラインインターフェイスをインストールします。詳細は、Azure ドキュメントの [How to install the Azure CLI](https://learn.microsoft.com/ja-jp/cli/azure/install-azure-cli) を参照してください。

### Get the Required Azure Permissions

You'll need to have subscription owner permission so that you can create a resource group.

When that's done, you only need contributor permission on the resource group level for that resource.

For details, see [Azure roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/rbac-and-directory-admin-roles#azure-roles) in the Azure documentation.

When all prerequisite steps are complete, you're ready to deploy the UID2 Private Operator. See [Deployment](#deployment).

### Install the kubectl CLI

Kubernetesの `kubectl` コマンドラインインターフェイスをインストールします。詳細は、Kubernetes ドキュメントの [Install Tools](https://kubernetes.io/docs/tasks/tools/) を参照してください。

### Install the Helm CLI

`helm` コマンドラインインターフェイスをインストールします。詳細は、[Installing Helm](https://helm.sh/docs/intro/install/) を参照してください。

## Deployment Environments

The following environments are available. As a best practice, we recommend that you test and verify your implementation in the integration environment before deploying in the production environment.

:::note
You'll receive separate `{OPERATOR_KEY}` values for each environment. Be sure to use the correct key for the environment you're using. The deployment artifacts and the process flow are the same for both environments.
:::

| Environment | Details |
| :--- | :--- |
| Integration (`integ`) | For testing only. Debug mode is available in the integration environment. |
| Production (`prod`) | For managing production traffic. |

## Deployment

To deploy a new UID2 Private Operator for AKS, you'll need to complete the following high-level steps:

- [Download ZIP File and Extract Files](#download-zip-file-and-extract-files)
- [Prepare Environment Variables](#prepare-environment-variables)
- [Set Up AKS and Node Pool](#set-up-aks-and-node-pool)
- [Set Up AKS Cluster](#set-up-aks-cluster)
- [Complete Key Vault and Managed Identity Setup](#complete-key-vault-and-managed-identity-setup)
- [Complete the UID2 Private Operator Setup](#complete-the-uid2-private-operator-setup)

### Download ZIP File and Extract Files

To get set up with the installation files, follow these steps:

1. Ask your UID2 contact for the ZIP file containing the deployment files.

1. Download and unzip the file.

<!-- Download the ZIP file linked in the following table, AKS Download column, for the latest version. 

1. Unzip the ZIP file to extract the following files, needed for the deployment:

   - `operator.yaml` -->

### Prepare Environment Variables

Run the following commands to prepare environment variables that you'll use later. Choose variable names to suit your needs.

```
export RESOURCE_GROUP="my-vn-aks"
export LOCATION="eastus"
export VNET_NAME="my-vnet"
export PUBLIC_IP_ADDRESS_NAME="my-public-ip"
export NAT_GATEWAY_NAME="my-nat-gateway"
export AKS_CLUSTER_NAME="myvncluster"
export KEYVAULT_NAME="my-vn-aks-vault"
export KEYVAULT_SECRET_NAME="my-vn-aks-opr-key-name"
export MANAGED_IDENTITY="my-vn-aks-opr-id"
export AKS_NODE_RESOURCE_GROUP="MC_${RESOURCE_GROUP}_${AKS_CLUSTER_NAME}_${LOCATION}"
export SUBSCRIPTION_ID="$(az account show --query id --output tsv)"
export DEPLOYMENT_ENV="integ"
```

### Set Up AKS and Node Pool

To set up AKS and the node pool, complete the following steps:

- [Create Resource Group](#create-resource-group)
- [Create Virtual Network](#create-virtual-network)
- [Create Subnets](#create-subnets)
- [Create Public IP Address](#create-public-ip-address)
- [Create NAT Gateway](#create-nat-gateway)
- [Configure NAT Service for Source Subnet](#configure-nat-service-for-source-subnet)
- [Get the AKS Subnet ID](#get-the-aks-subnet-id)
- [Create an AKS Service](#create-an-aks-service)
- [Get the Principal ID of the Managed Identity](#get-the-principal-id-of-the-managed-identity)
- [Create Contributor Role for the Two Resource Groups](#create-contributor-role-for-the-two-resource-groups)

#### Create Resource Group
In Azure, run the following command to create a resource group to run the UID2 Private Operator:

```
az group create --name "${RESOURCE_GROUP}" --location "${LOCATION}"
```

:::info
All the resources are provisioned later under the name you provide as the value for `${RESOURCE_GROUP}`.
:::

There are some limitations with regard to location:
- UID2 Private Operator for AKS is not supported in these areas: Europe, China.

- Azure 仮装ネットワークのデプロイメントの可用性は、[Resource availability & quota limits for ACI](https://learn.microsoft.com/ja-jp/azure/container-instances/container-instances-resource-and-quota-limits#confidential-container-resources-preview) を確認してください。

- To get the alias for the location, run the following command:

   ```
   az account list-locations -o table
   ```

#### Create Virtual Network

To create the virtual network, run the following command, using your own values as needed:

```
az network vnet create \
    --resource-group ${RESOURCE_GROUP} \
    --name ${VNET_NAME} \
    --location ${LOCATION} \
    --address-prefixes 10.0.0.0/8
```

#### Create Subnets

To create the subnets, run the following command, using your own values as needed:

```
# Default Subnet (10.0.0.0/24)
az network vnet subnet create \
    --resource-group ${RESOURCE_GROUP} \
    --vnet-name ${VNET_NAME} \
    --name default \
    --address-prefixes 10.0.0.0/24

# AKS Subnet (CIDR /16)
az network vnet subnet create \
    --resource-group ${RESOURCE_GROUP} \
    --vnet-name ${VNET_NAME} \
    --name aks \
    --address-prefixes 10.1.0.0/16

# Container Groups Subnet (CIDR /16) with Delegation
az network vnet subnet create \
    --resource-group ${RESOURCE_GROUP} \
    --vnet-name ${VNET_NAME} \
    --name cg \
    --address-prefixes 10.2.0.0/16 \
    --delegations Microsoft.ContainerInstance/containerGroups
```

#### Create Public IP Address

To create the public IP address, run the following command, using your own values as needed:

```
az network public-ip create --name ${PUBLIC_IP_ADDRESS_NAME} --resource-group ${RESOURCE_GROUP} --sku standard --allocation static
```

#### Create NAT Gateway

To create the [Azure Network Address Translation (NAT) gateway](https://learn.microsoft.com/en-us/azure/nat-gateway/nat-overview), run the following command, using your own values as needed:

```
az network nat gateway create \
    --resource-group ${RESOURCE_GROUP} \
    --name ${NAT_GATEWAY_NAME} \
    --public-ip-addresses ${PUBLIC_IP_ADDRESS_NAME} \
    --idle-timeout 4
```

#### Configure NAT Service for Source Subnet

To configure the NAT service, run the following command, using your own values as needed:

```
az network vnet subnet update \
    --resource-group ${RESOURCE_GROUP} \
    --vnet-name ${VNET_NAME} \
    --name cg \
    --nat-gateway ${NAT_GATEWAY_NAME}
```

#### Get the AKS Subnet ID

To create the AKS subnet ID, run the following command, using your own values as needed:

```
export AKS_SUBNET_ID=$(az network vnet subnet show \
    --resource-group ${RESOURCE_GROUP} \
    --vnet-name ${VNET_NAME} \
    --name aks \
    --query id \
    --output tsv)
```

#### Create an AKS Service

To create the AKS service, run the following command, using your own values as needed:

```
# Create the AKS cluster
az aks create \
    --resource-group ${RESOURCE_GROUP} \
    --name ${AKS_CLUSTER_NAME} \
    --location ${LOCATION} \
    --kubernetes-version 1.33 \
    --network-plugin azure \
    --network-policy calico \
    --vnet-subnet-id ${AKS_SUBNET_ID} \
    --service-cidr 10.4.0.0/16 \
    --dns-service-ip 10.4.0.10 \
    --node-vm-size Standard_D4d_v5 \
    --node-count 2 \
    --enable-cluster-autoscaler \
    --min-count 2 \
    --max-count 5 \
    --auto-upgrade-channel patch \
    --enable-managed-identity \
    --nodepool-name oprnodepool \
    --os-sku Ubuntu
```
:::note
必ず最新のサポートされている Kubernetes バージョンを使用してください。`--kubernetes-version` フラグを使用します。以前のバージョンを使用する場合は、長期サポート（LTS）を有効にする必要があります。詳細は、Microsoft ドキュメントの [Long-term support for Azure Kubernetes Service (AKS) versions](https://learn.microsoft.com/en-us/azure/aks/long-term-support) を参照してください。
:::

#### Get the Principal ID of the Managed Identity

To get the Principal ID, run the following command, using your own values as needed:

```
export MANAGED_IDENTITY_PRINCIPAL_ID="$(az aks show --resource-group ${RESOURCE_GROUP} --name ${AKS_CLUSTER_NAME} --query "identityProfile.kubeletidentity.clientId" --output tsv)"
```

For details, see [Get the principal ID of the system-assigned managed identity](https://learn.microsoft.com/en-us/azure/aks/use-managed-identity#get-the-principal-id-of-the-system-assigned-managed-identity) in the Microsoft Azure documentation.

#### Create Contributor Role for the Two Resource Groups

To create the contributor role for each group, run the following commands, using your own values as needed:

```
az role assignment create \
  --assignee ${MANAGED_IDENTITY_PRINCIPAL_ID} \
  --scope /subscriptions/${SUBSCRIPTION_ID}/resourceGroups/${AKS_NODE_RESOURCE_GROUP} \
  --role Contributor

az role assignment create \
  --assignee ${MANAGED_IDENTITY_PRINCIPAL_ID} \
  --scope /subscriptions/${SUBSCRIPTION_ID}/resourceGroups/${RESOURCE_GROUP} \
  --role Contributor
```

Additional reference information in external documentation:
- [Tutorial: Deploy virtual nodes on Azure Container Instances in your Azure Kubernetes Service cluster](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-virtual-nodes-helm)
- [Setting up a virtual node Environment](https://github.com/microsoft/VirtualNodesOnAzureContainerInstances?tab=readme-ov-file#setting-up-a-virtual-node-environment)

### Set Up AKS Cluster

To set up the AKS cluster, run the following command, using your own values as needed:

```
az aks get-credentials --name ${AKS_CLUSTER_NAME} --resource-group ${RESOURCE_GROUP}

az provider register -n Microsoft.ContainerInstance

git clone git@github.com:microsoft/virtualnodesOnAzureContainerInstances.git

helm install virtualnode virtualnodesOnAzureContainerInstances/Helm/virtualnode
 
# Wait for ~1 minute for virtualnode-0 to appear.
kubectl get nodes
```

### Complete Key Vault and Managed Identity Setup

The next step is to set up a [key vault](https://learn.microsoft.com/en-us/azure/key-vault/general/overview) and save the operator key in it. When you've created the key vault, you can create a [managed identity](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/overview) and grant it permission to access the key vault.

後で、AKS クラスターが起動すると、この ID を使用します。詳細は、Microsoft Azure ドキュメントの [Running pods with an Azure Managed Identity](https://github.com/microsoft/virtualnodesOnAzureContainerInstances/blob/main/Docs/PodCustomizations.md#running-pods-with-an-azure-managed-identity) を参照してください。

Follow these steps:

```
az identity create --name "${MANAGED_IDENTITY}" --resource-group "${RESOURCE_GROUP}" --location "${LOCATION}"

az keyvault create --name "${KEYVAULT_NAME}" --resource-group "${RESOURCE_GROUP}" --location "${LOCATION}" --enable-purge-protection --enable-rbac-authorization

export KEYVAULT_RESOURCE_ID="$(az keyvault show --resource-group "${RESOURCE_GROUP}" --name "${KEYVAULT_NAME}" --query id --output tsv)"

az keyvault secret set --vault-name "${KEYVAULT_NAME}" --name "${KEYVAULT_SECRET_NAME}" --value "<some value>"

export IDENTITY_PRINCIPAL_ID="$(az identity show --name "${MANAGED_IDENTITY}" --resource-group "${RESOURCE_GROUP}" --query principalId --output tsv)"

az role assignment create --assignee-object-id "${IDENTITY_PRINCIPAL_ID}" --role "Key Vault Secrets User" --scope "${KEYVAULT_RESOURCE_ID}" --assignee-principal-type ServicePrincipal
```

### Complete the UID2 Private Operator Setup

To complete the Private Operator setup, follow these steps:

- [Update Placeholder Values](#update-placeholder-values)
- [Deploy Operator](#deploy-operator)

#### Update Placeholder Values

前のステップを完了した後、プレースホルダ値を更新するには、次の手順に従ってください:

1. 以下のコマンドを実行して、Managed Identity ID を取得します:

   ```
   MANAGED_IDENTITY_ID=$(az identity show --name "${MANAGED_IDENTITY}" --resource-group "${RESOURCE_GROUP}" --query id --output tsv)
   ```

2. `operator.yaml` ファイルの `microsoft.containerinstance.virtualnode.identity` を、取得した Managed Identity ID で更新します:

   - For Linux, run:

   ```
   sed -i "s#IDENTITY_PLACEHOLDER#$MANAGED_IDENTITY_ID#g" "operator.yaml"
   ```

   - For MacOS, run:

   ```
   sed -i '' "s#IDENTITY_PLACEHOLDER#$MANAGED_IDENTITY_ID#g" "operator.yaml"
   ```

3. Vault Key と Secret 名を環境変数で更新します:

   - For Linux, run:


   ```
   sed -i "s#VAULT_NAME_PLACEHOLDER#$KEYVAULT_NAME#g" "operator.yaml"
   sed -i "s#OPERATOR_KEY_SECRET_NAME_PLACEHOLDER#$KEYVAULT_SECRET_NAME#g" "operator.yaml"
   sed -i "s#DEPLOYMENT_ENVIRONMENT_PLACEHOLDER#$DEPLOYMENT_ENV#g" "operator.yaml"
   ```

   - For MacOS, run:

   ```
   sed -i '' "s#VAULT_NAME_PLACEHOLDER#$KEYVAULT_NAME#g" "operator.yaml"
   sed -i '' "s#OPERATOR_KEY_SECRET_NAME_PLACEHOLDER#$KEYVAULT_SECRET_NAME#g" "operator.yaml"
   sed -i '' "s#DEPLOYMENT_ENVIRONMENT_PLACEHOLDER#$DEPLOYMENT_ENV#g" "operator.yaml"
   ```


#### Deploy Operator

Follow these steps to deploy the Private Operator:

1. Kubernetes configuration credentials を取得するには、次のコマンドを実行します:

   ```
   az aks get-credentials --name ${AKS_CLUSTER_NAME} --resource-group ${RESOURCE_GROUP}
   ```

2. Kubernetes configuration credentials を取得したら、次のコマンドを実行して Private Operator をデプロイします:

   ```
   kubectl apply -f operator.yaml
   ```

## Running the Health Check

ヘルスチェックエンドポイントを呼び出して、実装のヘルスチェックをテストします。

ヘルスチェックの実行は、エンドポイントを除き、インテグレーション環境と本番環境で同じです。

Follow these steps:

1. Get the public IP address for the service:

   ```
   IP=$(az network public-ip list --resource-group ${AKS_NODE_RESOURCE_GROUP} --query "[?starts_with(name, 'kubernetes')].ipAddress" --output tsv)
   ```

2. Operator のステータスをテストするには、ブラウザでヘルスチェックエンドポイントにアクセスします: `http://${IP}/ops/healthcheck`。

   HTTP 200 とレスポンスボディ `OK` が表示される場合、ステータスは正常です。

import AttestFailure from '../snippets/_private-operator-attest-failure.mdx';

<AttestFailure />

## Upgrading

ASK 用の UID2 Private Operator の新しいバージョンがリリースされると、独自の Private Operator をホストしている参加者は、更新のメール通知を受け取り、新しいリリースリンクまたはインストールファイルを取得するための手順が記載されます。アップグレードのための期間があり、その後、古いバージョンは非アクティブになり、サポートされなくなります。

To upgrade, complete the following steps:

1. Follow the instructions in [Download ZIP File and Extract Files](#download-zip-file-and-extract-files) to download the deployment file for the new version and then unzip it.

2. [Complete the UID2 Private Operator Setup](#complete-the-uid2-private-operator-setup) の手順に従い、新しいファイルを使用して AKS 実装を新しいバージョンでデプロイします。

3. 新しい AKS デプロイメントのヘルスを確認し、ステータスが正常であることを確認します。

4. 古い AKS ポッドが適切にシャットダウンされていることを再確認します:

   ```
   kubectl get pods
   ```
