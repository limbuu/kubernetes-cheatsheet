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

## Pods
* Create Commands.
```
$ kubectl apply -f pod.yaml                                          # create a pod from yaml file
$ kubectl run my-pod --image image-name                              # create a pod from image
$ kubectl run my-pod --image image-name --port port-number --expose  # create pod and export it as a service

```
* Get Commands.
```
$ kubectl get pods --all-namespaces                                     # get all pods in all namespaces
$ kubectl get pods -o wide                                              # get all pods in the current namespace, with more details
$ kubectl get pods                                                      # get all pods in the namespace
$ kubectl get pod my-pod -o yaml                                        # get a pod's YAML
$ kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'# get pods Sorted by Restart Count 
$ kubectl get pods --field-selector=status.phase=Running                # get all running pods in the namespace
```

* Describe Commands.
```
$ kubectl describe pods my-pod                # describe pod with verbose output
```

* Labels & Annotations.
```
$ kubectl label pods my-pod new-label=awesome                   # add label to a pod
$ kubectl annotate pods my-pod icon-url=http://goo.gl/XXBTWq    # dd a annotation to a pod
$ kubectl get pods --show-labels                                # show labels for all pods
```

* Patch commands.
```
$ kubectl patch pod valid-pod -p '{"spec":{"containers":[{"name":"kubernetes-serve-hostname","image":"new image"}]}}'           # update a container's image; spec.containers[*].name is required because it's a merge key

$ kubectl patch pod valid-pod --type='json' -p='[{"op": "replace", "path": "/spec/containers/0/image", "value":"new image"}]'    # update a container's image using a json patch with positional arrays

```

* Delete Commands.
```
$ kubectl delete pod my-pod                             # delete pod by name
$ kubectl delete pod my-pod --force --grace-period=0    # delete pod with grace shutdown period=0
$ kubectl delete pods,services abcd                     # delete pods/services with same names `abcd`
$ kubectl delete pods,svc --all -n my-namespace         # delete pods/service within same namespace
```

* Trobleshooting and Interacting pods.
```
## portforward
$ kubectl port-forward my-pod 5000:6000          # Listen on port 5000 on the local machine and forward to port 6000 on my-pod

## access/execute pods
$ kubectl exec -it my-pod bash                  # execute a pod
$ kubectl exec my-pod -- ls /                   # run commad in existing pod(1 container)
$ kubectl exec --stdin --tty my-pod -- /bin/sh  # interactive shell access to running pod(1 container)
$ kubectl exec my-pod -c my-container -- ls /   # run command in exsiting pod(multiple conatiner case)

## show metrics
$ kubectl top pod my-pod                           # Show metrics for a given pod
$ kubectl top pod my-pod --containers              # Show metrics for a given pod and its containers
$ kubect

## logs
$ kubectl logs my-pod                              # dump pod logs (stdout)
$ kubectl logs my-pod -c my-container              # dump pod container logs (stdout, multi-container case)
$ kubectl logs -f my-pod                           # stream pod logs (stdout)
$ kubectl logs -f my-pod -c my-container           # stream pod container logs (stdout, multi-container case)

## ContainerIds
$ kubectl get pods --all-namespaces -o jsonpath='{range .items[*].status.initContainerStatuses[*]}{.containerID}{"\n"}{end}' | cut -d/ -f3              # get all containerIDs of initContainer of all pods

## Secrets
$ kubectl get pods -o json | jq '.items[].spec.containers[].env[]?.valueFrom.secretKeyRef.name' | grep -v null | sort | uniq            # get all secrets currently in use by a pod

```


