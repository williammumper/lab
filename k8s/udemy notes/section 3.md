# Section 3 Notes

### Manual Scheduling

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
