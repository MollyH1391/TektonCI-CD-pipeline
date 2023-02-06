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

## Tekton pipeline







