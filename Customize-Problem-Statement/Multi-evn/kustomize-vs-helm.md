# Kustomize vs Helm --- Beginner-Friendly Guide

## 1. What problem do they solve?

Kubernetes applications are usually defined using YAML files. For
example, you may have YAML files for:

-   Deployment --- runs your application Pods
-   Service --- provides network access to the Pods
-   ConfigMap --- stores non-secret configuration
-   Ingress --- exposes an application outside the cluster

When you deploy the same application to **Development**, **QA**, and
**Production**, some settings change. For example:

-   Number of replicas
-   Container image tag
-   CPU and memory limits
-   Environment-specific configuration
-   Hostname or namespace

**Kustomize** and **Helm** help you manage these differences without
manually maintaining a completely separate set of YAML files for every
environment.

------------------------------------------------------------------------

## 2. What is Kustomize?

**Kustomize** customizes existing Kubernetes YAML files using a
`kustomization.yaml` file. It is built into modern versions of
`kubectl`, so you can use it without installing a separate command-line
tool.

Think of it as a way to take common YAML files and apply
environment-specific changes.

### Example folder structure

``` text
my-app/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    ├── qa/
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml
```

-   `base/` contains the common Kubernetes resources.
-   `overlays/dev/` contains changes for Development.
-   `overlays/qa/` contains changes for QA.
-   `overlays/prod/` contains changes for Production.

### Base Deployment example

File: `base/deployment.yaml`

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
        - name: web-app
          image: nginx:1.27
          ports:
            - containerPort: 80
```

File: `base/service.yaml`

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app
spec:
  selector:
    app: web-app
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

File: `base/kustomization.yaml`

``` yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
```

### Development overlay example

File: `overlays/dev/kustomization.yaml`

``` yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

namespace: dev

replicas:
  - name: web-app
    count: 1

images:
  - name: nginx
    newTag: "1.27"
```

This overlay reuses the base resources, sets the namespace to `dev`, and
changes the Deployment to one replica.

### How to use Kustomize

Preview the generated YAML:

``` bash
kubectl kustomize overlays/dev
```

Apply it to the cluster:

``` bash
kubectl apply -k overlays/dev
```

Check the resources:

``` bash
kubectl get deployments,services -n dev
```

**Important:** `kubectl kustomize` renders YAML; it does not deploy it.
`kubectl apply -k` renders and applies the resources.

------------------------------------------------------------------------

## 3. What is Helm?

**Helm** is a Kubernetes package manager. It groups application
resources into a package called a **chart**.

A chart can contain templates, default values, metadata, and other
files. Helm fills in the templates using values you provide.

Think of Helm as an application template or package that can be
configured for different environments.

### Example chart structure

``` text
web-app-chart/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    └── service.yaml
```

-   `Chart.yaml` describes the chart.
-   `values.yaml` contains default configuration.
-   `templates/` contains Kubernetes YAML templates.

### Default values example

File: `values.yaml`

``` yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.27"

service:
  type: ClusterIP
  port: 80
```

### Deployment template example

File: `templates/deployment.yaml`

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-web-app
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}-web-app
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}-web-app
    spec:
      containers:
        - name: web-app
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
```

The expressions such as `{{ .Values.replicaCount }}` are Helm template
syntax. Helm replaces them with values when it renders the chart.

### Production values example

File: `values-prod.yaml`

``` yaml
replicaCount: 4

image:
  repository: nginx
  tag: "1.27"

service:
  type: ClusterIP
  port: 80
```

### How to use Helm

Render the chart to inspect the YAML:

``` bash
helm template web-app ./web-app-chart -f values-prod.yaml
```

Install or upgrade the application:

``` bash
helm upgrade --install web-app ./web-app-chart \
  --namespace prod \
  --create-namespace \
  -f values-prod.yaml
```

Check the release:

``` bash
helm list -n prod
kubectl get deployments,services -n prod
```

Helm tracks releases and supports actions such as upgrade, rollback, and
uninstall.

------------------------------------------------------------------------

