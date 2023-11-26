# k8s-exam

## Pod

## Labels

k get pod --selector env=dev --no-headers | wc -l
### Multipod 
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multi-pod
  name: multi-pod
spec:
  containers:
  - image: nginx
    name: jupiter
    env:
    - name: type
      value: planet
  - image: busybox
    name: europa
    command: ["/bin/sh","-c","sleep 4800"]
    env:
     - name: type
       value: moon
```
### Daemonset
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: elasticsearch
  template:
    metadata:
      labels:
        name: elasticsearch
    spec:
      containers:
      - name: elasticsearch
        image: registry.k8s.io/fluentd-elasticsearch:1.20
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
      terminationGracePeriodSeconds: 30
```
## admission-plugins
ps -ef | grep kube-apiserver | grep admission-plugins

kube-apiserver -h | grep enable-admission-plugins

## Services

kubectl expose deploy my-webapp --port=80 --target-port=80 --name front-end-service --labels=tier=frontend --type=NodePort
## Volumes

### Persistent Volumes

```
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: custom-volume
spec:
  accessModes: ["ReadWriteMany"]
  capacity:
    storage: 50Mi
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /opt/data
```

## RBAC

### Cluster Role

kubectl create clusterrole storage-admin --verb=get,list,watch,create --resource=persistentvolumes,storageclasses

kubectl create clusterrolebinding michelle-storage-admin --clusterrole=storage-admin --user=michelle

# Secrets 

## Secret Volume/ nodeSelector
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: my-busybox
  name: my-busybox
  namespace: dev2406
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: dotfile-secret
  nodeSelector:
    kubernetes.io/hostname: controlplane
  containers:
  - command:
    - sleep
    args:
    - "3600"
    image: busybox
    name: secret
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```

## TLS Certificate
```
kubectl create secret tls webhook-server-tls \
  --cert=/root/keys/webhook-server-tls.crt\
  --key=/root/keys/webhook-server-tls.key
```

```
apiVersion: v1
kind: Secret
metadata:
  name: webhook-server-tls
type: kubernetes.io/tls
data:
  # the data is abbreviated in this example
  tls.crt: |
    M/root/keys/webhook-server-tls.crt
  tls.key: |
    MIIEpgIBAAKCAQEA7yn3bRHQ5FHMQ ...  
```

### Literal Secret 
```
kubectl create secret generic db-secret-xxdf --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123
```

```
kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3
```
### Docker registry Secret 

```
kubectl create secret docker-registry private-reg-cred --docker-username=dock_user --docker-password=dock_password --docker-email=dock_user@myprivateregistry.com --docker-server=myprivateregistry.com:5000
```
#### Pod Declaration Docker registry Secret 
```
    spec:
      containers:
      - image: myprivateregistry.com:5000/nginx:alpine
        imagePullPolicy: IfNotPresent
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      imagePullSecrets:
      - name: private-reg-cred
```
## Deployment - Scale

kubectl scale deployment --replicas=1 frontend-v2

## Configmap

### Configmap literal
kubectl create configmap cm-3392845 --from-literal=DB_NAME=SQL3322 --from-literal=DB_HOST=sql322.mycompany.com --from-literal=DB_PORT=3306

### Configmap Pod
```	
apiVersion: v1
kind: Pod
metadata:
  name: time-check
  namespace: dvl1987
spec:
  containers:
    - name: time-check
      image: busybox
      command: ['sh', '-c', 'while true; do date; sleep $TIME_FREQ;done > /opt/time/time-check.log']
      env:
        - name: TIME_FREQ
          valueFrom:
            configMapKeyRef:
              name: time-config
              key: TIME_FREQ
      volumeMounts:
      - name: vol1
        mountPath: /opt/time
  volumes:
    - name: vol1
      hostPath:
        path: /opt/time
```				
## livenessProbe/StartupPobre
```
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx1401
  namespace: default
spec:
  containers:
    - name: nginx1401
      image: nginx
      livenessProbe:
        exec:
          command: ["ls /var/www/html/probe"]
        initialDelaySeconds: 10
        periodSeconds: 60
```
```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: my-image
      livenessProbe:
        httpGet:
          path: /
          port: 8080
        initialDelaySeconds: 15
        periodSeconds: 10
      startupProbe:
        exec:
          command:
            - sh
            - -c
            - |
              sleep 10
              while true; do
                if [ -f /var/www/html/file_check ]; then
                  break
                fi
                sleep 60
              done
        initialDelaySeconds: 10
        periodSeconds: 60
