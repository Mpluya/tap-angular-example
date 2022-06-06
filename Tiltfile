SOURCE_IMAGE = os.getenv("SOURCE_IMAGE")
LOCAL_PATH = os.getenv("LOCAL_PATH", default='.')
NAMESPACE = os.getenv("NAMESPACE")

update_settings ( max_parallel_updates = 3 , k8s_upsert_timeout_secs = 120 , suppress_unused_image_warnings = None )

k8s_custom_deploy(
   'tap-angular-example',
   apply_cmd="tanzu apps workload apply -f workload.yaml --live-update" +
       " --local-path " + LOCAL_PATH +
       " --source-image " + SOURCE_IMAGE +
       " --namespace " + NAMESPACE +
       " --yes > /dev/null" +
       " && kubectl get workload tap-angular-example --namespace " + NAMESPACE + " -o yaml",
   delete_cmd="tanzu apps workload delete -f workload.yaml --namespace " + NAMESPACE + " --yes" ,
   container_selector='workload',
   deps=[],
   live_update=[
       fall_back_on(['package.json', 'package-lock.json']),
       sync('.', '/src'),
   ]
)

k8s_resource('tap-angular-example', port_forwards=["3000:3000"],
   extra_pod_selectors=[{'serving.knative.dev/service': 'tap-angular-example'}])

allow_k8s_contexts('gke_cso-pcfs-emea-mewald_europe-west3_tap-cluster-demo')
