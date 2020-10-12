# Working with Kubernetes Part - 3

### **Before you start**
You need minikube up and running before going through the examples.

Start minikube using the command:
```
$ minikube start
```

If you do not have this setup please go to the [pre-lab] and follow those instructions before continuing.

If you are coming from [lab 1] please make sure your minikube is cleaned, by deleting all contexts, resources, configs, pods, and namespaces made.

You can do it using the `kubectl delete <resource> <name>` command or go with the scorched earth method and run ...

```
$ minikube stop
$ minikube delete
$ minikube start
```
---
## create namespace and context
If you switched your context or deleted your namespace/context from the previous part please follow the below instructions

Create a new namespace - 

```
$ kubectcl create namespace dev
```

Lets make a new context within our minikube environment with our dev namespace and use the same minikube user.

```
$ kubectl config set-context kubedev --cluster=minikube --namespace=dev --user minikube
```

switch to this context.

```
$ kubectl config use-context kubedev
```


## Notes
You will be creating manifest files as we go through this walkthrough - 
**These will be turned in and graded**

**While working through this go through the lecture slides, and online resources**

---

# Kubernetes Configuration

In this guide we will go over the patterns Kubernetes uses to decouple configuration from applications - namely ConfigMaps and Secrets.


Make sure you have read the slides [TODO LINK] before going through this so you have a general understanding. 

Understanding the core components, how they are used, and what they are used for are necessary before starting to use the higher-level kubernetes objects and resources.


---

# ConfigMaps

### refresher
- Externalized data
- KV pairs
- Can be created several ways, like manifests or literals.
- Can be used via:
  - env variable
  - cli
  - injected via volume mount
 


## Task 1 (Create a ConfigMap)
- make sure to get a copy of `config_tasks.md` from [TODO](todo) to fill out while you go through the tasks.
  
Your first task is to create a `ConfigMap` several different ways.

### 1 - via literal

1. Using the `create` command and flags, create a `ConfigMap` called `literal-data` with the following kv pairs:
   - make : bmw
   - model : m3

-- hint [create-cm] reference link

2. Save the command used in the `config_tasks.md` file under *task 1* 

### 2 - via manifest

1. Creates a manifest file of where `kind` is `ConfigMap`
2. The name should be `manifest-data`
3. Add the following KV pairs:
- make : subaru
- model: wrx
4. Save the file as `manifest-cm.yml`
5. Using `create` or `apply` create the ConfigMap in your minikube

### 3 & 4 - via directory or file.

1. create a directory called `cm`
2. inside `cm` create a file called `make` and add text `audi`
3. inside `cm` create a file called `model` and add text `rs4`
4. Using the `kubectl create cm`, create two more config maps, one called `file-data` and one called `dir-data`
   - You should use the command for creating a `ConfigMap` with a directory for `dir-data` and supplying the directory `cm`
   - For `file-data` your command should supply the files we made in `cm` 
   - Write the commands down in the `config_tasks.md` under the appropriate section *Task 1*


These are the 3 ways to create `ConfigMaps`, now lets see how we can use them.

---

## Task 2 (Using ConfigMaps)

1. Create the following the following `Job` as `cm_job_1.yml`

```
apiVersion: batch/v1
kind: Job
metadata:
  name: cm-job
spec:
  template:
    spec:
      containers:
      - name: job
        image: alpine:latest
        command: ["/bin/sh", "-c"]
        args: [" TODO "]
        env: TODO
      restartPolicy: Never

```

1. Update the `cm_job_1.yml` so that it prints the value of  `make` and `model` from one of the `ConfigMap`'s we made in the previous Task.
2. The output should look like "make: [MAKE] model: [MODEL]"  


## Task 3 (Using Secrets)

see [here](https://kubernetes.io/docs/concepts/configuration/secret/) for help

1. Secrets are similar to `ConfigMap`, create a secret similar to how we made a `ConfigMap` manifest and save this as `secret.yml`
2. Copy `cm_job_1.yml` and `secret_job_1.yml` and change it to use the secret created instead of the configmap.

[pre-lab]: https://github.com/cscc-afarag/kubernetes-lab1/blob/master/pre-lab.md
[lab 1]: https://github.com/cscc-afarag/kubernetes-lab1/blob/master/lab1.md
[create-cm]: https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#create-configmaps-from-literal-values
