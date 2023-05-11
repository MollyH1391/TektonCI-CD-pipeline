# Building a CI/CD Pipeline for Deploying an Application to GKE with TektonHub Tasks
This article presents the use of Tekton tasks and pipelines to deploy a Node.js app to a GKE cluster. The steps involve configuring a workspace and PVC for file transfer between different tasks, creating a service account and binding secrets and roles to grant permissions for pushing and pulling images from a Harbor registry, and granting permissions for deploying the application to the GKE cluster. Finally, the tasks are sequenced and configured into a pipeline, and the pipeline is instantiated and executed on-cluster using Pipelinerun to complete the application deployment.

## Prerequisite
- A GKE cluster
- Tekton set up in the cluster

## Steps
- [Create a PVC for data transfer between Tekton tasks](https://github.com/MollyH1391/TektonCI-CD-pipeline/tree/main/tektonhub-ci-cd-pipl#create-a-pvc-for-data-transfer-between-tekton-tasks)
- [Create a secret for Tekton pipeline to push Kaniko-built images to Harbor registry and bind to a service account](https://github.com/MollyH1391/TektonCI-CD-pipeline/tree/main/tektonhub-ci-cd-pipl#create-a-secret-for-tekton-pipeline-to-push-kaniko-built-images-to-harbor-registry-and-bind-to-a-service-account)
- [Create a secret to allow Tekton pipeline to pull images from the Harbor registry](https://github.com/MollyH1391/TektonCI-CD-pipeline/tree/main/tektonhub-ci-cd-pipl#create-a-secret-to-allow-tekton-pipeline-to-pull-images-from-the-harbor-registry)
- [Grant Kubernetes cluster deployment permission to a service account with a cluster role binding](https://github.com/MollyH1391/TektonCI-CD-pipeline/tree/main/tektonhub-ci-cd-pipl#grant-kubernetes-cluster-deployment-permission-to-a-service-account-with-a-cluster-role-binding)
- [Configure a Tekton pipeline and pipelinerun](https://github.com/MollyH1391/TektonCI-CD-pipeline/tree/main/tektonhub-ci-cd-pipl#configure-a-tekton-pipeline-and-pipelinerun)
- [Deployment Result]()

### Create a PVC for data transfer between Tekton tasks
This PVC must be bound to the workspace of the pipelinerun to enable data transfer between tasks.
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: kaniko-source-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

### Create a secret for Tekton pipeline to push Kaniko-built images to Harbor registry and bind to a service account
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: basic-user-pass
  annotations:
    tekton.dev/docker-0: <HARBOR_REGISTRY_url>
type: kubernetes.io/basic-auth
stringData:
  username: <HARBOR_REGISTRY_USERNAME>
  password: <HARBOR_REGISTRY_PASSWORD>

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: kaniko-service-account
secrets:
  - name: basic-user-pass
```

### Create a secret to allow Tekton pipeline to pull images from the Harbor registry
```bash
kubectl create secret docker-registry harbor-registry-key --docker-server=<HARBOR_REGISTRY_url> --docker-username=<HARBOR_REGISTRY_USERNAME> --docker-password=<HARBOR_REGISTRY_PASSWORD> --docker-email=<YOUR_EMAIL>
```

Add the 'imagePullSecret' property to the deployment.yaml file.
```yaml
    spec:
      imagePullSecrets:
        - name: harbor-registry-key 
      containers:
        - name: server
          image: <HARBOR_REGISTRY_URL>/molly/nodejs-hello-world:latest
```

### Grant Kubernetes cluster deployment permission to a service account with a cluster role binding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: nodejs-pipeline-role
rules:
- apiGroups: ["extensions", "apps", ""]
  resources: ["services", "deployments", "pods","pvc","job"]
  verbs: ["get", "create", "update", "patch", "list", "delete"]
---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: nodejs-pipeline-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: nodejs-pipeline-role
subjects:
- kind: ServiceAccount
  name: kaniko-service-account
```

### Configure a Tekton pipeline and pipelinerun
- pipeline: [tektonhub-ci-cd-pipeline.yaml](https://github.com/MollyH1391/TektonCI-CD-pipeline/blob/8eef94df49c096f1ef3508bd8364d204af22bec8/tektonhub-ci-cd-pipl/pipelines/tektonhub-ci-cd-pipeline.yaml)
    - Arrange tasks in the correct order within the pipeline
    - Deploy Kubernetes manifests to the cluster using the 'kubectl' command.

- pipelinerun: [run-tektonhub-ci-cd-pipeline.yaml](https://github.com/MollyH1391/TektonCI-CD-pipeline/blob/8eef94df49c096f1ef3508bd8364d204af22bec8/tektonhub-ci-cd-pipl/pipelines/run-tektonhub-ci-cd-pipeline.yaml)
    - Bind the service account and PVC
    - Instantiate and execute pipeline on cluster

### Result
The following outcome can be expected: a Node.js app deployed to a GKE cluster, and its deployment status can be confirmed using the kubectl command.

```bash
 kubectl get deployments.apps nodejs-hello-world

NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
nodejs-hello-world   1/1     1            1           3h28m

kubectl get pods | grep nodejs-hello-world

nodejs-hello-world-54d98b7b78-hgc9m               1/1     Running     0          3h28m
```
![tekton-pipeline](https://github.com/MollyH1391/TektonCI-CD-pipeline/blob/846855ff050f70a00373b695b4f16951c6e37701/tektonhub-ci-cd-pipl/GUI/tekton-pipelinerun.png)

![nodejs-app](https://github.com/MollyH1391/TektonCI-CD-pipeline/blob/65b4c730189c5bc85e04739321ee1b333cd80731/tektonhub-ci-cd-pipl/GUI/nodejs-app.png)