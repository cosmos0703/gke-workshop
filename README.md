# The Google Kubernetes Engine Workshop

**Welcome to the Google Cloud Kubernetes Workshop. In this lab, you'll go through tasks that will help you master the basic and more advanced topics required to deploy a multi-container application to Kubernetes on [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine/docs/concepts/service?hl=ko).**

**You can use this guide as a Kubernetes tutorial and as study material to help you get started to learn Kubernetes.**

**Some of the things you’ll be going through:**

**- Kubernetes deployments, services and ingress**
**- Deploying MongoDB using Helm version 3**
**- Google Monitor for Containers, Horizontal Pod Autoscaler and the Cluster Autoscaler**

## 1. k8sbasics

There is an assumption of some prior knowledge of Kubernetes and its concepts. If you are new to Kubernetes, start with the [Kubernetes Learning Path](https://kubernetes.io/docs/tutorials/kubernetes-basics/) to learn Kubernetes basics, then go through the concepts of [what Kubernetes is and what it isn't](https://kubernetes.io/docs/concepts/overview/).

- Kubernetes deployments
- Kubernetes services
- Kubernetes ingress

### 1.1. prerequisites

Set up the GKS Cluster
시작하기 전에 다음 태스크를 수행했는지 확인합니다. [GKE Cluster 생성](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster?hl=ko)
아래 내용은 대략적인 내용만 담고 있으니 Cluster 생성을 위해서는 공식 문서를 확인 바랍니다.

- gcloud project id 설정
- network 설정
- Google Kubernetes Engine API 사용 설정

```sh
gcloud config set project PROJECT_ID
gcloud container clusters create example-cluster \
    --zone us-central1-a \
    --node-locations us-central1-a,us-central1-b,us-central1-c
```

kubectl 설치 및  클러스터 액스 구성은 다음 문서를 확인합니다. [Kubectl 설치 및 클러스터 액세스 구성](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl?hl=ko)

### 1.2 Application Overview
You will be deploying a customer-facing order placement and fulfillment application that is containerized and is architected for a microservice implementation.

![Application diagram](media/overview.png)

The application consists of 3 components:

* A public facing Order Capture swagger enabled API
* A public facing frontend
* A MongoDB database

### 1.3 Challenges
Useful resources are provided to help you work through each task. If you're working through this as part of a team based hack, ensure you make progress at a good pace by dividing the workload between team members where possible. This may mean anticipating work that might be required in a later task.

> **Hint**: If you get stuck, you can ask for help from the proctors. You may also choose to peek at the solutions.

### Core tasks

Running through this as part of a one day workshop, you should be able to complete all of the **Getting up and running** section. This involves setting up a Kubernetes cluster, deploying the application containers from Docker Hub, setting up monitoring and scaling your application.

### DevOps tasks (To be scheduled)

Once you're done with the Core tasks, next would be to include some DevOps. **Complete as many tasks as you can**. You'll be setting up a Continuous Integration and Continuous Delivery pipeline for your application and then using Helm to deploy it.

### Advanced cluster tasks

If you're up to it, explore configuring the Google Kubernetes Engine with Auto pilot Nodes, enabling MongoDB replication, and using the Google Secret Manager for secrets.

## 2. Getting up and running

### 2.1. Deploy MongoDB

Google has a managed Kubernetes service, GKE (Google Kubernetes Engine), we'll use this to easily deploy and standup a Kubernetes cluster.

### Tasks

#### Before you begin

{% collapsible %}

Before you start, make sure you have performed the following tasks:
* Enable the Google Kubernetes Engine API.
```sh
Enable Google Kubernetes Engine API (https://console.cloud.google.com/flows/enableapi?apiid=container.googleapis.com&_ga=2.63667439.1539050351.1679471184-1748947644.1679383172)
```

* If you want to use the Google Cloud CLI for this task, install and then initialize the gcloud CLI.


Google Kubernetes Engine API를 사용 설정합니다.
Google Kubernetes Engine API 사용 설정
이 태스크에 Google Cloud CLI를 사용하려면 gcloud CLI를 설치한 후 초기화합니다.
참고: 기존 gcloud CLI 설치의 경우 compute/region 및 compute/zone 속성을 설정해야 합니다. 기본 위치를 설정하면 gcloud CLI에서 One of [--zone, --region] must be supplied: Please specify location과 같은 오류를 방지할 수 있습니다.
멀티 영역 클러스터는 단일 영역 클러스터보다 리소스를 더 많이 사용합니다. 멀티 영역 클러스터를 만드는 경우 적절한 할당량이 있는지 확인하세요.
클러스터를 만들 수 있는 올바른 권한이 있는지 확인합니다. 최소한 Kubernetes Engine 클러스터 관리자 권한이 있어야 합니다.






Get the latest available Kubernetes version in your preferred region and store it in a bash variable. Replace `<region>` with the region of your choosing, for example `eastus`.

```sh
version=$(az aks get-versions -l <region> --query 'orchestrators[?!isPreview] | [-1].orchestratorVersion' -o tsv)
```

The above command lists all versions of Kubernetes available to deploy using AKS. Newer Kubernetes releases are typically made available in "Preview". To get the latest non-preview version of Kubernetes, use the following command instead

```sh
version=$(az aks get-versions -l <region> --query 'orchestrators[?isPreview == null].[orchestratorVersion][-1]' -o tsv)
```

{% endcollapsible %}

#### Create a Resource Group

> **Note** You don't need to create a resource group if you're using the lab environment. You can use the resource group created for you as part of the lab. To retrieve the resource group name in the managed lab environment, run `az group list`.

{% collapsible %}

```sh
az group create --name <resource-group> --location <region>
```

{% endcollapsible %}

#### Create the AKS cluster

**Task Hints**
* It's recommended to use the Azure CLI and the `az aks create` command to deploy your cluster. Refer to the docs linked in the Resources section, or run `az aks create -h` for details
* The size and number of nodes in your cluster is not critical but two or more nodes of type `Standard_DS2_v2` or larger is recommended

> **Note** You can create AKS clusters that support the [cluster autoscaler](https://docs.microsoft.com/en-us/azure/aks/cluster-autoscaler#about-the-cluster-autoscaler).

##### **Option 1:** Create an AKS cluster without the cluster autoscaler (recommended)

Create AKS using the latest version (if using the provided lab environment)
  
{% collapsible %}
  
> **Note** If you're using the provided lab environment, you'll not be able to create the Log Analytics workspace required to enable monitoring while creating the cluster from the Azure Portal unless you manually create the workspace in your assigned resource group. Additionally, if you're running this on an Azure Pass, please add `--load-balancer-sku basic` to the flags, as the Azure Pass only supports the basic Azure Load Balancer. Additionaly, please pass in the service prinipal and secret provided.

  ```sh
  az aks create --resource-group <resource-group> \
    --name <unique-aks-cluster-name> \
    --location <region> \
    --kubernetes-version $version \
    --generate-ssh-keys \
    --load-balancer-sku basic \
    --service-principal <APP_ID> \
    --client-secret <APP_SECRET>
  ```

  {% endcollapsible %}
  
  Create AKS using the latest version (on your own subscription)
  
  {% collapsible %}

  ```sh
  az aks create --resource-group <resource-group> \
    --name <unique-aks-cluster-name> \
    --location <region> \
    --kubernetes-version $version \
    --generate-ssh-keys
  ```

  {% endcollapsible %}

##### **Option 2 ** Create an AKS cluster with the cluster autoscaler

 
  AKS clusters create worker nodes in Virtual Machine Scale Sets by default. The number of nodes can be easily scaled up and down as required. AKS also supports the Kubernetes Cluster Autoscaler, which will automatically scale the number of nodes on demand to meet current system requirements. 
  
  To enable the Cluster Autoscaler, use the `az aks create` command specifying the `--enable-cluster-autoscaler` parameter, and a node `--min-count` and `--max-count`.
  
Create AKS using the latest version (if using the provided lab environment)

{% collapsible %}

> **Note** If you're running this on an Azure Pass or the provided lab environment, please add `--load-balancer-sku basic` to the flags, as the Azure Pass only supports the basic Azure Load Balancer. Additionaly, please pass in the service prinipal and secret provided.

   ```sh
  az aks create --resource-group <resource-group> \
    --name <unique-aks-cluster-name> \
    --location <region> \
    --kubernetes-version $version \
    --generate-ssh-keys \
    --vm-set-type VirtualMachineScaleSets \
    --enable-cluster-autoscaler \
    --min-count 1 \
    --max-count 3 \
    --load-balancer-sku basic \
    --service-principal <APP_ID> \
    --client-secret <APP_SECRET>
  ```

{% endcollapsible %}
  
Create AKS using the latest version (on your own subscription)

{% collapsible %}

   ```sh
  az aks create --resource-group <resource-group> \
    --name <unique-aks-cluster-name> \
    --location <region> \
    --kubernetes-version $version \
    --generate-ssh-keys \
    --vm-set-type VirtualMachineScaleSets \
    --enable-cluster-autoscaler \
    --min-count 1 \
    --max-count 3
  ```

  {% endcollapsible %}

#### Ensure you can connect to the cluster using `kubectl`

**Task Hints**
* `kubectl` is the main command line tool you will be using for working with Kubernetes and AKS. It is already installed in the Azure Cloud Shell
* Refer to the AKS docs which includes [a guide for connecting kubectl to your cluster](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough#connect-to-the-cluster) (Note. using the cloud shell you can skip the `install-cli` step).
* A good sanity check is listing all the nodes in your cluster `kubectl get nodes`.
* [This is a good cheat sheet](https://linuxacademy.com/site-content/uploads/2019/04/Kubernetes-Cheat-Sheet_07182019.pdf) for kubectl.
* If you run kubectl in PowerShell ISE , you can also define aliases :
```sh
function k([Parameter(ValueFromRemainingArguments = $true)]$params) { & kubectl $params }
function kubectl([Parameter(ValueFromRemainingArguments = $true)]$params) { Write-Output "> kubectl $(@($params | ForEach-Object {$_}) -join ' ')"; & kubectl.exe $params; }
function k([Parameter(ValueFromRemainingArguments = $true)]$params) { Write-Output "> k $(@($params | ForEach-Object {$_}) -join ' ')"; & kubectl.exe $params; }
```

{% collapsible %}

> **Note** `kubectl`, the Kubernetes CLI, is already installed on the Azure Cloud Shell.

Authenticate

```sh
az aks get-credentials --resource-group <resource-group> --name <unique-aks-cluster-name>
```

List the available nodes

```sh
kubectl get nodes
```

{% endcollapsible %}

> **Resources**
> * <https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough>
> * <https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-create>
> * <https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal>
> * <https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough#connect-to-the-cluster>
> * <https://linuxacademy.com/site-content/uploads/2019/04/Kubernetes-Cheat-Sheet_07182019.pdf>


## 3. Legal Notices

### 3.1 Legal Notices



