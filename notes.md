# Kubernetes Components

[Official documentation](https://kubernetes.io/docs/concepts/overview/components/)

## Control Plane

Makes global decisions about the cluster (schedule, detect and respond to cluster events).  
Usually, no more containers run alongside the control plane.

### kube-apiserver

[Official Kubernetes API documentation](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)

Serves the kubernetes API, the primary interface to control plane and the cluster itself.
```kubectl``` will make use of this to interact with the cluster.

**QUESTION:** What happens if I scale it down to 0 across all nodes? How can I brige it up again?

### Etcd

Backend data store for the cluster. Provides HA distributed storage for all data relating to the state of the cluster
Issuing ```kubectl``` commands will go to **kube-api-server** which will then go to **etcd** to read/write information

**QUESTION:** As said in the official documentation, we should have a backup plan for the etcd data. [How](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)? 
**QUESTION:** As said in the officual documentation, "_if_ we're using etc as backing store", which means there are others?

### kube-scheduler

Handles _scheduling_, the process of reading what is defined in POD and selecting a node that matches given criteria as well as availability to then schedule the pod there.

The flow here, once we run ```kubectl apply -f deployment.yaml```, most likely is:

```flow
kube-api-server > etcd > kube-api-server > kube-scheduler > kube-api-server > etcd > kube-api-server > output
```

Factors taken into account for scheduling include:

* individual and collective resource requirements
* hardware/software/policy constraints
* affinity and anti-affinity specifications
* data locality
* inter-workload interference
* deadlines

**QUESTION:** What are those requirements??

### kube-controller-manager

Has multiple controllers running in a single process. Handles bunch of stuff:

* Node controller: notice and react to nodes going down
* job controller: watch job objects that represents one-off tasks, creates the pod for it 'till completion
* endpoints controller: populates endpoint objects (Joins services and pods)
    **QUESTION:** Why not the kube-proxy here? Or this controls it and delegates to kube-proxy?
* service account & token controllers: Create default accounts and API access tokens for new namespaces

**QUESTION:** What does it exactly do? When is it called? What happens if I scale it down or it malfunctions?

_The kubelet doesn't manage containers which were not created by Kubernetes_

### cloud-controller-manager

Used to interface to/from cloud providers. Use when k8s cluster is integrated with cloud platform stuff. Most likely used in AKS/EKS... 

As per official documentation _"If you are running Kubernetes on your own premises, or in a learning environment inside your own PC, the cluster does not have a cloud controller manager."_

## Nodes

A VM (most-likely) that is a part of the cluster.  
Also known as _worker machines_

### kubelet

Kubernetes agent that runs on each node.  
Communicates with control plane and makes sure everything is as expected, meaning that pods are running where/how they are supposed to be.

**kubelet** also has to check pod status and report them to the control plane.

### * container runtime

Good thing to know is that kubernetes support multiple different implementations of conteriner runtimes - **Docker**, **CRI-O** and **containerd** are examples.   
This is what enables containers to actually run in the node.

(*) Not acually part of the kubernetes but is a requirement that needs to be running in each node.  

### kube-proxy

Network proxy.  
Also needs to run in each node of the cluster.  
Implements part of the Service concept.
Provides network rules on nodes. These allow network comm. to Pods from network sessions inside or outside of the cluster.

## Addons

Addons use k8s features to implement cluster features.  
Since they provide cluster-level features, namespaced resources for addons belong within the **kube-system** namespace