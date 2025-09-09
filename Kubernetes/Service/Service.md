# Kubernetes Service Overview

In Kubernetes, a **Service** is an abstraction that defines a logical set of Pods and a policy to access them. Since Pods are ephemeral and their IPs can change, a Service provides a **stable network endpoint** (IP and DNS name) to reliably communicate with those Pods.

## Purpose of a Service

1. **Stable Endpoint:** Pods have dynamic IPs; Services give a fixed IP or DNS name for consistent access.
2. **Load Balancing:** Distributes traffic among multiple Pods behind the Service.
3. **Decoupling:** Clients don't need to know the actual Pod IPs; they only access the Service.
4. **Discovery:** Supports internal DNS-based discovery within the cluster.

## Types of Kubernetes Services

| Service Type | Description | Accessibility |
|--------------|-------------|---------------|
| **ClusterIP** | Default type. Provides a stable IP accessible **only inside the cluster**. | Internal only (in-cluster) |
| **NodePort** | Exposes Service on a static port of each node. Routes traffic to the pods. | Accessible externally via node IP |
| **LoadBalancer** | Uses cloud provider integration to create an external load balancer. | External access with LB IP |
| **ExternalName** | Maps Service to an external DNS name. | Internal pods resolve external DNS |

## How Services Work

1. Service selects Pods using **labels and selectors**.
2. Traffic sent to the Service IP or DNS is automatically **load-balanced** to the selected Pods.
3. ClusterDNS allows Pods to communicate using the Service name instead of IP.

## Key Points for Placement Interviews

* Services **abstract Pod networking** for reliability and stability.
* ClusterIP is the default and most commonly used internally.
* NodePort and LoadBalancer are used to expose services to external traffic.
* Services are essential for **microservices communication**, **load balancing**, and **high availability**.