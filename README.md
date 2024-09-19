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
_________________________________________________
_________________________________________________
<br><br>
<br><br>


# Uninstall
- Uninstall releases
- https://helm.sh/docs/helm/helm_uninstall/
```bash
helm --namespace namespaceNameHere delete platformNameHere
```
- If you cancel helm install and you can not re-install your deploymonet you maybe get `Error: UPGRADE FAILED: another operation (install/upgrade/rollback) is in progress`. In this case you could completly uninstall your release and then install it again






























<br><br>
<br><br>
_________________________________________________
_________________________________________________
<br><br>
<br><br>


# List
- List releases
- [https://helm.sh/docs/helm/helm_rollback/](https://helm.sh/docs/helm/helm_list/)

<br><br>

## Alle Releases auflisten
```bash
helm list -n namespaceNameHere
```









<br><br>
<br><br>
_________________________________________________
_________________________________________________
<br><br>
<br><br>


# Rollback
- https://helm.sh/docs/helm/helm_rollback/

<br><br>

## Rollback zu der letzten Revision:
```bash
helm --namespace namespaceNameHere rollback platform
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

## Sub Charts
1. Lege das Subchart in das charts-Verzeichnis: Wenn dein Subchart lokal ist, musst du es in den Ordner charts deines übergeordneten Helm-Charts kopieren. Beispielstruktur:
```
my-parent-chart/
├── charts/
│   └── my-subchart/
├── Chart.yaml
├── values.yaml
├── templates/
└── ...
```

2. Keine Chart.yaml-Abhängigkeit notwendig: Du musst das Subchart nicht als Abhängigkeit in der Chart.yaml deines übergeordneten Charts hinzufügen, da es lokal vorliegt und Helm es automatisch erkennt, wenn es im charts-Verzeichnis liegt.

3. Konfiguration des Subcharts in values.yaml: In der values.yaml deines übergeordneten Charts kannst du die Werte für dein Subchart konfigurieren. Helm wird das Subchart unter dem Subchart-Namen erkennen. Beispiel:
```yaml
my-subchart:
  someKey: someValue
```

4. Chart bereitstellen: Du kannst dein übergeordnetes Chart direkt installieren, und Helm wird das lokale Subchart automatisch verwenden:
```shell
helm install my-release ./my-parent-chart
```












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

# Helm Charts Third Party



## MongoDB
<details><summary>Click to expand..</summary>
- You can see all available options inside of the values.yaml or by running `helm show values bitnami/mongodb`:
  - https://github.com/bitnami/charts/blob/main/bitnami/mongodb/values.yaml
    
  - For more details about the install process please check https://github.com/CyberT33N/helm-cheat-sheet/blob/main/README.md#helm-install 

- custom-values.yaml
```yaml
architecture: standalone

auth:
  rootUser: root
  rootPassword: test

persistence:
  size: 10Gi
  storageClass: standard

service:
  type: NodePort
  nodePorts:
    mongodb: 30644

## MongoDB(&reg;) pods' liveness probe. Evaluated as a template.
## ref: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes
## @param livenessProbe.enabled Enable livenessProbe
## @param livenessProbe.initialDelaySeconds Initial delay seconds for livenessProbe
## @param livenessProbe.periodSeconds Period seconds for livenessProbe
## @param livenessProbe.timeoutSeconds Timeout seconds for livenessProbe
## @param livenessProbe.failureThreshold Failure threshold for livenessProbe
## @param livenessProbe.successThreshold Success threshold for livenessProbe
##
# livenessProbe:
#   enabled: false
#   initialDelaySeconds: 30
#   periodSeconds: 1000
#   timeoutSeconds: 10
#   failureThreshold: 6
#   successThreshold: 1
## MongoDB(&reg;) pods' readiness probe. Evaluated as a template.
## ref: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes
## @param readinessProbe.enabled Enable readinessProbe
## @param readinessProbe.initialDelaySeconds Initial delay seconds for readinessProbe
## @param readinessProbe.periodSeconds Period seconds for readinessProbe
## @param readinessProbe.timeoutSeconds Timeout seconds for readinessProbe
## @param readinessProbe.failureThreshold Failure threshold for readinessProbe
## @param readinessProbe.successThreshold Success threshold for readinessProbe
##
# readinessProbe:
#   enabled: false
#   initialDelaySeconds: 5
#   periodSeconds: 1000
#   timeoutSeconds: 5
#   failureThreshold: 6
#   su
# service:
#   type: "NodePort"
#   nodePort: 30018ccessThreshold: 1
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
# startupProbe:
#   enabled: false
#   initialDelaySeconds: 5
#   periodSeconds: 20
#   timeoutSeconds: 10
#   successThreshold: 1
#   failureThreshold: 30

# Minimum 900 cpu needed that timeout will not trigger
# 1250 cpu at limit is maybe too low because sometimes mongodb service will crash
# resources:
#   limits:
#     memory: "2048Mi"
#     cpu: "1500m"
#   requests:
#     memory: "1024Mi"
#     cpu: "900m"

# arbiter:
#   resources:
#     limits:
#       memory: "2048Mi"
#       cpu: "300m"
#     requests:
#       memory: "1024Mi"
#       cpu: "100m"
```

<br><br>
<br><br>

### Add repo
```shell
# Add bitnami repo
helm repo add bitnami https://charts.bitnami.com/bitnami

# Update helm repo
helm repo update

# Auflisten der verfügbaren Helm Chart Versionen
helm search repo bitnami/mongodb --versions
```

<br><br>
<br><br>

### Install Helm Chart
```shell
# This will download the mongodb helm chart to the folder ./mongodb/Chart
cd ~/Projects/minikube
mkdir -p ./mongodb/Chart

# 15.6.12 = MongoDB 7
helm pull bitnami/mongodb --version 15.6.12 --untar --untardir ./tmp
cp -r ./tmp/mongodb/* ./mongodb/Chart
rm -rf ./tmp

# Create custom-values.yaml
touch ./mongodb/custom-values.yaml

# Change context
kubectl config use-context minikube

# Install
helm install mongodb-dev ./mongodb/Chart --namespace dev -f ./mongodb/custom-values.yaml
```

<br><br>
<br><br>

### Upgrade Helm Chart
```shell
kubectl config use-context minikube
helm upgrade mongodb-dev ./mongodb/Chart --namespace dev -f ./mongodb/custom-values.yaml --atomic
```

### Delete Deployment
```shell
kubectl config use-context minikube
helm --namespace dev delete mongodb-dev
```
</details>












































<br><br>
<br><br>



## Gitlab
<details><summary>Click to expand..</summary>

  - The Gitlab helm Chart will use if configured with the minikube example from official docs their own self-signed certificates and we do not have to have to worry about and this is what we will do. If needed you can create own certs and include them but it is not recommended because it will be a lot of work. You basicly only have to include the genereated secret to the gitlab-runner with:
```yaml
gitlab-runner:
  install: true
  certsSecretName: gitlab-dev-wildcard-tls-chain
```

<br><br>
<br><br>

### Guides
- https://docs.gitlab.com/charts/development/minikube/

<br><br>

### Links
- https://gitlab.local.com/users/sign_in

<br><br>

#### UI
- https://gitlab.local.com
- root:69aZc996


<br><br>
<br><br>

### Hosts
- Add this to your `/etc/hosts` file. In your custom-values.yaml you can also add instead global.hosts.domain=192.168.49.2.nip.io
```shell
sudo gedit /etc/hosts

# ==== MINIKUBE ====
192.168.49.2 gitlab.local.com
192.168.49.2 minio.local.com
```





<br><br>
<br><br>

### setup.sh
```shell
#!/bin/bash
cd "$(dirname "$0")"; printf "\nCurrent working directory:"; pwd

kubectl config use-context minikube

# ==== INSTALL ====
helm install gitlab-dev ./Chart --namespace dev -f ./custom-values.yaml

# Wait until gitlab UI is ready..
until kubectl get pods --namespace dev | grep gitlab-dev-webservice-default | grep Running | grep 2/2
do
    echo "Wait for healthy gitlab-dev-webservice-default Pods..."
    sleep 10
done
echo "Gitlab UI is ready.."
sleep 10

# ==== CHANGE GITLAB ROOT PASSWORD ====
NAMESPACE="dev"
POD_NAME=$(kubectl get pods -n dev | grep gitlab-dev-toolbox | awk '{print $1}')

# Prüfen, ob der Pod-Name gefunden wurde
if [ -z "$POD_NAME" ]; then
  echo "Kein Pod mit dem Namen 'gitlab-dev-toolbox' gefunden."
  exit 1
fi

# Befehl im Pod ausführen
kubectl exec -it $POD_NAME -n $NAMESPACE -- bash -c "gitlab-rails runner \"user = User.find_by(username: 'root'); user.password = '69aZc996'; user.password_confirmation = '69aZc996'; user.save!\""
```




<br><br>
<br><br>

### custom.values.yaml
```shell
# values-minikube.yaml
# This example intended as baseline to use Minikube for the deployment of GitLab
# - Services that are not compatible with how Minikube runs are disabled
# - Configured to use 192.168.49.2, and nip.io for the domain

# initialRootPassword:
#   secret: gitlab-root-password-custom
#   key: password

# Minimal settings
global:
  minio:
    enabled: true

  ingress:
    configureCertmanager: false
    class: "nginx"
    tls:
      external: true

  hosts:
    domain: local.com
    externalIP: 192.168.49.2
    
  shell:
    # Configure the clone link in the UI to include the high-numbered NodePort
    # value from below (`gitlab.gitlab-shell.service.nodePort`)
    port: 32022

# Don't use certmanager, we'll self-sign
certmanager:
  install: false

# Use the `ingress` addon, not our Ingress (can't map 22/80/443)
nginx-ingress:
  enabled: false

# Map gitlab-shell to a high-numbered NodePort cloning over SSH since
# Minikube takes port 22.
gitlab:
  gitlab-shell:
    service:
      type: NodePort
      nodePort: 32022

# Provide gitlab-runner with secret object containing self-signed certificate chain
gitlab-runner:
  install: true
  certsSecretName: gitlab-dev-wildcard-tls-chain

  runners:
    cache:
    ## S3 the name of the secret.
      secretName: minio-dev
    ## Use this line for access using gcs-access-id and gcs-private-key
    # secretName: gcsaccess
    ## Use this line for access using google-application-credentials file
    # secretName: google-application-credentials
    ## Use this line for access using Azure with azure-account-name and azure-account-key
    # secretName: azureaccess

    config: |
      [[runners]]
        image = "ubuntu:22.04"

        {{- if .Values.global.minio.enabled }}
        [runners.cache]
          Type = "s3"
          Path = "gitlab-runner"
          Shared = true
          [runners.cache.s3]
            AccessKey = "test69696969"
            SecretKey = "test69696969"
            ServerAddress = "192.168.49.2.nip.io:30000"
            BucketName = "runner-cache"
            BucketLocation = "us-east-1"
            Insecure = true
        {{ end }}
```












<br><br>
<br><br>


### Minio
- I did not find a way to get the included minio release running because of the tls self signed cert problem
    - https://gitlab.com/gitlab-org/charts/gitlab-runner/-/issues/75#note_211405230

- But we can deploy our own minio instance and then use it inside of our gitlab-runner. Please check the MinIO Install section (https://github.com/CyberT33N/minio-cheat-sheet/blob/main/README.md) above or run `./minio/setup.sh`
  - The setup will also create the needed bucket `runner-cache` for the gitlab-runner
  - It will also create the needed secret `minio-dev` inside of our `dev` namespace
  ```yaml
  gitlab-runner:
    runners:
      cache:
      ## S3 the name of the secret.
        secretName: minio-dev
  ```

- In order to use our own minio instance with the gitlab runner we have to make sure to set the correct config:
```yaml
# Provide gitlab-runner with secret object containing self-signed certificate chain
gitlab-runner:
  install: true
  certsSecretName: gitlab-dev-wildcard-tls-chain

  runners:
    cache:
    ## S3 the name of the secret.
      secretName: minio-dev
    ## Use this line for access using gcs-access-id and gcs-private-key
    # secretName: gcsaccess
    ## Use this line for access using google-application-credentials file
    # secretName: google-application-credentials
    ## Use this line for access using Azure with azure-account-name and azure-account-key
    # secretName: azureaccess

    config: |
      [[runners]]
        image = "ubuntu:22.04"

        {{- if .Values.global.minio.enabled }}
        [runners.cache]
          Type = "s3"
          Path = "gitlab-runner"
          Shared = true
          [runners.cache.s3]
            AccessKey = "test69696969"
            SecretKey = "test69696969"
            ServerAddress = "192.168.49.2.nip.io:30000"
            BucketName = "runner-cache"
            BucketLocation = "us-east-1"
            Insecure = true
        {{ end }}
```
- **AccessKey & SecretKey must be set or you get error that the url is not found. The values must be valid**
- In our case the minio instance is not https so set `Insecure = true`
- As mentioned before the bucket must already exists `mc mb minio/runner-cache`
- **runners.cache.secretName must be set**. As mentioned above the secret will be created from our side. However this is what it will look inside:
```
MINIO_ROOT_PASSWORD: test69696969                                                                                               │
MINIO_ROOT_USER: test69696969      
```
- **certsSecretName: gitlab-dev-wildcard-tls-chain** must be set or you will get tls error that gitlab-runner can not connect to gitlab.local.com



<br><br>
<br><br>

### Certs
- This section is not needed for the gitlab helm chart for minikube because of automated creation for self-signed certs. However, it is maybe usefully for somebody

<br><br>

#### Create self signed cert
```shell
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout gitlab.local.com.key -out gitlab.local.com.crt -subj "/C=DE/ST=Some-State/L=City/O=Organization/OU=Department/CN=local.com"
```

<br><br>

### Download cert and create secret

```shell
# Create cert
openssl s_client -showcerts -connect gitlab.local.com:443 -servername gitlab.local.com < /dev/null 2>/dev/null | openssl x509 -outform PEM > ./gitlab/gitlab.local.com.crt

if kubectl get secret -n dev gitlab-cert-self >/dev/null 2>&1; then
    kubectl delete secret -n dev gitlab-cert-self
fi

kubectl create secret generic gitlab-cert-self \
--namespace dev \
--from-file=./gitlab/gitlab.local.com.crt
```



<br><br>
<br><br>

### Git
- We use NodePort for gitlab-shell in order to be able to push into our repos. Git is available over port 32022. Check this guide for how to to create and to add SSH Key (https://github.com/CyberT33N/git-cheat-sheet/blob/main/README.md#ssh)
  - Then after this you run `ssh-add ~/.ssh/github/id_ecdsa` and then after this:
```shell
git remote add gitlabInternal ssh://git@gitlab.local.com:32022/websites/test.git
```



<br><br>
<br><br>


### Add repo
```shell
# Add gitlab repo
helm repo add gitlab https://charts.gitlab.io/

# Update helm repo
helm repo update

# Auflisten der verfügbaren Helm Chart Versionen
helm search repo gitlab --versions
```

<br><br>
<br><br>

### Install Helm Chart
```shell
# This will download the gitlab helm chart to the folder ./gitlab/Chart
cd ~/Projects/minikube
mkdir -p ./gitlab/Chart

helm pull gitlab/gitlab --version 8.1.2  --untar --untardir ./tmp
cp -r ./tmp/gitlab/* ./gitlab/Chart
rm -rf ./tmp

# Create custom-values.yaml
touch ./gitlab/custom-values.yaml

```
     - If you get error `download failed after attempts=6: net/http: TLS handshake timeout` in your gitlab-runner deployment try:
     ```shell
     unset all_proxy
     ```

<br><br>

### Upgrade Helm Chart
```shell
kubectl config use-context minikube
helm upgrade gitlab-dev ./gitlab/Chart --namespace dev -f ./gitlab/custom-values.yaml --atomic
```

<br><br>

### Delete release
```shell
kubectl config use-context minikube
helm --namespace dev delete gitlab-dev
```

<br><br>

### Retrieve IP addresses
```shell
kubectl get ingress -lrelease=gitlab-dev -n dev
```

<br><br>

### Change password

<br><br>

#### Method #1 - UI
- You can access the GitLab instance by visiting the domain specified, https://gitlab.192.168.99.100.nip.io is used in these examples. If you manually created the secret for initial root password, you can use that to sign in as root user. If not, GitLab automatically created a random password for the root user. This can be extracted by the following command (replace <name> by name of the release - which is gitlab if you used the command above). 
```shell
kubectl get -n dev secret gitlab-dev-gitlab-initial-root-password -ojsonpath='{.data.password}' | base64 --decode ; echo
```
- You can change the password by sign in > right click on your avater > edit > password

<br><br>

#### Method #2 - gitlab-rails
- Use gitlab-rails. The pod gitlab-dev-toolbox is able to dit
```shell
kubectl create secret generic gitlab-cert-self \
--namespace dev \
--from-file=./gitlab/gitlab.local.com.crt

NAMESPACE="dev"
POD_NAME=$(kubectl get pods -n dev | grep gitlab-dev-toolbox | awk '{print $1}')

# Prüfen, ob der Pod-Name gefunden wurde
if [ -z "$POD_NAME" ]; then
  echo "Kein Pod mit dem Namen 'gitlab-dev-toolbox' gefunden."
  exit 1
fi

# Befehl im Pod ausführen
kubectl exec -it $POD_NAME -n $NAMESPACE -- bash -c "gitlab-rails runner \"user = User.find_by(username: 'root'); user.password = 'passwordHere'; user.password_confirmation = 'passwordHere'; user.save!\""
```

<br><br>

#### Method 3 - Helm chart (Not tested)
You create a secret and set it to your custom-values.yaml like we did in this guide
```shell
kubectl create secret -n dev generic gitlab-root-password-custom --from-literal='password=test'

```
```yaml
# initialRootPassword:
#   secret: gitlab-root-password-custom
#   key: password

```

<br><br>

### Check ingress object
```shell
kubectl describe ingress gitlab-dev-webservice-default -n dev
```

<br><br>



</details>

