![RX-M, llc.](https://rx-m.com/rxm-cnc.svg)


# Kubernetes


## Deployments and ReplicaSets Challenge Solutions

### 11.1. CHALLENGE: labels

Recreate the website deployment, the cache-pod, and dev-pod resources from earlier in the lab:

```
~$ cd ~/dep

~/dep kubectl apply -f ~/dep/mydep.yaml

deployment.apps/website created

~/dep kubectl run cache-pod -l app=cache --image redis

pod/cache-pod created

~/dep kubectl run dev-pod -l targetenv=dev --image httpd:2.2

pod/dev-pod created

~/dep
```

- Run a pod named `labelpod` with the label `targetenv=prod` and a container from `nginx:1.7.9`

```
~/dep$ kubectl run labelpod -l targetenv=prod --image nginx:1.7.9

pod/labelpod created

~/dep$
```

- Enter a command to display all of the pods with either the “demo” or “prod” value for targetenv

```
~/dep kubectl get po -l "targetenv in (prod,demo)" --show-labels

NAME                       READY   STATUS    RESTARTS   AGE    LABELS
labelpod                   1/1     Running   0          81s    targetenv=prod
website-86895ff58d-5xwj9   1/1     Running   0          100s   app=website,pod-template-hash=86895ff58d,targetenv=demo
website-86895ff58d-d96ts   1/1     Running   0          100s   app=website,pod-template-hash=86895ff58d,targetenv=demo
website-86895ff58d-prccz   1/1     Running   0          100s   app=website,pod-template-hash=86895ff58d,targetenv=demo

~/dep
```

- Find all pods other than those with the “demo” or “prod” value for targetenv

```
~/dep kubectl get po -l "targetenv notin (prod,demo)" --show-labels

NAME        READY   STATUS    RESTARTS   AGE   LABELS
cache-pod   1/1     Running   0          86s   app=cache
dev-pod     1/1     Running   0          84s   targetenv=dev

~/dep
```

- Enter a command to display all of the pods with either the “demo” or “prod” value for targetenv and the app key
  set to website

```
~/dep kubectl get po -l "targetenv in (prod,demo), app=website" --show-labels

NAME                       READY   STATUS    RESTARTS   AGE   LABELS
website-86895ff58d-5xwj9   1/1     Running   0          79s   app=website,pod-template-hash=86895ff58d,targetenv=demo
website-86895ff58d-d96ts   1/1     Running   0          79s   app=website,pod-template-hash=86895ff58d,targetenv=demo
website-86895ff58d-prccz   1/1     Running   0          79s   app=website,pod-template-hash=86895ff58d,targetenv=demo

~/dep
```

- Delete the pods you created for this challenge at the end

```
~/dep kubectl delete deploy/website pod/cache-pod pod/dev-pod pod/labelpod

deployment.apps "website" deleted
pod "cache-pod" deleted
pod "dev-pod" deleted
pod "labelpod" deleted

~/dep
```

### 11.2. Challenge: Deployments

- Create a deployment named `webserver-special`
  - Ensure the deployment is labeled with `app=commerce`, `vendor=student-tech`, `component=webserver`
  - The deployment should maintain 3 pods that have the labels `component=webserver` and `server=nginx` present
  - The containers of pods in the deployment should be created from the `nginx:1.20.1` image

```
~/dep$ nano websvr-spec.yaml && cat $_
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: commerce
    vendor: student-tech
    component: webserver
  name: webserver-special
spec:
  replicas: 3
  selector:
    matchLabels:
      component: webserver      # Must be present in template label
      server: nginx             # Must be present in template label
  template:
    metadata:
      labels:
        component: webserver    # Must be present if included in matchLabels
        server: nginx           # Must be present if included in matchLabels
        app: commerce           # Optional: not all pod labels need to be in matchLabels
    spec:
      containers:
      - image: nginx:1.20.1
        name: nginx
```
```
~/dep$ kubectl apply -f websvr-spec.yaml

deployment.apps/webserver-special created

~/dep$
```

- After creating the deployment, modify it so its pods:
  - run from the `nginx:1.23.2` image
  - Have the environment variable `HTTPD_PROXY=main`
  - Have 5 copies instead of 3

```
~/dep$ nano websvr-spec.yaml && cat $_
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: commerce
    vendor: student-tech
    component: webserver
  name: webserver-special
spec:
  replicas: 5                 # Change this
  selector:
    matchLabels:
      component: webserver
      server: nginx
  template:
    metadata:
      labels:
        component: webserver
        server: nginx
        app: commerce
    spec:
      containers:
      - image: nginx:1.23.2   # Change this
        name: nginx
        env:                  # Add this
        - name: HTTPD_PROXY   # Add this
          value: main         # Add this
```
```
~/dep$ kubectl apply -f websvr-spec.yaml

deployment.apps/webserver-special configured
~/dep$
```

- What is different about the pods now in `kubectl get`?

```
~/dep$ kubectl get pods,rs,deployment

NAME                                     READY   STATUS              RESTARTS   AGE
pod/webserver-special-6b5658457-5fgvx    1/1     Running             0          21s
pod/webserver-special-6b5658457-fxps6    1/1     Running             0          21s
pod/webserver-special-6b5658457-p78xt    1/1     Running             0          21s
pod/webserver-special-6b5658457-rnhks    1/1     Running             0          4s
pod/webserver-special-8568dff779-8nc6k   0/1     ContainerCreating   0          4s
pod/webserver-special-8568dff779-g799l   0/1     ContainerCreating   0          4s
pod/webserver-special-8568dff779-k9f52   0/1     ContainerCreating   0          4s

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/webserver-special-6b5658457    4         4         4       21s
replicaset.apps/webserver-special-8568dff779   3         3         0       4s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webserver-special   4/5     3            4           21s

~/dep$
```

The replicaSet hash is different and there are 5

- Delete the deployment when finished

```
~/dep$ kubectl delete -f websvr-spec.yaml

deployment.apps "webserver-special" deleted

~/dep$
```


_Copyright (c) 2023-2024 RX-M LLC, Cloud Native & AI Training and Consulting, all rights reserved_
