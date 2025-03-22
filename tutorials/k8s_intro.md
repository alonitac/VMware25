# Kubernetes introduction

Like the Linux OS, Kubernetes (or shortly, k8s) is shipped by [many different distributions](https://nubenetes.com/matrix-table/#), each aimed for a specific purpose.

> [!NOTE]
> Throughout this module, we will use YAML manifests to work with k8s. You can fetch all files into your control plane machine by:
> 
> ```bash
> git clone https://github.com/alonitac/VMware25.git
> cd VMware25
> ```


## Deploy application in the cluster

Let's see Kubernetes cluster in all his glory! 

**Online Boutique** is a microservices demo application, consists of an 11-tier microservices.
The application is a web-based e-commerce app where users can browse items, add them to the cart, and purchase them.

Here is the app architecture and description of each microservice:

![k8s_online-boutique-arch][k8s_online-boutique-arch]

| Service                                              | Language      | Description                                                                                                                       |
| ---------------------------------------------------- | ------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| frontend                           | Go            | Exposes an HTTP server to serve the website. Does not require signup/login and generates session IDs for all users automatically. |
| cartservice                     | C#            | Stores the items in the user's shopping cart in Redis and retrieves it.                                                           |
| productcatalogservice | Go            | Provides the list of products from a JSON file and ability to search products and get individual products.                        |
| currencyservice             | Node.js       | Converts one money amount to another currency. Uses real values fetched from European Central Bank. It's the highest QPS service. |
| paymentservice               | Node.js       | Charges the given credit card info (mock) with the given amount and returns a transaction ID.                                     |
| shippingservice             | Go            | Gives shipping cost estimates based on the shopping cart. Ships items to the given address (mock)                                 |
| emailservice                   | Python        | Sends users an order confirmation email (mock).                                                                                   |
| checkoutservice             | Go            | Retrieves user cart, prepares order and orchestrates the payment, shipping and the email notification.                            |
| recommendationservice | Python        | Recommends other products based on what's given in the cart.                                                                      |
| adservice                         | Java          | Provides text ads based on given context words.                                                                                   |
| loadgenerator                 | Python/Locust | Continuously sends requests imitating realistic user shopping flows to the frontend.                                              |

To deploy the app in you cluster, perform the below command from the root directory of our course repo (make sure the YAML file exists): 

```bash 
kubectl apply -f k8s/online-boutique.yaml
```

By default, **applications running within the cluster are not accessible from outside the cluster.**
There are various techniques available to enable external access, we will cover some of them later on.

Using **port forwarding** allows developers to establish a temporary tunnel for debugging purposes and access applications running inside the cluster from their local machines.

```bash
kubectl port-forward svc/frontend 8080:80 --address 0.0.0.0
```

Visit the service using the external lab url (e.g. http://ip-123-123-123-123.ec2.internal.exit-zero.click:8080).

## Pods and namespaces


Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.
A Pod is a group of **one or more containers**, with **shared storage** and **network resources**, and a specification for how to run the containers.

You can list the Online Boutique pods by:

```bash
kubectl get pods
```

Make sure all pods are `Running`. 

Pods and other resources are aggrandized in **namespaces**.
Namespace isolates resources within a cluster, usually for better organization and access control.

If you don't specify namespace, the `default` namespace is used. To list pods from all namespaces:

```bash
kubectl get pods -A
```

# Exercises 

> [!TIP]
> ### `kubectl` quick reference
> 
> | Description                                | Examples                                                |
> |--------------------------------------------|---------------------------------------------------------|
> | List cluster objects - basic information   | `kubectl get pods`, `kubectl get nodes`                 |
> | List cluster objects - from all namespaces | `kubectl get pods -A`.                                  |
> | List cluster objects - certain namespace   | `kubectl get pods -n kube-system`.                      |
> | List cluster objects - wider information   | `kubectl get pods -o wide`, `kubectl get nodes -o wide` |
> | Get full description of an object          | `kubectl describe pod POD_NAME`                         |
> | Apply a YAML manifest                      | `kubectl apply -f my-manifest.yaml`.                    |
> | Apply all manifests in a given dir         | `kubectl apply -f dir/`.                                |
> | Delete an applied manifest                 | `kubectl delete -f my-manifest.yaml`                    |
> | Watch pod logs                             | `kubectl logs my-pod`                                   |


### :pencil2: Using `kubectl`

Use `kubectl` to answer the following questions: 

1. How many pod replicas does the **frontend** microservice have? 
2. How many **containers** does a **frontend** pod have? 
3. Use the `kubectl describe` command to get the IP address of the **frontend** pod. 
4. For the single **frontend** running pod, how many environment variables does a container named `server` have? 
5. For the single **frontend** running pod, what is the Docker image the container named `server` based on? 
6. What is the node name that the **emailservice** pod was scheduled on (by the k8s scheduler)?
7. What is the port do the **checkoutservice** pods listend on? 
8. How many containers does the pod starting with `fluentbit` in the `kube-system` namespace have? 

### :pencil2: Pod troubleshoot I

Apply the `k8s/customers-db.yaml` to deploy a MySQL pod in the cluster.

1. When a pod is in `Pending` status, one of the first places for debugging is the pod's events. Use the `kubectl describe` to investigate the root cause for the `Pending` status of the `customers` pod.
2. Use the `kubectl describe node YOUR_NODE_NAME` to get information about the current available capacity in the node (you can also get the same information from your node's **Node details** page in the GCP console), use your common sense to modify the YAML manifest so the customers-db pod can run in the cluster. 
3. When a pod is in `CrashLoopBackOff` status, the pod's containers were started successfully, but crashed due to internal error. 
   One of the first places for debugging in the pod's logs. Print the pod's log to address the issue. 
4. Inspired by the `k8s/online-boutique.yaml` YAML manifest, try to add the required env var to the `k8s/customers-db.yaml` manifest to make the MySQL pod running. 


Use the `kubectl delete` command to clean-up your cluster from the resources of the `k8s/customers-db.yaml` and `k8s/online-boutique.yaml` manifests.


[k8s_components]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/k8s_components.png
[k8s_online-boutique-arch]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/k8s_online-boutique-arch.png