```
livenessProbe
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx1401
  namespace: dev1401
spec:
  containers:
  - image: kodekloud/nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    ports:
    - containerPort: 9080
      protocol: TCP
    readinessProbe:
      httpGet:
        path: /
        port: 9080    
    livenessProbe:
      exec:
        command:
        - ls
        - /var/www/html/file_check
      initialDelaySeconds: 10
      periodSeconds: 60
```

##  Jobs
```
apiVersion: batch/v1
kind: Job
metadata:
  name: whalesay
spec:
  completions: 10
  backoffLimit: 6
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - sh 
        - -c
        - "cowsay I am going to ace CKAD!"
        image: docker/whalesay
        name: whalesay
      restartPolicy: Never
```

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: dice
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      completions: 1
      backoffLimit: 25 # This is so the job does not quit before it succeeds.
      activeDeadlineSeconds: 20
      template:
        spec:
          containers:
          - name: dice
            image: kodekloud/throw-dice
          restartPolicy: Never

```		  
## Ingress
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: ingress
spec:
  rules:
  - host: ckad-mock-exam-solution.com
    http:
      paths:
      - backend:
          service:
            name: my-video-service
            port:
              number: 8080
        path: /video
        pathType: Prefix
```
```		
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: ingress-vh-routing
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: watch.ecom-store.com
    http:
      paths:
      - pathType: Prefix
        path: "/video"
        backend:
          service:
            name: video-service
            port:
              number: 8080
  - host: apparels.ecom-store.com
    http:
      paths:
      - pathType: Prefix
        path: "/wear"
        backend:
          service:
            name: apparels-service
            port:
              number: 8080	  
```
### Network Policies
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Egress
  ingress:
  egress:
    - to:
        - podSelector:
            matchLabels:
              name: mysql
        - podSelector:
            matchLabels:
              name: payroll
      ports:
        - protocol: TCP
          port: 8080
        - protocol: TCP
          port: 3306
```
#### Egress
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
  namespace: space1
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
     - namespaceSelector:
        matchLabels:
         kubernetes.io/metadata.name: space2
  - ports:
    - port: 53
      protocol: TCP
    - port: 53
      protocol: UDP
```
#### Ingress
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
  namespace: space2
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
   - from:
     - namespaceSelector:
        matchLabels:
         kubernetes.io/metadata.name: space1
```
#### Verify
```
# these should work
k -n space1 exec app1-0 -- curl -m 1 microservice1.space2.svc.cluster.local
k -n space1 exec app1-0 -- curl -m 1 microservice2.space2.svc.cluster.local
k -n space1 exec app1-0 -- nslookup tester.default.svc.cluster.local
k -n kube-system exec -it validate-checker-pod -- curl -m 1 app1.space1.svc.cluster.local

# these should not work
k -n space1 exec app1-0 -- curl -m 1 tester.default.svc.cluster.local
k -n kube-system exec -it validate-checker-pod -- curl -m 1 microservice1.space2.svc.cluster.local
k -n kube-system exec -it validate-checker-pod -- curl -m 1 microservice2.space2.svc.cluster.local
k -n default run nginx --image=nginx:1.21.5-alpine --restart=Never -i --rm  -- curl -m 1 microservice1.space2.svc.cluster.local
```

## Logs 
kubectl logs dev-pod-dind-878516 -c log-x | grep WARNING > /opt/dind-878516_logs.txt	  


## Taint/Toleration 

kubectl taint nodes node01 app_type=alpha:NoSchedule

```
apiVersion: v1
kind: Pod
metadata:
  name: alpha
spec:
  containers:
  - name: redis-container
    image: redis
  tolerations:
  - key: app_type
    value: alpha
    operator: Exists
    effect: NoSchedule
```

# Node Affinity
```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: beta-apps
  name: beta-apps
spec:
  replicas: 3
  selector:
    matchLabels:
      app: beta-apps
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: beta-apps
    spec:
      affinity:
        nodeAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
           nodeSelectorTerms:
           - matchExpressions:
             - key: app_type
               values: ["beta"]
               operator: In
      containers:
      - image: nginx
        name: nginx
```
 
## Maintenance 

### Drain
kubectl drain node01 --ignore-daemonsets

### Cordon

kubectl cordon node01


kubectl uncordon node01

### Cluster Upgrade
https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

#### kubeadm

kubeadm upgrade plan

apt update

##### install the kubeadm version 1.27.0

apt-get install kubeadm=1.27.0-00


##### upgrade Kubernetes controlplane node.

kubeadm upgrade apply v1.27.0


This will update the kubelet with the version 1.27.0.

apt-get install kubelet=1.27.0-00 


#### reload the daemon and restart kubelet service after it has been upgraded.

systemctl daemon-reload
systemctl restart kubelet

#### Etcd

 describe pod -n kube-system etcd-controlplane | grep -i crt

#### Backup 
 ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379  snapshot save --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key /opt/snapshot-pre-boot.db

##### Restore 
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/pki/etcd.pem --key=/etc/etcd/pki/etcd-key.pem snapshot restore /root/cluster2.db --data-dir /var/lib/etcd-data-new
 watch "crictl ps | grep etcd"

######
standalone

vi /etc/systemd/system/etcd.service

```
ExecStart=/usr/local/bin/etcd \
  --name etcd-server \
  --data-dir=/var/lib/etcd-data-new \
