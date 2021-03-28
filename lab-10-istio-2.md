# Lab 10 (Part 2): Expose the app with the Istio Ingress Gateway and Route

The components deployed on the service mesh by default are not exposed outside the cluster. External access to individual services so far has been provided by creating an external load balancer or node port on each service.

An Ingress Gateway resource can be created to allow external requests through the Istio Ingress Gateway to the backing services.

![](images/lab-10-images/withistio.svg)

### Expose the BookInfo app with Ingress Gateway

1.  Take a look at the `bookinfo-gateway.yaml` file and correct `<your-initials>` in the VirtualService section. This will ensure that your applications will receive their own custom URI prefix

2.  Configure the bookinfo default route with the Istio Ingress Gateway.

```shell
oc create -f bookinfo-gateway.yaml
```

3. Get the **ROUTE** of the Istio Ingress Gateway.

```shell
oc get routes -n istio-system istio-ingressgateway
```

Output:

```shell
NAME                   HOST
istio-ingressgateway   istio-ingressgateway-istio-system.rvennamocpcluster-c8427b5c054eb1823b50328ad3aeeb58-0000.us-south.containers.appdomain.cloud
```

4. Make note of the HOST address that you retrieved in the previous step, as it will be used to access the BookInfo app in later parts of the course. Create an environment variable called $INGRESS_HOST with your address.

Example:

```
export INGRESS_HOST=istio-ingressgateway-istio-system.rvennamocpcluster-c8427b5c054eb1823b50328ad3aeeb58-0000.us-south.containers.appdomain.cloud
```

_or for windows:_

```
set INGRESS_HOST=istio-ingressgateway-istio-system.rvennamocpcluster-c8427b5c054eb1823b50328ad3aeeb58-0000.us-south.containers.appdomain.cloud
```

Congratulations! You extended the base Ingress features by providing a DNS entry to the Istio service.

Visit the application by going to `http://<INGRESS_HOST>/<your-initials>/productpage` in a new tab. If you keep hitting Refresh, you should see different versions of the page in random order (v1, v2, v3).

![](images/lab-10-images/bookinfo.png)
