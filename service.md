Deployment.yaml
===============

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-minimal
  labels:
    foo: deployment-label
  annotations:
    bar: deployment-annotation
spec:
  replicas: 3
  selector:
    matchLabels:
      baz: pod-label
  template:
    metadata:
      labels:
        baz: pod-label
      annotations:
        bing: pod-annotation
    spec:
      containers:
        - name: nginx
          image: nginx:1.26.0


$ kubectl apply -f Deployment.yaml


Service.nginx-clusterip.yaml
============================
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
  labels:
    foo: service-label
  annotations:
    bar: service-annotation
spec:
  type: ClusterIP # This is the default value
  selector:
    baz: pod-label
  ports:
    - protocol: TCP
      port: 80 # Port the service is listening on
      targetPort: 80 # Port the container is listening on (if unset, defaults to equal port value)


$ kubectl apply -f Service.nginx-clusterip.yaml


$ kubectl get service
NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
nginx-clusterip   ClusterIP   10.96.148.84   <none>        80/TCP    6m37s


Service.nginx-nodeport.yaml
===========================
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    baz: pod-label
  ports:
    - protocol: TCP
      port: 80 # Port the service is listening on
      targetPort: 80 # Port the container is listening on (if unset, defaults to equal port value)
      # nodePort: 30XXX (if unset, kubernetes will assign a port within 30000-32767)


$ kubectl apply -f Service.nginx-nodeport.yaml


$ kubectl get svc
NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-clusterip   ClusterIP   10.96.148.84   <none>        80/TCP         10m
nginx-nodeport    NodePort    10.96.5.200    <none>        80:31698/TCP   2m29s


Service.nginx-loadbalancer.yaml
===============================
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  type: LoadBalancer # Will only work if cluster is configured to provision one from an external source (e.g. cloud provider)
  selector:
    baz: pod-label
  ports:
    - protocol: TCP
      port: 80 # Port the service is listening on
      targetPort: 80 # Port the container is listening on (if unset, defaults to equal port value)

$ kubectl apply -f Service.nginx-loadbalancer.yaml


$ kubectl get svc
NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-clusterip      ClusterIP      10.96.148.84    <none>        80/TCP         13m
nginx-loadbalancer   LoadBalancer   10.96.130.166   <pending>     80:31909/TCP   2m25s
nginx-nodeport       NodePort       10.96.5.200     <none>        80:31698/TCP   6m8s


# while its nice that it create an IP address for us that would be all that useful from a service discovery standpoint
# so there's another service running within our cluster called 'coredns' within the 'kube-system' namespace and what this
# does it looks at all these services and sets up DNS that resolves internal to the cluster so that based on the service
# name and the namespace we can create a DNS name that will route us to that service
$ kubectl get pods --namespace kube-system
NAME                                         READY   STATUS    RESTARTS       AGE
coredns-7db6d8ff4d-7cwk2                     1/1     Running   5 (13h ago)    362d  # DNS
coredns-7db6d8ff4d-j7wwv                     1/1     Running   5 (13h ago)    362d  # DNS
...


# to demo this create a temporary pod (works within the same namespace, but won't work from a different namespace)
$ kubectl run curl-demo-pod -it --rm --image=curlimages/curl --command -- sh
If you don't see a command prompt, try pressing enter.
~ $ curl nginx-clusterip:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>


# try from a different namespace, does not work
# if we want to make calls across namespaces we have to use the service name followed by the namespace
# where the service lives followed by svc.cluster.local i.e. <service-name>.<namespace>.svc.cluster.local.
# so, even though we are making the call from the 'default' namespace by using that FQDN coredns is going to
# resolve it to the service in the appropriate namespace
# this is very important as you work with services and want your applications to address each other within the cluster
$ kubectl run curl-demo-pod -it --rm -n default --image=curlimages/curl --command -- sh
If you don't see a command prompt, try pressing enter.
~ $ curl nginx-clusterip:80
curl: (6) Could not resolve host: nginx-clusterip
~ $ curl nginx-clusterip.04--service.svc.cluster.local
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>


# if we did nothing the LoadBalancer service would be in pending forever because kind out of the box 
# does not have support for LoadBalance service
$ kubectl get svc
NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-clusterip      ClusterIP      10.96.148.84    <none>        80/TCP         13m
nginx-loadbalancer   LoadBalancer   10.96.130.166   <pending>     80:31909/TCP   2m25s
nginx-nodeport       NodePort       10.96.5.200     <none>        80:31698/TCP   6m8s


# so run sigs.k8s.io/cloud-provider-kind@latest to enable load balancer services with KinD
$ task kind:03-run-cloud-provider-kind
task: [kind:03-run-cloud-provider-kind] sudo cloud-provider-kind
[sudo] password for rossi: rossivan


# this is not truly an external IP but will be accessible from my laptop
$ kubectl get svc
NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-clusterip      ClusterIP      10.96.148.84    <none>        80/TCP         66m
nginx-loadbalancer   LoadBalancer   10.96.130.166   172.18.0.5    80:31909/TCP   55m
nginx-nodeport       NodePort       10.96.5.200     <none>        80:31698/TCP   58m


# connecting to LoadBalance on IP 172.18.0.5 does not work

# that EXTERNAL-IP is the giveaway: 172.18.0.5 is a private, internal cluster/network address, not 
# something your laptop can reach directly. Here’s what’s going on:
# Why you can’t access it?
# 172.18.0.5 is in the RFC1918 private range (172.16.0.0/12)
# it’s likely assigned by your container runtime network (e.g., Docker bridge if you’re using something like 
# Kind, Minikube, or Docker Desktop)
# that IP exists inside the Kubernetes node network, not on your host machine’s network Your laptop has no 
# route to that subnet → connection fails. so even though Kubernetes labels it “LoadBalancer,” it’s not a 
# real external IP in your environment.

# need to port forward to the LoadBalancer service and connect to it at http://localhost:8080/
$ kubectl port-forward svc/nginx-loadbalancer 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
Handling connection for 8080


# loadbalancer service in kind cluster stuck terminating. resolution?

# why this happens in kind?
# LoadBalancer services don’t work natively in kind (since there’s no real cloud provider).
# Tools like MetalLB simulate this, but if something crashes or is misconfigured, the cleanup 
# finalizer never finishes—so the Service gets stuck in Terminating.

# check the service
kubectl get svc <service-name> -n <namespace> -o yaml

# look for something like
metadata:
  finalizers:
    - service.kubernetes.io/load-balancer-cleanup

# remove the finalizer (force cleanup)
kubectl patch svc <service-name> -n <namespace> \
  -p '{"metadata":{"finalizers":[]}}' --type=merge

# if it’s still stuck, force delete
kubectl delete svc <service-name> -n <namespace> --grace-period=0 --force