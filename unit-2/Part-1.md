# Working with Kubernetes Part - 1

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

## Notes
You will be creating manifest files as we go through this walkthrough - 
**These will be turned in and graded**

**While working through this go through the lecture slides, and online resources**

---

# Kubernetes Core

In this guide we will go over the core features that make up Kubernetes. Make sure you have read the slides [TODO LINK] before going through this so you have a general understanding. 

Understanding the core components, how they are used, and what they are used for are necessary before starting to use the higher-level kubernetes objects and resources.

---

# namespaces and context

In kubernetes, namespaces are logical partitions of a cluster or environment. Its primary use is to scope access, or partition a cluster.

For example we might want to partition our cluster with a development namespace, a test namespace, and a prod namespace.

  Lets get the current namespaces - 

```
$ kubectl get namespaces
```

Create a new namespace - 

```
$ kubectl create namespace dev
```


To make things easy lets look at all the contexts we have then create a context for this namespace - 

```
$ kubectl config get-contexts
```

you should see something like:

```
$ kubectl config get-contexts

CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
*         minikube   minikube   minikube   
```

Lets make a new context within our minikube environment with our dev namespace and use the same minikube user.

```
$ kubectl config set-context kubedev --cluster=minikube --namespace=dev --user minikube
```

Now when running  `kubectl config get-contexts`, we should see our new context.

```
$ kubectl config get-contexts

CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
          kubedev    minikube   minikube   dev
*         minikube   minikube   minikube   

```

Now lets switch to this context.

```
$ kubectl config use-context kubedev
```

Now that we have a context we can switch between them easily. In a real environment I might have a context for each namespace.

Creating namespaces serve has providing scope to names, and access to resources.

---

# Pods

### refresher
- Pods serve as the atomic unit in kubernetes. 
- It is the single unit of work. When containers are run the containers are assigned to pods.
- We can deploy one or more containers in a pod, and mount volumes to containers within a pod

## Task 1 (Single Pod)
- Before begining this task download a copy of the `pod_task.md` file.

Your first task is to create a manifest file that has the following properties:

1. Create a manifest of `kind` Pod using the image -  `nginx:stable-alpine`
2. The name of the pod should be `pod-nginx`
3. The name of the container should be `nginx`
4. The container should have a port opened at `80`
   
- Save this manifest file as `pod_1.yml` in your manifest directory.
- Use `create` or `apply` the manifest file
  - Save the command you used to create the pod in the `pod_task.md` file
- Verify the pod is deployed using the `describe` or `proxy` command
  - Save the command you used to verify in the `pod_task.md` file
  - if using the `proxy` command also save the url you used to view the pod.

---

## Task 2 (Multi Pod with Volume mount)

In this task we will be adding a second container to the pod and mounting a volume with content.

- Start by copying the `pod_1.yml` as `pod_2.yml`

In the `pod_2.yml`, add an additional pod/volume to the manifest with the following properties:

1. Change the name of the pod to `multi-pod-volume-nginx`
2. Add a volume called `html-content` using an `emptyDir`
3. Add a second container to the pod using the image -  `alpine:latest`
   -  The name of the container should be `pod-content`
4. On the `nginx` container add a `volumeMount`
   - mount the created `html-content` volume to the pod using `/user/share/nginx/html` as the `mountPath`
5. Similarly on the `pod-content` container add a `volumeMount`
   - mount the created `html-content` volume to the pod using `/html` as the `mountPath`
6. On the `pod-content` container add a `command` with value `["/bin/sh", "-c"]`
7. On the `pod-content` and to `args` the following code:
  
  ```
while true; do
    echo "current time<br/>"$(date)"<br/>" >> /html/index.html;
    sleep 5;
done
   ```
   
- Save this manifest file as `pod_2.yml` in your manifest directory.
- Use `create` or `apply` the manifest file
- Using `proxy` visit the url for this pod
- In `pod_task.txt` breifly describe how the containers are working together to display what you see on the browser.

delete the pod using the `kubectl delete` command.

---

# Labels/Selectors

