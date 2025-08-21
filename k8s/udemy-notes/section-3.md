# Section 3 Notes
---
## Manual Scheduling

How does a scheduler work? Every pod starts with a nodeName: which is not set by default. The k8s algorithm looks through the pods and finds pods without names to schedule. It then finds the proper node for the pod, then sends that pod to a node by naming it something like node02, binding it to the second node.

If there is no scheduler, what happens? The pods stay in a pending state. Then, you can manually schedule the pod. The easiest way to do this is to create a name for the pod. You can only specify the node name at creation time. If the pod is already created, you can create a binding object then send that to the pods binding API. This looks like:

```
Pod-bind-definition.yaml

apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02

# Then send to node:

curl --header "Content-type:application/json" --request POST --data '{"apiVersion":"v1", "kind":"Binding"...}' http://$SERVER/api/v1/namespaces/pods/$PODNAME/binding
```

So, you convert the definition file to json, then send it through a curl request to bind the proper node name to the pod.

## Labels and Selectors

Labels and selectors are a standard way of grouping things together.
Labels are ways of identifiying objects. For the example of colors of animals, we might want to label based on class, kind, and color. These would be labels.
Selectors help us select between these objects. For example, Class=Mammal filters for only animals that are mammals. Combinding with Color=Green, we get green mammals.
Keywords and labels are *everywhere* from YouTube shorts to k8s.
We might want to be able to view everything in k8s by type, or by application, or by functionality. We attach lables as per need to objects. Then, we filter to find specific objects, for example app=App1. To select the pod with the labels, use, for example,

```
kubectl get pods --selector app=App1
```

This finds all the pods with the label App1.
To create a replica set with 3 pods, we want to make the replica set first, then apply the labels for the replica set. We want the label under the replica set field to match the labels defined on the pods. This looks like, for example, both the replica set and the pods having the same app: name. On creation of the replica set, if the labels match, the replica set is created. A service works the same way. Annotations in the definition file can hold on to a version number.

## Taints and Tolerations

In this example we don't want a bug to land on a person, so we apply a taint (bugspray) to the person, which the bug is intolerant of. Some bugs are tolerant to that smell, so the taint doesn't affect them.
So the determining factors for whether a bug can land on a person are the persons taint and the bugs tolerance level. 
The person is a node and the bug is a pod. Taints and tolerations are what pods can be scheduled on a node.
For a setup with nodes 1 2 and 3, and pods a b c and d, at the start there are no limitations. Thus the scheduler places the pods according to random assignment. Lets say we don't want any pods on node 1, we apply a taint that says no nodes, we're calling it blue. Here, no pods can tolerate blue so no unwanted pods are placed on the node. Next, we can specify that we only want pod d for example to be tolerant, so we add a toleration to pod d.
With this setup, the scheduler tries to place a pod on node 1 but is thrown off due to the taint, leading it to node 2, then the rest of the pods are placed according to the schedulers, taints and tolerations. By the end, only pod d ends up in node 1 since it is the only pod with the tolerant for the taint blue.
Note: in our example the scheduler places pod a in node 2 and b in node 3, the next available node. Which means with a taint, the scheduler is going to fill out the nodes based on min number of pods in each node, rather than just filling up node 2 because of the taint on node 1.
To set a taint:

```
kubectl taint nodes node-name key=value:taint-effect
```

taint-effect determines what happens to the pods if they DO NOT tolerate the taint.
There are THREE taint effects:
- NoSchedule - pods are not scheduled on this node
- PreferNoSchedule - system tries to avoid placing pods on node but not guaranteed
- NoExecute - new pods are not scheduled

Example taint:

```
kubectl taint nodes node1 app=blue:NoSchedule
```

Under spec in pod-definitions add:

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx

  tolerations:
  - key: "app"
    operator: "Equal1"
    value: "blue"
    effect: "NoSchedule"
```

For the example of having nodes 1 2 and 3 and pods a b c and d, with node 1 containing c and d and node 2 containing a and node 3 containing b, if we are to add a NoExecute taint to node 1, pod c gets killed and stops running, while pod d stays running, since we configuered it to have a toleration to the taint blue.
Also, adding a taint means the scheduler won't schedule any pods to a node unless it has the specific taint toleration. 
Note: the master node has a taint that prevents any pods from being scheduled on the node.
Best practice is to not deploy workloads on a master server.

## Node Selectors

Example: starting with a 3 node cluster, with one large node and two smaller nodes. We want to route a certain dev pod to node 1, since that is the only node with enough processing power to handle that pods' requirements. Under the default selector, this pod can end up anywhere, so it ends up in node 3. In order to solve this, we can set a node selector on the pod, which is the simplist method. The pod definition file can be edited such that the pod gets routed to the correct pod:

```
pod-definition.yaml

