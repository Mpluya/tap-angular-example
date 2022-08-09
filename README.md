# TAP Angular Example

## Prerequisites

TAP 1.2 (Tanzu Application Platform) ships with TBS (Tanzu Build Service) 1.6.0 which now includes the `web-servers` buildpack in the `full` profile. Make sure to install the full dependencies package (https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.2/tap/GUID-install.html#tap-install-full-deps) during the TAP installation process.

UPDATE FOR TAP 1.2 TO COME SOON


## Running

```
export SOURCE_IMAGE="gcr.io/cso-pcfs-emea-mewald/tap-angular-example"
export NAMESPACE="mewald"
tilt up
```

