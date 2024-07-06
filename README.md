# Выполнено ДЗ № 1

 - [X] Основное ДЗ
 - [X] Задание со *

## В процессе сделано:
```
 - Created vm in yandex cloud
 - Created cluster minikube version: v1.32.0
 - Installed kubectl
 - Created docker image nginx : tulamelki/neweng (with port 8000 and edit config nginx)
 - Created Namespace: homework (manifest namespace.yaml)
 - Created Pod (static-web)
 - Created init container (image: busybox)
   - Share volume without two containers( for web-nginx /homework and for init /init)
   - Init container download page index.html and put in share volume
 - Before close container web-nginx preStop removed index.html
```
## Как запустить проект:
```
 - minikube start
 - kubectl cluster-info
   * Kubernetes control plane is running at https://192.168.148.2:8443
   * CoreDNS is running at https://192.168.148.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
 - kubectl create -f namespace.yaml
 - kubectl get namespace
    NAME              STATUS   AGE
    default           Active   8d
  * homework          Active   5d19h
    kube-node-lease   Active   8d
    kube-public       Active   8d
    kube-system       Active   8d
- kubectl apply -f pod.yaml --namespace=homework
   kubectl get pods --namespace=homework
   NAME         READY   STATUS    RESTARTS   AGE
   static-web   1/1     Running   0          38m
- kubectl describe pod static-web --namespace=homework`
```

```
Name:             static-web
Namespace:        homework
Priority:         0
Service Account:  default
Node:             minikube/192.168.148.2
Start Time:       Sun, 21 Jan 2024 14:18:33 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.0.23

- kubectl exec -it static-web --namespace=homework  bash
```
## Как проверить работоспособность:
 ```
 - netstat -tuln
         
   Active Internet connections (only servers)
   Proto Recv-Q Send-Q Local Address           Foreign Address         State
   tcp        0      0 0.0.0.0:8000            0.0.0.0:*               LISTEN
   tcp6       0      0 :::8000                 :::*                    LISTEN     

 - curl http://localhost:8000
 - Check init download index.html and save share volume
 - Check page: 
<!DOCTYPE HTML>
<html lang="en">
<head>
<!-- THIS IS A COMMENT -->
<title>Sample Web Page</title>
<META charset="utf-8">
<META name="viewport"
 content="width=device-width, initial-scale=1.0">
