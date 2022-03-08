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
  resources:
    requests:
      storage: <add capacity>
```

## Verification

Verify that your Persistent Volume Claim has moved from "Pending" into status "Bound". Examine the details of the PVC, as well as the generated PV.

```bash
$ oc get pvc
NAME                   STATUS
block-storage-gw-pvc   Bound


$ oc describe pvc/block-storage-gw-pvc
Name:          block-storage-gw-pvc
Namespace:     dev-gw
StorageClass:  ibmc-vpc-block-5iops-tier
Status:        Bound
Volume:        pvc-06bf7ced-c876-4e94-bf67-51a5f5acab26
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: vpc.block.csi.ibm.io
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      10Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Used By:       <none>
Events:
  Type    Reason                 Age                From                                                                                      Message
  ----    ------                 ----               ----                                                                                      -------
  Normal  Provisioning           53s                vpc.block.csi.ibm.io_ibm-vpc-block-csi-controller-0_ec28cb25-8072-4688-84fc-908ff4b02189  External provisioner is provisioning volume for claim "dev-gw/block-storage-gw-pvc"
  Normal  ExternalProvisioning   33s (x4 over 53s)  persistentvolume-controller                                                               waiting for a volume to be created, either by external provisioner "vpc.block.csi.ibm.io" or manually created by system administrator
  Normal  ProvisioningSucceeded  26s                vpc.block.csi.ibm.io_ibm-vpc-block-csi-controller-0_ec28cb25-8072-4688-84fc-908ff4b02189  Successfully provisioned volume pvc-06bf7ced-c876-4e94-bf67-51a5f5acab26
```