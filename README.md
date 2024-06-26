# Helm Cheat Sheet
Helm Cheat Sheet with the most needed stuff..


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




<br><br>
<br><br>

## Hooks
- https://helm.sh/docs/topics/charts_hooks/
```
pre-install	Executes after templates are rendered, but before any resources are created in Kubernetes
post-install	Executes after all resources are loaded into Kubernetes

pre-delete	Executes on a deletion request before any resources are deleted from Kubernetes
post-delete	Executes on a deletion request after all of the release's resources have been deleted

pre-upgrade	Executes on an upgrade request after templates are rendered, but before any resources are updated
post-upgrade	Executes on an upgrade request after all resources have been upgraded

pre-rollback	Executes on a rollback request after templates are rendered, but before any resources are rolled back
post-rollback	Executes on a rollback request after all resources have been modified

test	Executes when the Helm test subcommand is invoked ( view test docs)
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


