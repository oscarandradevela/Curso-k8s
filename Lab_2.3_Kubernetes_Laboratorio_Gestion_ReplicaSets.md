
# Laboratorio: Gestión de ReplicaSets en Kubernetes

## Objetivos
- Conocer el funcionamiento de un ReplicaSet.

## Requerimientos
- Tener configurado un clúster de Kubernetes.

---

## Creación de un ReplicaSet
El propósito de un **ReplicaSet** es mantener un conjunto estable de réplicas de Pods en ejecución en un momento dado, garantizando así la disponibilidad de una cantidad específica de Pods idénticos.

Crear un archivo `replicaset.yaml` con el siguiente contenido para definir un ReplicaSet llamado `sample-rs` que use la imagen `httpd:alpine`:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: sample-rs
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: apache
        image: httpd:alpine
```

Aplicar el archivo YAML para crear el ReplicaSet:
```bash
kubectl apply -f replicaset.yaml
```

Para listar los ReplicaSets, puede usar el siguiente comando:
```bash
kubectl get replicaset
# O bien, la versión abreviada:
kubectl get rs
```

**Descripción de Campos Importantes:**
- **Name:** Nombre del ReplicaSet.
- **Desired:** Número de réplicas que Kubernetes debe mantener.
- **Current:** Número actual de réplicas creadas.
- **Ready:** Número de Pods en estado `running`.
- **Age:** Tiempo desde la creación del ReplicaSet.

---

## Gestión de Pods en un ReplicaSet
Listar los pods y eliminar uno para verificar la re-creación automática:
```bash
kubectl get pods
kubectl delete pod sample-rs-xxxx
kubectl get pods
```

Al eliminar un Pod, Kubernetes automáticamente crea uno nuevo para mantener el número de réplicas especificado en el ReplicaSet (`replicas: 2`).

---

## Creación de un Segundo ReplicaSet
Crear un archivo `rs-2.yaml` para definir un nuevo ReplicaSet llamado `replicaset-2` usando la imagen `nginx`:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-2
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: testing
  template:
    metadata:
      labels:
        tier: testing
    spec:
      containers:
      - name: nginx
        image: nginx
```

Aplicar el archivo YAML:
```bash
kubectl apply -f rs-2.yaml
```

Para obtener más detalles del ReplicaSet:
```bash
kubectl describe replicaset replicaset-2
```

Listar los Pods y verificar que las 3 réplicas estén en estado `running`, utilizando la etiqueta `tier=testing`:
```bash
kubectl get pods --selector tier=testing
```

---

## Eliminación de ReplicaSets
Eliminar los ReplicaSets creados:
```bash
kubectl delete replicaset replicaset-2
kubectl delete rs sample-rs
```

Verificar que los ReplicaSets y sus Pods han sido eliminados:
```bash
kubectl get replicaset
kubectl get pods
```
