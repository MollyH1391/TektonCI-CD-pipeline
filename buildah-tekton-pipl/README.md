## Install tekton "git-clone" task
```bash
tkn hub install task git-clone

Task git-clone(0.9) installed in molly-tekton namespace
```

## Create pvc
```bash
kubectl create -f buildah-tekton-pipl/pvc/buildah-shared-pvc.yaml

persistentvolumeclaim/buildah-pvc created

kubectl get pvc
NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
buildah-pvc   Bound    pvc-a92585d1-015b-48b2-9780-bfe3a3201a16   1Gi        RWO            standard       12s
src-repo      Bound    pvc-72189a03-88e5-43da-85ef-db3d48f21ad1   1Gi        RWO            standard       5d2h
```

