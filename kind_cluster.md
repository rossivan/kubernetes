kind-config.yaml
================
# three node (two workers) cluster config
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
    extraMounts:
      - hostPath: /mnt/c/Users/rossi/my_interests/my_programs/kubernetes/devops-directive-kubernetes-course/03-installation-and-setup/kind-bind-mount-1
        containerPath: /some/path/in/container
  - role: worker
    extraMounts:
      - hostPath: /mnt/c/Users/rossi/my_interests/my_programs/kubernetes/devops-directive-kubernetes-course/03-installation-and-setup/kind-bind-mount-2
        containerPath: /some/path/in/container


# create a Kubernetes cluster using kind
kind create cluster --config kind-config.yaml


# delete an existing kind Kubernetes cluster
$ kind delete cluster