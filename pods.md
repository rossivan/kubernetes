$ kubectl get pods
No resources found in default namespace.


# get pods across all namespaces
$ kubectl get pods -A
NAMESPACE            NAME                                         READY   STATUS    RESTARTS       AGE
kube-system          coredns-7db6d8ff4d-7cwk2                     1/1     Running   5 (66m ago)    362d
kube-system          coredns-7db6d8ff4d-j7wwv                     1/1     Running   5 (66m ago)    362d
kube-system          etcd-kind-control-plane                      1/1     Running   1 (66m ago)    5d9h
kube-system          kindnet-5xtg8                                1/1     Running   5 (66m ago)    362d
kube-system          kindnet-l85g4                                1/1     Running   23 (66m ago)   362d
kube-system          kindnet-xrskk                                1/1     Running   5 (66m ago)    362d
kube-system          kube-apiserver-kind-control-plane            1/1     Running   1 (66m ago)    5d9h
kube-system          kube-controller-manager-kind-control-plane   1/1     Running   5 (66m ago)    362d
kube-system          kube-proxy-cgnqb                             1/1     Running   5 (66m ago)    362d
kube-system          kube-proxy-ppdkb                             1/1     Running   5 (66m ago)    362d
kube-system          kube-proxy-tbvg9                             1/1     Running   5 (66m ago)    362d
kube-system          kube-scheduler-kind-control-plane            1/1     Running   5 (66m ago)    362d
local-path-storage   local-path-provisioner-988d74bc-lgsln        1/1     Running   9 (65m ago)    362d


#
$ kubectl get pods -l key=<value>


# Create a pod the wrong way
$ kubectl run --image=nginx:1.26.0 -n ${NAMESPACE} created-the-wrong-way


$ kubectl port-forward -n 04--pod created-the-wrong-way 9000:80
Forwarding from 127.0.0.1:9000 -> 80
Forwarding from [::1]:9000 -> 80
Handling connection for 9000
Handling connection for 9000


Pod.nginx-minimal.yaml
======================
apiVersion: v1
kind: Pod
metadata:
  name: nginx-minimal
spec:
  containers:
    - name: nginx
      image: nginx:1.26.0


$ kubectl apply -n ${NAMESPACE} -f Pod.nginx-minimal.yaml


$ kubectl port-forward -n 04--pod nginx-minimal 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80


# Pod.nginx-better.yaml
# Based on https://github.com/BretFisher/podspec
apiVersion: v1
kind: Pod
metadata:
  name: nginx-better
  namespace: 04--pod
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


$ kubectl apply -n ${NAMESPACE} -f Pod.nginx-better.yaml


$ kubectl port-forward -n ${NAMESPACE} nginx-better 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080


$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
created-the-wrong-way   1/1     Running   0          26m
nginx-better            1/1     Running   0          2m49s
nginx-minimal           1/1     Running   0          24m


$ kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
fluentd-minimal-przvq   1/1     Running   0          77s   10.244.1.13   kind-worker    <none>           <none>
fluentd-minimal-vsht4   1/1     Running   0          77s   10.244.2.7    kind-worker2   <none>           <none>


# deleting the namespace, deletes all resource within that namespace
$ kubectl delete -f Namespace.yaml


$ kubectl delete pod nginx-minimal-l84l4
pod "nginx-minimal-l84l4" deleted


# delete all pods
$ kubectl delete pods --all
pod "nginx-minimal-hxc6c" deleted
pod "nginx-minimal-n58nw" deleted
pod "nginx-minimal-t2h5m" deleted


$ watch "kubectl get pods"