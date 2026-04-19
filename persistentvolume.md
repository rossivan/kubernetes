# persistent volume and persistent volume claim are kubernetes interface for creating, managing and consuming storage
# that lives beyond the life of an individual pod

# access modes
# one of the single most attribute of the storage are the access modes

# ReadWriteOnce - can mount this storage device with a single pod in a rw mode. so it already mounted to a pod and you
# try to mount it to another that second one would fail.
# slight nuance is that you could have the storage mounted by multiple mode in rw fashion if they all were on the same
# mode because its assocaited with which node the container is running on.
# so there is new version of it - ReadWriteOncePod - which actually limits it to a single pod regardless of whether or
# not the other pods are on the same node

# ReadOnlyMany - can have multiple pods mount the storage but only in read only fashion

# ReadWriteMany - can have multiple pods across multiple nodes all mounting this volume and reading and writing to it at
# the same time

# another important configuration spec if the reclaim policy for the storage class. this is whether or not when the persistent
# volume claim is deleted what should happen to the underlying persistent volume. in the 'retain' case that persistent volume
# and therefore the disk that it provisioned or the volume it provisioned in the cloud would remain versus if its set to
# 'delete' then persistent volume would be deleted when persistent volume claim is deleted

# you can provision persistent volume and persistent volume claims directly or within a statefulset you could provide a
# template that will provision them dynamically

# manualy provision a persistent volume and persistent volume claim
# this storage class was created by the kind cluster using the rancher.io/local-path provisioner
$ k get storageclasses
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  363d


PersistentVolume.manual-kind.yaml
=================================
apiVersion: v1
kind: PersistentVolume
metadata:
  name: manual-kind-worker
  labels:
    name: manual-kind
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 100Mi
  storageClassName: standard
  local:
    path: /some/path/in/container # Replace with the path to your local storage
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - kind-worker
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: manual-kind-worker2
  labels:
    name: manual-kind
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 100Mi
  storageClassName: standard
  local:
    path: /some/path/in/container # Replace with the path to your local storage
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - kind-worker2


$ kubectl apply -f kind/PersistentVolume.manual-kind.yaml


PersistentVolumeClaim.manual-pv-kind.yaml
=========================================
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: manual-pv-kind
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
  selector:
    matchLabels:
      name: manual-kind
  storageClassName: standard


$ kubectl apply -f kind/PersistentVolumeClaim.manual-pv-kind.yaml


Pod.manual-pv-and-pvc-kind.yaml
===============================
apiVersion: v1
kind: Pod
metadata:
  name: manual-pv-and-pvc
spec:
  containers:
    - name: nginx
      image: nginx:1.26.0
      volumeMounts:
        - name: storage
          mountPath: /some/mount/path
  volumes:
    - name: storage
      persistentVolumeClaim:
        claimName: manual-pv-kind


$ kubectl apply -f kind/Pod.manual-pv-and-pvc-kind.yaml


$ kubectl get pods
NAME                READY   STATUS    RESTARTS   AGE
manual-pv-and-pvc   1/1     Running   0          5s


$ kubectl get pods -o wide
NAME                READY   STATUS    RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
manual-pv-and-pvc   1/1     Running   0          3m44s   10.244.1.15   kind-worker   <none>           <none>


$ kubectl get pv
NAME                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                                 STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
manual-kind-worker    100Mi      RWO            Retain           Bound       04--persistentvolume/manual-pv-kind   standard       <unset>                          29s
manual-kind-worker2   100Mi      RWO            Retain           Available                                         standard       <unset>                          29s


$ kubectl get pvc
NAME             STATUS   VOLUME               CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
manual-pv-kind   Bound    manual-kind-worker   100Mi      RWO            standard       <unset>                 20s


$ kubectl exec -it ma
nual-pv-and-pvc -- bash
root@manual-pv-and-pvc:/# ls
bin  boot  dev  docker-entrypoint.d  docker-entrypoint.sh  etc  home  lib  lib64  media  mnt  opt  proc  product_uuid  root  run  sbin  some  srv  sys  tmp  usr  var


# to dynamically provision persistent volume claim and the underlying storage

PersistentVolumeClaim.dynamic-pv-kind.yaml
==========================================
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pv-kind
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
  storageClassName: standard


$ kubectl apply -f kind/PersistentVolumeClaim.dynamic-pv-kind.yaml


Deployment.shared-pvc-kind.yaml
===============================
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shared-pvc-kind
spec:
  # Both pods can only run simultaneously on the same node because
  # PVC has accessModes: [ReadWriteOnce]. Would be better to demo this
  # on multi-node cluster
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.26.0
          volumeMounts:
            - name: data
              mountPath: /some/mount/path
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: dynamic-pv-kind


$ kubectl apply -f kind/Deployment.shared-pvc-kind.yaml


