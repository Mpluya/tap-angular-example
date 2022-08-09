# TAP Angular Example

## Prerequisites

TAP 1.1 (Tanzu Application Platform) ships with TBS (Tanzu Build Service) which
is based on kpack. TBS brings a bunch of `ClusterBuilders` none of which
include the `nodejs` and `nginx` buildpack in the same `Group`. Therefore,
building and Angular app that is served as static files via Nginx is not
possible with the default configuration.

Luckily, TAP is very extensible and customizable so that we can make this work
with only a few tweaks.

0. Create a `ClusterStack` that is compatible with the `web-servers` buildpack

**custom-stack.yaml**
```
---
apiVersion: stacks.stacks-operator.tanzu.vmware.com/v1alpha1
kind: CustomStack
metadata:
  name: spa-stack
spec:
  source:
    stack:
      name: base
      kind: ClusterStack
  destination:
    build:
      tag: gcr.io/cso-pcfs-emea-mewald/build-service/spa-buildimage
    run:
      tag: gcr.io/cso-pcfs-emea-mewald/build-service/spa-runimage
    stack:
      name: spa
      kind: ClusterStack
  packages:
    - name: libexpat1
  mixins:
    - name: libexpat1
  serviceAccountName: kp-default-repository-serviceaccount
```
```
kubectl -n kpack apply -f custom-stack.yaml
```

1. Create `ClusterBuilder` that includes the `web-servers` buildpack
```
REGISTRY="gcr.io/cso-pcfs-emea-mewald"
kp clusterstore create spa \
  -b paketobuildpacks/web-servers

kp clusterbuilder create spa \
  --buildpack paketo-buildpacks/web-servers \
  --tag $REGISTRY/spa:v2 \
  --stack spa \
  --store spa
```

2. Create a `ClusterSupplyChain` that uses that new `ClusterBuilder`

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
      path: /spec/resources/2/params/2/default
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

