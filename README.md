# Install Tekton Operator
![Alt text](./static/this-is-the-way.webp "This is the way")
``` bash
kubectl apply -f https://storage.googleapis.com/tekton-releases/operator/latest/release.yaml
```

# Install tekton-pipelines by operator 

``` yaml
cat <<EOF | kubectl create -f -
apiVersion: operator.tekton.dev/v1alpha1
kind: TektonConfig
metadata:
  name: config
spec:
  profile: all
  targetNamespace: tekton-pipelines
  pruner:
    resources:
    - pipelinerun
    - taskrun
    keep: 2
    schedule: "0 8 * * *"
EOF
```

# Installing Tekton Results (optional) https://tekton.dev/docs/results/
![Alt text](./static/result.png "Result API")

## Prerequisites

1. Tekton Pipelines must be installed on the cluster.
2. Generating a database root password.

   A database root password must be generated by the user and stored in a
   [Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/)
   before installing. By default, Tekton Results expects this `Secret` to have
   the following properties:

   - namespace: `tekton-pipelines`
   - name: `tekton-results-postgres`
   - contains the fields:
     - `POSTGRES_USER=postgres`
     - `POSTGRES_PASSWORD=<your password>`

   If you are not using a particular password management strategy, the following
   command will generate a random password for you:

   ``` sh
   kubectl create secret generic tekton-results-postgres --namespace="tekton-pipelines" --from-literal=POSTGRES_USER=postgres --from-literal=POSTGRES_PASSWORD=$(openssl rand -base64 20)
   ```

3. Generate cert/key pair. Note: Feel free to use any cert management software
   to do this!

   Tekton Results expects the cert/key pair to be stored in a
   [TLS Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/#tls-secrets) named `tekton-results-tls`.

   ``` sh
   cd cert
   ```

   ``` sh
   # Generate new self-signed cert.
   openssl req -x509 \
   -newkey rsa:4096 \
   -keyout key.pem \
   -out cert.pem \
   -days 365 \
   -nodes \
   -subj "/CN=tekton-results-api-service.tekton-pipelines.svc.cluster.local" \
   -addext "subjectAltName = DNS:tekton-results-api-service.tekton-pipelines.svc.cluster.local"
   ```

   ``` sh
   # Create new TLS Secret from cert.
   kubectl create secret tls -n tekton-pipelines tekton-results-tls \
   --cert=cert.pem \
   --key=key.pem
   ```
## Installing latest release

```sh
kubectl apply -f https://storage.googleapis.com/tekton-releases/results/previous/v0.4.0/release.yaml
```

# Create Pipline

## Install tasks
### git-clone https://hub.tekton.dev/tekton/task/git-clone
``` bash
tkn hub install task git-clone -n default
```

### buildah https://hub.tekton.dev/tekton/task/buildah
``` bash
tkn hub install task buildah -n default
```

### Create hadolint task https://hub.tekton.dev/tekton/task/hadolint (created arm64 hadolint)
``` bash
kubectl apply -f tekton/workshop-task-hadolint.yaml
```

### Generate new github token 
https://github.com/settings/tokens/new

### Add secret with generated token
``` bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: hwsw-workshop-cr-push-secret
  namespace: default
  annotations:
    tekton.dev/docker-0: https://ghcr.io
type: kubernetes.io/basic-auth
stringData:
    username: ptzool # github user name
    password: ${TOKEN} 
EOF


# or


kubectl create secret generic hwsw-workshop-cr-push-secret -n default --type="kubernetes.io/basic-auth" --from-literal=username=USER --from-literal=password=TOKEN

kubectl annotate secret hwsw-workshop-cr-push-secret tekton.dev/docker-0=https://ghcr.io
```

### Add pipline

``` bash
cat tekton/workshop-pipeline-pvc.yaml | yq
kubectl apply -f tekton/workshop-pipeline-pvc.yaml
```

``` bash
cat tekton/workshop-serviceaccount.yaml | yq
kubectl apply -f tekton/workshop-serviceaccount.yaml
```

``` bash
cat tekton/workshop-pipeline.yaml | yq
kubectl apply -f tekton/workshop-pipeline.yaml
```

### Add eventlistener
``` bash
cat tekton/eventlistener/workshop-el.yaml | yq
kubectl apply -f tekton/eventlistener/workshop-el.yaml
```

``` bash
cat tekton/eventlistener/workshop-el-tt.yaml | yq
kubectl apply -f tekton/eventlistener/workshop-el-tt.yaml
```

``` bash
cat tekton/eventlistener/workshop-el-tb.yaml | yq
kubectl apply -f tekton/eventlistener/workshop-el-tb.yaml
```

### Start pipline
``` bash
cat cat tekton/workshop-pipelinerun.yaml | yq
kubectl create -f tekton/workshop-pipelinerun.yaml
```

### EL curl request
``` bash
curl -X POST -d '{"imageTag":"v.1.0.6"}' localhost:8080 | jq
```
