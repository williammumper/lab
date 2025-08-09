# Section 2 Notes

## Services

Services connect applications together. For example, one group could be running frontend, one running backend, one connecting to a data source. Services are what enable connectivity between the groups of pods.
Example: say you're trying to connect to a k8s pod from your computer. Both your computer and the node you're connecting to share the same IP range, but the pod you're interested in connecting to has a different IP range. Your options are to curl into the pod, but this is still inside the node. We want to access the server without sshing in. This is what services allow us to do.
A service is an object, like a pod, or a deployment, or a replica set. The service lives inside the node, and forwards requests from the node to the pods.
There are a couple types of nodes:
- NodePort
- ClusterIP 
- LoadBalancer
We're going in depth on NodePort.
NodePort can help us map a node on a port to a node on a pod...
So you would be connecting from the port (ex:80) from the service to the TargetPort (ex:80) on the pod.
The service has its own IP address in the node.
There is also the NodePort - the port address for the whole node. Valid ranges for the NodePort are 30,000 to 32,767.
To create a service, we need a definition file.

An example service-definition.yml would look like:

```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service

spec:
  type: NodePort
  ports:
   - targetPort: 80
     port: 80
     nodePort: 3008
  selector:
     app: myapp
     type: front-end
```
Notes: port is the only spec necessary. If no TargetPort is assigned, it is assumed to be the same as port. Also, if nodePort isn't assigned, it is randomly assigned between the range of possible values, from 30,000 to 23,767.
Also, make sure there is a - below ports: , since the ports section is an array. You can have multiple port mappings within a single service.
It is common to use labels and selectors to identify pods in k8s.

To start a service, run:

```
kubectl create -f service-definition.yml
```

Assuming the name of your file is service-definition.yml

To see the list of services, run:

```
kubectl get services
```

To access the web server, for the example of having IP 192.168.1.2 and the node having port 300008:

```
curl https://192.168.1.2:30008
```

This is for a service mapped to a single pod. When there are multiple pods, they all have the same label (myapp) and the service distributes load across all pods with the alogrithm Random. There is also SessionAffinity.
When there are multiple pods across multiple nodes, k8s automatically creates a service that functions across nodes, meaning it adds all the pods to a cluster. Thus, you can use the IP of any node in the cluster, and using the same port number for all nodes.

### Services Cluster IP

Take the example of having multiple pods running front-end, back-end, and redis. How do we create connectivity between the three?
Each pod has an IP.
The IPs are not static though, and when a pod goes down one is regenerated. So IPs are not static.
Also, what if there are many pods for back-end. What if a front-end pod needs to access a back-end pod? How is that handled?
Luckily, services can help group the pods together and route traffic randomly similar to the node traffic service described in the previous video.
This way, we can expand or collapse sections of these groups without having to worry about impacting functionaility elsewhere.
Each service gets a name assigned to it, which is the name that is used by other pods to access the service. It is known as Cluster Service IP.

For example:

```
apiVersion: v1
kind: Service
metadata:
  name: back-end

spec:
  type: ClusterIP
  ports:
   - targetPort: 80
     port: 80

  selector:
     app: myapp
     type: back-end
```
You can create this service using the same commands as described in the previous video.


### Service Load Balancer

Another type of service is the LoadBalancer service.
Take the example of a voting app. This is a four node cluster.
Taking the front-end nodes, there is a group of pods hosted on a node in a cluster. Here we have two groups of pods, (nodes). One node is for the voting-app, and one node is for the result-app.
You can access the application by getting any of the nodes and the high port with the services exposed on them. So here, it would be four combinations of IPs for the voting app and four combinations of IPs for the result app.
But, we want end users to be able to visit a URL and see the web app. One way to route traffic is to install a VM that can run nginx (for example) to route traffic. This can be tedious.
If we're using AWS or google cloud or Azure, we can use the native load balancer, which k8s has support for in configuring our load balancing.
To make your service Load Balancing, just change the type under spec to LoadBalancer instead of NodePort. This works with GCP AWS and Azure.

---

## Namespaces

