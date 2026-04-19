DaemonSet.fluentd-minimal.yaml
==============================
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-minimal
  namespace: 04--daemonset
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
        - name: fluentd
          image: fluentd:v1.16-1


$ kubectl apply -f DaemonSet.fluentd-minimal.yaml


$ kubectl get ds
NAME              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
fluentd-minimal   2         2         2       2            2           <none>          6s


$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
fluentd-minimal-przvq   1/1     Running   0          54s
fluentd-minimal-vsht4   1/1     Running   0          54s


# daemonset is expected to run an instance on each node
$ kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
fluentd-minimal-przvq   1/1     Running   0          77s   10.244.1.13   kind-worker    <none>           <none>
fluentd-minimal-vsht4   1/1     Running   0          77s   10.244.2.7    kind-worker2   <none>           <none>


$ kubectl get nodes
NAME                 STATUS   ROLES           AGE    VERSION
kind-control-plane   Ready    control-plane   362d   v1.30.0
kind-worker          Ready    <none>          362d   v1.30.0
kind-worker2         Ready    <none>          362d   v1.30.0