# TASK 3 report

## Task: 
- We published minio "outside" using nodePort. Do the same but using ingress.
- Publish minio via ingress so that minio by ip_minikube and nginx returning hostname (previous job) by path ip_minikube/web are available at the same time.
- Create deploy with emptyDir save data to mountPoint emptyDir, delete pods, check data.
- Optional. Raise an nfs share on a remote machine. Create a pv using this share, create a pvc for it, create a deployment. Save data to the share, delete the deployment, delete the pv/pvc, check that the data is safe.

### Changing volume type to emptyDir 
```bash
bash-3.2$ kubectl create namespace task3
namespace/task3 created
bash-3.2$
bash-3.2$ cat deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: minio
  namespace: task3
  labels:
    app: minio
spec:
  replicas: 1
  selector:
    matchLabels:
      app: minio
  template:
    metadata:
      labels:
        app: minio
    spec:
      containers:
        - name: minio
          image: minio/minio
          args: ["server", "/data", "--console-address", ":9001"]
          ports:
            - containerPort: 9001
          volumeMounts:
            - name: data
              mountPath: "/data"
              readOnly: false
      volumes:
        - name: data
          emptyDir: {}
bash-3.2$
```

### Exposing minio using Ingress
```bash
bash-3.2$ cat service-minio.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: minio-service
  namespace: task3
spec:
  type: ClusterIP
  selector:
    app: minio
  ports:
  - port: 9001
    targetPort: 9001
bash-3.2$
bash-3.2$ kubectl create -f service-minio.yaml
service/minio-service created
bash-3.2$
bash-3.2$ cat ingress-minio.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-minio
  namespace: task3
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: minio-service
            port:
              number: 9001
bash-3.2$
bash-3.2$ kubectl create -f ingress-minio.yaml
ingress.networking.k8s.io/ingress-minio created
```

### Checking concurrent work with the application from task 2
```bash
bash-3.2$ minikube ssh
Last login: Sun Apr 10 19:18:58 2022 from 192.168.49.1
docker@minikube:~$
docker@minikube:~$ curl localhost # minio console response
<!doctype html><html lang="en"><head><meta charset="utf-8"/><base href="/"/><meta content="width=device-width,initial-scale=1" name="viewport"/><meta content="#081C42" media="(prefers-color-scheme: light)" name="theme-color"/><meta content="#081C42" media="(prefers-color-scheme: dark)" name="theme-color"/><meta content="MinIO Console" name="description"/><link href="./styles/root-styles.css" rel="stylesheet"/><link href="./apple-icon-180x180.png" rel="apple-touch-icon" sizes="180x180"/><link href="./favicon-32x32.png" rel="icon" sizes="32x32" type="image/png"/><link href="./favicon-96x96.png" rel="icon" sizes="96x96" type="image/png"/><link href="./favicon-16x16.png" rel="icon" sizes="16x16" type="image/png"/><link href="./manifest.json" rel="manifest"/><link color="#3a4e54" href="./safari-pinned-tab.svg" rel="mask-icon"/><title>MinIO Console</title><script defer="defer" src="./static/js/main.5a7c25ee.js"></script><link href="./static/css/main.90d417ae.css" rel="stylesheet"></head><body><noscript>You need to enable JavaScript to run this app.</noscript><div id="root"><div id="preload"><img src="./images/background.svg"/> <img src="./images/background-wave-orig2.svg"/></div><div id="loader-block"><img src="./Loader.svg"/></div></div></body></html>docker@minikube:~$
docker@minikube:~$
docker@minikube:~$ curl localhost/nginx # task 2 application responses
Basic version
docker@minikube:~$ curl -H "canary:always" localhost/nginx
Canary version
docker@minikube:~$
```

### Checking data persistance with emptyDir
```bash
bash-3.2$ kubectl get po -n task3
NAME                     READY   STATUS    RESTARTS   AGE
minio-69f75574bd-5z8pv   1/1     Running   0          8m10s
bash-3.2$ # saving test data
bash-3.2$ kubectl exec -ti minio-69f75574bd-5z8pv -n task3 -- bash
[root@minio-69f75574bd-5z8pv /]# echo test > /data/file1
[root@minio-69f75574bd-5z8pv /]# cat /data/file1
test
[root@minio-69f75574bd-5z8pv /]#
[root@minio-69f75574bd-5z8pv /]# exit
exit
bash-3.2$ # deleting pod
bash-3.2$ kubectl delete po minio-69f75574bd-5z8pv -n task3
pod "minio-69f75574bd-5z8pv" deleted
bash-3.2$ # checking successful recreation
bash-3.2$ kubectl get po -n task3
NAME                     READY   STATUS    RESTARTS   AGE
minio-69f75574bd-np85j   1/1     Running   0          9s
bash-3.2$ # checking lost of data from emptyDir volume
bash-3.2$ kubectl exec -ti minio-69f75574bd-np85j -n task3 -- bash
[root@minio-69f75574bd-np85j /]#
[root@minio-69f75574bd-np85j /]# cat /data/file1
cat: /data/file1: No such file or directory
[root@minio-69f75574bd-np85j /]#
```