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


