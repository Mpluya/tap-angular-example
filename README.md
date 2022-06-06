# TAP Angular Example

```
export SOURCE_IMAGE="gcr.io/cso-pcfs-emea-mewald/tap-angular-example"
export NAMESPACE="mewald"
tilt up
```

## Current Situation 

The supply chain ran through completely:
```
$ tanzu -n mewald app workload get tap-angular-example
# tap-angular-example: Ready
---
lastTransitionTime: "2022-06-06T20:56:14Z"
message: ""
reason: Ready
status: "True"
type: Ready

Workload pods
NAME                                                   STATUS      RESTARTS   AGE
tap-angular-example-00001-deployment-c46d4f96c-s7vhk   Running     0          5m5s
tap-angular-example-build-1-build-pod                  Succeeded   0          10m
tap-angular-example-config-writer-6952c-pod            Succeeded   0          5m53s

Workload Knative Services
NAME                  READY     URL
tap-angular-example   Unknown   https://tap-angular-example-mewald.cnrs.tap.halusky.net
```

The knative `App` was created and is reconciled:
```
$ kubectl -n mewald get app
NAME                  DESCRIPTION           SINCE-DEPLOY   AGE
tap-angular-example   Reconcile succeeded   6m12s          11m
```

The `Deployment` was created but doesn't become ready. The `ReplicaSet` shows that Nginx should be listening on port 8080 (env PORT) but the readiness check goes against port 8012. Moreover, there are variables set like `JAVA_TOOL_OPTIONS` which doesn't seem right in an Angular + Nginx context.
```
$ kubectl -n mewald get deployment
NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
tap-angular-example-00001-deployment   0/1     1            0           7m9s
$ kubectl -n mewald get rs
NAME                                             DESIRED   CURRENT   READY   AGE
tap-angular-example-00001-deployment-c46d4f96c   1         1         0       7m12s
$ kubectl -n mewald describe rs tap-angular-example-00001-deployment-c46d4f96c
kubectl -n mewald describe rs tap-angular-example-00001-deployment-c46d4f96c
Name:           tap-angular-example-00001-deployment-c46d4f96c
Namespace:      mewald
Selector:       pod-template-hash=c46d4f96c,serving.knative.dev/revisionUID=d62358f1-3348-444f-947e-b15b4c89d6f4
Labels:         app=tap-angular-example-00001
                app.kubernetes.io/component=run
                app.kubernetes.io/part-of=tap-angular-example
                apps.tanzu.vmware.com/workload-type=web
                carto.run/workload-name=tap-angular-example
                pod-template-hash=c46d4f96c
                serving.knative.dev/configuration=tap-angular-example
                serving.knative.dev/configurationGeneration=1
                serving.knative.dev/configurationUID=e3147930-10bf-4b1e-8db4-0334efd91b44
                serving.knative.dev/revision=tap-angular-example-00001
                serving.knative.dev/revisionUID=d62358f1-3348-444f-947e-b15b4c89d6f4
                serving.knative.dev/service=tap-angular-example
                serving.knative.dev/serviceUID=2202f699-d6c2-4a70-bfed-41b689c2cc9a
Annotations:    apps.tanzu.vmware.com/live-update: true
                conventions.apps.tanzu.vmware.com/applied-conventions: developer-conventions/add-source-image-label
                deployment.kubernetes.io/desired-replicas: 1
                deployment.kubernetes.io/max-replicas: 2
                deployment.kubernetes.io/revision: 1
                developer.apps.tanzu.vmware.com/image-source-digest:
                  gcr.io/cso-pcfs-emea-mewald/tap-angular-example:latest@sha256:59bca5aaaee79f7ac3a2072c78bf062eca7ede78faafc9eae2ec0e8f8891b48e
                developer.conventions/target-containers: workload
                serving.knative.dev/creator: system:serviceaccount:mewald:default
Controlled By:  Deployment/tap-angular-example-00001-deployment
Replicas:       1 current / 1 desired
Pods Status:    1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:           app=tap-angular-example-00001
                    app.kubernetes.io/component=run
                    app.kubernetes.io/part-of=tap-angular-example
                    apps.tanzu.vmware.com/workload-type=web
                    carto.run/workload-name=tap-angular-example
                    pod-template-hash=c46d4f96c
                    serving.knative.dev/configuration=tap-angular-example
                    serving.knative.dev/configurationGeneration=1
                    serving.knative.dev/configurationUID=e3147930-10bf-4b1e-8db4-0334efd91b44
                    serving.knative.dev/revision=tap-angular-example-00001
                    serving.knative.dev/revisionUID=d62358f1-3348-444f-947e-b15b4c89d6f4
                    serving.knative.dev/service=tap-angular-example
                    serving.knative.dev/serviceUID=2202f699-d6c2-4a70-bfed-41b689c2cc9a
  Annotations:      apps.tanzu.vmware.com/live-update: true
                    conventions.apps.tanzu.vmware.com/applied-conventions: developer-conventions/add-source-image-label
                    developer.apps.tanzu.vmware.com/image-source-digest:
                      gcr.io/cso-pcfs-emea-mewald/tap-angular-example:latest@sha256:59bca5aaaee79f7ac3a2072c78bf062eca7ede78faafc9eae2ec0e8f8891b48e
                    developer.conventions/target-containers: workload
                    serving.knative.dev/creator: system:serviceaccount:mewald:default
  Service Account:  default
  Containers:
   workload:
    Image:      gcr.io/cso-pcfs-emea-mewald/build-service/tap-angular-example-mewald@sha256:7494a6d992b2ab0dbc656c30a68ec7edf75921fffa54ca08d9d83e07c661a5d0
    Port:       8080/TCP
    Host Port:  0/TCP
    Environment:
      JAVA_TOOL_OPTIONS:  -Dmanagement.endpoint.health.probes.add-additional-paths="true" -Dmanagement.health.probes.enabled="true"
      PORT:               8080
      K_REVISION:         tap-angular-example-00001
      K_CONFIGURATION:    tap-angular-example
      K_SERVICE:          tap-angular-example
    Mounts:               <none>
   queue-proxy:
    Image:       gcr.io/cso-pcfs-emea-mewald/tap-packages@sha256:afcfb8360ba0b196f941e75fd660d83827f62a546729b7344d6226d1279708cf
    Ports:       8022/TCP, 9090/TCP, 9091/TCP, 8012/TCP
    Host Ports:  0/TCP, 0/TCP, 0/TCP, 0/TCP
    Requests:
      cpu:      25m
    Readiness:  http-get http://:8012/ delay=0s timeout=1s period=1s #success=1 #failure=3
    Environment:
      SERVING_NAMESPACE:                 mewald
      SERVING_SERVICE:                   tap-angular-example
      SERVING_CONFIGURATION:             tap-angular-example
      SERVING_REVISION:                  tap-angular-example-00001
      QUEUE_SERVING_PORT:                8012
      CONTAINER_CONCURRENCY:             0
      REVISION_TIMEOUT_SECONDS:          300
      SERVING_POD:                        (v1:metadata.name)
      SERVING_POD_IP:                     (v1:status.podIP)
      SERVING_LOGGING_CONFIG:
      SERVING_LOGGING_LEVEL:
      SERVING_REQUEST_LOG_TEMPLATE:      {"httpRequest": {"requestMethod": "{{.Request.Method}}", "requestUrl": "{{js .Request.RequestURI}}", "requestSize": "{{.Request.ContentLength}}", "status": {{.Response.Code}}, "responseSize": "{{.Response.Size}}", "userAgent": "{{js .Request.UserAgent}}", "remoteIp": "{{js .Request.RemoteAddr}}", "serverIp": "{{.Revision.PodIP}}", "referer": "{{js .Request.Referer}}", "latency": "{{.Response.Latency}}s", "protocol": "{{.Request.Proto}}"}, "traceId": "{{index .Request.Header "X-B3-Traceid"}}"}
      SERVING_ENABLE_REQUEST_LOG:        false
      SERVING_REQUEST_METRICS_BACKEND:   prometheus
      TRACING_CONFIG_BACKEND:            none
      TRACING_CONFIG_ZIPKIN_ENDPOINT:
      TRACING_CONFIG_DEBUG:              false
      TRACING_CONFIG_SAMPLE_RATE:        0.1
      USER_PORT:                         8080
      SYSTEM_NAMESPACE:                  knative-serving
      METRICS_DOMAIN:                    knative.dev/internal/serving
      SERVING_READINESS_PROBE:           {"tcpSocket":{"port":8080,"host":"127.0.0.1"},"successThreshold":1}
      ENABLE_PROFILING:                  false
      SERVING_ENABLE_PROBE_REQUEST_LOG:  false
      METRICS_COLLECTOR_ADDRESS:
      CONCURRENCY_STATE_ENDPOINT:
      CONCURRENCY_STATE_TOKEN_PATH:      /var/run/secrets/tokens/state-token
      HOST_IP:                            (v1:status.hostIP)
      ENABLE_HTTP2_AUTO_DETECTION:       false
    Mounts:                              <none>
  Volumes:                               <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  7m23s  replicaset-controller  Created pod: tap-angular-example-00001-deployment-c46d4f96c-s7vhk
```
