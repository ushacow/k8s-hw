# TASK 1 report

## Task:
Create a deployment nginx. Set up two replicas. Remove one of the pods, see what happens.

### Creating namespace and deployment
```bash
bash-3.2$ kubectl create namespace task1
namespace/task1 created
bash-3.2$
bash-3.2$ cat nginx-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: task1
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
bash-3.2$
bash-3.2$ kubectl create -f nginx-deploy.yaml
deployment.apps/nginx-deployment created
```
### Checking deployment and pod status
```bash
bash-3.2$ kubectl get po -n task1
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-9456bbbf9-4ln4f   1/1     Running   0          58s
nginx-deployment-9456bbbf9-q9l25   1/1     Running   0          58s
bash-3.2$
bash-3.2$ kubectl get deploy -n task1
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           70s
```

### Deleting pod and it is automatically recreated
```bash
bash-3.2$ kubectl delete po nginx-deployment-9456bbbf9-4ln4f -n task1
pod "nginx-deployment-9456bbbf9-4ln4f" deleted
bash-3.2$
bash-3.2$ kubectl get deploy -n task1
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           4m56s
bash-3.2$
bash-3.2$ kubectl get po -n task1
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-9456bbbf9-q9l25   1/1     Running   0          5m1s
nginx-deployment-9456bbbf9-xt5fj   1/1     Running   0          9s
```
