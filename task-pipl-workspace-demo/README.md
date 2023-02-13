# TektonCI-CD-pipeline

## A Simple Workspace Transition in a Tekton Pipeline

### Create a PVC 
Create a pvc for pipeinerun manually.
```bash
kubectl create -f src-pvc.yaml
```

### Task & Pipeline
Create two tasks:
- clone-src
- read-src

Organize these two tasks in a sequential manner within a Tekton pipeline and run it using a PipelineRun.

Ensure that the PipelineRun's workspace is mounted to the PVC created previously.

```bash
kubectl apply -f demo-pipl.yaml
```
![tasks](https://github.com/MollyH1391/TektonCI-CD-pipeline/blob/5ba6b38069199af71995e9e0763c89a4f1b20389/task-pipl-workspace-demo/GUI/tekton-tasks.png)
![pipeline](https://github.com/MollyH1391/TektonCI-CD-pipeline/blob/5ba6b38069199af71995e9e0763c89a4f1b20389/task-pipl-workspace-demo/GUI/pipeline.png)
![pipeline-result](https://github.com/MollyH1391/TektonCI-CD-pipeline/blob/5ba6b38069199af71995e9e0763c89a4f1b20389/task-pipl-workspace-demo/GUI/pipeline-result.png)