Take the case of having two houses with two Marks. In the first house, the Smiths, everyone addresses each other by their first names. The other family is the Williams. To address the Mark in the Williams house, the father of the Smith house would call his son Mark and the other Mark 'Mark Williams'. These are essentially namespaces in k8s.
Creating pods, deployments, and clusters is all done within a namespace.
The namespace is known as the Defualt namespace and is created by k8s on startup.
k8s also has its own internal namespace, for example the DNS server and networking space. This is in the namespace kube-system.
A third namespace created by k8s automatically is kube-public.
When you're using k8s for a larger project or enterprise, having a larger namespace is usually a good idea. For exapmple, having a Dev environment is a good idea to not mess with anything in the Prod environment (or namespace).
You can also set a certain smaller amount of resource use per namespace, guarunteeing that a namespace does not use more than an allotted limit of whatever resource you have determined. 
The services within a namespace can refer to each other simply by their names. So in default, the webapp pod can connect to the db service using simply "db-service". Say we want to connect to a pod in a separate service. We can do this by specifiying the other namespace - for example, mysql.connect('db-srvice.dev.svc.cluster.local) would connect to the dev db-service. Cluster.local is the default domain name of the k8s cluster. svc is the subdomain for the service, and dev is the namespace, and the actual service itself is db-service.
If we want to get the pods in a different namespace, we can use 

```
kubectl get pods --namespace=kube-system

```

Essentially, modify most common kubectl commands with --namespace=(target namespace). If you want to default make a new pod go to a specific namespace, under the metadata section in your pod-definition.yml file, add namespace: (namespace).
To create a new namespace, use a yml file:

```
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

Then run
```
kubectl create -f namespace dev

```

Above example is for dev namespace.
To swap namespaces, like to swap to the dev namespace, run:

```
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

This should switch you to the dev environment. Swapping to prod is the same process.
To create a resource quota, you can create a compute-quota.yml file where the kind is ResourceQuota, the namespace is specficied, and under spec, it looks like:


```
...
spec:
  hard:
    pods: '10'
    requests.cpu: '4'
    requests.memory: 5Gi
    limits.cpu: '10'
    limits.memory: 10Gi
```

### Imperative vs. Declarative


Imperative and declarative are different ways of getting to the same place.
Imperative is like saying what to do at each turn to a taxi driver to get to a location.
Declarative is like specifiying the specific location and we're expecting the system to get to the destination.
In the IaC world, imperative is like provisioning a VM named 'web-server' and installing NGINX, setting port 8080, and, editing the config file to web path '/var/www/nginx/', loading the web pages from the GIT repo - X and then starting the NGINX server. Convoluted even by how uncomplicated it is.
For declarative, we're saying MAKE a VM named web-server, with package nginx, using port 8080, through /var/www/nginx, and using code from GIT Repo - X.
This is what Terraform and Ansible do.
Ideally you should be able to update from the declarative statement without needing to redo the whole system.
In k8s, the imperative way of doing stuff is using kubectl run etc for each pod you need to make.
For example, using the kubectl replace command, using the kubectl delete command, all of which are convoluted and take more time than is neccesary.
Declarative is like using the kubectl apply -f command, since it looks at the existing config and checks what changes need to be made to the system.
Imperative commands are really bad for working in large environments where working with multiple people requires that we know what commands were run and when and by whom.
The declarative approach is way easier to use as an admin. Using the same objects, we use the kubectl apply command to create an object if it doesn't exist. You can specifiy a directory instead of single file as well. Updating objects applies regardless of whether or not the object exists.
Thus, using apply is best practice.

### Kubectl Apply

This command checks the local file, the last applied config, and the kubernetes definition. So if the object doesn't exist, the object is created, and kubernetes fills out some extra info internally. When you use kubectl appy, the yml is converted to json and is saved in the last applied configuration. So if we update the local file, the last applied config gets updated, and the kubernetes version also gets updated. If something gets deleted in the local file and kubernetes also has that line deleted, the last applied configuration could have that line saved. Only kubectl apply keeps this info.

---