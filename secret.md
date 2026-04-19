Secret.string-data.yaml
=======================
apiVersion: v1
kind: Secret
metadata:
  name: string-data
type: Opaque
stringData:
  foo: bar


# Apply Secret using stringData field
$ kubectl apply -f Secret.string-data.yaml


$ kubectl get secret
NAME          TYPE     DATA   AGE
string-data   Opaque   1      10s


$ kubectl describe secret string-data
Name:         string-data
Namespace:    04--secret
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
foo:  3 bytes


$ kubectl get secret string-data -o yaml | yq
apiVersion: v1
data:
  foo: YmFy
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Secret","metadata":{"annotations":{},"name":"string-data","namespace":"04--secret"},"stringData":{"foo":"bar"},"type":"Opaque"}
  creationTimestamp: "2026-04-18T19:53:57Z"
  name: string-data
  namespace: 04--secret
  resourceVersion: "2292789"
  uid: 37440cf9-8a40-4e09-b8a6-d861753fef7b
type: Opaque


$ kubectl get secret string-data -o yaml | yq '.data'
foo: YmFy

$ kubectl get secret string-data -o yaml | yq '.data.foo'
YmFy

$ kubectl get secret string-data -o yaml | yq '.data.foo' | base64 -d
bar


# base64 encode a string. use printf instead of base64. you could use 'echo -n'
printf "bar" | base64
YmFy
echo "bar" | base64
YmFyCg==


Secret.base64-data.yaml
=======================
apiVersion: v1
kind: Secret
metadata:
  name: base64-data
type: Opaque
data:
  foo: YmFy


$ kubectl apply -f Secret.base64-data.yaml


$ k get secret
NAME          TYPE     DATA   AGE
base64-data   Opaque   1      2m32s
string-data   Opaque   1      14m


$ kubectl get secret base64-data -o yaml | yq '.data.foo' | base64 -d
bar


# there are other types of secrets beyond opaque. for the generic case where you are going to store a password
# opaque type is going to be right. there is also a docker config file json type secret. this is used by kubernetes
# to authenticate to a container registry to pull container images

Secret.dockerconfigjson.yaml
============================
apiVersion: v1
kind: Secret
metadata:
  name: dockerconfigjson
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: |
    eyJhdXRocyI6eyJodHRwczovL2luZGV4LmRvY2tlci5pby92MSI6eyJ1c2VybmFtZSI6InVzZXJuYW1lIiwicGFzc3dvcmQiOiJwYXNzd29yZCIsImVtYWlsIjoiZm9vQGJhci5jb20iLCJhdXRoIjoiZFhObGNtNWhiV1U2Y0dGemMzZHZjbVE9In19fQ==


$ echo "eyJhdXRocyI6eyJodHRwczovL2luZGV4LmRvY2tlci5pby92MSI6eyJ1c2VybmFtZSI6InVzZXJuYW1lIiwicGFzc3dvcmQiOiJwYXNzd29yZCIsImVtYWlsIjoiZm9vQGJhci5jb20iLCJhdXRoIjoiZFhObGNtNWhiV1U2Y0dGemMzZHZjbVE9In19fQ==" | base64 -d | jq
{
  "auths": {
    "https://index.docker.io/v1": {
      "username": "username",
      "password": "password",
      "email": "foo@bar.com",
      "auth": "dXNlcm5hbWU6cGFzc3dvcmQ="
    }
  }
}


# rather than construct the above by hand use the kubectl command to create the secret
$ kubectl create secret docker-registry dockerconfigjson \
    --docker-email=foo@bar.com \
    --docker-username=username \
    --docker-password=password \
    --docker-server=https://index.docker.io/v1/


$ kubectl get secret
NAME               TYPE                             DATA   AGE
base64-data        Opaque                           1      12m
dockerconfigjson   kubernetes.io/dockerconfigjson   1      4s
string-data        Opaque                           1      24m


$ kubectl describe secret dockerconfigjson
Name:         dockerconfigjson
Namespace:    04--secret
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/dockerconfigjson

Data
====
.dockerconfigjson:  143 bytes


$ kubectl get secret dockerconfigjson -o yaml | yq
apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6eyJodHRwczovL2luZGV4LmRvY2tlci5pby92MS8iOnsidXNlcm5hbWUiOiJ1c2VybmFtZSIsInBhc3N3b3JkIjoicGFzc3dvcmQiLCJlbWFpbCI6ImZvb0BiYXIuY29tIiwiYXV0aCI6ImRYTmxjbTVoYldVNmNHRnpjM2R2Y21RPSJ9fX0=
kind: Secret
metadata:
  creationTimestamp: "2026-04-18T20:18:26Z"
  name: dockerconfigjson
  namespace: 04--secret
  resourceVersion: "2295138"
  uid: cbe78f61-280a-4278-843a-90850edc914c
type: kubernetes.io/dockerconfigjson


# could store the output in a repository as opposed to creating it imperatively
$ kubectl create secret docker-registry dockerconfigjson \
    --docker-email=foo@bar.com \
    --docker-username=username \
    --docker-password=password \
    --docker-server=https://index.docker.io/v1/ --dry-run=client -o yaml
apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6eyJodHRwczovL2luZGV4LmRvY2tlci5pby92MS8iOnsidXNlcm5hbWUiOiJ1c2VybmFtZSIsInBhc3N3b3JkIjoicGFzc3dvcmQiLCJlbWFpbCI6ImZvb0BiYXIuY29tIiwiYXV0aCI6ImRYTmxjbTVoYldVNmNHRnpjM2R2Y21RPSJ9fX0=
kind: Secret
metadata:
  creationTimestamp: null
  name: dockerconfigjson
type: kubernetes.io/dockerconfigjson