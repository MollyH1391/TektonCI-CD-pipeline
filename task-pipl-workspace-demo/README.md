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

