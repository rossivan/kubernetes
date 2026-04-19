# enables environment specific configuration to be decoupled from container images

ConfigMap.file-like-keys.yaml
=============================
apiVersion: v1
kind: ConfigMap
metadata:
  name: file-like-keys
data:
  conf.yml: |
    name: YourAppName
    version: 1.0.0
    author: YourName


$ kubectl apply -f ConfigMap.file-like-keys.yaml


ConfigMap.property-like-keys.yaml
=================================
apiVersion: v1
kind: ConfigMap
metadata:
  name: property-like-keys
data:
  NAME: YourAppName
  VERSION: 1.0.0
  AUTHOR: YourName


$ kubectl apply -f ConfigMap.property-like-keys.yaml


$ kubectl get cm
NAME                 DATA   AGE
file-like-keys       1      23s
kube-root-ca.crt     1      6m59s
property-like-keys   3      14s


$ k describe cm file-like-keys
Name:         file-like-keys
Namespace:    04--configmap
Labels:       <none>
Annotations:  <none>

Data
====
conf.yml:
----
name: YourAppName
version: 1.0.0
author: YourName


BinaryData
====

Events:  <none>


$ k describe cm property-like-keys
Name:         property-like-keys
Namespace:    04--configmap
Labels:       <none>
Annotations:  <none>

Data
====
AUTHOR:
----
YourName
NAME:
----
YourAppName
VERSION:
----
1.0.0

BinaryData
====

Events:  <none>


Pod.configmap-example.yaml
==========================
apiVersion: v1
kind: Pod
metadata:
  name: configmap-example
spec:
  containers:
    - name: nginx
      image: nginx:1.26.0
      volumeMounts:
        - name: configmap-file-like-keys
          mountPath: /etc/config
      envFrom:
        - configMapRef:
            name: property-like-keys
  volumes:
    - name: configmap-file-like-keys
      configMap:
        name: file-like-keys


$ kubectl apply -f Pod.configmap-example.yaml


# cat /etc/config/conf.yaml within the container
$ kubectl exec configmap-example -c nginx -- cat /etc/config/conf.yml
name: YourAppName
version: 1.0.0
author: YourName


# print the environment variables within the container
$ kubectl exec configmap-example -c nginx -- printenv
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=configmap-example
NGINX_VERSION=1.26.0
NJS_VERSION=0.8.4
NJS_RELEASE=2~bookworm
PKG_RELEASE=1~bookworm
AUTHOR=YourName
NAME=YourAppName
VERSION=1.0.0
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
HOME=/root