# TASK 4 report

## Task:
- Create users deploy_view and deploy_edit. Give the user deploy_view rights only to view deployments, pods. Give the user deploy_edit full rights to the objects deployments, pods.
- Create namespace prod. Create users prod_admin, prod_view. Give the user prod_admin admin rights on ns prod, give the user prod_view only view rights on namespace prod.
- Create a serviceAccount sa-namespace-admin. Grant full rights to namespace default. Create context, authorize using the created sa, check accesses.

### part1
```bash
bash-3.2$ openssl genrsa -out deploy_view.key 2048
Generating RSA private key, 2048 bit long modulus
.......................................................................................+++
...............+++
e is 65537 (0x10001)
bash-3.2$
bash-3.2$ openssl req -new -key deploy_view.key -out deploy_view.csr -subj "/CN=deploy_view"
bash-3.2$
bash-3.2$ ls -la
total 24
drwxr-xr-x  5 olgaukhina  staff   160 Apr 10 22:47 .
drwxr-xr-x  6 olgaukhina  staff   192 Apr 10 22:32 ..
-rw-r--r--@ 1 olgaukhina  staff   606 Apr 10 22:36 README.md
-rw-r--r--  1 olgaukhina  staff   895 Apr 10 22:47 deploy_view.csr
-rw-r--r--  1 olgaukhina  staff  1679 Apr 10 22:45 deploy_view.key
bash-3.2$ 
bash-3.2$ openssl x509 -req -in deploy_view.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out deploy_view.crt -days 500
Signature ok
subject=/CN=deploy_view
Getting CA Private Key
bash-3.2$
bash-3.2$ kubectl config set-credentials deploy_view --client-certificate=deploy_view.crt --client-key=deploy_view.key
User "deploy_view" set.
bash-3.2$ kubectl config set-context deploy_view --cluster=minikube --user=deploy_view
Context "deploy_view" created.
bash-3.2$
bash-3.2$ kubectl apply -f role_deploy_view.yaml
clusterrole.rbac.authorization.k8s.io/deploy_view created
bash-3.2$ cat role_deploy_view.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: deploy_view
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["deployments", "pods"]
  verbs: ["get", "watch", "list"]


bash-3.2$ kubectl apply -f  cluster_role_binding_deploy_view.yaml
clusterrolebinding.rbac.authorization.k8s.io/deploy_view created
bash-3.2$ cat cluster_role_binding_deploy_view.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: deploy_view
subjects:
- kind: User
  name: deploy_view
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: deploy_view
  apiGroup: rbac.authorization.k8s.io


 bash-3.2$ kubectl get pods --all-namespaces
NAMESPACE       NAME                                       READY   STATUS      RESTARTS      AGE
ingress-nginx   ingress-nginx-admission-create-rmptn       0/1     Completed   0             22h
ingress-nginx   ingress-nginx-admission-patch-ct6hz        0/1     Completed   1             22h
ingress-nginx   ingress-nginx-controller-cc8496874-5pfgj   1/1     Running     0             22h
kube-system     coredns-64897985d-h668q                    1/1     Running     0             23h
kube-system     etcd-minikube                              1/1     Running     0             23h
kube-system     kube-apiserver-minikube                    1/1     Running     0             23h
kube-system     kube-controller-manager-minikube           1/1     Running     0             23h
kube-system     kube-proxy-bzs64                           1/1     Running     0             23h
kube-system     kube-scheduler-minikube                    1/1     Running     0             23h
kube-system     storage-provisioner                        1/1     Running     1 (23h ago)   23h
task1           nginx-deployment-9456bbbf9-q9l25           1/1     Running     0             23h
task1           nginx-deployment-9456bbbf9-xt5fj           1/1     Running     0             23h
task2           nginx-base-6c88f974cf-s9628                1/1     Running     0             20h
task2           nginx-canary-745d845c87-hbldh              1/1     Running     0             20h
task3           minio-69f75574bd-np85j                     1/1     Running     0             18h
bash-3.2$ kubectl get nodes
Error from server (Forbidden): nodes is forbidden: User "deploy_view" cannot list resource "nodes" in API group "" at the cluster scope    
bash-3.2$ openssl genrsa -out deploy_edit.key 2048    
bash-3.2$ openssl req -new -key deploy_edit.key -out deploy_edit.csr -subj "/CN=deploy_edit"
bash-3.2$ openssl x509 -req -in deploy_edit.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out deploy_edit.crt -days 500
bash-3.2$ kubectl config use-context minikube
Switched to context "minikube".
bash-3.2$ kubectl apply -f cluster_role_deploy_edit.yaml
clusterrole.rbac.authorization.k8s.io/deploy_edit created
bash-3.2$ kubectl apply -f cluster_role_binding_deploy_edit.yaml
clusterrolebinding.rbac.authorization.k8s.io/deploy_edit created
bash-3.2$ kubectl config set-credentials deploy_edit --client-certificate=deploy_edit.crt --client-key=deploy_edit.key
User "deploy_edit" set.
bash-3.2$
bash-3.2$
bash-3.2$ kubectl config set-context deploy_edit --cluster=minikube --user=deploy_edit
Context "deploy_edit" created.
bash-3.2$
bash-3.2$
bash-3.2$ kubectl config use-context deploy_edit
Switched to context "deploy_edit".
bash-3.2$ kubectl create deployment nginx --image=nginx -n prod
deployment.apps/nginx created
```        
### part2

