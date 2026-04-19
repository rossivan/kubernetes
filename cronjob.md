CronJob.echo-date-better.yaml
=============================
apiVersion: batch/v1
kind: CronJob
metadata:
  name: echo-date-better
  namespace: 04--cronjob
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      parallelism: 1
      completions: 1
      activeDeadlineSeconds: 100
      backoffLimit: 1
      template:
        metadata:
          labels:
            app: echo-date
        spec:
          containers:
            - name: echo
              image: cgr.dev/chainguard/busybox:latest
              command: ["date"]
              resources:
                limits:
                  memory: "50Mi"
                requests:
                  memory: "50Mi"
                  cpu: "250m"
              securityContext:
                allowPrivilegeEscalation: false
                privileged: false
                runAsUser: 1001
                runAsGroup: 1001
                runAsNonRoot: true
          restartPolicy: Never
          securityContext:
            seccompProfile:
              type: RuntimeDefault


# job will be scheduled not at creation time but on cron schedule time
$ kubectl apply -f CronJob.echo-date-better.yaml


$ kubectl get cronjobs
NAME               SCHEDULE    TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
echo-date-better   * * * * *   <none>     False     0        <none>          56s


$ kubectl get jobs
NAME                        STATUS     COMPLETIONS   DURATION   AGE
echo-date-better-29608920   Complete   1/1           4s         9s


$ kubectl get pods
NAME                              READY   STATUS      RESTARTS   AGE
echo-date-better-29608920-vmf9h   0/1     Completed   0          12s


# for debugging purposes and forcing a cronjob run you could create an adhoc
# job from a cronjob i.e. manually create a job using a cronjob as the template
$ kubectl create job --from=cronjob/echo-date-better manually-triggered


$ kubectl get jobs
NAME                        STATUS     COMPLETIONS   DURATION   AGE
echo-date-better-29608927   Complete   1/1           4s         63s
echo-date-better-29608928   Complete   1/1           3s         3s
manually-triggered          Complete   1/1           4s         13s