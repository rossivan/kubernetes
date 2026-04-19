Pod.echo-date-minimal.yaml
==========================
apiVersion: v1
kind: Pod
metadata:
  name: echo-date-minimal
spec:
  containers:
    - name: echo
      image: busybox:1.36.1
      command: ["date"]
  restartPolicy: Never


$ kubectl apply -f Pod.echo-date-minimal.yaml


$ kubectl logs echo-date-minimal
Sat Apr 18 16:27:19 UTC 2026


Job.echo-date-minimal.yaml
==========================
apiVersion: batch/v1
kind: Job
metadata:
  name: echo-date-minimal
spec:
  template:
    spec:
      containers:
        - name: echo
          image: busybox:1.36.1
          command: ["date"]
      restartPolicy: Never
  backoffLimit: 1


$ kubectl apply -f Job.echo-date-minimal.yaml


$ kubectl logs echo-date-minimal-kqrcj
Sat Apr 18 17:08:38 UTC 2026


$ kubectl explain Job.spec.backoffLimit
GROUP:      batch
KIND:       Job
VERSION:    v1

FIELD: backoffLimit <integer>

DESCRIPTION:
    Specifies the number of retries before marking this job failed. Defaults to 6


Job.echo-date-better.yaml
=========================
apiVersion: batch/v1
kind: Job
metadata:
  name: echo-date-better
  namespace: 04--job
spec:
  parallelism: 2
  completions: 2
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


$ kubectl apply -f Job.echo-date-better.yaml


$ kubectl get jobs
NAME                STATUS     COMPLETIONS   DURATION   AGE
echo-date-better    Running    0/2           4s         4s
echo-date-minimal   Complete   1/1           3s         10m


$ kubectl get pods
NAME                      READY   STATUS      RESTARTS   AGE
echo-date-better-snhnv    0/1     Completed   0          30s
echo-date-better-v6bl6    0/1     Completed   0          30s
echo-date-minimal         0/1     Completed   0          51m
echo-date-minimal-kqrcj   0/1     Completed   0          10m


$ kubectl get jobs
NAME                STATUS     COMPLETIONS   DURATION   AGE
echo-date-better    Complete   2/2           4s         115s
echo-date-minimal   Complete   1/1           3s         11m


# for debugging purposes and forcing a cronjob run you could create an adhoc
# job from a cronjob i.e. manually create a job using a cronjob as the template
$ kubectl create job --from=cronjob/echo-date-better manually-triggered