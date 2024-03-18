## Helm Rendering

Helm renders the pod from the sub-chart with `readOnlyRootFilesystem: true`:

```
helm template --debug .
install.go:200: [debug] Original chart version: ""
install.go:217: [debug] CHART PATH: /Users/alehall/projects/trivy-subcharts

---
# Source: trivy-subcharts/charts/nginx/templates/pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 8080
    securityContext:
      readOnlyRootFilesystem: true
```

## Trivy Scan

Trivy scanning treats the pod as if it had `readOnlyRootFilesystem: false`

```
$ trivy conf --severity HIGH,CRITICAL .
2024-03-18T17:55:53.052-0400	INFO	Misconfiguration scanning is enabled
2024-03-18T17:55:54.316-0400	INFO	Detected config files: 1

charts/nginx/templates/pod.yaml (helm)

Tests: 65 (SUCCESSES: 64, FAILURES: 1, EXCEPTIONS: 0)
Failures: 1 (HIGH: 1, CRITICAL: 0)

HIGH: Container 'nginx' of Pod 'nginx' should set 'securityContext.readOnlyRootFilesystem' to true
══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
An immutable root file system prevents applications from writing to their local disk. This can limit intrusions, as attackers will not be able to tamper with the file system or write foreign executables to disk.

See https://avd.aquasec.com/misconfig/ksv014
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 charts/nginx/templates/pod.yaml:8-13
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   8 ┌   - name: nginx
   9 │     image: nginx:1.14.2
  10 │     ports:
  11 │     - containerPort: 8080
  12 │     securityContext:
  13 └       readOnlyRootFilesystem: false
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

```