```
bash-3.2$ kubectl create ns prod
namespace/prod created
bash-3.2$ openssl genrsa -out prod_admin.key 2048
bash-3.2$ openssl req -new -key prod_admin.key -out prod_admin.csr -subj "/CN=prod_admin"
bash-3.2$ openssl req -new -key prod_admin.key -out prod_admin.csr -subj "/CN=prod_admin"
bash-3.2$ openssl x509 -req -in prod_admin.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out prod_admin.crt -days 500
Signature ok
subject=/CN=prod_admin
Getting CA Private Key
bash-3.2$ kubectl apply -f role_binding_prod_admin.yaml
rolebinding.rbac.authorization.k8s.io/prod_admin created
bash-3.2$ kubectl config use-context prod_admin
Switched to context "prod_admin".
bash-3.2$ kubectl create deployment nginx --image=nginx -n default
error: failed to create deployment: deployments.apps is forbidden: User "prod_admin" cannot create resource "deployments" in API group "apps" in the namespace "default"
bash-3.2$ kubectl create deployment nginx --image=nginx -n prod
deployment.apps/nginx created
bash-3.2$ openssl genrsa -out prod_viewer.key 2048
bash-3.2$ openssl req -new -key prod_viewer.key -out prod_viewer.csr -subj "/CN=prod_viewer"
bash-3.2$ openssl x509 -req -in prod_viewer.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out prod_viewer.crt -days 500
Signature ok
subject=/CN=prod_viewer
Getting CA Private Key
bash-3.2$ kubectl apply -f  role_binding_prod_viewer.yaml
rolebinding.rbac.authorization.k8s.io/prod_viewer created
bash-3.2$ kubectl config set-credentials prod_viewer --client-certificate=prod_viewer.crt --client-key=prod_viewer.key
User "prod_viewer" set.
bash-3.2$ kubectl config set-context prod_viewer --cluster=minikube --user=prod_viewer
Context "prod_viewer" created.
bash-3.2$ kubectl config use-context prod_viewer
Switched to context "prod_viewer".
bash-3.2$ kubectl get po -n prod
NAME                     READY   STATUS    RESTARTS   AGE
nginx-85b98978db-mb2jd   1/1     Running   0          3m57s
bash-3.2$ kubectl delete po nginx-85b98978db-mb2jd  -n prod
Error from server (Forbidden): pods "nginx-85b98978db-mb2jd" is forbidden: User "prod_viewer" cannot delete resource "pods" in API group "" in the namespace "prod"
```
### part3

