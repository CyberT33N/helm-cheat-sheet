# Helm Cheat Sheet
Helm Cheat Sheet with the most needed stuff..






# Install
```shell
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```















<br><br>
<br><br>
_____________________________________________________
_____________________________________________________
<br><br>
<br><br>



## Alle Releases auflisten
```bash
helm list -n namespaceNameHere
```
  - Wenn man den Namespace hier nicht findet helfen auch NICHT die Helm Rollback und Uninstall Befehle um es zu fixen

<br><br>

## Rollback zu der letzten Revision:
```bash
helm --namespace namespaceNameHere rollback platform
```

## Uninstall Platform:
 - Error: UPGRADE FAILED: another operation (install/upgrade/rollback) is in progress

```bash
helm --namespace namespaceNameHere delete platformNameHere
```








































<br><br>
<br><br>
_________________________________________________
_________________________________________________
<br><br>
<br><br>


# Helm Template
- [https://helm.sh/docs/helm/helm_upgrade/](https://helm.sh/docs/helm/helm_template/)
- Render chart templates locally and display the output.

Any values that would normally be looked up or retrieved in-cluster will be faked locally. Additionally, none of the server-side testing of chart validity (e.g. whether an API is supported) is done.
```shell
helm template [NAME] [CHART] [flags]
```


































<br><br>
<br><br>
_________________________________________________
_________________________________________________
<br><br>
<br><br>


# Helm Upgrade
https://helm.sh/docs/helm/helm_upgrade/
- This command upgrades a release to a new version of a chart.

The upgrade arguments must be a release and chart. The chart argument can be either: a chart reference('example/mariadb'), a path to a chart directory, a packaged chart, or a fully qualified URL. For chart references, the latest version will be specified unless the '--version' flag is set.
```shell
helm upgrade --set foo=bar --set foo=newbar redis ./redis
```


















<br><br>
<br><br>
_________________________________________________
_________________________________________________
<br><br>
<br><br>

# Helm Install
- https://helm.sh/docs/helm/helm_install/
```shell
$ helm install -f myvalues.yaml myredis ./redis
or

$ helm install --set name=prod myredis ./redis
or

$ helm install --set-string long_int=1234567890 myredis ./redis
or

$ helm install --set-file my_script=dothings.sh myredis ./redis
or

$ helm install --set-json 'master.sidecars=[{"name":"sidecar","image":"myImage","imagePullPolicy":"Always","ports":[{"name":"portname","containerPort":1234}]}]' myredis ./redis

```






<br><br>
<br><br>


## --set
- https://helm.sh/docs/intro/using_helm/#the-format-and-limitations-of---set
```shell
--set outer.inner=value
```


<br><br>
<br><br>

## Install helm repo
```
helm repo add [repo-name] [repo-url]
# helm repo add stable https://charts.helm.sh/stable

helm repo update

helm install [release-name] [repo-name]/[chart-name]
# helm install my-nginx stable/nginx

# prüfe installation
helm list
```































<br><br>
<br><br>
_________________________________________________
_________________________________________________
<br><br>
<br><br>

# Helm Charts

<br><br>
<br><br>
<br><br>
<br><br>



## Hooks
- https://v2.helm.sh/docs/charts_hooks/
```
The following hooks are defined:

pre-install: Executes after templates are rendered, but before any resources are created in Kubernetes.
post-install: Executes after all resources are loaded into Kubernetes
pre-delete: Executes on a deletion request before any resources are deleted from Kubernetes.
post-delete: Executes on a deletion request after all of the release’s resources have been deleted.
pre-upgrade: Executes on an upgrade request after templates are rendered, but before any resources are loaded into Kubernetes (e.g. before a Kubernetes apply operation).
post-upgrade: Executes on an upgrade after all resources have been upgraded.
pre-rollback: Executes on a rollback request after templates are rendered, but before any resources have been rolled back.
post-rollback: Executes on a rollback request after all resources have been modified.
crd-install: Adds CRD resources before any other checks are run. This is used only on CRD definitions that are used by other manifests in the chart.
test-success: Executes when running helm test and expects the pod to return successfully (return code == 0).
test-failure: Executes when running helm test and expects the pod to fail (return code != 0).
```
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}"
  labels:
    app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
    app.kubernetes.io/instance: {{ .Release.Name | quote }}
    app.kubernetes.io/version: {{ .Chart.AppVersion }}
    helm.sh/chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
  annotations:
    # This is what defines this resource as a hook. Without this line, the
    # job is considered part of the release.
    "helm.sh/hook": post-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    metadata:
      name: "{{ .Release.Name }}"
      labels:
        app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
        app.kubernetes.io/instance: {{ .Release.Name | quote }}
        helm.sh/chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
    spec:
      restartPolicy: Never
      containers:
      - name: post-install-job
        image: "alpine:3.3"
        command: ["/bin/sleep","{{ default "10" .Values.sleepyTime }}"]
```

### Hooks weight
- Tiller sorts hooks by weight (assigning a weight of 0 by default) and by name for those hooks with the same weight in ascending order.
  - This means 0 will be executed first and 1 as next one. So you can create different files 
```
apiVersion: batch/v1
kind: Job
metadata:
  name: pre-deploy-hook-2
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "1"
spec:
  template:
    spec:
      containers:
        - name: pre-deploy-container-2
          image: busybox
          command: ['sh', '-c', 'echo "Running second pre-deploy hook"; sleep 10']
      restartPolicy: Never
