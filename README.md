# daskernetes-gpu

An notebook example showcasing RAPIDS at scale with Dask on Kubernetes. 


## Software Stack
*  Kubernetes
*  Kubeflow
*  Dask cuDF
*  Dask XGBoost

## Getting Started
The following notebook was run on a baremetal kubernetes cluster with 3 nodes and 4xT4 GPUs per node.

The Notebook uses runs on a RAPIDS NGC container (nvcr.io/nvidia/rapidsai/rapidsai:0.7-cuda10.0-runtime-ubuntu16.04-gcc5-py3.7) within the cluster. If kubeflow was used to run 
the container then there is no need to provide an authorized service account for the container. Otherwise, client should make sure the container started by Kubernetes has an
authorized service account.

Specify `/bin/bash -c source activate rapids && jupyter notebook  --no-browser --allow-root --port=8888 --NotebookApp.token='' --NotebookApp.password='' --NotebookApp.allow_origin='*'` 
as the start up command in the yaml file for the pod in your deployment.

After the jupyter notebook server starts access it using the pod ip (If Kubeflow is being used the notebook server provides you the direct access to the Jupyter Notebook).
Run `pip install dask-kubernetes` on top of the running container.

If a NFS Server is being used to access the dataset then a Persistent Volume(PV) and a Persistent Volume Claim(PVC) has to be setup on the cluster. Below is an example for the same. The PVC needs to be mounted on the container running the jupyter notebook.

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pvname
spec:
  capacity:
    storage: 500Gi
  accessModes:
    - ReadWriteMany
  mountOptions:
    - nfsvers=3
  nfs:
    server: nfsserverip
    path: "/path/to/dataset/in/the/nfsserver"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvcname
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 500Gi
```

The repository has a config.yaml file where the name of PVC(`nfs` in my case) created above needs to be specified. Also the mount path of the PVC on the dask worker(`"/home/jovyan/nyc-taxi-data"`) needs to be the same as mount path on the jupyter container.




