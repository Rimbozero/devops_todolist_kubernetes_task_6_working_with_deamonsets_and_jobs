# Deploy and validate the Todo app workloads

The DaemonSet and CronJob run in the `mateapp` namespace and call the existing
ClusterIP service `todoapp-service` in the `todoapp` namespace. Ensure that the
Todo app service is deployed and has ready endpoints before applying these
manifests.

## Deploy

Create the namespace and apply both manifests:

```sh
kubectl create namespace mateapp --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f daemonset.yml
kubectl apply -f cronjob.yml
```

Check that the workloads are ready and the CronJob is scheduled:

```sh
kubectl get daemonset,cronjob -n mateapp
kubectl get pods -n mateapp -o wide
```

## Validate

The DaemonSet makes a request to the Todo app service every five seconds. Follow
its logs to see the HTTP status for each request:

```sh
kubectl logs -n mateapp -l app=todoapp-curl --all-containers=true --follow
```

The CronJob calls `/api/health` every four minutes. List its Jobs and Pods, then
inspect the logs of a run (replace `<job-name>` with a name from the `jobs`
output):

```sh
kubectl get jobs -n mateapp
kubectl get pods -n mateapp
kubectl logs -n mateapp job/<job-name>
```

Successful requests print an HTTP status such as `HTTP 200`. Also verify that
the CronJob has the requested schedule and retained run counts:

```sh
kubectl get cronjob todoapp-health -n mateapp
kubectl describe cronjob todoapp-health -n mateapp
```

The CronJob starts every four minutes, so a new Job may not be available
immediately after deployment.
