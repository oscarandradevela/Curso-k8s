# Simulación del Examen CKA - Resolución

## Pregunta 1: Cluster Architecture, Installation & Configuration
```sh
kubectl create clusterrole read-nodes-role --verb=get,list,watch --resource=nodes
kubectl create rolebinding read-nodes-binding --clusterrole=read-nodes-role --user=dev-user --namespace=default
```

## Pregunta 2: Workloads & Scheduling 
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
  namespace: default
spec:
  selector:
    matchLabels:
      app: log-collector
  template:
    metadata:
      labels:
        app: log-collector
    spec:
      containers:
      - name: fluentd
        image: fluentd:latest
        volumeMounts:
        - name: logs
          mountPath: /logs
      volumes:
      - name: logs
        hostPath:
          path: /var/log
```

## Pregunta 3: Workloads & Scheduling 
```sh
kubectl taint nodes worker-node-1 dedicated=database:NoSchedule
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db-deployment
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
      tolerations:
      - key: "dedicated"
        operator: "Equal"
        value: "database"
        effect: "NoSchedule"
```

## Pregunta 4: Workloads & Scheduling 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
  namespace: default
spec:
  volumes:
  - name: shared-volume
    emptyDir: {}
  containers:
  - name: web-container
    image: nginx
    ports:
    - containerPort: 80
  - name: health-checker
    image: busybox
    command: ["/bin/sh", "-c", "while true; do wget -q --spider http://localhost:80 && echo 'OK'; sleep 10; done"]
    volumeMounts:
    - name: shared-volume
      mountPath: /shared
```

## Pregunta 5: Services & Networking 
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: default
spec:
  selector:
    app: backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

## Pregunta 6: Services & Networking 
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
  namespace: default
spec:
  selector:
    app: web-app
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
  type: LoadBalancer
```

## Pregunta 7: Services & Networking 
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

## Pregunta 8: Storage 
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/aws-ebs
```
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: fast-storage
```

## Pregunta 9: Storage 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-logs
  namespace: default
spec:
  volumes:
  - name: log-storage
    persistentVolumeClaim:
      claimName: pvc-logs
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: log-storage
      mountPath: /usr/share/nginx/html
      subPath: html
    - name: log-storage
      mountPath: /var/log/nginx
      subPath: logs
```

## Pregunta 10: Troubleshooting 
```sh
kubectl describe pod broken-pod
kubectl logs broken-pod
```
**Corrección:** Revisar configuraciones erróneas y reiniciar el pod.

## Pregunta 11: Troubleshooting 
```sh
kubectl describe service api-service
kubectl get endpoints api-service
```
**Corrección:** Verificar si los pods están etiquetados correctamente y asegurarse de que el selector del servicio coincide con ellos.

## Pregunta 12: Troubleshooting 
```sh
kubectl get pods -A -o wide
kubectl get nodes
kubectl describe nodes
```
**Corrección:** Comprobar el estado de los complementos de red y reiniciar los nodos si es necesario.

## Pregunta 13: Backup y Restauración de etcd 
```sh
ETCDCTL_API=3 etcdctl snapshot save /var/lib/etcd-backup/snapshot.db
rm -rf /var/lib/etcd/*
ETCDCTL_API=3 etcdctl snapshot restore /var/lib/etcd-backup/snapshot.db --data-dir /var/lib/etcd
```

## Pregunta 14: Troubleshooting 
```sh
kubectl get nodes
kubectl describe node worker-node
journalctl -u kubelet -f
```
**Corrección:** Revisar logs de kubelet y estado del nodo, reiniciar servicios según sea necesario.
