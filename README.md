# Harness CI/CD Pipeline for Modern Microservices Application

This repository contains a **Harness CI/CD pipeline** for building, and deploying a modern microservices-based application. The pipeline automates Docker image creation, pushing to a container registry, updating Helm charts, and rolling out deployments to VKS Kubernetes cluster.

---

## Prerequisites

Before running this pipeline, ensure the following **connectors and references** are created in your Harness account. These are the names as referenced in the pipeline:

### Fork the Following Repositories

1. **Application Source Code Repository**  
   GitHub Repo: [vks-gitops-demo](https://github.com/rahulk010/vks-gitops-demo.git)  

   It contains the source code for the microservices in this pipeline.

2. **Helm Charts Repository**  
   GitHub Repo: [helm-charts](https://github.com/rahulk010/helm-charts.git)  

   It contains Helm chart configurations, including the necessary values and deployment configurations to deploy the microservices in a Kubernetes cluster.
   

### Connectors and References

1. **GitHub Connector** – `gitrepoconnector`  
   **Note:** When creating the GitHub connector, **tick "Enable API access"** (recommended), as it's needed by Harness to interact with the GitHub API.  
   [Harness GitHub Connector Documentation](https://developer.harness.io/docs/platform/connectors/code-repositories/connect-to-code-repo/)

2. **Quay.io Docker Registry Connector** – `quayrepo`  
   [Harness Docker Registry Connector Documentation](https://developer.harness.io/docs/platform/connectors/cloud-providers/ref-cloud-providers/docker-registry-connector-settings-reference/)

3. **Kubernetes Connector** – `vks-cluster`  
   [Harness Kubernetes Connector Documentation](https://developer.harness.io/docs/platform/connectors/cloud-providers/add-a-kubernetes-cluster-connector/)

4. **Service Reference** – `appdeployservice`  
   [Harness Service Reference Documentation](https://developer.harness.io/docs/continuous-delivery/x-platform-cd-features/services/services-overview/)

5. **Environment Reference** – `prodenv`  
   [Harness Environment Reference Documentation](https://developer.harness.io/docs/continuous-delivery/x-platform-cd-features/environments/environment-overview/)

Additionally, ensure that **Redis** is available as a prerequisite, as it is required by the `cartservice` microservice to be deployed as a pod in the Kubernetes cluster. **`redis:latest`** should be available for this purpose, as it is not part of the source code.

> **Note:** If you choose to create connectors with different names, you must update the pipeline YAML accordingly to reference the correct connector names before running the pipeline.

---

## Kubernetes Delegate Setup

To allow Harness SaaS to connect to your **Kubernetes cluster** (`vks-cluster`), you need to install the **Harness Delegate** in your Kubernetes cluster. The Delegate is responsible for performing various tasks like building, deploying, and managing services.

You can install the Delegate using **Helm**, which is the recommended method for deployment. Here’s the Helm command to install the Delegate:

### Helm Command Example

```bash
helm upgrade -i helm-delegate-vks --namespace harness-delegate \
  harness-delegate/harness-delegate-ng \
  --set delegateName=helm-delegate-vks \
  --set accountId=axs4qKS3SYmPeIACSysCuA \
  --set delegateToken=NjBiNzA0MDE3OWVlYjlmMWQ1ZjViMTNhZjI1YjA3ZGQ= \
  --set managerEndpoint=https://app.harness.io \
  --set delegateDockerImage=us-docker.pkg.dev/gar-prod-setup/harness-public/harness/delegate:25.10.87101 \
  --set replicas=1 --set upgrader.enabled=true
```
### Explanation of the Helm Command

* helm upgrade -i helm-delegate-vks: Installs or upgrades the Harness Delegate with the name helm-delegate-vks in the harness-delegate namespace.
* --set delegateName=helm-delegate-vks: Sets the name of the delegate to helm-delegate-vks.
* --set accountId=axs4qKS3SYmPeIACSysCuA: Specifies your Harness account ID.
* --set delegateToken=NjBiNzA0MDE3OWVlYjlmMWQ1ZjViMTNhZjI1YjA3ZGQ=: This is your delegate token, which can be retrieved from the Harness UI (ensure to replace it with your actual token).
* --set managerEndpoint=https://app.harness.io: Points to the Harness SaaS instance endpoint.
* --set delegateDockerImage=us-docker.pkg.dev/gar-prod-setup/harness-public/harness/delegate:25.10.87101: Specifies the version of the Harness Delegate Docker image to use.
* --set replicas=1: Deploys a single instance of the Delegate. You can scale this as needed.
* --set upgrader.enabled=true: Enables automatic upgrades for the Delegate.


### Installation Notes

1. Install the Helm chart with the command above to set up the Harness Delegate in your Kubernetes cluster.
2. After installing the Delegate, you should be able to link the Kubernetes cluster to Harness by using the Kubernetes Connector (vks-cluster).
3. If you wish to scale the number of Delegate replicas, you can modify the replicas value in the Helm command accordingly.

### Pipeline Overview

* Pipeline Name: cicd-pipeline
* Project: mymodernapp
* Type: CI/CD
* Stages:
  1. Build and Push: Builds Docker images for multiple microservices and pushes them to Quay.io.
  2. Deploy: Deploys the updated images to a Kubernetes cluster using Helm charts.
---

## Pipeline Details

### Stage 1: Build and Push

- **Purpose:** Build Docker images for all microservices and push to Quay.io.
- **Key Features:**
  - Clone the microservices repository.
  - Cache builds to improve pipeline speed.
  - Build Docker images for all services with the execution ID as the tag.
  - Push images to Quay.io Docker registry.
  - Update Helm chart values (`values-prod.yaml`) with the new image tags and push changes back to GitHub.

**Microservices built:**
- `adservice`
- `cartservice`
- `checkoutservice`
- `currencyservice`
- `emailservice`
- `frontend`
- `loadgenerator`
- `paymentservice`
- `productcatalogservice`
- `recommendationservice`
- `shippingservice`
- `shoppingassistantservice`

---

### Stage 2: Deploy

- **Purpose:** Deploy the application to Kubernetes using Harness.
- **Deployment Type:** Rolling Kubernetes deployment.
- **Environment:** `prodenv`
- **Infrastructure:** `vkscluster`
- **Rollback Strategy:** Rollback deployment in case of failure.

---

## How to Use

1. **Fork Repositories**  
   Fork the following repositories into your GitHub account:
   - [vks-gitops-demo](https://github.com/rahulk010/vks-gitops-demo) (application source code)
   - [helm-charts](https://github.com/rahulk010/helm-charts) (Helm charts for deployment)
2. **Configure Harness CI/CD Connectors**  
   Set up the following connectors in Harness:
   - **GitHub Connector**: Use `gitrepoconnector` with **API access enabled**.
   - **Quay Docker Registry Connector**: Use `quayrepo`.
   - **Kubernetes Connector**: Use `vks-cluster`.
   - **Service Reference**: Use `appdeployservice`.
   - **Environment Reference**: Use `prodenv`.
3. **Set Up Redis**  
   Ensure **Redis (redis:latest)** is deployed as a pod in your Kubernetes cluster for the `cartservice`.
4. **Install Harness Delegate**  
   Install the **Harness Delegate** in your Kubernetes cluster. Follow the [official Harness documentation](https://support.harness.io/) for installation instructions.
5. **Update Pipeline YAML**  
   Ensure connector names in `cicd-pipeline.yaml` match the ones configured in Harness. Modify if necessary.
6. **Create the Pipeline**  
   Create a new pipeline in Harness using the `cicd-pipeline.yaml`.
7. **Trigger the Pipeline**  
   Trigger the pipeline manually or via a Git webhook for automatic builds.
8. **Monitor Pipeline**  
   Monitor pipeline execution in Harness to check build and deployment results.

---

## Notes

- The pipeline uses `<+pipeline.executionId>` as a dynamic tag for Docker images.
- Helm charts are automatically updated with the new image tags and committed back to the Git repository.
- Caching is enabled for Docker builds to reduce build time.

---

## Contributing

Feel free to open issues or submit pull requests for improvements, including:
- Additional microservices
- Integration with other Docker registries
- Enhanced deployment strategies

---

## References

- [Harness Documentation](https://docs.harness.io/)
- [Quay.io Documentation](https://quay.io/)
- [Kubernetes Rolling Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Helm Charts](https://helm.sh/docs/)

---


