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
gcloud container clusters create-auto hello-cluster \
    --region=COMPUTE_REGION
```

## Legal Notices

Microsoft and any contributors grant you a license to the Microsoft documentation and other content
in this repository under the [Creative Commons Attribution 4.0 International Public License](https://creativecommons.org/licenses/by/4.0/legalcode),
see the [LICENSE](LICENSE) file, and grant you a license to any code in the repository under the [MIT License](https://opensource.org/licenses/MIT), see the
[LICENSE-CODE](LICENSE-CODE) file.

Microsoft, Windows, Microsoft Azure and/or other Microsoft products and services referenced in the documentation
may be either trademarks or registered trademarks of Microsoft in the United States and/or other countries.
The licenses for this project do not grant you rights to use any Microsoft names, logos, or trademarks.
Microsoft's general trademark guidelines can be found at http://go.microsoft.com/fwlink/?LinkID=254653.

Privacy information can be found at https://privacy.microsoft.com/en-us/

Microsoft and any contributors reserve all other rights, whether under their respective copyrights, patents,
or trademarks, whether by implication, estoppel or otherwise.
