# Lab 11: Using and developing Helm templates

## Prerequisites

- Access to Openshift
- Openshift CLI
- [Helm 3 CLI](https://helm.sh/docs/intro/install/) - check with `helm version` that you have Helm 3 installed

## Supporting Information

- Install local Helm charts with `helm install my-chart-name ./my-chart-dir/`
- Upgrade local charts with `helm upgrade my-chart-name ./my-chart-dir/`
- Rollback charts with `helm rollback my-chart-name`
- Uninstall charts with `helm uninstall my-chart-name`
- [Helm Command Reference](https://www.ibm.com/cloud/architecture/content/course/helm-fundamentals/helm-commands)

## Challenges to be solved

This time you'll create your first Helm Chart to install a Postgress database onto your Openshift.

## Intro

Firstly open your terminal and navigate to a directory where you want to store your helm chart, let's create our template chart:

```
helm create <your-initial>-postgresql
```

This will create a folder with following structure:

- < your-initial >-postgresql/
- - charts/
- - templates/
- - - tests/
- - - \_helpers.tpl
- - - deployment.yaml
- - - hpa.yaml
- - - ingress.yaml
- - - NOTES.txt
- - - service.yaml
- - - serviceaccount.yaml
- - .helmignore
- - Chart.yaml
- - values.yaml

Look trough the files, maybe you recognize few resources like Deployment and ServiceAccount that we've seen previously.

If you run `helm list` in your terminal, it will show all installed helm charts. Since we didn't install anything yet - you should see an empty list:

| NAME | NAMESPACE | REVISION | UPDATED | STATUS | CHART | APP VERSION |
| ---- | --------- | -------- | ------- | ------ | ----- | ----------- |

<br/>

In your terminal navigate to a parent folder of the newly created helm chart directory `<your-initial>-postgresql`. We want to install this example helm chart onto project in Openshift. To install helm chart from the local directory you can use `helm install chart-name location-of-the-chart/`. Run `helm install <your-initial>-postgresql ./<your-initial>-postgresql/`. You should see `STATUS: deployed` in your terminal nce deployed successfully.

```
NAME: <your-initial>-postgresql
LAST DEPLOYED:
NAMESPACE: dev-<your-initials>
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
...
```

Run `helm list` again and you'll see that our helm chart is installed:

| NAME                        | NAMESPACE | REVISION | UPDATED    | STATUS   | CHART                             | APP VERSION |
| --------------------------- | --------- | -------- | ---------- | -------- | --------------------------------- | ----------- |
| < your-initial >-postgresql | dev-      | 1        | 2022-01-01 | deployed | < your-initial >-postgresql-0.1.0 | 1.16.0      |

Run `oc get all` to see all the things that have been created by just running `helm install`. Otherwise you can explore all the resources created by going to the Openshift UI.

<br/>

Since this is just an example and we'll modify it later let's just delete it from our Openshift project by running `helm uninstall < your-initial >-postgresql`. You should see:

```
release "<your-initial>-postgresql" uninstalled
```

<br/>

## Real stuff

The next step is to allocate some Storage for our Postgresql, let's create a PersistenVolumeClaim, create a file `pvc.yaml` under `templates/` folder with the following content:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: <your-initials>-postgresql-data-pvc
spec:
  storageClassName: ibmc-vpc-block-5iops-tier
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Now Postgresql requires some configuration, let's create a `ConfigMap` for that. Create a new file under `templates/` called `configmap.yaml`, copy the following content and change the name for the future database:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
data:
  POSTGRES_DB: <your-db-name>
```

We also have some parts of configuration that should be protected, so let's create `Secret` for those. Put the following content into the new file under `templates/` and call it 'secret.yaml`, put also some values for `POSTGRES_USER`and`POSTGRES_PASSWORD`. Hint: Don't forget base64

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
data:
  POSTGRES_USER: <user>
  POSTGRES_PASSWORD: <password>
```

This approach of hard coding values is okay if we wish to use this helm chart just for us, if you intend to reuse or share it we need an approach to inject values when installing. We have a file `values.yaml` in the root of `<your-initial>-postgresql`. Let's add `POSTGRES_DB`, `POSTGRES_USER` and `POSTGRES_PASSWORD` with values to `values.yaml` but this time you don't have to base64 encode anything, append to `values.yaml`:

```yaml
config:
  POSTGRES_DB: db123
  POSTGRES_USER: user
  POSTGRES_PASSWORD: user
```

To use those values we have to update `secret.yaml` with the following:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
data:
  POSTGRES_USER:
    {{ default "admin" .Values.config.POSTGRES_USER | b64enc | quote }}
  POSTGRES_PASSWORD:
    {{ default "admin" .Values.config.POSTGRES_PASSWORD | b64enc | quote }}
```

and `configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
data:
  POSTGRES_DB: {{ .Values.config.POSTGRES_DB }}
```

We have all pices but deployment for our Postgresql is still missing, let's create it. Replace the contents of the `deployment.yaml` under `templates/` with following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgresql
spec:
  selector:
    matchLabels:
      app: postgresql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: postgresql
    spec:
      containers:
        - name: postgresql
          image: bitnami/postgresql
          ports:
            - containerPort: 5432
          env:
            - name: POSTGRES_DB
              valueFrom:
                configMapKeyRef:
                  name: db-config
                  key: POSTGRES_DB
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: POSTGRES_USER
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: POSTGRES_PASSWORD
          volumeMounts:
            - name: sql-storage
              mountPath: /var/lib/postgresql/data
      volumes:
        - name: sql-storage
          persistentVolumeClaim:
            claimName: <your-initials>-postgresql-data-pvc
```

We have now created all the pieces required for our Postgresql helm chart, let's install it similarly as we've done it in the beginning of this challege.

## Rollbacks

Let's change something in our `deployment.yaml`! Change name of the deployment to `postgresql-deployment`. Let's also bump the version of our chart, in `Chart.yaml` you'll find field called `version`, update value to be `0.2.0`.

Run `helm list` to see what chart & version we have currently installed


| NAME                        | NAMESPACE | REVISION | UPDATED    | STATUS   | CHART                             | APP VERSION |
| --------------------------- | --------- | -------- | ---------- | -------- | --------------------------------- | ----------- |
| < your-initial >-postgresql | dev-      | 2        | 2022-01-01 | deployed | < your-initial >-postgresql-0.1.0 | 1.16.0      |

Now let's make an upgrade of our installed helm chart with local changes we've just made. Run `helm install <your-initial>-postgresql ./<your-initial>-postgresql/`. You should see

```
Release "vd-postgresql" has been upgraded. Happy Helming!
...
```

Once upgraded verify in the Openshift UI that those changes are applied. Now if you run `helm list`, you should see that your chart version is bumped to 0.2.0:


| NAME                        | NAMESPACE | REVISION | UPDATED    | STATUS   | CHART                             | APP VERSION |
| --------------------------- | --------- | -------- | ---------- | -------- | --------------------------------- | ----------- |
| < your-initial >-postgresql | dev-      | 3        | 2022-01-01 | deployed | < your-initial >-postgresql-0.2.0 | 1.16.0      |

Now let's rollback to the previous version with the following command:

```
helm rollback <your-initial>-postgresql
```

As a result you should see the following in your terminal:

```
Rollback was a success! Happy Helming!
```

If you run `helm list` again, you'll see that the version of the chart was downgraded:


| NAME                        | NAMESPACE | REVISION | UPDATED    | STATUS   | CHART                             | APP VERSION |
| --------------------------- | --------- | -------- | ---------- | -------- | --------------------------------- | ----------- |
| < your-initial >-postgresql | dev-      | 4        | 2022-01-01 | deployed | < your-initial >-postgresql-0.1.0 | 1.16.0      |

## Verification

Navigate to the Openshift UI and make sure that all the resources are created and the Postgresql Pod has state "Running". Go to "Logs" inside the postgresql Pod and validate that database system is ready to accept connections. Once validated, delete your helm chart from your Openshift Project.
