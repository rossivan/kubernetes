Service.nginx.yaml
==================
apiVersion: v1
kind: Service
metadata:
  name: nginx # singular since it points to a single cluster IP
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80


$ kubectl apply -f Service.nginx.yaml


Service.nginxs.yaml
===================
apiVersion: v1
kind: Service
metadata:
  name: nginxs # Plural because it sets up DNS for each replica in the StatefulSet (e.g. nginx-0.nginxs.default.svc.cluster.local)
spec:
  type: ClusterIP
  clusterIP: None # This makes it a "headless" service
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80


$ kubectl apply -f Service.nginxs.yaml


StatefulSet.nginx-with-init-container.yaml
==========================================
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-with-init-conainer
spec:
  serviceName: nginxs
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      initContainers:
        - name: populate-default-html
          image: nginx:1.26.0
          # Nginx is a silly example to use for a stateful application (you should use a deployment for nginx)
          # but this demonstrates how you can use an init container to pre-populate a pod specific config file
          # For example, you might configure a database StatefulSet with some pods having read/write access, and
          # others only providing read access.
          #
          # See: https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/
          command:
            - bash
            - "-c"
            - |
              set -ex
              [[ $HOSTNAME =~ -([0-9]+)$ ]] || exit 1
              ordinal=${BASH_REMATCH[1]}
              echo "<h1>Hello from pod $ordinal</h1>" >  /usr/share/nginx/html/index.html
          volumeMounts:
            - name: data
              mountPath: /usr/share/nginx/html
      containers:
        - name: nginx
          image: nginx:1.26.0
          volumeMounts:
            - name: data
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: "standard"
        resources:
          requests:
            storage: 100Mi



$ kubectl apply -f StatefulSet.nginx-with-init-container.yaml


$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
nginx-with-init-conainer-0   1/1     Running   0          87s
nginx-with-init-conainer-1   1/1     Running   0          81s
nginx-with-init-conainer-2   1/1     Running   0          78s


$ kubectl port-forward nginx-with-init-conainer-0 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
Handling connection for 8080

$ kubectl port-forward nginx-with-init-conainer-1 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
Handling connection for 8080

$ kubectl port-forward nginx-with-init-conainer-2 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
Handling connection for 8080