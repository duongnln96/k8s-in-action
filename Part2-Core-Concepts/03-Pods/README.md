# **Chapter 03: Pods**

## **3.1. Introducing Pods**

### **3.1.1. Why we need pods ?**

***Noted:*** *Multiple containers are better than one container running multiple process*

- In Kubernetes you always run processes in containers and each container is much like an isolated.
machine, you may think it makes sense to run multiple processes in a single container, but you shouldn’t do that.

- Containers are designed to run only a single process per container.

- If you run multiple unrelated processes in a single container, it is your responsibility to keep all those processes running, manage their logs, and so on.

### **3.1.2. Understanding pods**

#### **The partial isolation between containers of the same Pod**

- K8S want to isolate groups of containers instead of individual ones. The containers inside each group to share certain resources, although not all, so that they’re not fully isolated. And K8S achieves this by configuring Docker to have all containers of a pod share the same set of Linux namespaces instead of each container having its own set.

- Because all containers of a pod run under the same Network and UTS namespaces (we’re talking about Linux namespaces here), they all share the same hostname and network interfaces.

#### **How the container share the same IP and Port space**

- One thing to stress here is that because containers in a pod run in the same Network namespace, they share the same IP address and port space.

- This means processes running in containers of the same pod need to take care not to bind to the same port numbers or they’ll run into port conflicts. But this only concerns containers in the same pod. Containers of different pods can never run into port conflicts, because each pod has a separate port space.

#### **Summary**

- Pods are logical hosts and behave much like physical hosts or VMs in the non-container world. Processes running in the same pod are like processes running on the same physical or virtual machine, except that each process is encapsulated in a container.

## **3.2 Pods YAML Description**

### **3.2.1 YAML Description**

- ***Metadata*** includes the name, namespace, labels, and other information about the pod.

- ***Spec*** contains the actual description of the pod’s contents, such as the pod’s containers, volumes, and other data.

- ***Status*** contains the current information about the running pod, such as what condition the pod is in, the description and status of each container, and the pod’s internal IP and other basic info.

```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubernetes.io/created-by: ...
  generateName: kubia
  labels:
    run: kubia
  name: kubia-zxzij
  namespace: default

spec:
  containers:
  - image: duongtomho/kubia-js
    name: kubia
    ports:
    - containerPort: 8080
      protocol: TCP
    
    resources:
      requests:
        cpu: 100m

    volumeMounts:
    - mountPath: /var/run/secrets/k8s.io/servacc
      name: default-token-kvcqa
      readOnly: true

  volumes:
  - name: default-token-kvcqa
    secret:
      secretName: default-token-kvcqa
```

### **3.2.2. Using *kubectl***

```console
$ kubectl create -f kubia-manual.yaml
pod "kubia-manual" created

$ kubectl get pods
NAME          READY   STATUS    RESTARTS  AGE
kubia-manual  1/1     Running   0         32s

$ kubectl logs kubia-manual
Kubia server starting...

$ kubectl logs kubia-manual -c kubia
Kubia server starting...

$ kubectl port-forward kubia-manual 8888:8080
... Forwarding from 127.0.0.1:8888 -> 8080
... Forwarding from [::1]:8888 -> 8080

$ curl localhost:8888
You’ve hit kubia-manual
```

## **3.3. Pods With Labels**

```yaml
metadata:
  name: kubia-manual-v2
  labels:
    creation_method: manual
    env: prod
```

```console
# Modifying labels existing pods
$ kubectl label po kubia-manual creation_method=manual
pod "kubia-manual" labeled

$ kubectl label po kubia-manual-v2 env=debug --overwrite
pod "kubia-manual-v2" labeled

# Listing pods with labels
$ kubectl get po -l creation_method,env
NAME            READY     STATUS    RESTARTS    AGE     CREATION_METHOD    ENV
kubia-manual    1/1       Running   0           16m     manual             <none>
kubia-manual-v2 1/1       Running   0           2m      manual             debug
kubia-zxzij     1/1       Running   0           1d      <none>             <none>
```

## **3.4. Using Namespaces to group resources**
