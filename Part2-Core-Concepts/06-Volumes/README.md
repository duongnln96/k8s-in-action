# **Chapter 06: Volumes**

Kubernetes volumes are a component of a pod and are thus defined in the *pod’s specification*.

A volume is available to all containers in the pod, but it must be mounted in each container that needs to access it. In each container, you can mount the volume in any location of its filesystem.

## **6.1. Introducing volumes**

### **6.1.2. Available volume types**

- ***emptyDir*** — A simple empty directory used for storing transient data.

- ***hostPath*** — Used for mounting directories from the worker node’s filesystem into the pod.

- ***gitRepo*** — A volume initialized by checking out the contents of a Git repository.

- ***nfs*** — An NFS share mounted into the pod.

- ***configMap, secret, downwardAPI*** — Special types of volumes used to expose certain Kubernetes resources and cluster information to the pod.

- ***persistentVolumeClaim*** — A way to use a pre- or dynamically provisioned persistent storage. (We’ll talk about them in the last section of this chapter.)

- ...

***Note:*** A single pod can use multiple volumes of different types at the same time.

## **6.2. Using volumes to share data between containers**

### **6.2.1. Using *emptyDir***

An *emptyDir* volume is especially useful for sharing files between containers running in the same pod. It can also be used by single container at a monent.

```yaml
kind: Pod
spec:
  containers:
  - image: luksa/fortune
    name: html-generator
    volumeMounts:
    - name: html
      mountPath: /var/htdocs
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
      readOnly: true
    ports:
    - containerPort: 80
      protocol: TCP
  volumes:
  - name: html
    emptyDir: {}
```

The *emptyDir* you used as the volume was created on the actual disk of the worker node hosting your pod, so its performance depends on the type of the node’s disks.

```yaml
volumes:
- name: html
  emptyDir:
  medium: Memory  # This emptyDir’s files should be stored in memory.
```

### **6.2.2 Using gitRepo**

```yaml
volumes:
  - name: html
    gitRepo:    # creating a gitRepo volume
      repository: https://github.com/luksa/kubia-website-example.git  # gitrepo
      revision: master  # The master branch will be checked out.
      directory: .  # Git repo wil be cloned to root folder
```

## **6.3. Accessing files on the worker node’s filesystem**

***Note:*** Remember, these will usually be managed by a DaemonSet

A *hostPath* volume points to a specific file or directory on the node’s filesystem

*hostPath* volumes are the first type of persistent storage, because both the gitRepo and emptyDir volumes’ contents get deleted when a pod is killed, whereas a hostPath volume’s contents don’t

## **6.4. Using persistent storage**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
spec:
  volumes:
  - name: mongodb-data
    gcePersistentDisk:  # The type of the volume is a GCE Persistent Disk.
      pdName: mongodb
      fsType: ext4   # The filesystem type is EXT4 (a type of Linux filesystem).
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - name: mongodb-data
      mountPath: /data/db # The path where MongoDB stores its data
    ports:
    - containerPort: 27017
      protocol: TCP
```

## **6.5. PersistentVolumes and PersistentVolumeClaims**

Using GCE Persistent Disk

## **6.5.. Creating a PersistentVolumes**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv
spec:
  capacity:
    storage: 1Gi  # PersistentVolume’s size
  accessModes:
  - ReadWriteOnce
  - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain   # After the claim is released, the PersistentVolume should be retained (not erased or deleted).
  gcePersistentDisk:
    pdName: mongodb
    fsType: ext4
```
