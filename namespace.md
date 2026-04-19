# Create a namespace in the cluster
kubectl create namespace 04--namespace-cli


Namespace.yaml
==============
apiVersion: v1
kind: Namespace
metadata:
  name: 04--namespace-file


# Apply the namespace configuration to the cluster
kubectl apply -f Namespace.yaml


# List all namespaces in the cluster
$ kubectl get namespaces
NAME                 STATUS   AGE
04--namespace-cli    Active   29s
04--namespace-file   Active   17s
default              Active   362d
kube-node-lease      Active   362d
kube-public          Active   362d
kube-system          Active   362d
local-path-storage   Active   362d


# Delete the namespace created using the CLI
kubectl delete namespace 04--namespace-cli


# Delete the namespace created using the YAML file
kubectl delete -f Namespace.yaml