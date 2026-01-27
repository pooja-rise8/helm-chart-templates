# Frontend Chart Template

A minimal Helm chart template for deploying a locally built container image on OpenShift Local by pushing to OpenShift's internal registry.

### Prerequisites

- OpenShift Local running
- `helm` CLI installed
- `oc` CLI logged in to your OpenShift cluster
- `kubectl` CLI for interacting with kubernetes resources
- `podman` for building and pushing container images

### Setup

1. Stop & Clean Existing Cluster (if needed)

- crc stop - Stop any running OpenShift Local instance
- crc cleanup - Clean up resources

2. Initialize Cluster

- crc setup - Configure networking and hypervisor
- crc start -p "$PULL_SECRET_PATH" - Start cluster with pull secret

3. Login & Configure Environment

- eval $(crc oc-env) - Set environment variables for oc CLI
- oc login -u kubeadmin https://api.crc.testing:6443 - Login as admin

4. Create Project & Configure Registry

- oc new-project "$TARGET_NAMESPACE" - Create a namespace for your app

---

### Overview

The workflow involves:

1. Building your container image locally using Podman
2. Pushing the image to OpenShift's internal registry
3. Deploying the image using this Helm chart

### Prerequisites for Local Images

- **Podman installed**: Required for building and pushing container images
- **Separate repository with Dockerfile**: Your application code and Dockerfile should be in a separate repository
- **OpenShift logged in**: You must be logged in with `oc login`

### OpenShift Internal Registry

OpenShift Local includes a built-in container registry that's automatically exposed:

- **External URL** (for pushing): `default-route-openshift-image-registry.apps-crc.testing`
- **Internal URL** (for pulling): `image-registry.openshift-image-registry.svc:5000`

The registry uses your OpenShift authentication token, so no separate credentials are needed.

To verify the registry is accessible:

```bash
# Check if registry route exists
oc get route -n openshift-image-registry

# Get registry URL
oc registry info
```

### Step 1: Build Your Container Image

In your application repository (containing the Dockerfile):

```bash
cd /path/to/your/app

# Log into private Docker/Podman registry (stores creds in podman machine)
# this step is OPTIONAL - only if your build references a private registry
podman login <YOUR_REGISTRY_SERVER_DOMAIN> -u '<YOUR_LOGIN_USERNAME>'

podman build -t <image-name>:<tag> .
```

Alternatively, just pull this public image to verify this internal registry setup

```bash
podman pull nginxinc/nginx-unprivileged:stable-alpine
```

### Step 2: Log in to OpenShift Registry

```bash
# Get your OpenShift token
OC_TOKEN=$(oc whoami -t)

# Log in to the registry
podman login -u $(oc whoami) -p $OC_TOKEN default-route-openshift-image-registry.apps-crc.testing
```

You may have to include --tls-verify=false

### Step 3: Tag Image for OpenShift Registry

```bash
podman tag <image-name>:<tag> default-route-openshift-image-registry.apps-crc.testing/<namespace>/<image-name>:<tag>
```

### Step 4: Push Image to Registry

```bash
podman push default-route-openshift-image-registry.apps-crc.testing/<namespace>/<image-name>:<tag>
```

You may have to include `--tls-verify=false`

If you run into errors running `podman push`, try the following [workaround](docs/podman-push-workaround.md).

### Step 5: Deploy with Helm

Basic Installation

1. Navigate to the chart directory
2. Install the chart with default values:

```bash
helm install <image-name> . -f values.yaml -n <namespace>
```

Upgrading

```bash
helm upgrade <image-name> . -f your-values.yaml -n <namespace>
```

Uninstalling

```bash
helm uninstall <image-name> -n <namespace>
```

### Step 6: Verify Deployment

```bash
# Check pods
kubectl get pods -n <namespace>

# Check route to get application URL
kubectl get route -n <namespace>

# View logs
kubectl logs -l app.kubernetes.io/name=frontend-chart-template -n <namespace>
```

---

### Container Port

Ensure the `containerPort` in your values file matches the port your application listens on:

```yaml
# values.yaml
containerPort: 8080 # Change to match your app (e.g., 3000 for React dev server)
```

---

## Troubleshooting

### Image Pull Errors

If pods fail to pull images:

```bash
# Check if image exists in registry
oc get is -n <namespace>

# Check image stream tags
oc describe is <image-name> -n <namespace>

# Verify service account can pull images
oc policy add-role-to-user system:image-puller system:serviceaccount:<namespace>:default --namespace=<namespace>
```

### Registry Login Issues

If you can't log in to the registry:

```bash
# Verify you're logged in to OpenShift
oc whoami

# Check registry route is exposed
oc get route -n openshift-image-registry

# Try logging in again with explicit token
podman login -u $(oc whoami) -p $(oc whoami -t) default-route-openshift-image-registry.apps-crc.testing
```

### Podman Build Failures

If builds fail:

```bash
# Check Podman is working
podman version

# Try building with more verbose output
podman build -t my-app:latest . --log-level=debug

# On macOS, ensure Podman machine is running
podman machine list
podman machine start
```

### Application Not Accessible

If your application deploys but isn't accessible:

```bash
# Check pod status
kubectl get pods -n <namespace>

# Check pod logs for errors
kubectl logs <pod-name> -n <namespace>

# Verify route exists and is correct
kubectl get route -n <namespace>
kubectl describe route <route-name> -n <namespace>

# Check service is correctly routing to pods
kubectl get endpoints -n <namespace>
```

---

## Additional Resources

- [Podman Documentation](https://docs.podman.io/)
- [OpenShift Local Documentation](https://access.redhat.com/documentation/en-us/red_hat_openshift_local/)
- [OpenShift Image Registry](https://docs.openshift.com/container-platform/latest/registry/index.html)
- [Helm Documentation](https://helm.sh/docs/)
