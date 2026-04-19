# rbac grants applications (or users) access to the kubernetes api.
# access could be granted cluster wide or by namespace

$ kubectl get ServiceAccount
NAME      SECRETS   AGE
default   0         80s     # the default service account has access to the default namespace


Job.no-permissions.yaml
=======================
apiVersion: batch/v1
kind: Job
metadata:
  name: no-permissions
spec:
  template:
    spec:
      containers:
        - name: kubectl
          image: cgr.dev/chainguard/kubectl
          args: ["get", "pods", "-A"]
      restartPolicy: Never
  backoffLimit: 1


$ kubectl apply -f Job.no-permissions.yaml


$ kubectl get pods
NAME                   READY   STATUS   RESTARTS   AGE
no-permissions-grgl5   0/1     Error    0          39s
no-permissions-vjwcd   0/1     Error    0          25s


# the error suggests the default service account is not allowed to run the 'kubectl get pods -A' command in the non-default
# namespace
$ kubectl logs no-permissions-grgl5
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:04--rbac:default" cannot list resource "pods" in API group "" at the cluster scope


# the error suggests the default service account is not allowed to run the 'kubectl get pods -A' command in the non-default
# namespace
$ kubectl logs no-permissions-vjwcd
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:04--rbac:default" cannot list resource "pods" in API group "" at the cluster scope


# so the below steps suggests how we permission the default service account

ServiceAccount.namespaced-pod-permissions.yaml
==============================================
apiVersion: v1
kind: ServiceAccount
metadata:
  name: namespaced-pod-reader


$ kubectl apply -f ServiceAccount.namespaced-pod-permissions.yaml


Role.pod-reader.yaml
====================
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]


$ kubectl apply -f Role.pod-reader.yaml


RoleBinding.pod-reader.yaml
===========================
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
subjects:
  - kind: ServiceAccount
    name: namespaced-pod-reader


$ kubectl apply -f RoleBinding.pod-reader.yaml


Job.namespaced-pod-reader-succeed.yaml
======================================
apiVersion: batch/v1
kind: Job
metadata:
  name: namespaced-pod-permissions-succeed
spec:
  template:
    spec:
      automountServiceAccountToken: true
      containers:
        - name: kubectl
          image: cgr.dev/chainguard/kubectl
          args: ["get", "pods", "-n", "04--rbac"]
      serviceAccountName: namespaced-pod-reader
      restartPolicy: Never
  backoffLimit: 1


$ kubectl apply -f Job.namespaced-pod-reader-succeed.yaml


Job.namespaced-pod-reader-fail.yaml
===================================
apiVersion: batch/v1
kind: Job
metadata:
  name: namespaced-pod-permissions-fail
spec:
  template:
    spec:
      automountServiceAccountToken: true
      containers:
        - name: kubectl
          image: cgr.dev/chainguard/kubectl
          args: ["get", "pods", "-A"]
      serviceAccountName: namespaced-pod-reader
      restartPolicy: Never
  backoffLimit: 1


$ kubectl apply -f Job.namespaced-pod-reader-fail.yaml


$ kubectl get pods
NAME                                       READY   STATUS      RESTARTS   AGE
namespaced-pod-permissions-fail-kfj9l      0/1     Error       0          6m39s     # does not have cluster wide permissions
namespaced-pod-permissions-fail-ph7dg      0/1     Error       0          6m50s
namespaced-pod-permissions-succeed-q4gn4   0/1     Completed   0          6m50s


# how to provide for cluster role permissions

ServiceAccount.cluster-pod-permissions.yaml
===========================================
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cluster-pod-reader


$ kubectl apply -f ServiceAccount.cluster-pod-permissions.yaml


ClusterRole.pod-reader.yaml
===========================
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]


$ kubectl apply -f ClusterRole.pod-reader.yaml


ClusterRoleBinding.pod-reader.yaml
==================================
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: pod-reader
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: pod-reader
subjects:
  - kind: ServiceAccount
    name: cluster-pod-reader
    namespace: 04--rbac


$ kubectl apply -f ClusterRoleBinding.pod-reader.yaml


Job.cluster-pod-reader.yaml
===========================
apiVersion: batch/v1
kind: Job
metadata:
  name: cluster-pod-permissions
spec:
  template:
    spec:
      automountServiceAccountToken: true
      containers:
        - name: kubectl
          image: cgr.dev/chainguard/kubectl
          args: ["get", "pods", "-A"]
      serviceAccountName: cluster-pod-reader
      restartPolicy: Never
  backoffLimit: 1

  
$ kubectl apply -f Job.cluster-pod-reader.yaml