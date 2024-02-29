# **Chapter 05: Sevices**

Understanding how to expose Kubernetes services is key for building robust applications.

Services in Kubernetes allow pods to communicate with each other and provide a stable endpoint that doesn't change as pods are created or deleted.

## Creating the service

- Console

```console
kubectl expose rc kubia --type=LoadBalancer --name kubia-http
```

- Creating the service via YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia
spec:
  sessionAffinity: ClientIP # Default: None. It helps redirect all requests originating from the same client IP to the same pod
  ports:
    - port: 80 # The port this service will be available on
      targetPort: 8080 # The container port the service will forward to
  selector:
    app: kubia # All pods with the app=kubia label will be part of thi service.
```

- Exposing muiltiple ports in the same service

```yaml
spec:
  ports:
    - name: http # Specifing a namefor each port
      port: 80
      targetPort: 8080
    - name: https
      port: 443
      targetPort: 8443
    - name: https
      port: 443
      targetPort: https # Using named ports
```

- Using named ports

```yaml
kind: Pod
spec:
  containers:
    - name: kubia
      ports:
        - name: http
          containerPort: 8080
        - name: https # Port 8443 is called https
          containerPort: 8443
```

## There are several types of services:

**1. ClusterIP**

Exposes the service on a cluster-internal IP only. This makes the service only reachable from within the cluster.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia
spec:
  sessionAffinity: ClientIP # Default: None. It helps redirect all requests originating from the same client IP to the same pod
  ports:
    - port: 80 # The port this service will be available on
      targetPort: 8080 # The container port the service will forward to
  selector:
    app: kubia # All pods with the app=kubia label will be part of thi service.
```

**2. NodePort**

Exposes the service on each Node's IP at a static port. You can contact the NodePort service from outside the cluster by requesting <NodeIP>:<NodePort>.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia-nodeport
spec:
  type: NodePort # Set the service
  ports:
    - port: 80 # Port of the service’s internal cluster IP
      targetPort: 8080 # Port of the backing pod
      nodePort: 30123 # The service will be accessible via this port of each of your cluster nodes (not mandatory, it can choose random via K8S, port in range 30000-32767)
  selector:
    app: kubia
```

```console
$ kubectl get svc kubia-nodeport
NAME            CLUSTER-IP      EXTERNAL-IP   PORT(S)       AGE
kubia-nodeport  10.111.254.223  <nodes>       80:30123/TCP  2m

# The service is accessible at the following addresses
# 10.11.254.223:80
# <1st node’s IP>:30123
# <2nd node’s IP>:30123, and so on
```

**3. LoadBalancer**

Creates an external load balancer and assigns a fixed, external IP to the service. The load balancer routes to NodePorts of cluster nodes.

Kubernetes clusters running on cloud providers usually support the automatic provision of a load balancer from the cloud infrastructure.

If Kubernetes is running in an environment that _doesn’t support LoadBalancer services_, the load balancer will not be provisioned, but the service will _still behave like a NodePort service_

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia-loadbalancer
spec:
  type: LoadBalancer # The service type is set to LoadBalancer instead of NodePort
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app: kubia
```

```console
$ kubectl get svc kubia-loadbalancer
NAME                CLUSTER-IP      EXTERNAL-IP     PORT(S)       AGE
kubia-loadbalancer  10.111.241.153  130.211.53.173  80:32143/TCP  1m
```

**4. Ingress**

An ingress is really just a set of rules to pass to a controller that is listening for them. You can deploy a bunch of ingress rules, but nothing will happen unless you have a controller that can process them. A LoadBalancer service could listen for ingress rules, if it is configured to do so.

One important reason is that **each LoadBalancer service requires its own load balancer with its own public IP address**, whereas an Ingress **only requires one**, even when providing access to dozens of services.

An Ingress Controller is simply a pod that is configured to interpret ingress rules. One of the most popular ingress controllers supported by kubernetes is NGINX. Popular Ingress controllers include NGINX, Traefik, HAProxy and more.

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubia
spec:
  rules:
    - host: kubia.example.com # This Ingress maps the kubia.example.com domain name
      http:
        paths:
          - path: / # All requests will be
            backend: # sent to port 80 of the kubianodeport service.
              serviceName: kubia-nodeport
              servicePort: 80
```

## Note

**1. Load Balancer service of cloud**

A kubernetes LoadBalancer service is a service that points to external load balancers that are NOT in your kubernetes cluster, but exist elsewhere. They can work with your pods, assuming that your pods are externally routable. Google and AWS provide this capability natively

## References

1. [Ingress vs Load Balancer](https://stackoverflow.com/questions/45079988/ingress-vs-load-balancer)

2. [Kubernetes Services and Ingress Demystified](https://www.linkedin.com/posts/brijpandeyji_kubernetes-services-and-ingress-demystified-activity-7168205635682033664-kLCe?utm_source=combined_share_message&utm_medium=member_desktop)

##

<img "src" = >