```
bash-3.2$ kubectl create sa sa-namespace-admin -n default
bash-3.2$ kubectl apply -f role_sa_namespace_admin.yaml
bash-3.2$ kubectl get secret sa-namespace-admin-token-r48j2 -n default -o yaml
apiVersion: v1
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCakNDQWU2Z0F3SUJBZ0lCQVRBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwdGFXNXAKYTNWaVpVTkJNQjRYRFRJeU1EUXdPVEUwTVRFME9Gb1hEVE15TURRd056RTBNVEUwT0Zvd0ZURVRNQkVHQTFVRQpBeE1LYldsdWFXdDFZbVZEUVRDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTUVlCk8vZkwxQVdYQXpSQld0eFg0UERCNzhqcHN5KzQ3WnRlWnN5TUlYUHVqVnlSLzVJcGNjWENJVmFrWVBiLzJ6UTYKREtuc2t3Q2xBUmYyekNIeHVXTUlCU1ZiRlZxMVYvQ05vTmc2cGQveFZ6clUrRGh1K3JSSjNBZ1czODIwVnVyNgpwR1BkRUtZT1VYNUZoNkw5UGJzdEd0VUFpV1gxWlI4TEoxN3h4RzUxbDZMVzlwOTBhQVQzUlNtRGFxSS9hKytECjJUMEppcjgwUTVtY1MvdWdUVnJtaG9xT2FNdm9lY3dpT1kzU2Rtc0JpNGd0dytDZ0VUdXFrUis1MUEwS3dZMWEKSDNtVDY5bCt3YVM5U0g3NzVXcnB6TVV6WVI2YlVyRTY4cDYveDJXUHlnSDl4OUJ6UmlpMC9TamlES2FNQUw3agpYUDM3MUxhcnNkdEFOTFRvdkFjQ0F3RUFBYU5oTUY4d0RnWURWUjBQQVFIL0JBUURBZ0trTUIwR0ExVWRKUVFXCk1CUUdDQ3NHQVFVRkJ3TUNCZ2dyQmdFRkJRY0RBVEFQQmdOVkhSTUJBZjhFQlRBREFRSC9NQjBHQTFVZERnUVcKQkJTZUh3K0RsV0VkQzFyMm1VeDBxNWFGRFlXN0dUQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFhMkxjV2JLeQorOTNFU2JjK1orbHFCVVhzZ0F1NFVFUk9IcVlqd1A0OVpwYTV0WHRIRG45TnNhR1FMT3pGZDJMb2ZuV1F1Qll2CnRKZFdSbU5RaGVzZ1k2Q0FmdElTdWpLMitXRU02U3d3UmtMenluSG9oN29KUCtJcXN5NHA0azhoODljRUxIdVcKOWhRNEhicEdZekVkdU5GanZQd2dmK2pUcVIzZXdRbS9FekpHSytVRzc1VHVWVFJwZC9zdGFTM055R25KUnBuUgpQSndDU2tBeFpyTGxJOW0xWEI4RnZYdlRiWDRGY0VmNE81cGNsaXV6UnAzc21vQURuSXMzdnkxMmd2ZitoTlRKCktjSmlyeUROYVZlVW1RWThwM3JiOHVURnFqMmF5eEhmTzhNWHBNd2VWK2ZWMFNaNG40dHBIbU8yenk5RFhiYTYKdTViR1h6cms0R3J1Q1E9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  namespace: ZGVmYXVsdA==
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklqRlJjMWg1V2kxRWQzaGlMWEZFV1VOT00wbEdjbTQzV1cxUlpYcE5WamhFVXpoS1lXbGpReTA0UmxVaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUprWldaaGRXeDBJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpXTnlaWFF1Ym1GdFpTSTZJbk5oTFc1aGJXVnpjR0ZqWlMxaFpHMXBiaTEwYjJ0bGJpMXlORGhxTWlJc0ltdDFZbVZ5Ym1WMFpYTXVhVzh2YzJWeWRtbGpaV0ZqWTI5MWJuUXZjMlZ5ZG1salpTMWhZMk52ZFc1MExtNWhiV1VpT2lKellTMXVZVzFsYzNCaFkyVXRZV1J0YVc0aUxDSnJkV0psY201bGRHVnpMbWx2TDNObGNuWnBZMlZoWTJOdmRXNTBMM05sY25acFkyVXRZV05qYjNWdWRDNTFhV1FpT2lKaE5XWmlaR1k0WWkwNE9XUTJMVFF3T0RFdFlqTTRNeTAxT1RWa1lXTTNPVFUxTlRJaUxDSnpkV0lpT2lKemVYTjBaVzA2YzJWeWRtbGpaV0ZqWTI5MWJuUTZaR1ZtWVhWc2REcHpZUzF1WVcxbGMzQmhZMlV0WVdSdGFXNGlmUS54cmZqVFJkYVIxQ2laeXREYmU0RHlFTVdJVnoyWWo5R1cxdWk5X01IdjQ3SlFSd2hoMmVYWUcxSzhsdTJISE1JRXBJMGhmdXFYRzVOcGhuWDNqS0FDRHZvekQ1T0FreW15c19HSjNMNFlaajRyejRlMDJ0LWpTOXlaNWVvX1FCNExGOEZDUWstY083OTc1SzNMTzNTczNqUGx4RXl4YTZXaTk1TmlJay1UR2xGRnU4MkJ4SXc3VnY0MjlUWjJyMGJBLXl3NUdZUU53N3FaeFZKWVQ2bzUxdEJlbmJGNzhrT3VIdGRqUVNwNEpHalVJNE5sbjR1Q3YydTlkWkRFek1SQThpM2NFaDNFY3F0S1ZRNFpTUzRmRmJMSHdDRVhadkJocGRNc3Rqc3RXeVJpTWF0OWY3RGN0dlpyR2RuT21LTDlVc0RCZHVVSm9vNVNOR2Q3QTFxV3c=
kind: Secret
metadata:
  annotations:
    kubernetes.io/service-account.name: sa-namespace-admin
    kubernetes.io/service-account.uid: a5fbdf8b-89d6-4081-b383-595dac795552
  creationTimestamp: "2022-04-11T16:24:49Z"
  name: sa-namespace-admin-token-r48j2
  namespace: default
  resourceVersion: "51715"
  uid: bc2cf104-682e-4771-adaf-9f307703950d
type: kubernetes.io/service-account-token
bash-3.2$ echo -n "ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklqRlJjMWg1V2kxRWQzaGlMWEZFV1VOT00wbEdjbTQzV1cxUlpYcE5WamhFVXpoS1lXbGpReTA0UmxVaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUprWldaaGRXeDBJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpXTnlaWFF1Ym1GdFpTSTZJbk5oTFc1aGJXVnpjR0ZqWlMxaFpHMXBiaTEwYjJ0bGJpMXlORGhxTWlJc0ltdDFZbVZ5Ym1WMFpYTXVhVzh2YzJWeWRtbGpaV0ZqWTI5MWJuUXZjMlZ5ZG1salpTMWhZMk52ZFc1MExtNWhiV1VpT2lKellTMXVZVzFsYzNCaFkyVXRZV1J0YVc0aUxDSnJkV0psY201bGRHVnpMbWx2TDNObGNuWnBZMlZoWTJOdmRXNTBMM05sY25acFkyVXRZV05qYjNWdWRDNTFhV1FpT2lKaE5XWmlaR1k0WWkwNE9XUTJMVFF3T0RFdFlqTTRNeTAxT1RWa1lXTTNPVFUxTlRJaUxDSnpkV0lpT2lKemVYTjBaVzA2YzJWeWRtbGpaV0ZqWTI5MWJuUTZaR1ZtWVhWc2REcHpZUzF1WVcxbGMzQmhZMlV0WVdSdGFXNGlmUS54cmZqVFJkYVIxQ2laeXREYmU0RHlFTVdJVnoyWWo5R1cxdWk5X01IdjQ3SlFSd2hoMmVYWUcxSzhsdTJISE1JRXBJMGhmdXFYRzVOcGhuWDNqS0FDRHZvekQ1T0FreW15c19HSjNMNFlaajRyejRlMDJ0LWpTOXlaNWVvX1FCNExGOEZDUWstY083OTc1SzNMTzNTczNqUGx4RXl4YTZXaTk1TmlJay1UR2xGRnU4MkJ4SXc3VnY0MjlUWjJyMGJBLXl3NUdZUU53N3FaeFZKWVQ2bzUxdEJlbmJGNzhrT3VIdGRqUVNwNEpHalVJNE5sbjR1Q3YydTlkWkRFek1SQThpM2NFaDNFY3F0S1ZRNFpTUzRmRmJMSHdDRVhadkJocGRNc3Rqc3RXeVJpTWF0OWY3RGN0dlpyR2RuT21LTDlVc0RCZHVVSm9vNVNOR2Q3QTFxV3c=" | base64 -D
eyJhbGciOiJSUzI1NiIsImtpZCI6IjFRc1h5Wi1Ed3hiLXFEWUNOM0lGcm43WW1RZXpNVjhEUzhKYWljQy04RlUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6InNhLW5hbWVzcGFjZS1hZG1pbi10b2tlbi1yNDhqMiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJzYS1uYW1lc3BhY2UtYWRtaW4iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJhNWZiZGY4Yi04OWQ2LTQwODEtYjM4My01OTVkYWM3OTU1NTIiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpzYS1uYW1lc3BhY2UtYWRtaW4ifQ.xrfjTRdaR1CiZytDbe4DyEMWIVz2Yj9GW1ui9_MHv47JQRwhh2eXYG1K8lu2HHMIEpI0hfuqXG5NphnX3jKACDvozD5OAkymys_GJ3L4YZj4rz4e02t-jS9yZ5eo_QB4LF8FCQk-cO7975K3LO3Ss3jPlxEyxa6Wi95NiIk-TGlFFu82BxIw7Vv429TZ2r0bA-yw5GYQNw7qZxVJYT6o51tBenbF78kOuHtdjQSp4JGjUI4Nln4uCv2u9dZDEzMRA8i3cEh3EcqtKVQ4ZSS4fFbLHwCEXZvBhpdMstjstWyRiMat9f7DctvZrGdnOmKL9UsDBduUJoo5SNGd7A1qWwbash-3.2$
bash-3.2$
bash-3.2$
bash-3.2$ kubectl --token=eyJhbGciOiJSUzI1NiIsImtpZCI6IjFRc1h5Wi1Ed3hiLXFEWUNOM0lGcm43WW1RZXpNVjhEUzhKYWljQy04RlUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6InNhLW5hbWVzcGFjZS1hZG1pbi10b2tlbi1yNDhqMiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJzYS1uYW1lc3BhY2UtYWRtaW4iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJhNWZiZGY4Yi04OWQ2LTQwODEtYjM4My01OTVkYWM3OTU1NTIiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpzYS1uYW1lc3BhY2UtYWRtaW4ifQ.xrfjTRdaR1CiZytDbe4DyEMWIVz2Yj9GW1ui9_MHv47JQRwhh2eXYG1K8lu2HHMIEpI0hfuqXG5NphnX3jKACDvozD5OAkymys_GJ3L4YZj4rz4e02t-jS9yZ5eo_QB4LF8FCQk-cO7975K3LO3Ss3jPlxEyxa6Wi95NiIk-TGlFFu82BxIw7Vv429TZ2r0bA-yw5GYQNw7qZxVJYT6o51tBenbF78kOuHtdjQSp4JGjUI4Nln4uCv2u9dZDEzMRA8i3cEh3EcqtKVQ4ZSS4fFbLHwCEXZvBhpdMstjstWyRiMat9f7DctvZrGdnOmKL9UsDBduUJoo5SNGd7A1qWw get nodes
```

