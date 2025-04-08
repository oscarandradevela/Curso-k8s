# Simulación de Examen CKAD - Resolución

---

## Pregunta 1: Core Concepts
```sh
kubectl create namespace ckad-test
kubectl run busybox-pod --image=busybox --namespace=ckad-test --command -- sleep 3600
kubectl get pods -n ckad-test
```

---

## Pregunta 2: Configuration
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
  namespace: default
data:
  DB_HOST: db.example.com
  DB_PORT: "5432"
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        envFrom:
        - configMapRef:
            name: db-config
```

---

## Pregunta 3: Pod Design
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: logging-pod
  namespace: default
spec:
  volumes:
  - name: shared-logs
    emptyDir: {}
  containers:
  - name: nginx
    image: nginx:1.21
  - name: logger
    image: busybox
    command: ["/bin/sh", "-c", "while true; do echo 'Log entry $(date)' >> /var/log/app.log; sleep 5; done"]
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log
```

---

## Pregunta 4: Observability
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: metrics-pod
  namespace: default
spec:
  containers:
  - name: nginx
    image: nginx
    livenessProbe:
      httpGet:
        path: /metrics
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 5
```
```sh
kubectl logs metrics-pod
```

---

## Pregunta 5: Services & Networking
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: default
spec:
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
```

---

## Pregunta 6: State Persistence
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-data
spec:
  capacity:
    storage: 2Gi
  accessModes:
  - ReadWriteMany
  storageClassName: manual
```
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-data
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 500Mi
  storageClassName: manual
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod
  namespace: default
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c", "echo 'Test File' > /data/testfile.txt; sleep 3600"]
    volumeMounts:
    - name: storage-volume
      mountPath: /data
  volumes:
  - name: storage-volume
    persistentVolumeClaim:
      claimName: pvc-data
```
---

## Pregunta 7: Multi-Container Pods
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
  namespace: default
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
  - name: checker
    image: busybox
    command: ["/bin/sh", "-c", "while true; do wget -q --spider http://localhost:80 && echo 'Accessible'; sleep 10; done"]
```

---

## Pregunta 8: Networking Policies
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx
  namespace: ckad-test
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

---

## Pregunta 9: Configuración Dinámica con Secrets
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
  namespace: default
stringData:
  username: admin
  password: SuperSecretPassword
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-backend
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        envFrom:
        - secretRef:
            name: db-credentials
```

---

## Pregunta 10: Gestión de Jobs y CronJobs
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-backup
  namespace: default
spec:
  template:
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["/bin/sh", "-c", "echo 'Backup Complete'"]
      restartPolicy: Never
```
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-task
  namespace: default
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: busybox
            image: busybox
            command: ["/bin/sh", "-c", "date"]
          restartPolicy: OnFailure
```

---

## Pregunta 11: Escalado Horizontal
```sh
kubectl autoscale deployment webapp --cpu-percent=50 --min=1 --max=5
```

---

## Pregunta 12: Volúmenes Configurados con SubPaths
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-with-logs
  namespace: default
spec:
  volumes:
  - name: shared-data
    emptyDir: {}
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
      subPath: html
    - name: shared-data
      mountPath: /var/log/nginx
      subPath: logs
```

---

## Pregunta 13: Canary Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-v1
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-v2
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.23
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
  namespace: default
spec:
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

---

## Pregunta 14: Rollbacks
```sh
kubectl create deployment api-deployment --image=nginx:1.21
kubectl set image deployment/api-deployment nginx=nginx:1.22
kubectl rollout undo deployment/api-deployment
```

---