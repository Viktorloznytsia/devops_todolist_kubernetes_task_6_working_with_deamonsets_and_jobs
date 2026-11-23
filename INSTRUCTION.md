# Task 6: Working with DaemonSets and Jobs

## 1. Deploy to the cluster

The DaemonSet and the CronJob call the ClusterIP service `todoapp-service`
(namespace `todoapp`), so the application must be deployed first.
Apply the manifests in this order:

```
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/clusterIp.yml
kubectl apply -f .infrastructure/mateapp-namespace.yml
kubectl apply -f .infrastructure/daemonset.yml
kubectl apply -f .infrastructure/cronjob.yml
```

- `namespace.yml`, `deployment.yml`, `clusterIp.yml` create the `todoapp` namespace,
  the ToDo application and the ClusterIP service `todoapp-service`.
- `mateapp-namespace.yml` creates the `mateapp` namespace. The `daemonset.yml` and
  `cronjob.yml` manifests belong to this namespace.
- The pods in `mateapp` reach the service from another namespace by the DNS name
  `todoapp-service.todoapp` (`<service>.<namespace>`).

## 2. Validate the solution

Check that the DaemonSet pod is running (one pod per node) and that the CronJob exists:

```
kubectl get daemonset,pods -n mateapp
kubectl get cronjob -n mateapp
```

### DaemonSet logs

The DaemonSet runs `curl` against the ClusterIP service every 5 seconds:

```
kubectl logs daemonset/curl-daemonset -n mateapp
```

Expected result: the line `Health OK` is repeated, a new line every 5 seconds.

### CronJob logs

The CronJob runs every 4 minutes (`*/4 * * * *`), so wait until the next minute
divisible by 4. Then list the jobs and read the logs of one of them:

```
kubectl get jobs,pods -n mateapp
kubectl logs job/<job-name> -n mateapp
```

Replace `<job-name>` with the name from the first command, for example
`curl-cronjob-29924108`. Expected result: `Health OK`, and the job is `Complete 1/1`.

The CronJob keeps 10 successful and 5 failed jobs in history
(`successfulJobsHistoryLimit: 10`, `failedJobsHistoryLimit: 5`) and has
`concurrencyPolicy: Allow`.
