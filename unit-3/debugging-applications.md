## Debugging pods

In this guide we will learn how to debug kubernetes resources

- Make sure all resources are deleted
- Make sure you are using the dev context and namepace
- Have two terminals open. 
  - In the second terminal run the following command
    - `watch kubectl get all`
  
## Part 1  - Restarting Pod
- create the following resource as `bad.yml`:
  
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bad-pod
spec:
  selector:
    matchLabels:
      app_name: myapp
  replicas: 1
  template:
    metadata:
      labels:
        app_name: myapp
    spec:
      containers:
        - name: shell
          image: centos:7
          command:
            - sh
            - '-c'
            - echo "Iam here now i am exiting "
  ```


  and apply it with `kubectl apply`

  - In the second terminal we are watching we can see it is failing to start. you should see something like

```
NAME                           READY   STATUS             RESTARTS   AGE
pod/bad-pod-58f7fc776b-fzvbj   0/1     CrashLoopBackOff   5          3m28s

```

- Lets find out whats wrong. first lets use describe on the deployment

```
$ kubectl describe deploy/bad-pod
```
Well see information about the deployment. Go through the output.

Lets get some more info using describe on the pod itself.

```
$ BADPOD=$(kubectl get po -l=app_name=myapp --output=jsonpath={.items[*].metadata.name})

$ kubectl describe po/$BADPOD

