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

### 2.1. Create GKE

Google has a managed Kubernetes service, GKE (Google Kubernetes Engine), we'll use this to easily deploy and standup a Kubernetes cluster.

### Tasks

#### Before you begin

You can check in the link [create a zonnal gke cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-zonal-cluster)

Before you start, make sure you have performed the following tasks:
* Enable the Google Kubernetes Engine API.
**Note** [Enable Google Kubernetes Engine API](https://console.cloud.google.com/flows/enableapi?apiid=container.googleapis.com&_ga=2.63667439.1539050351.1679471184-1748947644.1679383172)

* If you want to use the Google Cloud CLI for this task, install and then initialize the gcloud CLI.

#### Create a GKE zonal cluster
Replace the following:

* CLUSTER_NAME: the name of your new cluster.
* CHANNEL: the type of release channel, which can be one of rapid, regular, stable, or None. By default, the cluster is enrolled in the regular release channel if the following flags aren't specified: --cluster-version, --release-channel, --no-enable-autoupgrade, and --no-enable-autorepair.
* COMPUTE_ZONE: the compute zone for the cluster control plane.
* VERSION: the version you wish to specify for your cluster.
* COMPUTE_ZONE,COMPUTE_ZONE1,[...]: the zones in which nodes are created. You can specify as many zones as needed for your cluster. All zones must be in the same region as the cluster's control plane, specified by the --zone flag. For zonal clusters, --node-locations must contain the cluster's primary zone.

In the following commands, you can optionally use the --service-account=SERVICE_ACCOUNT_NAME@PROJECT_ID.iam.gserviceaccount.com flag to specify a different IAM service account that nodes in your cluster's first node pool uses instead of the Compute Engine default service account. This flag is optional, but we strongly recommend that you create and use a minimally-privileged service account so that your nodes don't have more privileges that they require.

##### Using a specific release channel:

To create a new cluster using a specific release channel, run the following command:
```sh 
gcloud container clusters create CLUSTER_NAME \
    --release-channel CHANNEL \
    --zone COMPUTE_ZONE \
    --node-locations COMPUTE_ZONE,COMPUTE_ZONE1
```

##### Using a specific version:
To create a new cluster using a specific cluster version, run the following command:
```sh
gcloud container clusters create CLUSTER_NAME \
    --cluster-version VERSION \
    --zone COMPUTE_ZONE \
    --node-locations COMPUTE_ZONE,COMPUTE_ZONE1
```

#### Using the static default version:
To create a new cluster using the static default cluster version, you don't need to specify a cluster version, but you do need to set the release channel to None:

```sh 
gcloud container clusters create CLUSTER_NAME \
    --release-channel None \
    --zone COMPUTE_ZONE \
    --node-locations COMPUTE_ZONE,COMPUTE_ZONE1
```

#### Example

```sh
gcloud container clusters create example-cluster \
    --zone us-central1-a \
    --node-locations us-central1-a,us-central1-b,us-central1-c
```    


#### Ensure you can connect to the cluster using `kubectl`

**Task Hints**

* `kubectl` is the main command line tool you will be using for working with Kubernetes and GKE. It is already installed in the Google Cloud Shell
* Refer to the GKE docs which includes [Kubectl 설치 및 클러스터 액세스 구성](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl?hl=ko)
* A good sanity check is listing all the nodes in your cluster `kubectl get nodes`.
* [This is a good cheat sheet](https://linuxacademy.com/site-content/uploads/2019/04/Kubernetes-Cheat-Sheet_07182019.pdf) for kubectl.

> **Note** `kubectl`, the Kubernetes CLI, is already installed on the Google Cloud Shell.

1. Install Kubectl

kubectl 구성요소를 설치합니다.
```sh
gcloud components install kubectl
```

2. kubectl version

kubectl version을 확인합니다.
```sh
kubectl version
```
3. Check the nodes

List the available nodes

```sh
kubectl get nodes
```

> **Resources**
> * <https://cloud.google.com/kubernetes-engine/docs/concepts/service?hl=ko>
> * <https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl?hl=ko>
> * <https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster?hl=ko>
> * <https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-zonal-cluster>
> * <https://console.cloud.google.com/flows/enableapi?apiid=container.googleapis.com&_ga=2.63667439.1539050351.1679471184-1748947644.1679383172>
> * <https://linuxacademy.com/site-content/uploads/2019/04/Kubernetes-Cheat-Sheet_07182019.pdf>


### 2.2. Deploy MongoDB
You need to deploy MongoDB in a way that is scalable and production ready.

**Task Hints**
* Use Helm and a standard provided Helm chart to deploy MongoDB.
* Be careful with the authentication settings when creating MongoDB. It is recommended that you create a standalone username/password. The username and password can be anything you like, but make a note of them for the next task. 

> **Important**: If you install using Helm and then delete the release, the MongoDB data and configuration persists in a Persistent Volume Claim. You may face issues if you redeploy again using the same release name because the authentication configuration will not match. If you need to delete the Helm deployment and start over, make sure you delete the Persistent Volume Claims created otherwise you'll run into issues with authentication due to stale configuration. Find those claims using `kubectl get pvc`.

### Tasks

#### Setup Helm

Helm is an application package manager for Kubernetes, and a way to easily deploy applications and services into Kubernetes, via what are called charts. To use Helm you will need the `helm` command (This is already installed if you're using the Google Cloud Shell).

**Task Hints**
* These instructions use [Helm version 3](https://helm.sh/blog/helm-3-released/).
* Helm version 3 does not come with any repositories predefined, so you'll need initialize the [stable chart repository](https://v3.helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository)


Add the `stable` Helm charts repository
`helm repo add stable https://kubernetes-charts.storage.googleapis.com/`

Upate the repositories
`helm repo update`



#### Deploy an instance of MongoDB to your cluster

A standard repository of Helm charts is available for many different software packages, and it has one for [MongoDB](https://github.com/helm/charts/tree/master/stable/mongodb) that is easily replicated and horizontally scalable. 

**Task Hints**
* When installing a chart, Helm uses a concept called a "release" and each release needs a name. We recommend you name your release `orders-mongo` to make it easier to follow later steps in this workshop
* When deploying a chart you provide parameters with the `--set` switch and a comma separated list of `key=value` pairs. There are MANY parameters you can provide to the MongoDB chart, but pay attention to the `mongodbUsername`, `mongodbPassword` and `mongodbDatabase` parameters 

> **Note** The application expects a database named `gkechallenge`. Using a different database name will cause the application to fail!

The recommended way to deploy MongoDB would be to use a Helm Chart. 

```sh
helm install orders-mongo stable/mongodb --set mongodbUsername=orders-user,mongodbPassword=orders-password,mongodbDatabase=gkechallenge
```

> **Hint** Using this command, the Helm Chart will expose the MongoDB instance as a Kubernetes Service accessible at ``orders-mongo-mongodb.default.svc.cluster.local``

Remember to use the username and password from the command above when creating the Kubernetes secrets in the next step.


#### Create a Kubernetes secret to hold the MongoDB details

In the previous step, you installed MongoDB using Helm, with a specified username, password and a hostname where the database is accessible. You'll now create a Kubernetes secret called `mongodb` to hold those details, so that you don't need to hard-code them in the YAML files.

**Task Hints**
* A Kubernetes secret can hold several items, indexed by key. The name of the secret isn't critical, but you'll need three keys to store your secret data:
  * `mongoHost`
  * `mongoUser`
  * `mongoPassword`
* The values for the username & password will be those you used with the `helm install` command when deploying MongoDB.
* Run `kubectl create secret generic -h` for help on how to create a secret, clue: use the `--from-literal` parameter to allow you to provide the secret values directly on the command in plain text.
* The value of `mongoHost`, will be dependent on the name of the MongoDB service. The service was created by the Helm chart and will start with the release name you gave. Run `kubectl get service` and you should see it listed, e.g. `orders-mongo-mongodb`
* All services in Kubernetes get DNS names, this is assigned automatically by Kubernetes, there's no need for you to configure it. You can use the short form which is simply the service name, e.g. `orders-mongo-mongodb` or better to use the "fully qualified" form `orders-mongo-mongodb.default.svc.cluster.local`
  

```sh
kubectl create secret generic mongodb --from-literal=mongoHost="orders-mongo-mongodb.default.svc.cluster.local" --from-literal=mongoUser="orders-user" --from-literal=mongoPassword="orders-password"
```

You'll need to reference this secret when configuring the Order Capture application later on.


> **Resources**
> * <https://helm.sh/docs/intro/using_helm/>
> * <https://github.com/helm/charts/tree/master/stable/mongodb>
> * <https://kubernetes.io/docs/concepts/configuration/secret/>

### Architecture Diagram
Here's a high level diagram of the components you will have deployed when you've finished this section (click the picture to enlarge)

<a href="media/architecture/mongo.png" target="_blank"><img src="media/architecture/mongo.png" style="width:500px"></a>

* The **pod** holds the containers that run MongoDB
* The **deployment** manages the pod
* The **service** exposes the pod to the Internet using a public IP address and a specified port


## 3. Legal Notices

### 3.1 Legal Notices



