# Lab 10 (Part 1): Deploy the BookInfo app in to the mesh

The BookInfo app is a sample app for users to leave comments. It consists of a web front end, Redis master for storage, and a replicated set of Redis slaves.

![](images/lab-10-images/withistio.svg)

### Enable the automatic sidecar injection for the bookinfo namespace

In Kubernetes, a sidecar is a utility container in the pod, and its purpose is to support the main container. For Istio to work, Envoy proxies must be deployed as sidecars to each pod of the deployment. There are two ways of injecting the Istio sidecar into a pod: manually using the istioctl CLI tool or automatically using the Istio sidecar injector. In this exercise, we will use the automatic sidecar injection provided by Istio.

1.  Create a project called "bookinfo" and postfix it with your initials

    ```shell
    oc new-project bookinfo-<your initials>
    oc new-project bookinfo-jd
    ```

2.  Make sure your bookinfo-<your initials> project is enrolled in the ServiceMeshMemberRoll resource. If you are not ensuring this, you won't be able to access the bookinfo app due to a missing NetworkPolicy that is created for members. Use the attribute `spec` - `members` to add your project.
    
    ![](images/lab-10-images/servicemesh_memberrole.png)

3.  Annotate the bookinfo namespace to enable automatic sidecar injection:

    ```shell
    oc label namespace bookinfo-<your initials> istio-injection=enabled
    oc label namespace bookinfo-jd istio-injection=enabled
    ```

4.  Validate the namespace is annotated for automatic sidecar injection:

    ```shell
    oc get namespace -L istio-injection
    ```

    Sample output:

    ```shell
    NAME                STATUS   AGE    ISTIO-INJECTION
    bookinfo-jd         Active   271d   enabled
    istio-system        Active   5d2h
    ...
    ```

## Clone application source repo

```shell
git clone https://github.com/ibm-garage-dach/cloud-native-bootcamp-ocp-full
cd bookinfo
```

## Install the BookInfo app

1. Inject the Istio Envoy sidecar into the bookinfo pods, and deploy the BookInfo app on to the Kubernetes cluster. Deploy both the v1 and v2 versions of the app:

   ```shell
   oc apply -f bookinfo.yaml
   ```

These commands deploy the BookInfo app on to the cluster. Since we enabled automation sidecar injection, these pods will be also include an Envoy sidecar as they are started in the cluster. Here we have two versions of deployments, a new version (`v2`) in the current directory, and a previous version (`v1`) in a sibling directory. They will be used in future sections to showcase the Istio traffic routing capabilities.

1. Verify that the pods are up and running.

   ```shell
   oc get pods
   ```

   Sample output:

   ```shell
   NAME                              READY     STATUS    RESTARTS   AGE
   details-v1-789c5f58f4-9twtw       2/2       Running   0          4m12s
   productpage-v1-856c8cc5d8-xcx2q   2/2       Running   0          4m11s
   ratings-v1-5786768978-tr8z9       2/2       Running   0          4m12s
   reviews-v1-5874566865-mxfgm       2/2       Running   0          4m12s
   reviews-v2-86865fc7d9-mf6t4       2/2       Running   0          4m12s
   reviews-v3-8d4cbbbbf-rfjcz        2/2       Running   0          4m12s
   ```

   Note that each bookinfo pods has 2 containers in it. One is the bookinfo container, and the other is the Envoy proxy sidecar.

Your bookinfo app is running, but you can't access it! In the next exercise you will expose the productpage service to allow incoming traffic.
