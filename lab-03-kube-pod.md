# Lab 3: Kubernetes Pod Creation

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

- https://cloudnative101.dev/lectures/kube-core-concepts/ (how to create namespaces and pods with kubectl)
- https://cloudnative101.dev/lectures/kube-configuration/ (how to refine the config of your pods, e.g. commands and arguments)

Hint: make sure to select **more** on the Kubernetes YAML examples in above supporting information.

## Challenge to be solved

Create your own OpenShift project.
Write a pod definition named yoda-service-pod.yml. Then create a pod in the cluster using this definition to make sure it works.

The specifications of this pod are as follows:

- Use the bitnami/nginx container image.
- The container needs a containerPort of 80.
- Set the command to run `["nginx"]`
- Pass in the `["-g", "daemon off;", "-q"]` args to run nginx in quiet mode.
- Create the pod in the "dev-**yourinitials**" namespace.

### Verification

When you have completed this lab, use the following commands to validate your solution.

```bash
$ oc get pods -n dev-yourinitials
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          8s

$ oc describe pod nginx -n dev-yourinitials
[...]
Events:
  Type    Reason          Age   From               Message
  ----    ------          ----  ----               -------
  Normal  Scheduled       35s   default-scheduler  Successfully assigned dev-gw/nginx to 10.134.237.201
  Normal  AddedInterface  32s   multus             Add eth0 [172.30.18.150/32]
  Normal  Pulling         32s   kubelet            Pulling image "bitnami/nginx"
  Normal  Pulled          23s   kubelet            Successfully pulled image "bitnami/nginx" in 8.925497856s
  Normal  Created         23s   kubelet            Created container nginx
  Normal  Started         22s   kubelet            Started container nginx
```