kind: Pod
metadata:
  name: mypod-app
spec:
  containers:
  - name: data-processor
    image: data-processor

  nodeSelector:
    size: Large
```

Size 'Large' comes from the labels assigned to nodes. Therefore, the nodes labeled 'Large' allow us to use node selection to send pods to those nodes.
To label:

```
kubectl label nodes <node-name> <label-key>=<label-value>
```

Example for setting to 'Large':

```
kubectl label nodes node-1 size=Large
```

Now that we have the node, create the pod, and the node selector will route the pod to the correct node (or nodes).
What about the case where we have a large or medium node, or any node that is not small. We need node affinity for this.

## Node Affinity

If we want to have OR or NOT selectors for node selection, we need node affinity.
The yaml for setting a pod definition to go to large nodes using node affinity is:

```
pod-definition.yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor

affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: size
          operator: In
          values:
          - Large
```

Awesome. Say we wanted to also let this pod go to nodes with size value 'medium' as well. We then want to add that to the values section:

```
...
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - Large
            - Medium
```

Or conversely if we wanted to have the pod not go in a small node, we can say:

```
...
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: NotIn
            values:
            - Small
```

This acts as a NOT operator on node affinity.
There are a number of other operators as well, documentation has the list.
If there is a label on a node that no longer exists, what happens? The answer is based on the *type* of node affinity.

The two types of node affinity (as of the udemy course release) are:

- requiredDuringSchedulingIgnoredDuringExecution
- prefferedDuringSchedulingIgnoredDuringExecution

DuringScheduling is the state in which a pod does not exist and is created for the first time.
So, lets say we want to schedule a pod with affinity to large, but none of our nodes have that label. Our type 1 affinity requires that the label exists during pod creation, or else the pod is not scheduled to any nodes. In the case with type 2, or preffered, then if no label matching the affinity is found, our scheduler will simply place the pod on any node according to the scheduler.
Next, lets look at how DuringExecution behaves. If an admin removes the 'Large' label from our node, then for both types of affinity, the change is 'ignored'. That is, changing the label after the pod has been created won't unschedule the pod from that node.
In the future there is a plan for a third type of affinity:

- requiredDuringSchedulingRequiredDuringExecution

This type of affinity means any pods running this will be evicted from nodes that do not match the affinity rules. So if our admin decides to change the label on the node, our pod using this type of affinity will be evicted (or terminated. Bottom line: it stops running).

## Node Affinity vs. Taints and Tolerations

I was wondering about this.
Here, we have three nodes and three pods. The pods are blue, red, and green. We want to place the blue pod in the blue node, the red pod in the red node, etc. There are other pods and other nodes on the same setup as well. We don't want any other pods on our node or vice versa. How is this done?
Using taints and tolerations: we apply a taint to the nodes to mark them with their colors, then we wet a specific toleration on the pods to tolerate those taints. When pods are created, the nodes accept only pods with the correct toleration. This, however, doesn't garuntee the pods will only prefer these nodes. How do we fix this?
Solving using node affinity: we can label the nodes with their respective colors, then set nodeSelectors on the pods to tie them to the nodes. Thus, the pods end up on the right nodes. This setup doesn't garuntee that other pods won't be scheduled on these nodes. 
Therefore, the best solution to this case is to use taints and tolerations in combination with nodeAffinity in order to keep the nodes and pods tied together.
So, using both methods in conjunction is enough to prevent specific nodes and pods from being scheduled elsewhere.

## Resource Limits

In determining how many resources a node gets, kube-scheduler determines how resources are divvied. If two nodes are filled up and don't have the resources to place another pod on them, the scheduler will place the pod on a new node. If there are no resources available, the pod will be held back from scheduling, in which case you will get an error saying Insufficient cpu (and a pending state on the pod). You are able to specify how much CPU and MEM you want to dedicate to a pod. This is known as a resource request for a container. You can set the pod yaml to request a node with the correct available resources as such:

```
pod-defintion.yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    resources:
	 requests:
	   memory: 4Gi
	   cpu: 2
