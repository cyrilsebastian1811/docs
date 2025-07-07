# Kubernetes Services
Kubernetes Services is a critical abstraction that defines a logical set of Pods and a policy by which to access them. Services enable a loose coupling between dependent Pods.

## Service Types
Kubernetes Services offer several service types to handle varying use cases:

- ClusterIP: Exposes the service on a cluster-internal IP. This type makes the service only reachable from within the cluster.
- NodePort: Exposes the service on each Node’s IP at a static port. A ClusterIP service, to which the NodePort service routes, is automatically created.
- LoadBalancer: Exposes the service externally using a cloud provider’s load balancer. NodePort and ClusterIP services, to which the external load balancer routes, are automatically created.
- ExternalName: Maps the service to the contents of the externalName field by returning a CNAME record.


## Defining a Service
A Kubernetes Service is defined using YAML (preferred) or JSON, like all Kubernetes objects. The Service's API object includes a specification that determines its behavior. When you create a Service, you must specify selectors to determine which Pods will receive the traffic sent to the Service.

## Accessing a Service
For some parts of your applications, you may want to expose a Service onto an external IP address. Kubernetes supports two ways of doing this: NodePort and LoadBalancer. The Service’s type determines how the Service is exposed.

For more detailed information, please refer to the official Kubernetes documentation here.

Please note that this is a high-level overview. For a more detailed understanding, it's recommended to go through the official Kubernetes documentation.