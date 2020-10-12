# Working with Kubernetes Part - 2

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

# Kubernetes Workloads

In this guide we will go over Workloads, which are  the higher level kubernetes objects that manage pods or other higher level objects.

Make sure you have read the slides [TODO LINK] before going through this so you have a general understanding. 

- make sure to get a copy of `workload_tasks.md` from [TODO](todo) to fill out while you go through the tasks.

---

# ReplicaSet

### refresher
ReplicaSets allow a user to set a desired number of replicas of a given pod(s) that match a selector.

 The ReplicaSet handles the replica pod's lifecycle and ensures the number requested is upheld.

 
## Task 1 (Create a ReplicaSet)

Your first task is to create a `ReplicaSet` of the nginx application we made earlier.

1. Create a manifest file called `replica_1.yml`
2. In it create a resource of kind `ReplicaSet` and call it `nginx-replica`
3. Give it `2` replicas
4. For the template for the replica, use the nginx pod from before:
   - image : `nginx:stable-alpine` 
   - container name : nginx
   - labels: app_name = nginx and env = dev
   - port : 80
5. `apply` or `create` the resource
6. run `kubectl get rs` and view the results
   
7. Now, using [scale] command, scale up the replicas to 3. Save the command used in the `workload_tasks.md` file where appropriate

8. Now create the following pod and `apply` or `create` it (you dont need to save this file when you are finished):

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app_name: nginx
    env: dev
spec:
  containers:
  - name: nginx
    image: nginx:stable-alpine
    ports:
    - containerPort: 80

```

9. What happened? How many pods with our labels are there? Is there an event in the `ReplicaSet` that tells us? Write your findings in the `workload_tasks.md`
   - hint - `kubectl describe rs <name>` will shows events for the resource   
10. cleanup using the command `kubectl delete rs nginx-replica`

---
# Deployments

### refresher
Like ReplicaSet but has additional functionality, like rollback. Also allows for more fine grained control of pods.

 
## Task 2 (Create a Deployment)

In this task we will make a `Deployment` similar to the replica set we made in task 1.

visit [deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) for more information

1. In a new manifest file called `deployment_1.yml` create a `Deployment` with name 'nginx-deployment'
2. The deployment should have 2 `replicas` and the `revisionHistoryLimit` set to 3
3. under [strategy](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy) we wil use the `RollingUpdate` strategy, with `maxSurge` = 1 and `maxUnavailable` = 0
4. Use the same `template` from task 1 for the pods and the same selector
5. Create the manifest using `kubectl create`. Make sure to use the `--record` flag so we keep the command as an annotation
6. Use the `kubectl get` command to list the `ReplicaSet` that this deployment created with all its labels.
7. What do you notice about the `ReplicaSet` - what is this used for? Write down your answer in the `workload_tasks.md`
8. Try scaling the deployment like we did for `ReplicaSet`

## Task 3 (Update the Deployment)

1. Add some labels to the pod template to the deployment from Task 2
   - make sure to apply the changes with `--record` using the `apply` command
2. Write down what labels changes in the `workload_tasks.md` file. 
3. Run the command `kubectl rollout history deployment nginx-deployment` - and notice the revisions
4. In your worksheet - write down the commands I would use to rollback a deployment to a specific revision.
   -  see [here](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment) for help
5. Now that we are done - delete the deployment with `kubectl delete deploy nginx-deployment`

---
# DaemonSet

[see](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) for information on DaemonSets

### refresher
Similar to ReplicaSet/Deployment except the pods are ensured to be running on all nodes that match the selector.

This is good for pods you want to run on all nodes, like for log forwarding or capturing node health

## Task 4 (Create DaemonSet)

make sure you have no pods running from previous tasks

1. in a file called `daemon_1.yml` create a new manifest of kind `DaemonSet` with name 'nginx-ds'
2. set `revisionHistoryLimit` to 3
3. The pod spec should be the same as what we used in the previous tasks, but now we will add a `nodeSelector` with `nodeType` set to 'health'
4. `apply` the manifest with the `--record` flag
5. Does anything happen? Why? Write down your reasoning in worksheet.
6. run `kubectl label node minikube nodeType=health`
7. Did anything change? Write down what happened in the worksheet. 
8. Optional - just like with deployments, see if you can update the `DaemonSet` and roll it back to a different revision
9. run `kubectl delete ds nginx-ds`

---
# StatefulSets

see [here](https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/) and [here](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) for information of StatefulSets

### refresher
StatefulSets are made for pods that must persist data or keep their state. 

## Task 5 (Create StatefulSet)

1. copy the following into a manifest file and `kubectl apply` it, then use `kubectl get pods --show-labels --watch` to see what happens
  - you dont need to save this file after the task is done
  - *hint* - open a new terminal and run the command above. You wil see events happening as you work in another terminal
 ```
   
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stateful-nginx
spec:
  replicas: 3
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app_name: sts-nginx
  serviceName: nginx-sts
  updateStrategy:
    type: OnDelete
  template:
    metadata:
      labels:
        app_name: sts-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:stable-alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: html
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: standard
      resources:
        requests:
          storage: 100M
   ```

   1. In your worksheet, describe what happened and what you notice about the pods and the volumes for them
      - *hint* - `kubectl describe sts stateful-nginx` `kubectl get pvc` 
   2. Add a new label in the pod template - apply the change and write down what happened in the worksheet.
   3. Delete one of the pods then immediately run `kubectl get pods --show-labels --watch`
   4. What happened? What does `OnDelete` for `updateStrategy` do? What is another `updateStrategy` type? (read docs) - Write down your answer in the worksheet.
   5. run `kubectl delete sts stateful-nginx` 

---
# Jobs

see [here](https://kubernetes.io/docs/concepts/workloads/controllers/job/) for more information on Jobs

### refresher
Jobs ensure that pods run and successfully terminate, where jobs do one task. Acts like an executor with tasks/pods able to be run in parallel.

## Task 6 (Create a Job)

1. create a file called `job.yml` and copy the following into it:
   
```
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-app
spec:
  backoffLimit: 4
  completions: 8
  parallelism: 2
  template:
    spec:
      containers:
      - name: job
        image: alpine:latest
        command: ["/bin/sh", "-c"]
        args: ["echo running job from $HOSTNAME!"]
      restartPolicy: Never
```

2. Apply the manifest and observe the behavior. Observe how many pods are active at one time.
3. create a new `cron_job.yml` and make the `Job` above a `CronJob` that runs with the following `schedule` : "*/1 * * * *" and has name `cron-app`
4. set  `successfulJobsHistoryLimit` to 3 and `failedJobsHistoryLimit` to 3
5. Wait and observe what happens


[pre-lab]: https://github.com/cscc-afarag/kubernetes-lab1/blob/master/pre-lab.md
[lab 1]: https://github.com/cscc-afarag/kubernetes-lab1/blob/master/lab1.md
[scale]: https://kubernetes.io/docs/reference/kubectl/cheatsheet/#scaling-resources