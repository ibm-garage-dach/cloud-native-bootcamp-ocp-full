# OpenShift Security Context Constraints

## Overview

Similar to the way that RBAC resources control user access, administrators can use security context constraints (SCCs) to control permissions for pods. These permissions include actions that a pod can perform and what resources it can access. You can use SCCs to define a set of conditions that a pod must run with to be accepted into the system.

## Hands-On Intro to custom OpenShift Security Context Constraints

1. Create a custom OpenShift Security ContextConstraints resource by applying the following YAML file. Validate with `oc describe scc my-custom-scc` .
   ```
   apiVersion: v1
   kind: SecurityContextConstraints
   metadata:
     name: my-custom-scc
   # Privileges
   allowPrivilegedContainer: false
   # Access Control
   runAsUser:
     type: MustRunAsRange
     uidRangeMin: 1000
     uidRangeMax: 2000
   seLinuxContext:
     type: RunAsAny
   fsGroup:
     type: MustRunAs
     ranges:
     - min: 5000
       max: 6000
   supplementalGroups:
     type: MustRunAs
     ranges:
     - min: 5000
       max: 6000
   # Capabilities
   defaultAddCapabilities:
     - CHOWN
     - SYS_TIME
   requiredDropCapabilities:
     - MKNOD
   allowedCapabilites:
     - NET_ADMIN
   ```
2. Make this SCC available to a deployment by creating a custom service account and associate the new SCC. Do not modify the default service account.

   ```
   oc create sa my-custom-sa
   ```
   ```
   oc adm policy add-scc-to-user my-custom-scc -z my-custom-sa
   ```
3. Deploy this sample application that require specific access controls and capabilities. Validate with `oc get events | grep replicaset/my-test-app`.
   ```
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-test-app
   spec:
     selector:
       matchLabels:
         app: scc-article-sc-sa
     template:
       metadata:
         labels:
           app: scc-article-sc-sa
       spec:
         serviceAccountName: my-custom-sa
         securityContext:
           fsGroup: 5555
         containers:
         - image: ubi8/ubi-minimal
           name: ubi-minimal
           command: ['sh', '-c', 'echo "Hello from user $(id -u)" && sleep infinity']
           securityContext:
             runAsUser: 1234
             runAsGroup: 5678
             capabilities:
               add: ["SYS_TIME"]
           volumeMounts:
           - mountPath: /var/opt/app/data
             name: data
         volumes:
         - emptyDir: {}
           name: data
   ```
   
## Reference
See https://developer.ibm.com/learningpaths/secure-context-constraints-openshift/ for more detailed information about this scenario .