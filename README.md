# how-to-package-digirunner-using-helm
Step-by-step guide on how to package the digirunner open source project using Helm.

**Prerequisite**

- helm version: `v3.18.*`
- [helm installation](https://helm.sh/docs/intro/install/)

**Step 1: Create A New Helm Project**

```shell
helm create digirunner-opensource-helm # your custom directory name
```

The created file structure as below:

```
digirunner-opensource-helm/
├─ charts/
├─ templates/
│  ├─ tests/
│  ├─ _helpers.tpl
│  ├─ deployment.yaml
│  ├─ hpa.yaml
│  ├─ ingress.yaml
│  ├─ NOTES.txt
│  ├─ service.yaml
│  ├─ serviceaccount.yaml
├─ .helmignore
├─ Chart.yaml
├─ values.yaml
```

**Step 2: Keep it simple**

Keep only the following file structure; delete everything else.

```
digirunner-opensource-helm/
├─ charts/
├─ templates/
│  ├─ _helpers.tpl
│  ├─ deployment.yaml
│  ├─ service.yaml
├─ .helmignore
├─ Chart.yaml
├─ values.yaml
```

**Step 3: Setting Values**

Replace the settings in the `values.yaml` file parts according to the following example.

`image` part

```yaml
# This sets the container image more information can be found here: https://kubernetes.io/docs/concepts/containers/images/
image:
  repository: tpisoftwareopensource/digirunner-open-source
  # This sets the pull policy for images.
  pullPolicy: Always
  # Overrides the image tag whose default is the chart appVersion.
  tag: "release-v4.5.5" # any available digirunner image tag
```

`service` part
```yaml
# This is for setting up a service more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/
service:
  # This sets the service type more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types
  type: NodePort
  # This sets the ports more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#field-spec-ports
  port: 18080
  nodePort: 31080
```

`probe` part
```yaml
# This is to setup the liveness and readiness probes more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
livenessProbe:
  httpGet:
    path: /dgrv4/liveness
    port: http
readinessProbe:
  httpGet:
    path: /dgrv4/version
    port: http
```

**Step 4: Setting Service**

Add the `nodePort` setting in the `/templates/service.yaml` file

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "digirunner-opensource-helm.fullname" . }}
  labels:
    {{- include "digirunner-opensource-helm.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
      nodePort: {{ .Values.service.nodePort }} # <-- add this line
  selector:
    {{- include "digirunner-opensource-helm.selectorLabels" . | nindent 4 }}
```

**Step 5: Setting Deployment**

Delete `serviceAccountName` setting in `templates/deployment.yaml` file


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "digirunner-opensource-helm.fullname" . }}
# ... (ellipsis)
      serviceAccountName: ...(ellipsis)  # <-- delete this line
# ... (ellipsis)
```

### We Are Ready

Run the following command to launch digirunner with Helm

```shell
helm install digirunner digirunner-opensource-helm/
```

# Upload helm chart to dockerhub

**Step 1: Config helm chart**

Modify `appVersion`、`name` and `version` field in `Chart.yaml`

```yaml
name: digirunner-opensource-helm # your custom Helm chart name
version: 1.0.0 # your custom Helm chart version
appVersion: "release-v4.5.5" # digirunner image tag you choose
```

**Step 2: Package helm chart**

```shell
helm package digirunner-opensource-helm
```

This operation will then generate a `{chart:name}-{chart:version}.tgz` file.

**Step 3: Helm registry login**

```shell
helm registry login registry-1.docker.io -u {your_dockerhub_username}
```

[how to login to dockerhub using access token](https://docs.docker.com/security/for-developers/access-tokens/)

**Step 4: Helm upload**

```shell
helm push digirunner-opensource-helm-1.0.0.tgz oci://registry-1.docker.io/{your_dockerhub_username}
```

Upon successful upload, it will be generated at the following URL
```
https://hub.docker.com/repository/docker/{your_dockerhub_username}/{chart:name}/general
```

### We Are Ready...Again!

Launch digirunner with Helm using public link

```shell
helm install digirunner oci://registry-1.docker.io/{your_dockerhub_username}/{chart:name}
```

# Congratulations! 

You've successfully built a minimal digirunner Helm chart and uploaded it to Docker Hub!

---

- All example configurations are located in the [digirunner-opensource-helm](./digirunner-opensource-helm) directory.

- The Helm OCI address in the example is at https://hub.docker.com/repository/docker/vulcantpisoft/digirunner-opensource-helm/general.

- Quickly install the example Helm chart:

    ```shell
    helm install digirunner oci://registry-1.docker.io/vulcantpisoft/digirunner-opensource-helm
    ```