# Example Application



## PHP Guestbook Application with Redis

What were going to deploy
- A single-instance Redis master to store guestbook entries
- Multiple replicated Redis instances to serve reads
- Multiple web frontend instances

---
## Redis Master
*redis is used by the guestbook application to store its data*

The application will write to a master and read from multiple slave instances.

## Redis Deployment

##### Create a Deployment in a file called `redis-master-deployment.yml` that has the following:
  
1. Metadata: name = redis-master and labels app = redis
2. For the deployment spec use a selector and matchLabels 
   1. app_name = redis
   2. role = master
   3. tier = backend
3. 1 replica
4. for the pod template add the following labels:
   1. app_name = redis
   2. role = master
   3. tier = backend
5. The container for the pod template should have:
   1. name = master
   2. use the image  `k8s.gcr.io/redis:e2e`
   3. Have a resource request:
      1. cpu 100m
      2. memort 100Mi
   4. container port = 6379

---
Once your manifest is ready:
-  use `kubectl apply` with the created manifest.
-  verify your pod is up with `kubectl get pods`
-  get the logs via `kubectl logs -f <pod-name>`
---
## Redis Master Service

For the guestbook application to communicate with redis we need to create a service to proxy traffic to the redis master pod.

##### Create a Service in a file called `redis-master-service.yml` that has the following:

1. name = redis-master
2. metadata labels
   1. app_name = redis
   2. role = master
   3. tier = backend
3. the spec should map port 6379 to the port 6379 on the pod
4. Selector
   1. *What should we use as the* `selector`?


---
Once your manifest is ready:
-  use `kubectl apply` with the created manifest.
-  use `kubectl get service` to verify its running
---
## Redis Slaves

We will make our redis highly available by adding replica slaves. We will do this with another Deployment


##### Create a Deployment in a file called `redis-slave-deployment.yml` that has the following:

1. name = redis-slave
2. metadata labels -> app_name = redis
3. For the selector match labels in the deployment-spec:
   1. app_name = redis
   2. role = slave
   3. tier = backend
4. 2 replicas
5. Our pod template should have the following labels:
   1. app_name = redis
   2. role = slave
   3. tier = backend
6. The container should have:
   1. name = slave
   2. image = `gcr.io/google_samples/gb-redisslave:v3`
   3. resource requests just like the master:
      1. cpu = 100m
      2. memory = 100Mi
   4. we need an env variable of name GET_HOSTS_FROM and value 'env'
   5. container port should be 6379

---
Once your manifest is ready:
-  use `kubectl apply` with the created manifest.
-  use `kubectl get pods` to verify
---

## Redis Slave Service

Now that we have made the slaves, lets also expose them with a service.

#### Create a Service in a file called `redis-slave-service.yml`

*Using what we know and what we did in the master service manifest, create your service and apply it*

---
Once your manifest is ready:
-  use `kubectl apply` with the created manifest.
-  use `kubectl get service` to verify its running
---

## Front End Deployment

The guestbook application has a web frontend. It is preconfigured to connect to the redis-master service for writes and redis-slave for reads.

use the below manifest for your front end - save it as `guestbook-deployment.yml`:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app_name: guestbook
spec:
  selector:
    matchLabels:
      app_name: guestbook
      tier: frontend
  replicas: 3
  template:
    metadata:
      labels:
        app_name: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google-samples/gb-frontend:v4
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: GET_HOSTS_FROM
          value: env
        ports:
        - containerPort: 80
```

---
Once your manifest is ready:
-  use `kubectl apply` with the created manifest.
-  use `kubectl get pods` to verify its running
---

## Front End Service 

Finally, write a NodePort service that will expose the pods at port 80. Remember to use the correct selector to select the right `tier` and `app_name`

create and save this as `guestbook-service.yml` then apply it.

___
Once you have applied the manifest:
- verify using `kubectl get services`
- run `minikube service frontend --url`
- Copy and pasete the returned  url in your browser to view the guestbook
- Test the guestbook!
- Try scaling the deployment
  - `kubectl scale deployment frontend --replicas=5`
___

## cleanup

```
kubectl delete deployment -l app=redis
kubectl delete service -l app=redis
kubectl delete deployment -l app=guestbook
kubectl delete service -l app=guestbook
```
  
