# Lab 7: Services (based on VPC on IBM Cloud)

## Prerequisites

**Remove the existing jedi deployment in your namespace before starting this lab.**

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

https://cloudnative101.dev/lectures/kube-services-networking/

Hint: make sure you select **Show more** on the code snippets on above site to see all content.

## Challenges to be solved

We have a jedi-deployment and yoda-deployment that need to communicate with others. The jedi needs to communicate with the world (outside the cluster), while yoda only needs to talk to jedi council (others in the cluster).

Examine the two deployments, and create two services that meet the following criteria.

**jedi-svc**

- The service name is jedi-svc.
- The service exposes the pod replicas managed by the deployment named jedi-deployment.
- The service listens on port 80 and its targetPort matches the port exposed by the pods.
- The service type is ClusterIP.

**jedi-ingress**

- The ingress name is jedi-http-ingress.
- We will start with exposing the jedi-svc via HTTP.
- Ingress path is "/", Ingress type: "Prefix" and the Ingress host name is: "<dev-yourinitials>.<ocp-cluster-ingress-subdomain>"


**yoda-svc**

- The service name is yoda-svc.
- The service exposes the pod replicas managed by the deployment named yoda-deployment.
- The service listens on port 80 and its targetPort matches the port exposed by the pods.
- The service type is ClusterIP.

Create those two deployments as basis for your service definitions:

**07-jedi-deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jedi-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: jedi
  template:
    metadata:
      labels:
        app: jedi
    spec:
      containers:
        - image: bitnami/nginx:1.16.1
          name: c
          ports:
            - containerPort: 8080
```

**07-yoda-deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: yoda-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: yoda
  template:
    metadata:
      labels:
        app: yoda
    spec:
      containers:
        - image: bitnami/nginx:1.16.1
          name: c
          ports:
            - containerPort: 8080
```

## Verification

Validate that both Kubernetes Services. Both services are within a private IP address range that cannot be reached from the public internet.

```bash
$ oc get svc
NAME       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
jedi-svc   ClusterIP   172.21.96.12     <none>        80/TCP         57s
yoda-svc   ClusterIP   172.21.235.139   <none>        80/TCP         48s
```

Also validate that the Ingress has been created. This endpoint is exposed to the public internet.

```bash
$ oc get ingress
NAME                CLASS     HOSTS                                                     PORTS
jedi-http-ingress   <none>    dev-gw.bootcamp-cluster.eu-de.containers.appdomain.cloud  80
```


### Validating internet access through jedi-ingress and jedi-svc into jedi pod.

Accessing "<dev-yourinitials>.<ocp-cluster-ingress-subdomain>" should return an nginx welcome screen.


### Validating pod-to-service access from jedi pod to yoda-svc

To validate the communication from jedi pod to yoda-svc we will use a bash shell within a jedi pod to try to reach the
statically namespaced DNS name of the yoda-svc. Remember **servicename.namespace.svc.cluster.local** can be leveraged.

```bash
$ oc get pods
NAME                               READY   STATUS    RESTARTS   AGE
jedi-deployment-6fbb7bf48c-gbdnz   1/1     Running   0          28m
jedi-deployment-6fbb7bf48c-tr8wk   1/1     Running   0          28m
yoda-deployment-79bb79465d-6htpn   1/1     Running   0          27m
yoda-deployment-79bb79465d-fp92x   1/1     Running   0          27m

$ oc exec -t -i jedi-deployment-6fbb7bf48c-gbdnz bash
I have no name!@jedi-deployment-6fbb7bf48c-gbdnz:/app$

jedi-deployment-6fbb7bf48c-gbdnz:/app$ curl yoda-svc.dev-gw.svc.cluster.local
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```

### Clean-Up

Delete the ingress after you have completed your tests.

### Optional Activity / Homework - Secure the ingress route.

Follow these steps for exposing a public endpoint --> https://cloud.ibm.com/docs/openshift?topic=openshift-ingress-roks4#ingress-roks4-public