Labels are kv pairs added as meta data to resources that allow for identification, and grouping. Selectors allow a user to filter or select objects via labels. 

## Task 3 (Labels and Selectors)

In this task we will add some labels to our manifest from Task 2, so copy the `pod_2.yml` as `labels_1.yml`


1. In the `labels_1.yml` change the name of the pod to `labeled-pod` and `create` or `apply` the resource.
   
   
2. Now, using the [kubectl label] command, add the following labels to the pod
   - app_name=nginx
   - env=dev 
3. You can verify the labels were added via `kubectl get pods --show-labels`
4. In the `pod_task.md` file - write the command used to add the labels to the pod.
5. Again using the [kubectl label] command remove these labels. Write the command down in the `pod_task.md` file.
6. Add the labels to the `labels_1.yml` and `apply` the changes.
7. Using the [equality selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#equality-based-requirement) and the [set selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#set-based-requirement) run and write down the `kubectl get` commands in the `pod_task.md` that will do the following -
   1. using equality the command to get pods that have app_name=nginx
   2. using set the command to get pods that has app_name=nginx and env != prod
8. 
 
### Take note -
Although it might seem tedious from the command line to use selectors, labels and selectors are heavily used in higher level abstractions in kubernetes. 

Being comfortable with using them is also highly helpful when trying to filter/find resources in a kubernetes environment with alot of running things.

---

# Exposing applications with Services

## **Note - You will need the pods from the previous task running to complete this task**

Before starting this task -

1. Create the pod `pod-nginx` from Task 1
2. Add the label `app_name=nginx` to the `pod-nginx` pod using what we learned in Task 3
3. Create the pod `labeled-pod` from Task 3.
4. Verify you have done this by running `kubectl get pods --show-labels`
   - you should see something like:
    ```
    NAME          READY   STATUS    RESTARTS   AGE   LABELS
    labeled-pod   2/2     Running   0          33m   app_name=nginx,env=dev
    pod-nginx     1/1     Running   0          27s   app_name=nginx
    ```

Use this [service-reference](https://kubernetes.io/docs/concepts/services-networking/service/) for the next two tasks.

---
## Task 4 - (Cluster IP)

In this task you will create a service using `clusterIP`.

1. Create a file called `cluster-ip-service.yml`
2. Create a Service of type `ClusterIP` and call the service `nginx-service`
    - note: what is the default `type` for a object of `kind` == `Service`
3. Expose the app from our previous task `nginx` using the `app_name` label and expose it at port 80, with the TCP protocol
   - hint : use a `selector` 
4. `create` or `apply` the manifest.  
5. run `kube proxy` and visit the url of the service at  http://127.0.0.1:8001/api/v1/namespaces/dev/services/nginx-service/proxy/ in your browser or using curl
6. Refresh the browser, or execute curl multiple times
7. Describe what is happening and what you see in the `pod_tasks.md` (Question 8)

### notes -
ClusterIP is the most common way to expose application in kubernetes, and is how pod/services are consumed within kubernetes. Remember that the service is a static IP abd DNS entry that maps to pods.

---
## Task 5 - (NodePort)

In this task we will expose our application using the `NodePort` object. Like before you will need the pods from Task 3 and Task 3, please refer to the note in Task 4 to get it setup.

1. Create a file called `nodeport-service.yml`
2. Create a `Service` of type `NodePort` and call it `nodeport-nginx`
3. Similar to Task 4, expose our pods at port `80` using the selector for `app_name=nginx` and `env=dev` with the `nodePort` equal to `32410`
4. `create` or `apply` the manifest. 
5.  Run the command `minikube service -n dev nodeport-nginx`
6.  Under Question 9 in the `pod_task.md` desribe the behavior you see, then explain the *why* you think it is different than what we saw in Task 4

---

[pre-lab]: https://github.com/cscc-afarag/kubernetes-lab1/blob/master/pre-lab.md
[lab 1]: https://github.com/cscc-afarag/kubernetes-lab1/blob/master/lab1.md
[kubectl label]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#label