```
We should see something like this: 

```
...
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Mon, 03 Aug 2020 21:43:31 -0400
      Finished:     Mon, 03 Aug 2020 21:43:31 -0400
    Ready:          False
    Restart Count:  6
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-6vs7n (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  default-token-6vs7n:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-6vs7n
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                   From               Message
  ----     ------     ----                  ----               -------
  Normal   Scheduled  8m12s                 default-scheduler  Successfully assigned dev/bad-pod-58f7fc776b-fzvbj to minikube
  Normal   Pulling    8m11s                 kubelet, minikube  Pulling image "centos:7"
  Normal   Pulled     8m5s                  kubelet, minikube  Successfully pulled image "centos:7"
  Normal   Created    6m37s (x5 over 8m5s)  kubelet, minikube  Created container shell
  Normal   Started    6m37s (x5 over 8m5s)  kubelet, minikube  Started container shell
  Normal   Pulled     6m37s (x4 over 8m4s)  kubelet, minikube  Container image "centos:7" already present on machine
  Warning  BackOff    3m7s (x25 over 8m3s)  kubelet, minikube  Back-off restarting failed container


```

Looks like the container is failing to restart. Lets get logs on the pod.

```
$ kubectl logs $BADPOD
$ kubectl exec -it $BADPOD -- sh
```

The output shows the echo statement, but then it is exiting. If we remember, a deployment ensures a number of pods. So kubernetes keeps restarting it, but the container is designed to print and exit!

Fix this by replacing the `echo` statement with  `tail -f /dev/null` 

Now apply the changes without deleting and starting it.
    - remember `kubectl apply` will handle the differences if you edit a manifest created with `apply` and update it with `apply`

Once applied our container should be running and not restarting.


delete the deployment before moving on
```
kubectl delete -f bad.yml
```

---

## Part 2 -  Bad Image

Create a `bad_image.yml` manifest with the following:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bad-image
spec:
  selector:
    matchLabels:
      app_name: myapp
  replicas: 1
  template:
    metadata:
      labels:
        app_name: myapp
    spec:
      containers:
        - name: nginx
          image: nginx:107
```

Apply it. And in our watching terminal we should see something like:

```
pod/bad-image-6c659b967-zj4pw   0/1     ImagePullBackOff   0          2m20s

```

To quickly grep for errors:

```
kubectl get events | grep nginx | grep Error
```

should show something like this
```
Warning   Failed              pod/bad-image-6c659b967-zj4pw    Failed to pull image "nginx:107": rpc error: code = Unknown desc = Error response from daemon: manifest for nginx:107 not found: manifest unknown: manifest unknown
```

To fix the bad image lets patch the resource:

```
kubectl patch deployment bad-image  \
                --patch '{ "spec" : { "template" : { "spec" : { "containers" : [ { "name" : "nginx" , "image" : "nginx" } ] } } } }'

```

this should resolve our broken image issue and the deployment should be running



delete the deployment before moving on
```
kubectl delete -f bad_image.yml
```

---

## Part 3 - Memory issues

Create and apply the following deployment as oom.yml

```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: oom
spec:
  selector:
    matchLabels:
      app_name: oom
  replicas: 1
  template:
    metadata:
      labels:
        app_name: oom
    spec:
      containers:
        - name: memoryhog
          image: centos:7
          command:
            - sh
            - '-c'
            - "sleep 5 && yes | tr \\n x | head -c 500m | grep n && sleep 1000"
          resources:
            limits:
              memory: 400M
        - name: shell
          image: centos:7
          command:
            - sh
            - '-c'
            - sleep 1000
```

Wait a few seconds and then run (might have to retry a few time since it crashes)
```
kubectl exec -it $(kubectl get po -l=app_name=oom --output=jsonpath={.items[*].metadata.name}) -c memoryhog -- cat /sys/fs/cgroup/memory/memory.limit_in_bytes /sys/fs/cgroup/memory/memory.usage_in_bytes
```
Lets use describe
```
kubectl describe po $(kubectl get po -l=app_name=oom --output=jsonpath={.items[*].metadata.name})

```


How can we fix this issue?

*hint* - see memory request limit and the command running in the container

After you fix it - delete the deployment

```
kubectl delete -f oom.yml 
```
---

## Part 4 App Issue

make and apply a new manifest called `app_issue.yml` with the following

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oops
spec:
  selector:
    matchLabels:
      app: oops
  replicas: 1
  template:
    metadata:
      labels:
        app: oops
    spec:
      containers:
        - name: oopsapp
          image: centos:7
          command:
            - sh
            - '-c'
            - "for x in {1..20}; do echo working here at $x ; sleep 1; done; echo exiting; exit"

```
Wait some time till it restarts.

Lets try describe:
```
kubectl describe deploy/oops
```
Unfortuntley its not showing us really why its restarting.

lets follow the logs of the pod
```
kubectl logs --follow $(kubectl get po -l=app=oops --output=jsonpath={.items[*].metadata.name})
```

Similar to the first example, we can see clearly that the application is exiting, and the deployment restarts it because it ensures that there is atleast 1 instance available.

Using the log command is helpful to target pods/containers and see whats going on in the app.

Delete the deployment
```
kubectl delete -f oops.yml 
```
---

## Part 5 Storage

like before save and create the following manifest

```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: volumeissue
spec:
  selector:
    matchLabels:
      app: volumeissue
  replicas: 1
  template:
    metadata:
      labels:
        app: volumeissue
    spec:
      containers:
        - name: producer
          image: centos:7
          command:
            - sh
            - '-c'
            - "printf 'consumer writing out data to volume' > /tmp/out/data; sleep 5000"
          volumeMounts:
            - name: sharedmount
              mountPath: /tmp/out
        - name: consumer
          image: centos:7
          command:
            - sh
            - '-c'
            - "cat /tmp/in/data; sleep 5000"
          volumeMounts:
            - name: sharedmount
              mountPath: /tmp/data
      volumes:
      - name: sharedmount
        emptyDir: {}

```

First lets check if the data was written out or not

```

kubectl exec -it $(kubectl get po -l=app=volumeissue --output=jsonpath={.items[*].metadata.name}) -c producer -- cat /tmp/out/data

```

looks fine - what about the consumer?

```
kubectl exec -it $(kubectl get po -l=app=volumeissue --output=jsonpath={.items[*].metadata.name}) -c consumer -- cat /tmp/in/data

```

looks like the file doesnt exit lets describe the pod.

```
kubectl describe po $(kubectl get po -l=app=volumeissue --output=jsonpath={.items[*].metadata.name})

```

Can you see whats wrong?

*hint* how do volumes get attached to containers?

Fix it and verify then delete the deployment (make sure to wait for everything to be re applied)

---

In these quick examples we went through some basics of how to debug pods and containers. Althought the examples were simple, the tools used are very helpful in debugging more complicated applications.



---

Helpful links:


- [Kubernetes how to debug CrashLoopBackoff](https://stackoverflow.com/questions/44673957/kubernetes-how-to-debug-crashloopbackoff)
- [Kubernetes imagePullSecrets not working; getting “image not found”](https://stackoverflow.com/questions/32510310/kubernetes-imagepullsecrets-not-working-getting-image-not-found)
- [Trying to create a Kubernetes deployment but it shows 0 pods available](https://stackoverflow.com/questions/51139988/trying-to-create-a-kubernetes-deployment-but-it-shows-0-pods-available)
- [Kubernetes: five steps to well-behaved apps](https://medium.com/@betz.mark/kubernetes-five-steps-to-well-behaved-apps-a7cbeb99471a)

- [Troubleshoot Applications](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/)
- A site dedicated to [Kubernetes Troubleshooting](https://kubernetes.feisky.xyz/en/troubleshooting/) 
  
- CrashLoopBackoff, Pending, FailedMount and Friends: Debugging Common Kubernetes Cluster (KubeCon NA 2017): [video](https://www.youtube.com/watch?v=7FOCG5kua1w) and [slide deck](https://afontofuseless.info/debugging-kubernetes-app-deploys-kc2017/)
- 10 Most Common Reasons Kubernetes Deployments Fail: [Part 1](https://kukulinski.com/10-most-common-reasons-kubernetes-deployments-fail-part-1/) and [Part 2](https://kukulinski.com/10-most-common-reasons-kubernetes-deployments-fail-part-1/)

- [Debugging Microservices: How Google SREs Resolve Outages](https://www.infoq.com/presentations/google-debug-microservices)
- [Debugging Microservices: Lessons from Google, Facebook, Lyft](https://thenewstack.io/debugging-microservices-lessons-from-google-facebook-lyft/)
- [Troubleshooting Java applications on OpenShift](https://developers.redhat.com/blog/2017/08/16/troubleshooting-java-applications-on-openshift/)
- Google Kubernetes Engine [Troubleshooting](https://cloud.google.com/kubernetes-engine/docs/troubleshooting) docs

- [How to find out why mounting an emptyDir volume fails in Kubernetes?](https://stackoverflow.com/questions/51206154/how-to-find-out-why-mounting-an-emptydir-volume-fails-in-kubernetes)
- [Kubernetes NFS volume mount fail with exit status 32](https://stackoverflow.com/questions/34113569/kubernetes-nfs-volume-mount-fail-with-exit-status-32)

- [Debugging Kubernetes PVCs](https://itnext.io/debugging-kubernetes-pvcs-a150f5efbe95)
