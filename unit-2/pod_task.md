## Task 1 - 

### 1 - Enter command for creation of pod_1.yml

```
kubectl create -f pod_1.yml
```

### 2 - Enter command used to verify the pod_1.yml (describe or proxy with url)

```
kubectl describe pod pod-nginx
kubectl proxy; curl http://127.0.0.1:8001/api/v1/namespaces/dev/pods/pod-nginx/proxy/
```

### 3 - Describe how the containers are working together after applying pod_2.yml

```
Within the same POD, one container is running a continious bash loop that is writing an HTML timestamp string to a file.  This file is written to a volume shared by both containers and can be seen via a HTTP server on the second container.
```

### 4 - What is the command used to add labels to the pod

```
kubectl label pods labeled-pod app_name=nginx env=dev
```

### 5 - What is the command used to remove labels from the pod

```
kubectl label pods labeled-pod app_name- env-
```

### 6 - What is the command used to select pods using labels with equality

```
kubectl get pods -l app_name==nginx
```

### 7 - What is the command used to select pods using the set selector

```
kubectl get pods -l 'app_name in (nginx),env notin (prod)'
```

### 8 - What do you see and what is happening after exposing via ClusterIP?

```
I see nothing because it's not load balencing like it should, my browser never changes from the nginx welcome page!
```

### 8 - What is happening? How is the nodeport behaving different? Why?

```
It is using the 192.168.49.0\24 subnet; since I don't know much about this yet! I would imagine that is the clusters external IP?  I would imagine because I told it to without expressing an IP range, I have not read that chapter yet.
```
