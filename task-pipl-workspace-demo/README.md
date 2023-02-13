# TektonCI-CD-pipeline

## A Simple Workspace Transition in a Tekton Pipeline
path: demo-pipl.yaml
### Create a PVC 
Create a pvc for pipeinerun manually.
```bash
kubectl create -f src-pvc.yaml
```

### Task & Pipeline
```bash
kubectl apply -f demo-pipl.yaml
```

Create two tasks:
- clone-src
- read-src


![tasks](https://github.com/MollyH1391/TektonCI-CD-pipeline/blob/5ba6b38069199af71995e9e0763c89a4f1b20389/task-pipl-workspace-demo/GUI/tekton-tasks.png)

Organize these two tasks in a sequential manner within a Tekton pipeline and run it using a PipelineRun. And ensure that the PipelineRun's workspace is mounted to the PVC created previously.

![pipeline](https://github.com/MollyH1391/TektonCI-CD-pipeline/blob/5ba6b38069199af71995e9e0763c89a4f1b20389/task-pipl-workspace-demo/GUI/pipeline.png)

Result:

![pipeline-result](https://github.com/MollyH1391/TektonCI-CD-pipeline/blob/5ba6b38069199af71995e9e0763c89a4f1b20389/task-pipl-workspace-demo/GUI/pipeline-result.png)


## Utilize the Workspace Subpath
path: git-clone-workspace-subpath.yaml

Tasks:
- git clone
- list-files

### Clone a repository in the first task and list the files in one of its subdirectories
The repository contains a "src" folder. The goal is to list the files within it.
```bash
tree 
.
├── Dockerfile
├── README.md
├── img
├── kubernetes-manifests
├── package-lock.json
├── package.json
├── skaffold.yaml
└── src
    ├── assets
    ├── index.html.hbs
    └── index.js
```

Add <code>subPath</code> to the pipeline.

```yaml 
  - name: list-files
    runAfter: ["fetch-repo"]
    workspaces:
    - name: source
      workspace: shared-data
      subPath: src
    taskSpec:
      workspaces:
      - name: source
      steps:
      - image: zshusers/zsh:4.3.15
        script: | 
          #!/usr/bin/env zsh
          cd workspace/source
          ls
```

![subpath-workspace](https://github.com/MollyH1391/TektonCI-CD-pipeline/blob/cd9205c9242f2ea645faa11776f8a5045cf76c56/task-pipl-workspace-demo/GUI/subpath.png)