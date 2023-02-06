# Kaniko-Tekton-CI-CD-pipeline-DEMO

## prerequisite
- GKE cluster
- Tekton & Tekton Dashboard
- a repo that contains ready to containerized source code 
- a repo tahet contains kubernetes manifests
- Dockerfile

## Tekton tasks
path: TektonCI-CD-pipeline/kaniko-tekton-pipl/task/
Add three tasks
- clone-src
    - need to edit param: <gitea_src_repo_url> in this task 
- clone-manifests
    - need to edit param: <gitea_manifests_repo_url> in this task 
- kaniko-build-push
    - need to edit param: <harbor_image_registry> in this task 

Execute the command below and the result will look similar to the image below

```bash
kubectl apply -f clone-build.yaml 

task.tekton.dev/clone-src configured
task.tekton.dev/clone-manifests configured
task.tekton.dev/kaniko-build-push configured
```

![tekton-tasks](https://github.com/MollyH1391/TektonCI-CD-pipeline/blob/79a575283f8c303dd90558d3bdca1846dc6b019d/kaniko-tekton-pipl/GUI/task.png)

## Tekton pipeline
path: TektonCI-CD-pipeline/kaniko-tekton-pipl/pipeline/

Mount tasks sequentially on a pipeline .

The created pipeline will look similar to the image below.

![tekton-pipeline](https://github.com/MollyH1391/TektonCI-CD-pipeline/blob/aeba8e5bed5226866e61ad384b8b11dc33ec022c/kaniko-tekton-pipl/GUI/pipl.png)

## Create PVC in kubernetes cluster for sharing volumes between tasks
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: src-repo
spec:
  resources:
    requests:
      storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
```

```bash
kubectl get pvc

NAME       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
src-repo   Bound    pvc-72189a03-88e5-43da-85ef-db3d48f21ad1   1Gi        RWO            standard       47h
```

## Create a secret for harbor registry
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: basic-user-pass
  annotations:
    tekton.dev/docker-0: <HARBOR_REGISTRY_URL>
type: kubernetes.io/basic-auth
stringData:
  username: <harbor_registry_username>
  password: <harbor_registry_password>
```

```bash
kubectl get secrets

NAME              TYPE                       DATA   AGE
basic-user-pass   kubernetes.io/basic-auth   2      43h
```

## Create a serviceaccount and mount the secret above for pushing image to harbor registry
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kaniko-service-account
secrets:
  - name: basic-user-pass
```

```bash
kubectl get serviceaccounts

NAME                     SECRETS   AGE
default                  0         47h
kaniko-service-account   1         43h
```

## Create pipelinerun
path: TektonCI-CD-pipeline/kaniko-tekton-pipl/pipelinerun/

- Mount serviceaccount to pipelinerun
- Create a pipelinerun

```bash
kubectl create -f run-tekton-kaniko-pipl.yaml

pipelinerun.tekton.dev/fetch-src-2gzpj created
```

The pipeline will start after the command above and the details is presented in the images below.

![tekton-pipelinerun-first-step](https://github.com/MollyH1391/TektonCI-CD-pipeline/blob/aeba8e5bed5226866e61ad384b8b11dc33ec022c/kaniko-tekton-pipl/GUI/pipe01.png)

![tekton-pipelinerun-second-step](https://github.com/MollyH1391/TektonCI-CD-pipeline/blob/aeba8e5bed5226866e61ad384b8b11dc33ec022c/kaniko-tekton-pipl/GUI/pipl02.png)

![tekton-pipelinerun-thrid-step](https://github.com/MollyH1391/TektonCI-CD-pipeline/blob/aeba8e5bed5226866e61ad384b8b11dc33ec022c/kaniko-tekton-pipl/GUI/pipl03.png)




