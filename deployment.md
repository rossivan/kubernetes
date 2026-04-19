Deployment.nginx-minimal.yaml
=============================
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-minimal
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-minimal
  template:
    metadata:
      labels:
        app: nginx-minimal
    spec:
      containers:
        - name: nginx
          image: nginx:1.26.0


$ kubectl apply -f Deployment.nginx-minimal.yaml


$ kubectl get deployments
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
nginx-minimal   3/3     3            3           20s


$ k get replicasets
NAME                       DESIRED   CURRENT   READY   AGE
nginx-minimal-6666ccb9cc   3         3         3       57s


Deployment.nginx-better.yaml
============================
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-better
  namespace: 04--deployment
  labels:
    app: nginx-better
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-better
  template:
    metadata:
      labels:
        app: nginx-better
    spec:
      containers:
        - name: nginx
          image: cgr.dev/chainguard/nginx:latest
          ports:
            - containerPort: 8080
              protocol: TCP
          readinessProbe:
            httpGet:
              path: /
              port: 8080
          resources:
            limits:
              memory: "50Mi"
            requests:
              memory: "50Mi"
              cpu: "250m"
          securityContext:
            allowPrivilegeEscalation: false
            privileged: false
      securityContext:
        seccompProfile:
          type: RuntimeDefault
        runAsUser: 1001
        runAsGroup: 1001
        runAsNonRoot: true


$ kubectl apply -f Deployment.nginx-better.yaml


$ kubectl get deployments
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
nginx-better    3/3     3            3           50s
nginx-minimal   3/3     3            3           5m46s


$ kubectl get replicasets
NAME                       DESIRED   CURRENT   READY   AGE
nginx-better-69848d549d    3         3         3       107s
nginx-minimal-6666ccb9cc   3         3         3       6m43s


$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
nginx-better-69848d549d-259b9    1/1     Running   0          2m30s
nginx-better-69848d549d-4lxnv    1/1     Running   0          2m30s
nginx-better-69848d549d-4zv95    1/1     Running   0          2m30s
nginx-minimal-6666ccb9cc-4snbq   1/1     Running   0          7m26s
nginx-minimal-6666ccb9cc-4w9mn   1/1     Running   0          7m26s
nginx-minimal-6666ccb9cc-8trmh   1/1     Running   0          7m26s


# rollout restart the pods in one of the deployments
$ kubectl rollout restart deployment nginx-better


$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
nginx-better-699f4d86f-dz6vg     1/1     Running   0          10s
nginx-better-699f4d86f-jzfwf     1/1     Running   0          11s
nginx-better-699f4d86f-lnbrj     1/1     Running   0          13s
nginx-minimal-6666ccb9cc-4snbq   1/1     Running   0          11m
nginx-minimal-6666ccb9cc-4w9mn   1/1     Running   0          11m
nginx-minimal-6666ccb9cc-8trmh   1/1     Running   0          11m


$ kubectl get replicasets
NAME                       DESIRED   CURRENT   READY   AGE
nginx-better-69848d549d    0         0         0       10m
nginx-better-699f4d86f     3         3         3       4m12s
nginx-minimal-6666ccb9cc   3         3         3       15m


$ kubectl rollout undo deployment nginx-better


$ kubectl get replicasets
NAME                       DESIRED   CURRENT   READY   AGE
nginx-better-69848d549d    3         3         3       12m
nginx-better-699f4d86f     0         0         0       6m17s
nginx-minimal-6666ccb9cc   3         3         3       17m


# for Deployment.nginx-minimal.yaml, if we change the image to nginx:1.27.0 and apply it, we see the pods for this replicaset get restarted


$ kubectl rollout restart deployment nginx-minimal


$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
nginx-better-69848d549d-h4klm    1/1     Running   0          2m43s
nginx-better-69848d549d-rrl57    1/1     Running   0          2m41s
nginx-better-69848d549d-w8hmn    1/1     Running   0          2m42s
nginx-minimal-84b948d666-6cr9j   1/1     Running   0          14s
nginx-minimal-84b948d666-dts97   1/1     Running   0          13s
nginx-minimal-84b948d666-hrmsj   1/1     Running   0          15s


$ kubectl get replicasets
NAME                       DESIRED   CURRENT   READY   AGE
nginx-better-69848d549d    3         3         3       17m
nginx-better-699f4d86f     0         0         0       10m
nginx-minimal-6666ccb9cc   0         0         0       22m
nginx-minimal-84b948d666   3         3         3       2m1s


$ kubectl get deployments
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
nginx-better    3/3     3            3           17m
nginx-minimal   3/3     3            3           22m