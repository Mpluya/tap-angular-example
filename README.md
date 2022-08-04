# TAP Angular Example

## Prerequisites

TAP 1.1 (Tanzu Application Platform) ships with TBS (Tanzu Build Service) which
is based on kpack. TBS brings a bunch of `ClusterBuilders` none of which
include the `nodejs` and `nginx` buildpack in the same `Group`. Therefore,
building and Angular app that is served as static files via Nginx is not
possible with the default configuration.

Luckily, TAP is very extensible and customizable so that we can make this work
with only a few tweaks.

1. Create `ClusterBuilder` that includes the `web-servers` buildpack
```
REGISTRY="gcr.io/cso-pcfs-emea-mewald"
kp clusterstore create spa \
  -b paketobuildpacks/web-servers

kp clusterbuilder create spa \
  --buildpack paketo-buildpacks/web-servers \
  --tag $REGISTRY/spa:v2 \
  --stack full \
  --store spa
```

2. Create a `ClusterSupplyChain` that uses that new `ClusterBuilder`

**Option A:** Edit the `source-to-url` supply chain and add a parameter to
overwrite the `clusterBuilder` value. If rolled out via `PackageInstall` this
will be reverted through the reconciliation process of kapp-controller.

**Option B:** Create a separate `ClusterSupplyChain` that selects `spa`
`Workloads` and annotate your `Workload` accordingly.

```
mkdir source-to-url-spa
cd source-to-url-spa

kubectl get clustersupplychain source-to-url -o yaml > cluster-supplychain.yaml
```
```
cat <<EOF > kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- cluster-supplychain.yaml
patches:
- target:
    group: carto.run
    kind: ClusterSupplyChain
    version: v1alpha1
  patch: | 
    - op: replace
      path: /metadata/name
      value: source-to-url-spa
    - op: replace
      path: /spec/resources/2/params/1/value
      value: spa
    - op: replace
      path: /spec/selector/apps.tanzu.vmware.com~1workload-type
      value: spa
EOF
```
```
kubectl apply -k .
```

## Running

```
export SOURCE_IMAGE="gcr.io/cso-pcfs-emea-mewald/tap-angular-example"
export NAMESPACE="mewald"
tilt up
```

