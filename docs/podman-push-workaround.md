# Podman push workaround

## Errors

When running the `podman push ...` command from the README.md, you may run into issues with your local computer's network settings (for other projects or set up by your IT department) which will throw errors like the following:

```sh
Error: trying to reuse blob sha256:186...a8f at destination: pinging container registry default-route-openshift-image-registry.apps-crc.testing: Get "https://default-route-openshift-image-registry.apps-crc.testing/v2/": EOF
```

```sh
Error: trying to reuse blob sha256:186...a8f at destination: pinging container registry default-route-openshift-image-registry.apps-crc.testing: StatusCode: 404, "404 page not found\n"
```

## Workaround  - build the image inside the OpenShift cluster

```sh
# Create private registry pull secret (OPTIONAL, only if your build references a private registry)
oc create secret docker-registry <PULL-SECRET-NAME> \
  --docker-server=<YOUR_REGISTRY_SERVER_DOMAIN> \
  --docker-username='<YOUR_LOGIN_USERNAME>' \
  --docker-password='YOUR_PASSWORD' \
  --docker-email='<YOUR_EMAIL>' \
  -n <YOUR_NAMESPACE>

# Create BuildConfig
oc new-build --name=<BUILD_NAME> --binary --strategy=docker -n <YOUR_NAMESPACE>

# use pull secret from earlier step (OPTIONAL)
oc patch buildconfig <BUILD_NAME> -n <YOUR_NAMESPACE> -p '{"spec":{"strategy":{"dockerStrategy":{"pullSecret":{"name":"<PULL-SECRET-NAME>"}}}}}'
oc secrets link builder <PULL-SECRET-NAME> --for=pull -n <YOUR_NAMESPACE>
oc secrets link default <PULL-SECRET-NAME> --for=pull -n <YOUR_NAMESPACE>

# Build
cd /path/to/your/code/repo
oc start-build <BUILD_NAME> --from-dir=. --follow -n <YOUR_NAMESPACE>
```

At this point, you should be able to proceed with the Helm install steps from the README.