```
## Context
k describe pod kube-apiserver-controlplane  -n kube-system | grep -i cert
kubectl config get-contexts


#### CErtificates


openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout

### Cluster Role
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: node-admin
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "watch", "list", "create", "delete"]
```
### Cluster Role   Binding
```
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: michelle-binding
subjects:
- kind: User
  name: michelle
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-admin
  apiGroup: rbac.authorization.k8s.io
```
### ClusterRole - Storage
```
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: storage-admin
rules:
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["get", "watch", "list", "create", "delete"]
- apiGroups: ["storage.k8s.io"]
  resources: ["storageclasses"]
  verbs: ["get", "watch", "list", "create", "delete"]
```
### ClusterRoleBinding - Storage
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: michelle-storage-admin
subjects:
- kind: User
  name: michelle
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: storage-admin
  apiGroup: rbac.authorization.k8s.io
```
## Service account
kubectl create serviceaccount dashboard-sa

kubectl create token dashboard-sa
```
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2023-07-03T21:02:43Z"
  generation: 1
  name: web-dashboard
  namespace: default
  resourceVersion: "865"
  uid: d4d9ea03-0daa-43c8-bc94-90910a5decc5
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      name: web-dashboard
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        name: web-dashboard
    spec:
      serviceAccountName: dashboard-sa
      containers:
      - env:

```

### Network Policy troubleshooting

Incoming or outgoing connections are not working because of network policy. In the default namespace, we deployed a default-deny network policy which is interrupting the incoming or outgoing connections.

Now, create a network policy called test-network-policy to allow the connections, as follows:-
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: secure-pod
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: webapp-color
    ports:
    - protocol: TCP
      port: 80
```
then check the connectivity from the webapp-color pod to the secure-pod:-
```
root@controlplane:~$ kubectl exec -it webapp-color -- sh
/opt # nc -v -z -w 5 secure-service 80
```
### ConfigMap/command exec
Create a namespace called dvl1987 by using the below command:-

$ kubectl create namespace dvl1987
Solution manifest file to create a configMap called time-config in the given namespace as follows:-
```
apiVersion: v1
data:
  TIME_FREQ: "10"
kind: ConfigMap
metadata:
  name: time-config
  namespace: dvl1987
```  
Now, create a pod called time-check in the same namespace as follows:-
```
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: time-check
  name: time-check
  namespace: dvl1987
spec:
  volumes:
  - name: log-volume
    emptyDir: {}
  containers:
  - image: busybox
    name: time-check
    env:
    - name: TIME_FREQ
      valueFrom:
            configMapKeyRef:
              name: time-config
              key: TIME_FREQ
    volumeMounts:
    - mountPath: /opt/time
      name: log-volume
    command:
    - "/bin/sh"
    - "-c"
    - "while true; do date; sleep $TIME_FREQ;done > /opt/time/time-check.log
```
## Deployment Rolling update 

Run the following command to create a manifest for deployment nginx-deploy and save it into a file:-

kubectl create deployment nginx-deploy --image=nginx:1.16 --replicas=4 --dry-run=client -oyaml > nginx-deploy.yaml
and add the strategy field under the spec section as follows:-
```
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
```
So final manifest file for deployment called nginx-deploy should looks like below:-
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy
  name: nginx-deploy
  namespace: default
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx-deploy
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
      - image: nginx:1.16
        imagePullPolicy: IfNotPresent
        name: nginx
```
then run the kubectl apply -f nginx-deploy.yaml to create a deployment resource.

Now, upgrade the deployment's image version using the kubectl set image command:-
```
kubectl set image deployment nginx-deploy nginx=nginx:1.17
```
Run the kubectl rollout command to undo the update and go back to the previous version:-
```
kubectl rollout undo deployment nginx-deploy
```
### Deployment with Resources request
Solution manifest file to create a deployment redis as follows:-
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: redis
spec:
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      volumes:
      - name: data
        emptyDir: {}
      - name: redis-config
        configMap:
          name: redis-config
      containers:
      - image: redis:alpine
        name: redis
        volumeMounts:
        - mountPath: /redis-master-data
          name: data
        - mountPath: /redis-master
          name: redis-config
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "0.2"
```
