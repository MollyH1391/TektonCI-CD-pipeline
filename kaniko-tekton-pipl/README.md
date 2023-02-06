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







