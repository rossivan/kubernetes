ReplicaSet.nginx-minimal.yaml
=============================
apiVersion: apps/v1
kind: ReplicaSet
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


$ kubectl apply -f ReplicaSet.nginx-minimal.yaml


$ kubectl get replicaset
NAME            DESIRED   CURRENT   READY   AGE
nginx-minimal   3         3         3       18s
$ kubectl get rs
NAME            DESIRED   CURRENT   READY   AGE
nginx-minimal   3         3         3       18s


$ kubectl get pods
NAME                  READY   STATUS    RESTARTS   AGE
nginx-minimal-hxc6c   1/1     Running   0          56s
nginx-minimal-l84l4   1/1     Running   0          56s
nginx-minimal-t2h5m   1/1     Running   0          56s