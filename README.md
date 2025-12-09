# Frontend Chart Template

A minimal Helm chart template for deploying a simple nginx frontend on OpenShift Local.

### Prerequisites

- OpenShift Local (formerly CodeReady Containers) running
- `helm` CLI installed
- `oc` CLI logged in to your OpenShift cluster
- `kubectl` CLI for interacting with kubernetes resources

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

### Basic Installation

1. Navigate to the chart directory

2. Install the chart with default values:
```bash
helm install my-frontend . -f values.yaml -n your-namespace
```

3. Check the deployment status using oc or kubectl:
```bash
oc get pods -n your-namespace
oc get route -n your-namespace
```

4. Hit your app at the URL printed when you run ```kubectl get routes```

### Upgrading

```bash
helm upgrade my-frontend . -f your-values.yaml -n your-namespace
```

### Uninstalling

```bash
helm uninstall my-frontend -n your-namespace
```