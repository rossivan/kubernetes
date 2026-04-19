# kubernetes system components
$ kubectl get pods --namespace kube-system
NAME                                         READY   STATUS    RESTARTS       AGE
coredns-7db6d8ff4d-7cwk2                     1/1     Running   5 (13h ago)    362d  # DNS
coredns-7db6d8ff4d-j7wwv                     1/1     Running   5 (13h ago)    362d  # DNS
etcd-kind-control-plane                      1/1     Running   1 (13h ago)    5d21h # storage
kindnet-5xtg8                                1/1     Running   5 (13h ago)    362d
kindnet-l85g4                                1/1     Running   23 (13h ago)   362d
kindnet-xrskk                                1/1     Running   5 (13h ago)    362d
kube-apiserver-kind-control-plane            1/1     Running   1 (13h ago)    5d21h # API server
kube-controller-manager-kind-control-plane   1/1     Running   5 (13h ago)    362d  # controller manager
kube-proxy-cgnqb                             1/1     Running   5 (13h ago)    362d  # kube proxy
kube-proxy-ppdkb                             1/1     Running   5 (13h ago)    362d  # kube proxy
kube-proxy-tbvg9                             1/1     Running   5 (13h ago)    362d  # kube proxy
kube-scheduler-kind-control-plane            1/1     Running   5 (13h ago)    362d  # scheduler


