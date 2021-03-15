# Kubernetes CheatSheet
## Cluster
* Check cluster version.
```
$ kubectl version
```
* Get cluster Information.
```
# Display addresses of the master and services
$ kubectl clusterinfo

# Dump current cluster state to stdout
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
## Nodes
* Get all worker nodes.
```
$ kubectl get node
or 
$ kubectl get node --selector='!node-role.kubernetes.io/master'

## with detailed information
$ kubectl get node -o wide
```
* Mark node as unscheduable.
```
$ kubectl cordon <my-node-name>
```
* Drain node in preparation for maintenance.
```
$ kubectl drain <my-node-name>
```
* Mark node as schedulable.
```
$ kubectl uncordon <my-node-name>
```
* show metrics of given node.
```
$ kubectl top node <my-node-name>
```
* Delete node.
```
$ kubectl delete node <my-node-name>
```
* Get external Ips of nodes.
```
$ kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'
```
* check which nodes are ready.
```
$ JSONPATH='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' \
 && kubectl get nodes -o jsonpath="$JSONPATH" | grep "Ready=True"

```
* Describe node with verbose output.
```
$ kubectl describe node <my-node-name>
```
* Partially update a node.
```
$ kubectl patch node <my-node-name> -p '{"spec":{"unschedulable":true}}'
```