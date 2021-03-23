# **Chapter 07: ConfigMaps and Secret**

## **7.2 Passing command-line arguments to containers**

### **7.2.1 Docker**

In a Dockerfile, two instructions define the two parts:

- ENTRYPOINT defines the executable invoked when the container is started.
- CMD specifies the arguments that get passed to the ENTRYPOINT.

The correct way is to do it through the ENTRYPOINT instruction and to only specify the CMD if you want to define the default arguments. The image can then be run without specifying any arguments

```console
docker run <image>

docker run <image> <arguments>
```

### **7.2.2 Overriding the command, argument and set environment in Kube**

```yaml
kind: Pod
spec:
  containers:
  - image: some/image
    command: ["/bin/command"]       # overriding command
    args: ["arg1", "arg2", "arg3"]  # argument
    env:                            # environment
    - name: FIRST_VAR
    value: "foo"
    - name: SECOND_VAR
    value: "$(FIRST_VAR)bar"
```

## **7.3 ConfigMaps**

### **7.3.1 Creating a ConfigMap**

```console
# USING THE KUBECTL CREATE CONFIGMAP COMMAND
$ kubectl create configmap fortune-config --from-literal=sleep-interval=25
configmap "fortune-config" created

# CREATING A CONFIGMAP ENTRY FROM THE CONTENTS OF A FILE
$ kubectl create configmap my-config --from-file=config-file.conf
configmap "my-config" created

# CREATING A CONFIGMAP FROM FILES IN A DIRECTORY
$ kubectl create configmap my-config --from-file=/path/to/dir
configmap "my-config" created

# COMBINING DIFFERENT OPTIONS
$ kubectl create configmap my-config \
--from-file=foo.json \
--from-file=bar=foobar.conf \
--from-file=config-opts/ \
--from-literal=some=thing \
configmap "my-config" created
```

### **7.3.2 Passing a ConfigMap entry to a container as an environment variable**

```yaml
...
kind: Pod
...
spec:
  containers:
  - image: luksa/fortune:env
    env:
    - name: INTERVAL            # env var
      valueFrom:                # get value from configmap
        configMapKeyRef:
          name: fortune-config  # the name of configmap is referenced
          key: sleep-interval   # key in configmap

```

### **7.3.3 Passing all entries of a ConfigMap as environment variables at once**

```yaml
spec:
  containers:
  - image: some-image
    envFrom:                    # Using envFrom instead of env
    - prefix: CONFIG_           # All environment variables will be prefixed with CONFIG_.
      configMapRef:
        name: my-config-map
```

### **7.3.4 Passing a ConfigMap entry as a command-line argument**

```yaml
kind: Pod
spec:
  containers:
  - image: luksa/fortune:args
    env:
    - name: INTERVAL
      valueFrom:
        configMapKeyRef:
          name: fortune-config
          key: sleep-interval
    args: ["$(INTERVAL)"]
```

Using the $(ENV_VARIABLE_NAME) syntax to have Kubernetes inject the value of the variable into the argument.

### **7.3.5 Using a configMap volume to expose ConfigMap entries as files**

A configMap volume will expose each entry of the ConfigMap as a file. The process running in the container can obtain the entry’s value by reading the contents of the file.


