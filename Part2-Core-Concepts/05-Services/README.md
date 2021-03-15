# **Chapter 05: Sevices**

Unlike in the non-K8S world, where we would configure each client app by specifying the exact IP address or hostname of the server. In K8S environment it will not work because

- ***Pods are ephemeral*** - They may come and go at any time, whether it’s because a pod is removed from a node to make room for other pods, because someone scaled down the number of pods, or because a cluster node has failed.

- ***Kubernetes assigns an IP address to a pod after the pod has been scheduled to a node and before it’s started*** - Clients thus can’t know the IP address of the server pod up front.

- ***Horizontal scaling means multiple pods may provide the same service*** - Each of those pods has its own IP address. All those pods should be accessible through a single IP address

## **5.1. Introducing Services**

### **5.1.1. Creating the service**

#### **a. Creating the service via *kubectl expose***

```console
kubectl expose rc kubia --type=LoadBalancer --name kubia-http
```

#### **b. Creating the service via YAML descriptor**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia
spec:
  sessionAffinity: ClientIP # Default: None. It helps redirect all requests originating from the same client IP to the same pod
  ports:
  - port: 80            # The port this service will be available on
    targetPort: 8080    # The container port the service will forward to
  selector:
    app: kubia          # All pods with the app=kubia label will be part of thi service.
```

#### **c. Exposing muiltiple ports in the same service**

```yaml
spec:
  ports:
  - name: http          # Specifing a namefor each port
    port: 80
    targetPort: 8080
  - name: https
    port: 443
    targetPort: 8443
  - name: https
    port: 443
    targetPort: https   # Using named ports
```

#### **d. Using named ports**

```yaml
kind: Pod
spec:
  containers:
  - name: kubia
    ports:
    - name: http            
      containerPort: 8080
    - name: https           # Port 8443 is called https
      containerPort: 8443 
```

## **5.2. Connecting to services living outside the cluster**

## **5.3. Exposing serviecs to external clients**

A few ways to make a service accessible externally:

- ***Setting the service type to NodePort*** - For a NodePort service, each cluster node opens a port on the node itself and redirects traffic received on that port to the underlying service.

- ***Setting the service type to LoadBalancer, an extension of the NodePort type*** - This makes the service accessible through a dedicated load balancer. Clients connect to the service through the load balancer’s IP

- ***Creating an Ingress resource, a radically different mechanism for exposing multiple services through a single IP address***

### **5.3.1. Using NodePort Service**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia-nodeport
spec:
  type: NodePort        # Set the service
  ports:
  - port: 80            # Port of the service’s internal cluster IP
    targetPort: 8080    # Port of the backing pod
    nodePort: 30123     # The service will be accessible via this port of each of your cluster nodes (not mandatory, it can choose random via K8S, port in range 30000-32767)
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

### **5.3.2. Exposing a service through an external load balancer**

Kubernetes clusters running on cloud providers usually support the automatic provision of a load balancer from the cloud infrastructure.

If Kubernetes is running in an environment that *doesn’t support LoadBalancer services*, the load balancer will not be provisioned, but the service will *still behave like a NodePort service*

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia-loadbalancer
spec:
  type: LoadBalancer    # The service type is set to LoadBalancer instead of NodePort
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

## **5.4. Exposing services externally through an INGRESS resource**

One important reason is that **each LoadBalancer service requires its own load balancer with its own public IP address**, whereas an Ingress **only requires one**, even when providing access to dozens of services.

### **5.4.1. Creating an Ingress resource**

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
      - path: /                         # All requests will be  
        backend:                        # sent to port 80 of the kubianodeport service.
          serviceName: kubia-nodeport
          servicePort: 80
```

## **5.5 Signaling when a pod is ready to accept connections**

What happend if the pod isn’t ready to start serving requests immediately ?

----
**Readiness** - Similar to liveness probes, The readiness probe is invoked periodically and determines whether the specific pod should receive client requests or not.

**Types of Readiness probes** - three types of readiness probes exist:

- *Exec* probe, where a process is executed. The container’s status is determined by the process’ exit status code.

- *HTTP GET* probe, which sends an HTTP GET request to the container and the HTTP status code of the response determines whether the container is ready or no

- *TCP Socket* probe, which opens a TCP connection to a specified port of the container. If the connection is established, the container is considered ready.

```yaml
apiVersion: v1
kind: ReplicationController
...
spec:
  ...
  template:
    ...
    spec:
      containers:
      - name: kubia
        readinessProbe:
          exec:
            command:        # run ls /var/ready 
            - ls            # get return code zero or non-zero
            - /var/ready
        ...
```

**Noted:**

- You ***should always*** define a readiness probe, even if it’s as simple as sending an HTTP request to the base URL

- ***Do not*** include Pod shutdown logic into readiness probes
