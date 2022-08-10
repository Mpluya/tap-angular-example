# TAP Angular Example

## Prerequisites (TAP 1.2)

1. In version 1.2, TAP ships with TBS (Tanzu BuildService) 1.6.0 which now includes the `web-servers` buildpack in the `full` profile. Make sure to install the full dependencies package (https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.2/tap/GUID-install.html#tap-install-full-deps) during the TAP installation process.

2. The `full` `ClusterBuilder` includes the `web-servers` buildpack, but it won't get triggered for an Angular app. Instead the `nodejs` buildpack will be used as it comes earlier in the sequence. Therefore, we need to create our own `ClusterBuilder` to include only the `web-servers` buildpack. It needs to be based on the `full` `ClusterStack` to work:

```
REGISTRY="gcr.io/cso-pcfs-emea-mewald"
kp clusterbuilder create spa \
  --buildpack paketo-buildpacks/web-servers \
  --tag $REGISTRY/spa:v2 \
  --stack full \
  --store default
```

3. By default, the `source-to-url` `ClusterSupplyChain` uses the `default` `ClusterBuilder` but the `ClusterBuilder` can be parameterized like this

```
tanzu app workload create --param clusterBuilder=full
```

or in the `workload.yaml`:

```
...
kind: Workload
...
spec:
  params:
  - name: clusterBuilder
    value: spa
```

## Running

```
export SOURCE_IMAGE="gcr.io/cso-pcfs-emea-mewald/tap-angular-example"
export NAMESPACE="mewald"
tilt up
```

