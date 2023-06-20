# k8s-exam

## Pod

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
kubectl create secret tls webhook-server-tls \
  --cert=/root/keys/webhook-server-tls.crt\
  --key=/root/keys/webhook-server-tls.key

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

kubectl create secret generic db-secret-xxdf --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123

kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3



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
