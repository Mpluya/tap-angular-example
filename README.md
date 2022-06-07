# TAP Angular Example

```
export SOURCE_IMAGE="gcr.io/cso-pcfs-emea-mewald/tap-angular-example"
export NAMESPACE="mewald"
tilt up
```

## Current Situation 

The `Deployment` and `Pod` is created but logs in the `workload` container indicate a problem:

```
$ kubectl -n mewald logs -f tap-angular-example-00001-deployment-7cd97bf9f4-gjb2g workload
Warning: Running a server with --disable-host-check is a security risk. See https://medium.com/webpack/webpack-dev-server-middleware-security-issues-1489d950874a for more information.
- Generating browser application bundles (phase: setup)...
[[Command killed by ForceStop]]
Warning: Running a server with --disable-host-check is a security risk. See https://medium.com/webpack/webpack-dev-server-middleware-security-issues-1489d950874a for more information.
- Generating browser application bundles (phase: setup)...
Another process, with id 56, is currently running ngcc.
Waiting up to 250s for it to finish.
(If you are sure no ngcc process is running then you should delete the lock-file at /layers/tanzu-buildpacks_npm-install/launch-modules/node_modules/.ngcc_lock_file.)

Error: Timed out waiting 250s for another ngcc process, with id 56, to complete.
(If you are sure no ngcc process is running then you should delete the lock-file at /layers/tanzu-buildpacks_npm-install/launch-modules/node_modules/.ngcc_lock_file.)
    at AsyncLocker.create (file:///layers/tanzu-buildpacks_npm-install/launch-modules/node_modules/@angular/compiler-cli/bundles/chunk-EI6PFDB4.js:1702:11)
    at async AsyncLocker.lock (file:///layers/tanzu-buildpacks_npm-install/launch-modules/node_modules/@angular/compiler-cli/bundles/chunk-EI6PFDB4.js:1673:5)
    at async file:///layers/tanzu-buildpacks_npm-install/launch-modules/node_modules/@angular/compiler-cli/bundles/ngcc/main-ngcc.js:31:5
/layers/tanzu-buildpacks_npm-install/launch-modules/node_modules/@ngtools/webpack/src/ngcc_processor.js:135
            throw new Error(errorMessage + `NGCC failed${errorMessage ? ', see above' : ''}.`);
                  ^

Error: NGCC failed.
    at NgccProcessor.process (/layers/tanzu-buildpacks_npm-install/launch-modules/node_modules/@ngtools/webpack/src/ngcc_processor.js:135:19)
    at /layers/tanzu-buildpacks_npm-install/launch-modules/node_modules/@ngtools/webpack/src/ivy/plugin.js:147:27
    at Hook.eval [as call] (eval at create (/layers/tanzu-buildpacks_npm-install/launch-modules/node_modules/tapable/lib/HookCodeFactory.js:19:10), <anonymous>:16:1)
    at Hook.CALL_DELEGATE [as _call] (/layers/tanzu-buildpacks_npm-install/launch-modules/node_modules/tapable/lib/Hook.js:14:14)
    at Compiler.newCompilation (/layers/tanzu-buildpacks_npm-install/launch-modules/node_modules/webpack/lib/Compiler.js:1121:30)
    at /layers/tanzu-buildpacks_npm-install/launch-modules/node_modules/webpack/lib/Compiler.js:1166:29
    at eval (eval at create (/layers/tanzu-buildpacks_npm-install/launch-modules/node_modules/tapable/lib/HookCodeFactory.js:33:10), <anonymous>:31:1)
[[Command exited with 1]]
```

