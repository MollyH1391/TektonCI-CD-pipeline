# TektonCI-CD-pipeline

## A Simple Workspace Transition in a Tekton Pipeline

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


