# Building a CI/CD Pipeline for Deploying an Application to GKE with TektonHub Tasks
This document demonstrates how to deploy a Node.js app to a Kubernetes cluster using Tekton tasks and pipelines.
## Prerequisite
- A GKE cluster
- Tekton set up in the cluster

## Steps
- Create a PVC for data transfer between Tekton tasks
- Create a secret for Tekton pipeline to push Kaniko-built images to Harbor registry and bind to a service account
- Create a secret to allow Tekton pipeline to pull images from the Harbor registry
- Grant Kubernetes cluster deployment permission to a service account with a cluster role binding
- Configure a Tekton pipeline and pipelinerun

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

### Results:
The following output can be expected.

```bash
 kubectl get deployments.apps nodejs-hello-world

NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
nodejs-hello-world   1/1     1            1           3h28m

kubectl get pods | grep nodejs-hello-world

nodejs-hello-world-54d98b7b78-hgc9m               1/1     Running     0          3h28m
```
![tekton-pipeline](https://github.com/MollyH1391/TektonCI-CD-pipeline/blob/846855ff050f70a00373b695b4f16951c6e37701/tektonhub-ci-cd-pipl/GUI/tekton-pipelinerun.png)

