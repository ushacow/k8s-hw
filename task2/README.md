# TASK 2 report

## Task:
- In Minikube in namespace kube-system, there are many different pods running. Your task is to figure out who creates them, and who makes sure they are running (restores them after deletion).
- Implement Canary deployment of an application via Ingress. Traffic to canary deployment should be redirected if you add "canary:always" in the header, otherwise it should go to regular deployment. Set to redirect a percentage of traffic to canary deployment.

### Checking owners of kube-system pods
coredns-... pod is owned by ReplicaSet/deployment coredns which have restartPolicy - always
kube-proxy-... pod is owned by DaemonSet kube-proxy which have restartPolicy - always
etcd-minikube and all other pods available at the moment are owned directly by Node resource and have restartPolicy - always
```bash
bash-3.2$ kubectl get po -n kube-system
NAME                               READY   STATUS    RESTARTS      AGE
coredns-64897985d-h668q            1/1     Running   0             28m
etcd-minikube                      1/1     Running   0             28m
kube-apiserver-minikube            1/1     Running   0             28m
kube-controller-manager-minikube   1/1     Running   0             28m
kube-proxy-bzs64                   1/1     Running   0             28m
kube-scheduler-minikube            1/1     Running   0             28m
storage-provisioner                1/1     Running   1 (28m ago)   28m
bash-3.2$
bash-3.2$
bash-3.2$ kubectl get rs -n kube-system
NAME                DESIRED   CURRENT   READY   AGE
coredns-64897985d   1         1         1       28m
bash-3.2$ 
bash-3.2$ kubectl get deploy -n kube-system
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
coredns   1/1     1            1           29m
bash-3.2$
bash-3.2$ kubectl get deploy coredns -n kube-system -o yaml | grep restartPolicy
      restartPolicy: Always
bash-3.2$      
bash-3.2$ kubectl get ds -n kube-system
NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   29m
bash-3.2$ 
bash-3.2$ kubectl get ds kube-proxy -n kube-system -o yaml | grep restartPolicy
      restartPolicy: Always
bash-3.2$ 
bash-3.2$ kubectl get -o yaml pod etcd-minikube -n kube-system | grep ownerRef -A5
  ownerReferences:
  - apiVersion: v1
    controller: true
    kind: Node
    name: minikube
    uid: de232a58-7795-45e5-aa91-75845daf1dea
bash-3.2$   
bash-3.2$ kubectl get -o yaml pod etcd-minikube -n kube-system | grep restartPolicy
  restartPolicy: Always
bash-3.2$
bash-3.2$ # same staff for other kube-system pods
```

### Implementing canary deployment - part 1, setup Ingress
```bash
bash-3.2$ minikube addons enable ingress
💡  After the addon is enabled, please run "minikube tunnel" and your ingress resources would be available at "127.0.0.1"
    ▪ Using image k8s.gcr.io/ingress-nginx/controller:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
bash-3.2$
bash-3.2$
bash-3.2$ kubectl get pods -n ingress-nginx
NAME                                       READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-rmptn       0/1     Completed   0          7m21s
ingress-nginx-admission-patch-ct6hz        0/1     Completed   1          7m21s
ingress-nginx-controller-cc8496874-5pfgj   1/1     Running     0          7m21s
```

### Implementing canary deployment - part 2, prepare basic deployment and the canary one
```bash
bash-3.2$ kubectl create namespace task2
namespace/task2 created
bash-3.2$
bash-3.2$ cat nginx-deploy-base.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-base
  namespace: task2
  labels:
    app: nginx-base
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-base
  template:
    metadata:
      labels:
        app: nginx-base
    spec:
      containers:
      - name: nginx
        image: marfich/mynginx:base
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service-base
  namespace: task2
spec:
  type: ClusterIP
  selector:
    app: nginx-base
  ports:
  - port: 8001
    targetPort: 80
bash-3.2$
bash-3.2$ kubectl create -f nginx-deploy-base.yaml
deployment.apps/nginx-base created
service/nginx-service-base created
bash-3.2$
bash-3.2$ kubectl get po -n task2
NAME                          READY   STATUS    RESTARTS   AGE
nginx-base-6c88f974cf-s9628   1/1     Running   0          44s
bash-3.2$
bash-3.2$ cat nginx-deploy-canary.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-canary
  namespace: task2
  labels:
    app: nginx-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-canary
  template:
    metadata:
      labels:
        app: nginx-canary
    spec:
      containers:
      - name: nginx
        image: marfich/mynginx:canary
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service-canary
  namespace: task2
spec:
  type: ClusterIP
  selector:
    app: nginx-canary
  ports:
  - port: 8002
    targetPort: 80
bash-3.2$
bash-3.2$ kubectl create -f nginx-deploy-canary.yaml
deployment.apps/nginx-canary created
service/nginx-service-canary created
bash-3.2$
bash-3.2$ kubectl get po -n task2
NAME                            READY   STATUS    RESTARTS   AGE
nginx-base-6c88f974cf-s9628     1/1     Running   0          6m50s
nginx-canary-745d845c87-hbldh   1/1     Running   0          7s
```
### Implementing canary deployment - part 3, creating Ingress and testing
```bash
bash-3.2$ cat ingress-basic.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-basic
  namespace: task2
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /nginx
        pathType: Prefix
        backend:
          service:
            name: nginx-service-base
            port:
              number: 8001
bash-3.2$              
bash-3.2$ kubectl create -f ingress-basic.yaml
ingress.networking.k8s.io/ingress-basic created
bash-3.2$
bash-3.2$ cat ingress-canary.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-canary
  namespace: task2
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-by-header: "canary"
    nginx.ingress.kubernetes.io/canary-by-header-value: "always"
    nginx.ingress.kubernetes.io/canary-weight: "20"
spec:
  rules:
  - http:
      paths:
      - path: /nginx
        pathType: Prefix
        backend:
          service:
            name: nginx-service-canary
            port:
              number: 8002
bash-3.2$
bash-3.2$ kubectl create -f ingress-canary.yaml
ingress.networking.k8s.io/ingress-canary created
bash-3.2$
bash-3.2$ minikube ssh
docker@minikube:~$ # test without header for ~ 80/20 percentage
docker@minikube:~$ for i in $(seq 1 10); do curl localhost/nginx ; done
Basic version
Canary version
Basic version
Basic version
Basic version
Basic version
Basic version
Basic version
Canary version
Basic version
docker@minikube:~$
docker@minikube:~$ # test with header for 100% canary response
docker@minikube:~$ for i in $(seq 1 10); do curl -H "canary:always" localhost/nginx ; done
Canary version
Canary version
Canary version
Canary version
Canary version
Canary version
Canary version
Canary version
Canary version
Canary version
```