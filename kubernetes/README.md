## What is Kubernetes
	
	Kubernetes is a containerized deployment management tool. It helps to manage 
	containerized workloads.
	Kubernetes is written in Go

## Kubernetes cluster Components

### Kubernetes Proxy
	kubernetes proxy routes network traffic to loas-balanced servie in cluster. 
	To do the job proxy must be preset on ever node in cluster

### Kubernetes DNS
	kubernetes runs a DNS server which provides naming and discovery for the services 
	that are defined in the cluster. This server also runs a replicated service on the
	cluster. Depending on the size of the cluster there might be mutiple DNS servers
	running.

	$ kubectl get deployments --namespace=kube-system kube-dns

	For this there is a kubernetes service running which performs the load balancing for 
	the DNS server.

	$ kubectl get services --namespace=kube-system kube-dns

### Kubernetes UI
	The final componet is a GUI service running. There is a single replicla managed by
	kubernetes depolyment fro reliability and upgrades.

	$ kubectl get deplyments --namespace=kube-system kubernetes-dashboard
	
	This shows the running dashbaord server
	
	$ kubectl get services --namesapce=kube-system kubernetes-dashboard

	To acess the UI launch kubectl proxy
	
	$ kubectl proxy
	
	Now you cam acess the dashboard at http://localhost:8001/ui


## kubectl

### Namespaces 
	Kubernetes uses namespaces to organize objects in the cluster. Namespace can be taken
	as a folder that holds a set of objects. That is in namespace kubectl can only acess
	objects defined in that namespace. By deafult the kubectl will interact with default
	namespace. To change namespace you can use.

	$ kubectl --namespace=mystuff

### Contexts 

	Kubernetes uses contexts to manage different cluster or different user for
	authenticating to those clusters. Conetext make changes in the .kube/config 
	file. 

	$ Kubectl config set-context my-context --namespace=mystuff
	
	This will create a context with a differen namespace

	to start using this context you can use
	
	$ kubectl config use-context my-context

### Kubernetes Objects

	Everything contained in kubernetes is represented by a RESTFul resource.These objects
	can be referenced using an simple API call. Each resource object exists at a differen
	t unique HTTP path for example, https://your-k8s.com/api/v1/namespaces/default/pods/ 
	my-pod. The Kubectl command converts the get command in a simple http request.

	The kubectl configures the repsonse and extracts the human redable important data 
	and presents that in more readble from. 
	To get more detailed version you can use -o flag or other flags to get detailed data
	-o json or -o yaml will represent the returned object in the specified format

	The kubectl uses Json path query laguage to extarct information from the returned 
	object. To get specific information you can use specified flags to get it

	$ kubectl get pods my-pod -o jsonpath --template={.status.podIP}

	To get the ip of that specific pod
	

### creating, Updating, and destroying Kuberneets Objects
	Objects in kubernetes API are represente as JSON or YAML files. These files are 
	either returned by the server in response to a query or posted to the server
	as a part of an API request. 

	To create an object just run the yaml file pertaining to it the typeo of object
	and other data will be taken from the file

	$ Kubectl apply -f obj.yaml

	To upadate or moidify also make changes and re run 
	
	$ kubectl apply -f obj.yaml

	to delete an object run
	
	$ kubectl delete -f obj.yaml
	
	Thsi will delete the specific object

### Labeling and Annotating Object
	$ kubectl label pods

### Debugging Commands
	
	$ kubectl log <pod-name> 
	
	This displays the logs for the current conatiner

	$ kubectl exec -i <pod-name> --bash
	
	This runs a interactive bash shell inside the running container 

	$ kubectl cp <pod-name>:/path/to/remote/file /path/to/local/file

	This can be used to copy fils to and fro from cluster to local

	kubectl help
	kubectl help command-name
	
	TO get info about the command
	 

## Pods 

	The pod is a group of whales. The whale is a symbol of docker so you get what pods
	represents.
	You put containers in pod and  most of the time a sigle container is run in a single
	pod but a case might arise where you run more than one container in a single pod as
	in both the container share a  common syncronized 

### What do appilcations running on same pod share
	The applications running on same pod share the same IP adress and port space.
	(network namespace) have same hostname and can communicate using interprocess 
	communication channels over system V, IPC etc

### When to put containers in same pods
	One question that you have to ask is will the containers work fine if they land up 
	on different machines. Other things to keep in mind is how will the container 
	scalling take place and are their symbiotic processes.

