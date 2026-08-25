## Intro 
Kubernetes revolve around pods, one **pod** holds one or more closely connected **containers**. Each pod functions  as a separate VM on a **node**

node -> VM [pod -> container]

The following is a comparison between docker and K8s

| **Function** | **Docker**                       | **Kubernetes**                                |
| ------------ | -------------------------------- | --------------------------------------------- |
| `Primary`    | Platform for containerizing Apps | An orchestration tool for managing containers |
| `Scaling`    | Manual scaling with Docker swarm | Automatic scaling                             |
| `Networking` | Single network                   | Complex network with policies                 |
| `Storage`    | Volumes                          | Wide range of storage options                 |

## Types of Components
K8s architecture is divided into two types of components

- The Control Plane (master node), which is responsible for controlling the Kubernetes cluster
- The Worker Nodes (minions), where the containerized applications are run

#### Master Node
The master node hosts the Kubernetes `Control Plane`, which manages and coordinates all activities within the cluster, the `Minions` execute the actual applications and they receive instructions from the Control Plane.

The Control Plane serves as the management layer. It consists of several crucial components, including:

|**Service**|**TCP Ports**|
|---|---|
|`etcd`|`2379`, `2380`|
|`API server`|`6443`|
|`Scheduler`|`10251`|
|`Controller Manager`|`10252`|
|`Kubelet API`|`10250`|
|`Read-Only Kubelet API`|`10255`|
#### Worker Nodes
Within a containerized environment, the Minions serve as the designated location for running applications, each node is managed by the master node (Control Plane)

The `Scheduler`, based on the `API server`, understands the state of the cluster and schedules new pods on the nodes accordingly. After deciding which node a pod should run on, the API server updates the `etcd`.

## Kubernetes API
The API is the main point of contact for all internal and external interactions.

Within the Kubernetes framework, an API resource serves as an endpoint that houses a specific collection of API objects. These objects pertain to a particular category and include essential elements such as Pods, Services, and Deployments, among others. Each unique resource comes equipped with a distinct set of operations that can be executed, including but not limited to:

| **Request** | **Description**                                                |
| ----------- | -------------------------------------------------------------- |
| `GET`       | Retrieves information about a resource or a list of resources. |
| `POST`      | Creates a new resource.                                        |
| `PUT`       | Updates an existing resource.                                  |
| `PATCH`     | Applies partial updates to a resource.                         |
| `DELETE`    | Removes a resource.                                            |

## Authentication
Kubernetes supports multiple authentication methods such as client certificates, bearer tokens, an authentication proxy or an HTTP basic auth. Once the user is authenticated, K8s enforces authorization decisions using Role Based Access Control (`RBAC`).

In Kubernetes, the `kubelet` can be configured to permit `anonymous access`. By default, the Kubelet allows anonymous access, just as ftp anonymous access.  