$ kubectl get deployments
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
shared-pvc-kind   2/2     2            2           3m49s


$ kubectl get rs
NAME                         DESIRED   CURRENT   READY   AGE
shared-pvc-kind-65fc8c8c9b   2         2         2       3m52s


# note that the 2 pods has to be scheduled on the same worker node in order to mount the same persistent volume
$ kubectl get pods -o wide
NAME                               READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
manual-pv-and-pvc                  1/1     Running   0          20m   10.244.1.15   kind-worker    <none>           <none>
shared-pvc-kind-65fc8c8c9b-7wdjs   1/1     Running   0          88s   10.244.2.16   kind-worker2   <none>           <none>
shared-pvc-kind-65fc8c8c9b-vps6n   1/1     Running   0          88s   10.244.2.15   kind-worker2   <none>           <none>


$ kubectl get pvc
NAME              STATUS   VOLUME                CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
dynamic-pv-kind   Bound    manual-kind-worker2   100Mi      RWO            standard       <unset>                 95s
manual-pv-kind    Bound    manual-kind-worker    100Mi      RWO            standard       <unset>                 20m


$ kubectl get pv
NAME                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                  STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
manual-kind-worker    100Mi      RWO            Retain           Bound    04--persistentvolume/manual-pv-kind    standard       <unset>                          20m
manual-kind-worker2   100Mi      RWO            Retain           Bound    04--persistentvolume/dynamic-pv-kind   standard       <unset>                          20m


# this is important difference as to how deployment and statefulset work with persistent volumes. in the case of a
# deployment if you specify a persistent volume claim all of the replicase are going to mount the same one. in the case
# of a statefulset each replica will get an independent persistent volume claim


StatefulSet.individual-pvcs-kind.yaml
=====================================
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: individual-pvcs-kind
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx-sts-kind
  template:
    metadata:
      labels:
        app: nginx-sts-kind
    spec:
      containers:
        - name: nginx
          image: nginx:1.26.0
          ports:
            - containerPort: 80
          volumeMounts:
            - name: data
              mountPath: /some/mount/path
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 50Mi
        storageClassName: "standard"
---
apiVersion: v1
kind: Service
metadata:
  name: individual-pvcss
spec:
  clusterIP: None
  selector:
    app: nginx-sts-kind


$ kubectl apply -f kind/StatefulSet.individual-pvcs-kind.yaml


$ kubectl get statefulset
NAME                   READY   AGE
individual-pvcs-kind   2/2     108s


$ kubectl get pods -o wide
NAME                               READY   STATUS    RESTARTS   AGE    IP            NODE           NOMINATED NODE   READINESS GATES
individual-pvcs-kind-0             1/1     Running   0          113s   10.244.1.19   kind-worker    <none>           <none>
individual-pvcs-kind-1             1/1     Running   0          109s   10.244.2.18   kind-worker2   <none>           <none>
manual-pv-and-pvc                  1/1     Running   0          36m    10.244.1.15   kind-worker    <none>           <none>
shared-pvc-kind-65fc8c8c9b-7wdjs   1/1     Running   0          17m    10.244.2.16   kind-worker2   <none>           <none>
shared-pvc-kind-65fc8c8c9b-vps6n   1/1     Running   0          17m    10.244.2.15   kind-worker2   <none>           <none>


$ kubectl get pvc
NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
data-individual-pvcs-kind-0   Bound    pvc-2f583080-aca9-4818-9616-068bc54660a7   50Mi       RWO            standard       <unset>                 2m1s
data-individual-pvcs-kind-1   Bound    pvc-67d12ad2-04cd-4587-bc8b-2ffcea117db3   50Mi       RWO            standard       <unset>                 117s
dynamic-pv-kind               Bound    manual-kind-worker2                        100Mi      RWO            standard       <unset>                 17m
manual-pv-kind                Bound    manual-kind-worker                         100Mi      RWO            standard       <unset>                 36m


$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                              STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
manual-kind-worker                         100Mi      RWO            Retain           Bound    04--persistentvolume/manual-pv-kind                standard       <unset>                          36m
manual-kind-worker2                        100Mi      RWO            Retain           Bound    04--persistentvolume/dynamic-pv-kind               standard       <unset>                          36m
pvc-2f583080-aca9-4818-9616-068bc54660a7   50Mi       RWO            Delete           Bound    04--persistentvolume/data-individual-pvcs-kind-0   standard       <unset>                          2m8s
pvc-67d12ad2-04cd-4587-bc8b-2ffcea117db3   50Mi       RWO            Delete           Bound    04--persistentvolume/data-individual-pvcs-kind-1   standard       <unset>                          2m4s


$ kubectl get svc
NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
individual-pvcss   ClusterIP   None         <none>        <none>    2m40s