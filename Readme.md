# Kubernetes CheatSheet
## Cluster
* Cluster Information Comamnds.
```
$ kubectl version                           # get current kubernetes cluster version
$ kubectl clusterinfo                       # display addresses of the master and services
$ kubectl clusterinfo dump                  # dump current cluster state to stdout
$ kubectl get componentstatuses             # check cluster components health status
```
* Contexts Commands.
```
$ kubectl config get-contexts                                           # get all contexts
$ cat ~/.kube/config                                                    # get all contexts
$ kubectl config view -o=jsonpath='{.current-context}'                  # view current context
$ kubectl config current-context                                        # view current context
$ kubectl config set-context my-context --namespace=my-namespace        # set new context in a specific namespace
$ kubectl config use-context my-context                                 # use/swtich to specific context
$ kubectl config delete-context my-context                              # delete cluster context
```
* Namespace Commands. 
``` 
$ kubectl get namespace                                                                 # get all namespace
$ kubectl get namespaces -o=jsonpath='{range .items[*].metadata.name}{@}{"\n"}{end}'    # get all namespace
```
## Nodes
* Get Commands.
```
$ kubectl get node                                                                              # get all worker nodes
$ kubectl get node --selector='!node-role.kubernetes.io/master'                                 # get all worker nodes
$ kubectl get node -o wide                                                                      # get all nodes with detailed information
$ kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}' # get External Ips of all nodes
```
* Describe Commands.
```
$ kubectl describe nodes my-node                # describe node with verbose output
$ 
```
* Patch Commands.
```
$ kubectl patch node <my-node-name> -p '{"spec":{"unschedulable":true}}'    # partially update a node
```
* Delete Commands.
```
$ kubectl delete node my-node                                           # delete node with a given name
```
* Troubleshooting and Inteacting with Nodes
```
## make node scheduable/unscheduable
$ kubectl cordon my-node                                                # mark my-node as unschedulable
$ kubectl drain my-node                                                 # drain my-node in preparation for maintenance
$ kubectl uncordon my-node                                              # mark my-node as schedulable

## metrics
$ kubectl top node my-node                                              # show metrics for a given node

## check node status
$ JSONPATH='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' \
 && kubectl get nodes -o jsonpath="$JSONPATH" | grep "Ready=True"
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
## Deployments
* Create Commands.
```
$ kubectl apply -f deployment.yaml                   # create a deployment from yaml file
$ kubectl create deployment nginx --image=nginx  # start a single instance of nginx
```
* Get Commands.
```
$ kubectl get deploy                                   # get all deployments
$ kubectl get deployment                               # get all deployments
$ kubectl get deployment my-deployment                 # get a particular deployment
```
* Describe Commands.
```
$ kubectl describe deploy my-deployment                 # describe deployment with verbose output
```
* Patch Commands.
```
$ kubectl patch deployment valid-deployment  --type json   -p='[{"op": "remove", "path": "/spec/template/spec/containers/0/livenessProbe"}]'               # Disable a deployment livenessProbe using a json patch with positional arrays
```
* Update/Rollout Commands.
```
$ kubectl set image deployment/frontend www=image:v2               # rolling update "www" containers of "frontend" deployment, updating the image
$ kubectl rollout history deployment/frontend                      # check the history of deployments including the revision 
$ kubectl rollout undo deployment/frontend                         # rollback to the previous deployment
$ kubectl rollout undo deployment/frontend --to-revision=2         # rollback to a specific revision
$ kubectl rollout status -w deployment/frontend                    # watch rolling update status of "frontend" deployment until completion
$ kubectl rollout restart deployment/frontend                      # rolling restart of the "frontend" deployment

```
* Scale Commands.
```
$ kubectl scale --current-replicas=2 --replicas=3 deployment/mysql       # If mysql deployment current size is 2, scale mysql to 3
$ kubectl scale --replicas=2 deployment/mysql                            # scal mysql deployment to 2 irrespective of what its current size is
$ kubectl autoscale deployment my-deployment --min=2 --max=10                      # Auto scale a deployment with min=2 and max=10
```
* Delete Commands.
```
$ kubectl delete deploy --all                               # delete all deployments
$ kubectl delete deployment --all                           # delete all deployments
$ kubectl delete deploy my-deployment                       # delete a particular depoyment
```
* Troubleshooting and Interacting with Deployments Commands.
```
## port-forward
$ kubectl port-forward deploy/my-deployment 5000:6000       # listen on local port 5000 and forward to port 6000 on a Pod created by my-deployment

## access/execute deployments
$ kubectl exec deploy/my-deployment -- ls                   # run command in first Pod and first container in Deployment (single- or multi-container cases)
$ kubectl exec -it deploy/my-deployment bash                # execute a particular deployment

## logs
$ kubectl logs deploy/my-deployment                         # dump Pod logs for a Deployment (single-container case)
$ kubectl logs deploy/my-deployment -c my-container         # dump Pod logs for a Deployment (multi-container case)

```
## Service
* Create Commands.
```
$ kubectl apply -f service.yaml                            # create a service from yaml
```
* Get Commands.
```
$ kubectl get services                              # get all services in the namespace
$ kubectl get svc                                   # get all services in the namespace
$ kubectl get svc my-service                        # get a particular service
$ kubectl get services --sort-by=.metadata.name     # get Services Sorted by Name
 
```
* Describe Commands.
```
$ kubectl describe svc my-service                   # describe a particular service
```
* Delete Commands.
```
$ kubectl delete svc my-service                     # delete a particular service
$ kubectl delete svc --all                          # delete all services
```

* Troubleshooting and Interacting with Service Commands.
```
## port forward
$ kubectl port-forward svc/my-service 5000                 # listen on local port 5000 and forward to port 5000 on Service backend
$ kubectl port-forward svc/my-service 5000:my-service-port  # listen on local port 5000 and forward to Service target port with name my-servic-port

```