```









<br><br>
<br><br>
<br><br>
<br><br>


## Instances
```
abc_push: 
  hpa: 
    # enabling hpa disables 'replicas' field.
    enabled: true
    minReplicas: 1
    maxReplicas: 1

  resources:
    requests:
      memory: "250Mi"
      cpu: 100m
    limits:
      memory: "500Mi"
      cpu: 200m
```


<br><br>
<br><br>

## template Function
```
env:
  - name: 'APP_VERSION'
    value: {{ .Chart.AppVersion | squote }}
  - name: 'IMAGE_VERSION'
    value: {{ .Values.test_backend.version | squote }}
```


<br><br>
<br><br>


### toYaml

<br><br>
<br><br>

#### Use dashes
- {{ toYaml Values.jobs.update-es.resources | nindent 12 }} = error
- You can not import properties with dashes. As workaround you can use:
```yaml
{{- tpl (toYaml (index .Values "backup" "mongodb-data" "env")) . | nindent 12 }}

# Without templating
# {{ toYaml (index .Values "jobs" "update-es" "resources") | nindent 12 }}
```
- index is able to read properties with dashes
- tpl will resolve something like `{{ .Values.cluster }}-mongodb-data-bck`



















<br><br>
<br><br>
_________________________________________________
_________________________________________________
<br><br>
<br><br>

# Helm Charts Scripts

## MongoDB
- You can see all available options inside of the values.yaml or by running `helm show values bitnami/mongodb`:
  - https://github.com/bitnami/charts/blob/main/bitnami/mongodb/values.yaml
    
  - For more details about the install process please check https://github.com/CyberT33N/helm-cheat-sheet/blob/main/README.md#helm-install 
```
# Add bitnami repo
helm repo add bitnami https://charts.bitnami.com/bitnami

# Update helm repo
helm repo update

# Auflisten der verfügbaren Helm Chart Versionen
helm search repo bitnami/mongodb --versions

# This will download the mongodb helm chart to the folder ./mongodb/Chart
cd ~/Projects/minikube
mkdir -p ./mongodb/Chart
helm pull bitnami/mongodb --version 15.6.12 --untar --untardir ./tmp
cp -r ./tmp/mongodb/* ./mongodb/Chart
rm -rf ./tmp

# Change context
kubectl config use-context minikube

# Create custom-values.yaml
touch

# Install
helm install mongodb-dev ./mongodb/Chart --namespace dev -f ./mongodb/custom-values.yaml
```

- custom-values.yaml
```yaml
architecture: standalone

auth:
  rootUser: root
  rootPassword: test

persistence:
  size: 10Gi
  storageClass: standard


## MongoDB(&reg;) pods' liveness probe. Evaluated as a template.
## ref: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes
## @param livenessProbe.enabled Enable livenessProbe
## @param livenessProbe.initialDelaySeconds Initial delay seconds for livenessProbe
## @param livenessProbe.periodSeconds Period seconds for livenessProbe
## @param livenessProbe.timeoutSeconds Timeout seconds for livenessProbe
## @param livenessProbe.failureThreshold Failure threshold for livenessProbe
## @param livenessProbe.successThreshold Success threshold for livenessProbe
##
livenessProbe:
  enabled: false
  initialDelaySeconds: 30
  periodSeconds: 1000
  timeoutSeconds: 10
  failureThreshold: 6
  successThreshold: 1
## MongoDB(&reg;) pods' readiness probe. Evaluated as a template.
## ref: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes
## @param readinessProbe.enabled Enable readinessProbe
## @param readinessProbe.initialDelaySeconds Initial delay seconds for readinessProbe
## @param readinessProbe.periodSeconds Period seconds for readinessProbe
## @param readinessProbe.timeoutSeconds Timeout seconds for readinessProbe
## @param readinessProbe.failureThreshold Failure threshold for readinessProbe
## @param readinessProbe.successThreshold Success threshold for readinessProbe
##
readinessProbe:
  enabled: false
  initialDelaySeconds: 5
  periodSeconds: 1000
  timeoutSeconds: 5
  failureThreshold: 6
  successThreshold: 1
## Slow starting containers can be protected through startup probes
## Startup probes are available in Kubernetes version 1.16 and above
## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-startup-probes
## @param startupProbe.enabled Enable startupProbe
## @param startupProbe.initialDelaySeconds Initial delay seconds for startupProbe
## @param startupProbe.periodSeconds Period seconds for startupProbe
## @param startupProbe.timeoutSeconds Timeout seconds for startupProbe
## @param startupProbe.failureThreshold Failure threshold for startupProbe
## @param startupProbe.successThreshold Success threshold for startupProbe
##
startupProbe:
  enabled: false
  initialDelaySeconds: 5
  periodSeconds: 20
  timeoutSeconds: 10
  successThreshold: 1
  failureThreshold: 30

# Minimum 900 cpu needed that timeout will not trigger
# 1250 cpu at limit is maybe too low because sometimes mongodb service will crash
resources:
  limits:
    memory: "2048Mi"
    cpu: "1500m"
  requests:
    memory: "1024Mi"
    cpu: "900m"

# service:
#   type: "NodePort"
#   nodePort: 30018

arbiter:
  resources:
    limits:
      memory: "2048Mi"
      cpu: "300m"
    requests:
      memory: "1024Mi"
      cpu: "100m"

```
