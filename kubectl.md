# 
$ kubectl explain Namespace


#
$ kubectl explain Namespace.metadata


#
$ kubectl logs <pod_name>
$ kubectl logs deployment/<deployment_name>


#
$ kubectl exec -it <pod_name> -c <container_name> -- bash
$ kubectl debug -it <pod_name> --image=<debug_image> -- bash


#
$ kubectl port-forward <pod_name> <local_port>:<pod_port>
$ kubecel port-forward svc/<deployment_name> <local_port>:<pod_port>


#
$ kubectl run curl-demo-pod -it --rm --image=curlimages/curl --command -- sh


# session ended, resume using command when the pod is running
$ kubectl attach curl-demo-pod -c curl-demo-pod -i -t