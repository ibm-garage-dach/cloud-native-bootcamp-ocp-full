# Lab 8: Dynamically provision file storage on IBM Cloud

## Prerequisites

You can retrieve metadata from your OpenShift cluster.

```bash
$ oc get nodes
NAME             STATUS   ROLES           AGE   VERSION
10.134.237.197   Ready    master,worker   52d   v1.19.0+e49167a
10.134.237.201   Ready    master,worker   52d   v1.19.0+e49167a
10.134.237.236   Ready    master,worker   52d   v1.19.0+e49167a
```

Make sure everytime you create resources that you

- target the right OpenShift cluster
- target the right OpenShift Project / Kubernetes namespace

## Supporting Information

https://cloudnative101.dev/lectures/kube-state-persistence/

https://cloud.ibm.com/docs/containers?topic=containers-file_storage#file_storageclass_reference

https://cloud.ibm.com/docs/containers?topic=containers-file_storage

## Challenges to be solved

You need to provide persistent block storage via dynamic provisioning.
This persistent storage needs to conform to the following requirements:

- 10GB block storage with 5 IOPS per GB
- Storage access is restricted to one pod only (read + write)
- Data is NOT retained after PV deletion

Take this YAML template as starting point.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: block-storage-<yourinitials>-pvc
spec:
  storageClassName: <add storage class>
  accessModes:
    - <add access mode>
  volumeMode: <add volume mode>
  resources:
    requests:
      storage: <add capacity>
```

## Verification

Verify that your Persistent Volume Claim has moved from "Pending" into status "Bound". Examine the details of the PVC.

```bash
$ oc get pvc
NAME                   STATUS
block-storage-gw-pvc   Bound

$ oc describe pvc/block-storage-gw-pvc

```