### Creating Pods
	The kubernetes API accepts the pod manifest and then stores that in the etcd storage
	the sheduler finds the pods that haven't been placed and then places them on machine
	which satisy the constraints. Kubernetes tries not to place multiple replica on same
	to provide a workout single machine failiure.

	There are two ways of creating pods that is by using replicaset deployment and other
	by pod Manifest.

### Deploying using Replicaset


### Deploying using manifest
	Pod manifets can be written uisng YAML or JSON. YAML is mostly prefered given the
	comment enabled and human readability.
	
	a breif example
	  apiVersion: v1
	  kind: pod
	  metadata:
	    name: kuard
	  spec:
	    containers:
	      - image: gcr.io/kaur-demo/kuard-amd64:1
	        name: kuard
	        ports:
		  - containerPort: 8080
	            name: http
	            protocol: TCP	

### Runnig Pods
	kubectl apply -f kuard-pod.yaml
	
	The Pod manifest will be submitted to kubernetes API server. The kubernets system
	will then schedule that pod to run on a healthy node in cluster. It wil be monitered
	by kubelet daemon process

	kubcetl get pods
	  to get pods

	kubectl descibe pods kuard
	  to get a detailed picture of the pod

	kubeclt delete pods/kuard 
	or
	kubcetl delete -f kuard-pod.yaml
	  to delete a pod

### accessing the pod
	
	Port forwarding

	  to expose a service to world or other container using load balancer or to access 
	  you need a port forwarding
	    kubcetl port-forward kuard 8080:8080
	
	This creates a secure tunnel from your local machine, through the kubernetes master 
	to the instance.

	Getting Logs
	  kubectl logs kuard
	This gives the logs related to that pod it you can use -f for contious log stream
	or other log aggreagatiob service like fluentd and elacticsearch 

	Running Commands in container
	  kubectl exec kuard date or kubcelt exec -it kuard bash

	Copying files to and from Containers
	  kubectl cp <pod-name>:/capture/capture3.txt ./capture3.txt
	   this copies file form container to local machine
	  kubectl cp $HOME/config.txt <pod-name>:/config.txt

	Health checks
	
	  When you run the application as a container in kubernetes, it is automatically 
	  kept alive uisng a process health check. This health check ensures that the main
	  process is running. However simple process check might not be enough as the 
	  application might have broken inside

	  To adress this kubernetes introduced health checks for application liveness and 
	  readiness
	    The liveness probe checks if application is live while the readiness probe checks
	    if the application is in ready state aong wilth built in health check that are
	    dependent on HTTP, TCP-sockets 

### Resource Management
	Kubernets can also be used for optimum uitlization of available computing resources
	so that there is less waste fo resources and dollars are saved. with sheduling system
	such as kubernets utilization greater than 50% can be acheived

	Minimum required Resource
	  apiVersion: v1
	  kind: pod
	  metadat:
	    name: kuard
	  spec:
            conatiners:
	      - image: gcr.io/kuar-demo/kuard-amd64:1
	        name: kuard
	        resources:
                  requests:
                    cpu: "500m"
                    memory: "128Mi"
                ports:
                  - containerPort: 8080
                    name: http
                    protocol: TCP
	
	Kubernetes shedules this in such a manner the pod is placed on a node where the 
	minimun resource requirement is met

	There is a maximum resoucrce requirement too
	
	when you set the maximun requirement you also set a minimum requirement the 
	diffrence between two is when you do not set the maximum cap all the available
	resource on the machine are given making sure minimum requirement is met meawhile
	in other case only the minimum asked resource is allocate and can only be scaled 
	to max cap limit

	requests:
	  cpu: "500m"
          memory: "128Mi"
	limits:
	  cpu: "1000m"
	  memory: "256Mi"
 
	

	

## Canary Deploments

	During the old mining times in britain the miners used to put caged canaries in 
	the mine and when ever the level of methane would increase or oxygen decreased
	beyond the safety limit the canaries would die or faint giving a signal to the 
	miners to vacate the caves
	
	Something similar is used in containerized workloads where a certain portion
	of servers are kept as canary servers and any new cahanges are first rolled out 
	on these servers and tested and then rolled out full fleged.

## Kubernetes RBAC authorization

## cretaing user

## creating roles

## applying roles

## creating bindings

## applying bindings  