## 4. Kustomize vs Helm

  -----------------------------------------------------------------------
  Feature                 Kustomize               Helm
  ----------------------- ----------------------- -----------------------
  Main purpose            Customize existing YAML Package and template
                                                  applications

  Configuration approach  Base resources plus     Templates plus values
                          patches/overlays        files

  Template syntax         No Helm-style template  Uses Go-template syntax
                          syntax required         

  Built into `kubectl`    Yes, through            No, install Helm
                          `kubectl -k`            separately

  Environment differences Overlays and patches    Different values files
                                                  or supplied values

  Packaging and sharing   Shares directories or   Packages applications
                          repositories of YAML    as charts

  Release tracking        Does not manage         Tracks named releases
                          chart-style releases    

  Rollback support        No built-in release     Supports release
                          history                 rollback

  Best fit                Straightforward YAML    Reusable, configurable
                          customization           application packages
  -----------------------------------------------------------------------

Neither tool is universally better. The right choice depends on how your
team structures and deploys applications.

------------------------------------------------------------------------

## 5. Same application in Dev, QA, and Production

### With Kustomize

``` text
base YAML
   |
   +--> overlays/dev  --> 1 replica, namespace dev
   |
   +--> overlays/qa   --> 2 replicas, namespace qa
   |
   +--> overlays/prod --> 4 replicas, namespace prod
```

You maintain common resources in `base/` and place environment-specific
changes in each overlay.

### With Helm

``` text
Helm chart
   |
   +--> values-dev.yaml  --> 1 replica
   |
   +--> values-qa.yaml   --> 2 replicas
   |
   +--> values-prod.yaml --> 4 replicas
```

You maintain templates in the chart and supply the values appropriate
for each environment.

------------------------------------------------------------------------

## 6. Which should a beginner learn first?

A practical learning order is:

1.  **Kubernetes YAML:** understand Pods, Deployments, Services,
    ConfigMaps, and namespaces.
2.  **Kustomize:** learn `kustomization.yaml`, bases, overlays, patches,
    image changes, and namespaces.
3.  **Helm:** learn charts, `Chart.yaml`, `values.yaml`, templates,
    `helm template`, install/upgrade, and rollback.
4.  **Practice both:** deploy the same small application to Dev and
    Production using each approach.

Kustomize may feel easier at first if you already understand Kubernetes
YAML because you mostly work with regular YAML. Helm adds a templating
layer, which takes extra practice.

------------------------------------------------------------------------

## 7. Common commands cheat sheet

### Kustomize

``` bash
# Render the customized resources
kubectl kustomize overlays/dev

# Apply the resources
kubectl apply -k overlays/dev

# Delete resources managed by that Kustomize configuration
kubectl delete -k overlays/dev
```

### Helm

``` bash
# Render templates locally without installing
helm template web-app ./web-app-chart

# Install or upgrade a release
helm upgrade --install web-app ./web-app-chart -n dev --create-namespace

# List releases
helm list -n dev

# View release history
helm history web-app -n dev

# Roll back to a previous revision
helm rollback web-app 1 -n dev

# Uninstall a release
helm uninstall web-app -n dev
```

Review the rendered manifests before applying them to an important
environment. Be especially careful with changes to persistent storage,
databases, and other stateful resources.

------------------------------------------------------------------------

## 8. Important beginner notes

-   Kustomize and Helm both produce Kubernetes manifests, but they
    organize configuration differently.
-   Kustomize does not require you to convert your YAML into templates.
-   Helm templates are useful when you want a reusable chart with
    configurable values.
-   Helm does not replace Kubernetes; it helps install and manage
    Kubernetes applications.
-   Kustomize is available through `kubectl`, but you can also install a
    standalone `kustomize` CLI.
-   You can use Helm and Kustomize in the same workflow, but start by
    learning each one separately so you understand what each tool
    changes.
-   Do not store passwords or other secrets directly in plain-text
    configuration files committed to Git.

## Final summary

**Kustomize = customize Kubernetes YAML using bases and overlays.**

**Helm = package Kubernetes applications using templates and
configurable values.**

For a beginner who is already learning Kubernetes YAML, start with
Kustomize. Then learn Helm to understand how teams package, configure,
version, and roll out reusable applications.
