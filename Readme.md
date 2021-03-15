# Kubernetes CheatSheet

## Cluster
* Check cluster version.
```
$ kubectl version
```
* Get cluster Information.
```
$ kubectl clusterinfo

$ kubectl clusterinfo dump
```
* Check cluster components health status.
```
$ kubectl get componentstatuses
```
* List, create, view, switch and delete cluster contexts.
```
## list all contexts
$ kubectl config get-contexts 
or 
$ cat ~/.kube/config

## view current context
$ kubectl config view -o=jsonpath='{.current-context}'
or
$ kubectl config current-context

## set new context in a specific namespace
$ kubectl config set-context <my-context-name> --namespace=<my-namespace>

## use/swtich to specific context
$ kubectl config use-context <my-context-name>

## delete cluster context
$ kubectl config delete-context <my-context-name>
```
* List all namespace in a cluster. 
``` 
$ kubectl get namespace
or 
$ kubectl get namespaces -o=jsonpath='{range .items[*].metadata.name}{@}{"\n"}{end}'
```