```

This setup looks for a node with at least 4Gi of memory and 2 cpu cores available.
1 count for a cpu is equivalent to 1 vCPU core (AWS), 1 GCP core, 1 Azure core, you get the idea. You can always get more cores too.
The minimum MEM is 256M or G for gigabyte. Gi is a gibibyte.
! NOTE: A Gigabyte (G) is 1,000,000,000 bytes. A Gi is a gibibyte, which is 1,073,741,824 bytes.
Same goes for using M versus Mi.
A pod can use as much resources as it wants on a node. This can suffocate the other native processes on the nodes. You can set a limit to each pod on a node which means the pod only gets to consume a certain amount of resources. Same goes for memory.
To do this, add a 'limits' section to your yaml:

```
...
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    resources:
      requests:
        memory: 1Gi
        cpu: 1
    limits:
      memory: 2Gi
      cpu: 2
```

This makes the pod capable of being scheduled on any node with 1Gi of memory and 1 CPU and it can use up to 2Gi and 2 cpus of that node.
Keep in mind, there can be multiple containers in a pod as well, which can each have resources distributed as such.
If a CPU tries to go beyond the limit, the system throttles the CPU. For memory, a container can use more memory than its limit, if this pod uses too much memory consistently, the pod gets terminated. That is what OOM stands for.
Kubernetes does not have a CPU or memory request or limit set, by default.
So, in default Kubernetes, with no limits set, one pod on a node can use the entirety of the resources without allowing another pod to have any access to resources. This is without requests or limits set.
Next, the case with no requests set but limits set. Each pod can use up to their limit which means all pods should be able to run on the same node. Here, requests = limits.
Next, the case of limits and requests. Pods are not scheduled unless they can get access to the right amount of resources. But in the case where a pod needs more resources and the other pod does not but both have limits set such that they both get access to the same amount, it is not ideal.
In this case, we use requests but no limits. This means each pod gets as much resources as they request with no limits. This is actually ideal.
An example of limits in action are the KodeKloud labs, they are hosted on containers with limits set such that the public does not over use the resources.
Make sure you have set requests for all the pods, so no pod accidentally uses all the resources.

Working with memory, the setups are similar.
No resources no limits means one pod can use all the resources, starving the other pod.
No resources but limits means both pods get access to an equal or set amount of resources but one might want more than the other.
Resources and limits set means both pods get a minimum of resources but cannot dynamically scale if more resources are needed.
Finally, resources with no limits is ideal, as a pod can get more mem as needed that is available. Downside: say pod1 is using up most of the memory and pod2 wants more of it, the only way to free up enough memory for pod2 is to kill pod1.

How do we ensure each pod created has a resource value set?
We can use a limit-range yaml file:

```
limit-range-cpu.yaml
---
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default:
      cpu: 500m
    defaultRequest:
      cpu: 500m
    max:
      cpu: 1
    min:
      cpu: 100m
    type: Container
```

This applies at the namespace level. So if you're in default, all containers are going to have a default limit set of 500m for the CPU and a max of 1. This is an object. Max refers to the maximum limit that can be set on a container. min is the minimum request a pod can make. These are examples not necessarily good values. Same yaml definition file works for memory.

How about a total consumption for a whole kubernetes cluster?
You can create quotas at a namespace level for this. For example:

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: 4
    requests.memory: 4Gi
    limits.cpu: 10
    limits.memory: 10Gi
```

This hard sets the maximum number of CPUs requested and memory per pod to 4 and 4Gi. The limit of total CPUS across the pods is 10 and 10Gi for memory.

## Note on Editing Pods and Deployments

You cannot edit the specifications of a pod except as follows:
- spec.containers\[\*].image
- spec.initContainers\[\*].image
- spec.activeDeadlineSeconds
- spec.tolerations

This means you can't edit the environment variables, service accounts, or resource limits of a running pod.
If you really want to you can using

```
kubectl edit pod <pod-name>
```

This won't save the pod, but it will save a temporary file with the specified edits made to the pod. You can then bring the pod down using:

```
kubectl delete pod <pod-name>
```

And afterwards use the temp file to bring it back using: (note the exact dir location is given after running the edit command)

```
kubectl create -f/tmp/kubectl-edit-<xxxxx>.yaml
```

Alternatively, you can edit the yaml of the existing file:

```
kubectl get pod <pod-name> -o yaml > my-new-pod.yaml
```

Then:

```
vim my-new-pod.yaml
```

Delete the existing pod and bring the new one up using kubectl create on the new file.

You can easily edit any existing deployment using:

```
kubectl edit deployment <deployment-name>
```

Any edits made to a deployment will kill and restart the pods in the deployment. If asked to edit the pod part of a deployment, you can use the above command.


---

## Lab Tips:

To check all objects under a specific environment: (example prod)

```
kubectl get all --selector env=prod
```

Checking the node status for a pod can be done with:

```
kubectl describe pods -o wide
```