## Task 1 - 

### 1 - Enter command for creation of pod_1.yml

```
kubectl create -f pod_1.yml
```

### 2 - Enter command used to verify the pod_1.yml (describe or proxy with url)

```
kubectl describe pod pod-nginx
kubectl proxy
curl http://127.0.0.1:8001/api/v1/namespaces/dev/pods/pod-nginx/proxy/
```

### 3 - Describe how the containers are working together after applying pod_2.yml

```
Within the same POD, one container is running a continious bash loop that is writing an HTML timestamp string to a file.  This file is written to a volume shared by both containers and can be seen via a HTTP server on the second container.
```

### 4 - What is the command used to add labels to the pod

```
enter command here
```


### 5 - What is the command used to remove labels from the pod

```
enter command here
```

### 6 - What is the command used to select pods using labels with equality

```
enter command here
```

### 7 - What is the command used to select pods using the set selector

```
enter command here
```


### 8 - What do you see and what is happening after exposing via ClusterIP?

```
enter command here
```

### 8 - What is happening? How is the nodeport behaving different? Why?

```
enter command here
```