<style>
blockquote { margin-left:20px; margin-right:5px }
pre { overflow-x:auto }
.tt { font-family:monospace }
.nowrap { white-space:nowrap }
.example { font-family:monospace; white-space:pre; overflow-x:auto; }
h3 { border-top:1px solid grey }
blockquote { margin-top:0; margin-bottom:0 }
table.compact { border-collapse:collapse }
table.compact th { text-align:left; background:#eeeeee }
table.compact td,th { padding:0 4px 0 8px; border:1px solid grey }
</style>
</head>
.....
```
 ## PR checklist:
 - [ ] Выставлен label с темой домашнего задания



# Выполнено ДЗ № 2 

 - [X] Основное ДЗ
 - [X] Задание со *

## В процессе сделано:
```
    Create manifest namespace.yaml namespace=homework
    Create deployment.yaml
    Create deploy 3 pods (nginx)
    Create readiness, it check file /homework/index.html (every 4 seconds cat file)
    Create RollingUpdate,During the update process, a maximum of 1 pod
    Create labels homework=true for node minikube and add nodeSelector - homework: "true"
```
- Create manifest namespace.yaml namespace=homework
## Как запустить проект:
```
apiVersion: v1
kind: Namespace
metadata:
  name: homework
```
- Create manifest namespace.yaml namespace=homework
```
- kubectl create -f namespace.yaml
- kubectl get namespace
    NAME STATUS AGE
    default Active 27h
    homework Active 4m58s
    kube-node-lease Active 27h
    kube-public Active 27h
    kube-system Active 27h
      
```
- Create deployment.yaml
```
- kubectl apply -f deployment.yaml --namespace=homework

apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      nodeSelector:
        homework: "true"
      containers:
      - name: nginx           
        image: tulamelki/neweng
        lifecycle:
          preStop:                  
            exec:
              command: [ 'sh', '-c', 'rm /homework/index.html'] 
        ports:
        - containerPort: 8000
        volumeMounts:
        - mountPath: /homework     
          name: volume
        readinessProbe:
          exec:
            command: ['sh', '-c', 'cat /homework/index.html']
          initialDelaySeconds: 7
          periodSeconds: 4
      initContainers:         
      - name: init-containers      
        image: busybox             
        command: ['sh', '-c', 'wget http://www.columbia.edu/~fdc/sample.html -O /init/index.html']
        volumeMounts:
        - mountPath: /init         
          name: volume
      volumes:
      - name: volume              
        emptyDir:
          sizeLimit: 800Mi
```
- Create deploy 3 pods (nginx)
```

    - kubectl get deploy --namespace=homework
      NAME READY UP-TO-DATE AVAILABLE AGE
      web-deploy 3/3 3 3 2m2s
    - kubectl describe deploy --namespace=homework
      Name: web-deploy
      Namespace: homework
      CreationTimestamp: Sun, 28 Jan 2024 21:11:11 +0000
      Labels:
      Annotations: deployment.kubernetes.io/revision: 1
      Selector: app=nginx
      Replicas: 3 desired | 3 updated | 3 total | 3 available | 0 unavailable
      StrategyType: RollingUpdate
      MinReadySeconds: 0
      RollingUpdateStrategy: 1 max unavailable, 25% max surge
```
- Create readiness, it check file /homework/index.html

```
readinessProbe:
  exec:
    command: ['sh', '-c', 'cat /homework/index.html']
  initialDelaySeconds: 7
  periodSeconds: 4
```
- Create RollingUpdate,During the update process, a maximum of 1 pod
```
strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
```
- Create labels homework=true for node minikube and add nodeSelector - homework: "true"
```
    - kubectl label nodes minikube homework=true
    - kubectl get nodes --show-labels
    NAME STATUS ROLES AGE VERSION LABELS minikube Ready control-plane 29h v1.28.3 beta.kubernetes.io/arch=amd64,beta.kubernetes.io/ os=linux,homework=true,kubernetes.io/arch=amd64,kubernetes.io /hostname=minikube,kubernetes.io/os=linux,mini
```
## Как проверить работоспособность:

- if i delete file index.html , then status not read and raidness policy work
- kubectl get pods --namespace=homework
```
    NAME READY STATUS RESTARTS AGE
    web-deploy-5dc 0/1 Running 0 26m
    web-deploy-5dc 1/1 Running 0 26m
    web-deploy-5dcf 1/1 Running 0 26m
```
- kubectl describe deploy --namespace=homework
```
      Name: web-deploy
      Namespace: homework
      CreationTimestamp: Sun, 28 Jan 2024 21:11:11 +0000
      Labels:
      Annotations: deployment.kubernetes.io/revision: 1
      Selector: app=nginx
      Replicas: 3 desired | 3 updated | 3 total | 3 available | 0 unavailable
      StrategyType: RollingUpdate
      MinReadySeconds: 0
      RollingUpdateStrategy: 1 max unavailable, 25% max surge
```
 - i have 1 node minikube and i don't check create or not pod with lables homework=true

## PR checklist:

    Выставлен label с темой домашнего задания


# Выполнено ДЗ № 3

 - [X] Основное ДЗ
 - [X] Задание со *

## В процессе сделано:
```
- create readiness-probe httpGet
- create service.yaml for ClusterIP
- create ingress-controller
- create minikube tunnel
- create ingress.yaml for homework.otus
- create redirect homework.otus
```
## Как запустить проект:
- create raidness-probe
```
- kubectl apply -f deployment.yaml 
  readinessProbe:
    httpGet:
      path: /index.html
      port: 8000
  Readiness:    http-get http://:8000/index.html delay=3s timeout=1s period=4s #success=1 #failure=3
```
- Create minikube tunnel
```
- minikube tunnel

💡  Exiting due to TUNNEL_ALREADY_RUNNING: Another tunnel process is already running, terminate the existing instance to start a new one

```

- create service.yaml for ClusterIP
```
- kubectl apply -f service.yaml 
- kubectl describe svc

 Name:              server-nginx
 Namespace:         default
 Labels:            <none>
 Annotations:       <none>
 Selector:          app=nginx
 Type:              ClusterIP
 IP Family Policy:  SingleStack
 IP Families:       IPv4
 IP:                10.96.125.147
 IPs:               10.96.125.147
 Port:              web  80/TCP
 TargetPort:        8000/TCP
 Endpoints:         10.244.0.212:8000,10.244.0.213:8000,10.244.0.214:8000
 Session Affinity:  None
 Events:            <none>


- kubectl get svc -o=wide

 NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE   SELECTOR
 kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP   12d   <none>
 server-nginx   ClusterIP   10.96.125.147   <none>        80/TCP    98m   app=nginx

```
- create ingress-controller
```
- minikube addons enable ingress
```
- create ingress.yaml for homework.otus
```
- kubectl apply -f ingress.yaml 

- kubectl get ingress

  NAME           CLASS   HOSTS           ADDRESS        PORTS   AGE
  ingress-host   nginx   homework.otus   192.168.49.2   80      6h54m

- kubectl describe ingress
  ame:             ingress-host
  Labels:           <none>
  Namespace:        default
  Address:          192.168.49.2
  Ingress Class:    nginx
  Default backend:  <default>
  Rules:
  Host           Path  Backends
  ----           ----  --------
  homework.otus  
                 /                server-nginx:80 (10.244.0.212:8000,10.244.0.213:8000,10.244.0.214:8000)
                 /homework/(.*)   server-nginx:80 (10.244.0.212:8000,10.244.0.213:8000,10.244.0.214:8000)
  Annotations:     nginx.ingress.kubernetes.io/rewrite-target: /$1
                 nginx.ingress.kubernetes.io/use-regex: true
  Events:
  Type    Reason  Age                  From                      Message
  ----    ------  ----                 ----                      -------
  Normal  Sync    17m (x22 over 140m)  nginx-ingress-controller  Scheduled for sync
  
```
## Как проверить работоспособность:
- curl service ip
- curl 10.96.125.147
```
<!DOCTYPE HTML>
<html lang="en">
<head>
<!-- THIS IS A COMMENT -->
<title>Sample Web Page</title>
<META charset="utf-8">
<META name="viewport"
 content="width=device-width, initial-scale=1.0">
<style>
blockquote { margin-left:20px; margin-right:5px }
pre { overflow-x:auto }
.tt { font-family:monospace }
.nowrap { white-space:nowrap }
.example { font-family:monospace; white-space:pre; overflow-x:auto; }
h3 { border-top:1px solid grey }
blockquote { margin-top:0; margin-bottom:0 }
table.compact { border-collapse:collapse }
table.compact th { text-align:left; background:#eeeeee }
table.compact td,th { padding:0 4px 0 8px; border:1px solid grey }
</style>
</head>
....
```
- minikube ssh
- curl http://homework.otus
- curl -I http://homework.otus/index.html
```
HTTP/1.1 200 OK
Date: Fri, 09 Feb 2024 08:07:49 GMT
Content-Type: text/html
Content-Length: 34974
Connection: keep-alive
Last-Modified: Fri, 09 Feb 2024 06:21:01 GMT
ETag: "65c5c44d-889e"
Accept-Ranges: bytes

<!DOCTYPE HTML>
<html lang="en">
<head>
<!-- THIS IS A COMMENT -->
<title>Sample Web Page</title>
<META charset="utf-8">
<META name="viewport"
 content="width=device-width, initial-scale=1.0">
<style>
```
## PR checklist:
 - [ ] Выставлен label с темой домашнего задания



# Выполнено ДЗ № 4

 - [X] Основное ДЗ
 - [X] Задание со *

## В процессе сделано:
```
- create manifest pvc.yaml for PersistentVolumeClaim
- create manifest cm.yaml for
- edit  deployment.yaml (umount emptyDir and mount pvc)
- edit deployment and add configMap with volume for directory /homework/conf
- create storage class with: k8s.io/minikube-hostpath and edit Policy:reclaimPolicy Retain
- edit  manifest pvc for my storageClass
```
## Как запустить проект:
- create manifest pvc.yaml for PersistentVolumeClaim
- storage 4G storage class "myprovision"
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-volume2
spec:
  storageClassName: "myprovision"
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
```
```
- kubectl get pvc -o=wide
  pvc-volume2   Bound  pvc-5ccedf45-11a2-428f-8c29-d79bb33f2633  4Gi  RWO  myprovision  45h  Filesystem

- kubectl get pv
  NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
 pvc-5ccedf45-11a2-428f-8c29-d79bb33f2633   4Gi        RWO            Retain           Bound    default/pvc-volume2    myprovision             45h
 pvc-73a56c8d-51dd-4972-a937-0ec81fcbd256   2Gi        RWO            Delete           Bound    default/pvc-volume     standard                2d

- kubectl describe pvc
  Name:          pvc-volume2
  Namespace:     default
  StorageClass:  myprovision
  Status:        Bound
  Volume:        pvc-5ccedf45-11a2-428f-8c29-d79bb33f2633
  Labels:        <none>
  Annotations:   pv.kubernetes.io/bind-completed: yes
                 pv.kubernetes.io/bound-by-controller: yes
                 volume.beta.kubernetes.io/storage-provisioner: k8s.io/minikube-hostpath
                 volume.kubernetes.io/storage-provisioner: k8s.io/minikube-hostpath
  Finalizers:    [kubernetes.io/pvc-protection]
  Capacity:      4Gi
  Access Modes:  RWO
  VolumeMode:    Filesystem
  Used By:       web-deploy-975b86874-529vj
                 web-deploy-975b86874-6wjpb
                 web-deploy-975b86874-qbzt4
  Events:        <none>

```
- create manifest storageClass.yaml with provisioner: k8s.io/minikube-hostpath
- with polycy: Retain
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: myprovision
  labels:
    name: mystor
provisioner: k8s.io/minikube-hostpath
reclaimPolicy: Retain
volumeBindingMode: Immediate
```
```
- kubectl describe sc
  Name:            myprovision
  IsDefaultClass:  No
  Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io /v1","kind":"StorageClass","metadata":{"annotations":{},"labels": {"name":"mystor"},"name":"myprovision"},"provisioner":"k8s.io/minikube- hostpath","reclaimPolicy":"Retain","volumeBindingMode":"Immediate"}
,storageclass.kubernetes.io/is-default-class=false
  Provisioner:           k8s.io/minikube-hostpath
  Parameters:            <none>
  AllowVolumeExpansion:  <unset>
  MountOptions:          <none>
  ReclaimPolicy:         Retain
  VolumeBindingMode:     Immediate
  Events:                <none>
```
- create cm.yaml and mount in directory /homework/conf/file
- this is config  new html page in configmap
```
- mountPath: /homework/conf/file
  name: env
  subPath: file

apiVersion: v1
data:
  file: |+
    <!doctype html>
    <html lang="en-US">
      <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width" />
        <title>My World</title>
      </head>
      <body>
        <img src="images/firefox-icon.png" alt="My test image" />
      </body>
    </html>

kind: ConfigMap
metadata:
  creationTimestamp: null
  name: cm

- kubectl get cm
  NAME               DATA   AGE
  cm                 1      5m27s
  cmnginx            1      4m54s
  kube-root-ca.crt   1      18d
```
- create new config nginx (cmnginx.yaml)
- this is config map mount file with new preference nginx (home.conf in pods)
```
- mountPath: /etc/nginx/conf.d/home.conf
  name: nginx
  subPath: home.conf
  readOnly: true


- kubectl describe cm cmnginx
  Name:         cmnginx
  Namespace:    default
  Labels:       <none>
  Annotations:  <none>

  Data
  ====
  home.conf:
  ----
  server {
    listen       8000;
    listen  [::]:8000;
    server_name  localhost;

    location / {
        root   /homework;
        index  index.html index.htm;
    }
    location /conf/file {
        root  /homework;
   }
  }

  BinaryData
  ====
  Events:  <none>

```
- edit deploy and add PersistentVolumeClaim for container init and default container 

```
  containers:
      - name: webnginx          
        image: tulamelki/neweng
        lifecycle:
          preStop:                  
            exec:
              command: [ 'sh', '-c', 'rm /homework/index.html'] 
        ports:
        - containerPort: 8000
        volumeMounts:
        - mountPath: /homework
          name: volume
....
 initContainers:         
      - name: init-containers      
        image: busybox             
        command: ['sh', '-c', 'wget http://www.columbia.edu/~fdc/sample.html -O /init/index.html']
        volumeMounts:
        - mountPath: /init         
          name: volume
      volumes:
      - name: volume 
        persistentVolumeClaim:
           claimName: pvc-volume2 
```

## Как проверить работоспособность:
- check pod
- kubectl get po (pods runing)
```
NAME                          READY   STATUS    RESTARTS   AGE
web-deploy-568894b964-2xf2r   1/1     Running   0          34m
web-deploy-568894b964-k85nt   1/1     Running   0          34m
web-deploy-568894b964-l2dtl   1/1     Running   0          34m
```
- check service
- kubectl get svc (service work)
```
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP   18d
server-nginx   ClusterIP   10.100.173.218   <none>        80/TCP    3h31m

```
DONT forget enable minikube tunnel!
```
- curl 10.100.173.218
<!DOCTYPE HTML>
<html lang="en">
<head>
<!-- THIS IS A COMMENT -->
<title>Sample Web Page</title>
<META charset="utf-8">
<META name="viewport"
```
- check ingress
- kubectl get ingress
```
NAME           CLASS   HOSTS           ADDRESS        PORTS   AGE
ingress-host   nginx   homework.otus   192.168.49.2   80      3h37m
```
- check storage,pv,pvc
-  kubectl get sc
```
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
myprovision          k8s.io/minikube-hostpath   Retain          Immediate           false                  46h
standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  18d
```
-  kubectl get pv
```
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
pvc-5ccedf45-11a2-428f-8c29-d79bb33f2633   4Gi        RWO            Retain           Bound    default/pvc-volume2   myprovision             46h
pvc-73a56c8d-51dd-4972-a937-0ec81fcbd256   2Gi        RWO            Delete           Bound    default/pvc-volume    standard                2d
```
-  kubectl get pvc
```
NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-volume    Bound    pvc-73a56c8d-51dd-4972-a937-0ec81fcbd256   2Gi        RWO            standard       2d
pvc-volume2   Bound    pvc-5ccedf45-11a2-428f-8c29-d79bb33f2633   4Gi        RWO            myprovision    46h
```
- check configmap
- kubectl get cm
```
NAME               DATA   AGE
cm                 1      54m
cmnginx            1      53m
kube-root-ca.crt   1      18d
```
- minikube ssh
- curl http://homework.otus/
```
<!DOCTYPE HTML>
<html lang="en">
<head>
<!-- THIS IS A COMMENT -->
<title>Sample Web Page</title>
<META charset="utf-8">
<META name="viewport"
 content="width=device-width, initial-scale=1.0">
<style>
```
curl http://homework.otus/conf/file
- my new confmap
```
docker@minikube:~$ curl http://homework.otus/conf/file
<!doctype html>
<html lang="en-US">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>My World</title>
  </head>
  <body>
    <img src="images/firefox-icon.png" alt="My test image" />
  </body>
</html>
```
```
docker@minikube:~$ curl -I http://homework.otus/conf/file
HTTP/1.1 200 OK
Date: Wed, 14 Feb 2024 20:20:56 GMT
Content-Type: application/octet-stream
Content-Length: 260
Connection: keep-alive
Last-Modified: Wed, 14 Feb 2024 19:30:48 GMT
ETag: "65cd14e8-104"
Accept-Ranges: bytes
```

## PR checklist:
 - [ ] Выставлен label с темой домашнего задания
# Выполнено ДЗ № 5

 - [X] Основное ДЗ
 - [X] Задание со *

## В процессе сделано:
- create service account monitoring and add access for endpoint /metrics
- edit manifest deployment and pods started with service account monitoring
- edit deployment and download metrics in pods
- save this metrics in file metrics.html
- show metrics with endpoint  /metrics.html
- create service account cd in namespace homework and add access admin
- change kubeconfig for service account cd
- generate token for sa cd on the 1 hour

## Как запустить проект:
- create manifest for service account monitoring
- first create service account
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: monitoring
  namespace: homework
...
- kubectl get sa monitoring -n homework

   NAME         SECRETS   AGE
   monitoring   0         33h

```
- second create role for sa monitoring
- add access for endpoint /metrics
- add access for launching pods ["create","watch","delete","update","list"]
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: rmonitoring
  namespace: homework
rules:
  - apiGroups: [""]
    resources: ["nodes/metrics"]
    verbs: ["get", "list"]
  - nonResourceURLs: ["/metrics"]
    verbs: [ "get"]

  - apiGroups: [""]
    resources: ["pod"]
    verbs: ["create","watch","delete","update","list"]
...
- kubectl get clusterrole -n homework

  NAME                     CREATED AT
  rmonitoring              2024-02-24T10:22:34Z

```
- third combine service account + role (this combine rolebinding)
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: RBmonitoring
  namespace: homework
subjects:
  - kind: ServiceAccount
    name: monitoring
    namespace: homework
roleRef:
  kind: ClusterRole
  name: rmonitoring
  apiGroup: rbac.authorization.k8s.io

- kubectl get clusterrolebinding -n homework

  NAME                                ROLE                                                                                                  AGE
  RBmonitoring                        ClusterRole/rmonitoring                                                                               33h
  
```
- edit manifest deployment and pods started with service account monitoring
```
   spec:
      serviceAccountName: monitoring
      containers:
```
- edit deployment and download metrics in pods
- first we must create secret for sa monitoring
- this is secrcret create token automatically
```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
  namespace: homework
  annotations:
    kubernetes.io/service-account.name: monitoring
type: kubernetes.io/service-account-token
- kubectl get secret -n homework
  NAME       TYPE                                  DATA   AGE
  mysecret   kubernetes.io/service-account-token   3      33h
```
- second us need forward this token in pods
- edit the deployment and add env to the initialization container with the token
```
env:
  - name: TOKEN
    valueFrom:
      secretKeyRef:
        name: mysecret
        key: token
```
- download metrics in pods
- i pull new docker container curlimages/curl
- save this metrics in file metrics.html
- this command is for download, but if the account is not authorized, we have problems and cannot load the metrics
```
command: ["sh", "-c", 'curl https://192.168.49.2:8443/metrics --header "Authorization: Bearer $TOKEN" -k > /init/metrics.html']
```
- show metrics with endpoint  /metrics.html
- first i change configmap nginx
```
- kubectl describe cm cmnginx -n homework
Name:         cmnginx
Namespace:    homework
Labels:       <none>
Annotations:  <none>
Data
====
home.conf:
----
server {
    listen       8000;
    listen  [::]:8000;
    server_name  localhost;

    location = /metrics.html {
        root   /homework;
        index  metrics.html metrics.htm;
    }
 
}
BinaryData
====

Events:  <none>
...
- kubectl get cm cmnginx -n homework

  NAME      DATA   AGE
  cmnginx   1      8h

```
- second change preference ingress
```
- kubectl describe ingress -n homework

Name:             ingress-host
Labels:           <none>
Namespace:        homework
Address:          192.168.49.2
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host           Path  Backends
  ----           ----  --------
  homework.otus  
                 /metrics.html   server-nginx:80 (10.244.0.140:8000,10.244.0.141:8000,10.244.0.142:8000)
Annotations:     <none>
Events:          <none>
...
- kubectl get ingress ingress-host -n homework

  NAME           CLASS    HOSTS           ADDRESS        PORTS   AGE
  ingress-host   <none>   homework.otus   192.168.49.2   80      33h

```
- full deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
  namespace: homework
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      serviceAccountName: monitoring
      containers:
      - name: webnginx          
        image: tulamelki/neweng
        lifecycle:
          preStop:
            exec:
              command: ['sh', '-c', 'rm /homework/metrics.html']
        ports:
        - containerPort: 8000
        volumeMounts:
        - mountPath: /homework
          name: volume
        - mountPath: /etc/nginx/conf.d/home.conf
          name: nginx
          subPath: home.conf
          readOnly: true
        readinessProbe:
          httpGet:
            path: /metrics.html
            port: 8000
          initialDelaySeconds: 3
          periodSeconds: 4
      initContainers: 
      - name: init-containers      
        image: curlimages/curl
        command: ["sh", "-c", 'curl https://192.168.49.2:8443/metrics --header "Authorization: Bearer $TOKEN" -k > /init/metrics.html']
        env:
        - name: TOKEN
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: token
        volumeMounts:
        - mountPath: /init         
          name: volume
      volumes:
      - name: volume 
        persistentVolumeClaim:
           claimName: pvc-volume2     
      - name: nginx
        configMap:
          name: cmnginx
```
- create service account cd in namespace homework and add access admin
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: scd
  namespace: homework
...
- kubectl get sa scd -n homework
  NAME   SECRETS   AGE
  scd    0         33h
```
- create role for cd
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: RoleCd
  namespace: homework
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs:     ["*"]
...
- kubectl get role -n homework
  NAME     CREATED AT
  RoleCd   2024-02-24T10:22:34Z
```
- create rolebinding for cd
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: RBcd
  namespace: homework
subjects:
  - kind: ServiceAccount
    name: scd
    namespace: homework
roleRef:
    kind: Role
    name: RoleCd
    apiGroup: rbac.authorization.k8s.io
```
- create token for sa scd feb 25 2025 -> sun feb 26 2024
```
kubectl create token scd --namespace homework --duration 24h >> token
https://jwt.io/
{
  "aud": [
    "https://kubernetes.default.svc.cluster.local"
  ],
  "exp": 1708980704,
  "iat": 1708894304,
  "iss": "https://kubernetes.default.svc.cluster.local",
  "kubernetes.io": {
    "namespace": "homework",
    "serviceaccount": {
      "name": "scd",
      "uid": "31520227-a74c-410d-9a42-2ef819e43339"
    }
  },
  "nbf": 1708894304,
  "sub": "system:serviceaccount:homework:scd"
}
```
- change kubeconfig and add user scd
```
- kubectl config set-credentials scd--token=eyJhbGciOiJSUzI1NiIsImtpZCI6IjJlUmM3REF4NWVZZ1c2cUhoZDNaRzBkM....
  User "scd" set.
```
create context for user scd
```
- kubectl config set-context contextSCD --cluster=minicube --user=scd
  Context "contextSCD" created.
```
***  examle use default context
- kubectl config use-context <user_name>
*** example delete user from config
  kubectl config unset users.<user_name>
*** example delete context rom config
- kubectl config unset contexts.<context_name>

## Как проверить работоспособность:
- kubectl get sa monitoring -n homework
```
 NAME         SECRETS   AGE
 monitoring   0         33h
```
- kubectl get clusterrole -n homework
```
NAME                     CREATED AT
rmonitoring              2024-02-24T10:22:34Z
```
- kubectl get clusterrolebinding -n homework
```
NAME                                ROLE                                                                                                  AGE
  RBmonitoring                        ClusterRole/rmonitoring  
```
- kubectl get secret -n homework
```
 NAME       TYPE                                  DATA   AGE
 mysecret   kubernetes.io/service-account-token   3      33h
```
- kubectl get cm cmnginx -n homework
```
NAME      DATA   AGE
cmnginx   1      8h
```
- kubectl get ingress ingress-host -n homework
```
NAME           CLASS    HOSTS           ADDRESS        PORTS   AGE
ingress-host   <none>   homework.otus   192.168.49.2   80      33h
```
- kubectl get svc -n homework
```
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
server-nginx   ClusterIP   10.108.178.242   <none>        80/TCP    22h
```
- kubectl get sc -n homework
```
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
myprovision          k8s.io/minikube-hostpath   Retain          Immediate           false     
```
- kubectl get pvc -n homework
```
NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-volume2   Bound    pvc-11112082-97d4-4ee6-b366-3b88c5e152ff   4Gi        RWO            myprovision    35h
```
- kubectl get po -n homework
```
NAME                        READY   STATUS    RESTARTS   AGE
web-deploy-cf5986b7-5b9gg   1/1     Running   0          10h
web-deploy-cf5986b7-scj78   1/1     Running   0          10h
web-deploy-cf5986b7-xczd5   1/1     Running   0          10h
```
- curl http://homework.otus/metrics
```
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="0.001"} 315
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="0.01"} 315
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="0.1"} 315
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="1"} 315
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="10"} 315
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="+Inf"} 315
workqueue_work_duration_seconds_sum{name="APIServiceRegistrationController"} 0.0020561219999999996
workqueue_work_duration_seconds_count{name="APIServiceRegistrationController"} 315
workqueue_work_duration_seconds_bucket{name="AvailableConditionController",le="1e-08"} 0
```
- kubectl config view 
```
contexts:
- context:
    cluster: minicube
    user: scd
  name: contextSCD
....
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/vagrant/.minikube/profiles/minikube/client.crt
    client-key: /home/vagrant/.minikube/profiles/minikube/client.key
- name: scd
  user:
    token: REDACTED
```
user created and token add

## PR checklist:

    Выставлен label с темой домашнего задания

# Выполнено ДЗ № 6

 - [X] Основное ДЗ
 - [ ] Задание со *

## В процессе сделано:



first install ns
```
kubectl create ns homework
```
second deploy helm to namespace homework
```
helm install helm /tmp/helm-chart/ -n homework
NAME: helm
LAST DEPLOYED: Sun Jun  9 18:44:53 2024
NAMESPACE: homework
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Hi,this is information about your server 
Kubernetes control plane at https://192.168.49.2:8443
http://homework.otus/metrics.html - metrics
```
check helm 
```
helm list -n homework
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART 	APP VERSION
helm	homework 	1       	2024-06-09 18:44:53.260208578 +0000 UTC	deployed	metr-2
```

check pods and namespace 
- pods status runing
- i install my helm chart + plus helm-memcached
```
kubectl get pods -n homework
NAME                              READY   STATUS    RESTARTS   AGE
helm-memcached-569bddd748-2d6sw   1/1     Running   0          3m13s
helm-metr-5876c66c59-5h9cr        1/1     Running   0          3m13s
helm-metr-5876c66c59-68f7w        1/1     Running   0          3m13s
helm-metr-5876c66c59-fgmqd        1/1     Running   0          3m13s
helm-metr-5876c66c59-hth9f        1/1     Running   0          3m13s
helm-metr-5876c66c59-j2bh9        1/1     Running   0          3m13s

kubectl get ns
NAME              STATUS   AGE
default           Active   5h42m
dev               Active   16m
homework          Active   5h32m
prod              Active   16m
```
check pv and pvc 
```
kubectl get pvc -n homework
NAME            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS     VOLUMEATTRIBUTESCLASS   AGE
pvc-helm-metr   Bound    pvc-91305702-e54b-4323-923b-d372699d8d7a   5Gi        RWO            storclass-metr   <unset>                 6m23s
kubectl get pv -n homework

pvc-91305702-e54b-4323-923b-d372699d8d7a   5Gi        RWO            Retain           Bound      homework/pvc-helm-metr                    storclass-metr   <unset>                          7m9s

```
check ingress and service
```
kubectl get ingress -n homework
NAME                CLASS   HOSTS           ADDRESS        PORTS   AGE
ingress-helm-metr   nginx   homework.otus   192.168.49.2   80      10m
kubectl get svc -n homework

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
helm-memcached       ClusterIP   10.98.251.110   <none>        11211/TCP   10m
service-nginx-metr   ClusterIP   10.108.205.80   <none>        80/TCP      10m
```
 - curl http://homework.otus/metrics
```
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="0.001"} 315
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="0.01"} 315
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="0.1"} 315
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="1"} 315
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="10"} 315
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="+Inf"} 315
workqueue_work_duration_seconds_sum{name="APIServiceRegistrationController"} 0.0020561219999999996
workqueue_work_duration_seconds_count{name="APIServiceRegistrationController"} 315
workqueue_work_duration_seconds_bucket{name="AvailableConditionController",le="1e-08"} 0
```
create helmfile
```
cat helmfile.yaml 
repositories:
  - name: bitnami
    url: https://charts.bitnami.com/bitnami

helmDefaults:
  createNamespace: True
  timeout: 800
  wait: True

releases: 
  - name: kafka
    chart: bitnami/kafka
    namespace: prod
    set:
      - name: replicaCount
        value: 1
      - name: image.tag
        value: "3.5.2"
      - name: client.protocol
        value: "SASL_PLAINTEXT"
      - name: interbroker.protocol
        value: "SASL_PLAINTEXT"
  - name: dev
    namespace: dev
    chart: bitnami/kafka
    set:
      - name: image.tag
        value: "latest"
      - name: broker.replicaCount
        value: 1
      - name: listeners.client.protocol
        value: "PLAINTEXT"
      - name: listeners.interbroker.protocol
        value: "PLAINTEXT"
```
check ns
```
kubectl get ns dev
NAME   STATUS   AGE
dev    Active   11m

kubectl get ns prod
NAME   STATUS   AGE
prod   Active   11m
``
kubectl get pods -n dev
NAME                     READY   STATUS    RESTARTS   AGE
dev-kafka-broker-0       1/1     Running   0          12m
dev-kafka-controller-0   1/1     Running   0          12m
dev-kafka-controller-1   1/1     Running   0          12m
dev-kafka-controller-2   1/1     Running   0          12

```
# Выполнено ДЗ № 7

 - [X] Основное ДЗ
 - [X] Задание со *

## В процессе сделано:
- pull nginx container
- create configmap for nginx
- install prometheus
- create Prometheus,Grafana,kube-exporter
- create deploy,service,configmap,nginx prometheus exporter

## Как запустить проект:
infrastrukture k8s with 3 nodes
```
kubectl get nodes -o wide
NAME      STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION   CONTAINER-RUNTIME
control   Ready    control-plane   9h    v1.29.6   192.168.2.4   <none>        Debian GNU/Linux 12 (bookworm)   6.1.0-13-amd64   cri-o://1.31.0
worker    Ready    <none>          9h    v1.29.6   192.168.2.5   <none>        Debian GNU/Linux 12 (bookworm)   6.1.0-13-amd64   cri-o://1.31.0
worker2   Ready    <none>          9h    v1.29.6   192.168.2.6   <none>        Debian GNU/Linux 12 (bookworm)   6.1.0-13-amd64   cri-o://1.31.0
```
- create deployment (nginx + nginx exporter)
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginxmetrics
  labels:
    app: nginx
spec:
  replicas: 1
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
          image: nginx:1.27.0
          ports:
            - name: nginx
              containerPort: 80
              protocol: TCP
          volumeMounts:
            - name: nginx-conf
              mountPath: /etc/nginx/conf.d/default.conf
              subPath: default.conf
              readOnly: true

        - name: nginx-exporter
          image: nginx/nginx-prometheus-exporter:1.1.0
          args: ['--nginx.scrape-uri', 'http://127.0.0.1:80/metrics']
          ports:
            - name: metrics
              containerPort: 9113
              protocol: TCP
      volumes:
        - name: nginx-conf
          configMap:
            name: nginxconf


kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
nginxmetrics-7fdc6cc6b7-pt5nx   2/2     Running   0          47m
```
install helm and add repo bitnami
```
helm repo list
NAME    	URL                               
traefik 	https://traefik.github.io/charts  
longhorn	https://charts.longhorn.io        
bitnami 	https://charts.bitnami.com/bitnami
```
install helm chart 
```
helm list -n monitoring
NAME           	NAMESPACE 	REVISION	UPDATED                                	STATUS  CHART                	APP VERSION
grafana        	monitoring	1       	2024-06-23 13:11:44.407061227 +0300 MSK	deployedgrafana-11.3.5       	11.0.0     
kube-prometheus	monitoring	1       	2024-06-23 13:11:37.169970917 +0300 MSK	deployedkube-prometheus-9.5.2	0.74.0     
```
create service
```
apiVersion: v1
kind: Service
metadata:
  name: service-metric
  namespace: default
  labels:
    app: nginx
spec:
  selector:
    app: nginx
  ports:
    - name: nginx
      port: 80
      targetPort: 80
    - name: metrics
      port: 9113
      targetPort: 9113

kubectl get svc
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)           AGE
kubernetes       ClusterIP   10.96.0.1      <none>        443/TCP           9h
service-metric   ClusterIP   10.109.27.36   <none>        80/TCP,9113/TCP   7h38m
```
- create service monitor
```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: service-monitor
  labels:
    app: nginx-service-monitor
spec:
  selector:
    matchLabels:
      app: nginx
  endpoints:
    - port: metrics
      path: /metrics
  namespaceSelector:
    matchNames:
      - default
...
kubectl get servicemonitor

NAME              AGE
service-monitor   7h41m
```
create configmap for nginx
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginxconf
  creationTimestamp: null
data:
 default.conf: |
    server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;


    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    location = /basic_status {
    stub_status;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    }
   ```
check
```
root@control:/home/vagrant# curl http://10.244.171.115/basic_status
Active connections: 1 
server accepts handled requests
 159 159 159 
Reading: 0 Writing: 1 Waiting: 0 
``
curl http://10.109.27.36:9113/metrics

# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 3.8425e-05
go_gc_duration_seconds{quantile="0.25"} 0.000159387
go_gc_duration_seconds{quantile="0.5"} 0.000285061
go_gc_duration_seconds{quantile="0.75"} 0.000405764
go_gc_duration_seconds{quantile="1"} 0.000710465
go_gc_duration_seconds_sum 0.007525263
go_gc_duration_seconds_count 25
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 14
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.21.5"} 1
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 2.302176e+06
# HELP go_memstats_alloc_bytes_total Total number of bytes allocated, even if freed.
# TYPE go_memstats_alloc_bytes_total counter
go_memstats_alloc_bytes_total 2.0265176e+07
# HELP go_memstats_buck_hash_sys_bytes Number of bytes used by the profiling bucket hash table.
# TYPE go_memstats_buck_hash_sys_bytes gauge
go_memstats_buck_hash_sys_bytes 4625
# HELP go_memstats_frees_total Total number of frees.
# TYPE go_memstats_frees_total counter
go_memstats_frees_total 104518
```
port forward grafana and prometheus
```
kubectl port-forward svc/grafana --address 0.0.0.0 3000:3000 -n monitoring

Forwarding from 0.0.0.0:3000 -> 3000

kubectl port-forward svc/kube-prometheus-prometheus  --address 0.0.0.0 9090:9090 -n monitoring

Forwarding from 0.0.0.0:9090 -> 9090
```
![image](https://github.com/Kuber-2023-12OTUS/tulamelkii_repo/assets/130311206/be6f6cf6-d848-4a62-a928-066a6a0cba64)

![image](https://github.com/Kuber-2023-12OTUS/tulamelkii_repo/assets/130311206/1e3a5f98-5c38-404a-847a-54ed6a845435)

=======
```
kubectl get ingress -n homework
NAME                CLASS   HOSTS           ADDRESS        PORTS   AGE
ingress-helm-metr   nginx   homework.otus   192.168.49.2   80      10m
kubectl get svc -n homework

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
helm-memcached       ClusterIP   10.98.251.110   <none>        11211/TCP   10m
service-nginx-metr   ClusterIP   10.108.205.80   <none>        80/TCP      10m
```
 - curl http://homework.otus/metrics
```
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="0.001"} 315
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="0.01"} 315
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="0.1"} 315
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="1"} 315
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="10"} 315
workqueue_work_duration_seconds_bucket{name="APIServiceRegistrationController",le="+Inf"} 315
workqueue_work_duration_seconds_sum{name="APIServiceRegistrationController"} 0.0020561219999999996
workqueue_work_duration_seconds_count{name="APIServiceRegistrationController"} 315
workqueue_work_duration_seconds_bucket{name="AvailableConditionController",le="1e-08"} 0
```
create helmfile
```
cat helmfile.yaml 
repositories:
  - name: bitnami
    url: https://charts.bitnami.com/bitnami

helmDefaults:
  createNamespace: True
  timeout: 800
  wait: True

releases: 
  - name: kafka
    chart: bitnami/kafka
    namespace: prod
    set:
      - name: replicaCount
        value: 1
      - name: image.tag
        value: "3.5.2"
      - name: client.protocol
        value: "SASL_PLAINTEXT"
      - name: interbroker.protocol
        value: "SASL_PLAINTEXT"
  - name: dev
    namespace: dev
    chart: bitnami/kafka
    set:
      - name: image.tag
        value: "latest"
      - name: broker.replicaCount
        value: 1
      - name: listeners.client.protocol
        value: "PLAINTEXT"
      - name: listeners.interbroker.protocol
        value: "PLAINTEXT"
```
check ns
```
kubectl get ns dev
NAME   STATUS   AGE
dev    Active   11m

kubectl get ns prod
NAME   STATUS   AGE
prod   Active   11m
``
kubectl get pods -n dev
NAME                     READY   STATUS    RESTARTS   AGE
dev-kafka-broker-0       1/1     Running   0          12m
dev-kafka-controller-0   1/1     Running   0          12m
dev-kafka-controller-1   1/1     Running   0          12m
dev-kafka-controller-2   1/1     Running   0          12
