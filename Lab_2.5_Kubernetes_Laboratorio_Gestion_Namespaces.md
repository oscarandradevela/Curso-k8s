
# Laboratorio: Gestión de Namespaces en Kubernetes

## Objetivos
- Crear nuevos namespaces.
- Gestionar aplicaciones mediante namespaces.

## Requerimientos
- Tener configurado un clúster de Kubernetes.

---

## ¿Qué es un Namespace?
En Kubernetes, un **Namespace** permite segmentar los recursos dentro de un clúster, proporcionando aislamiento entre diferentes aplicaciones o grupos.

---

## Explorando los Namespaces
Listar los namespaces existentes en el clúster:
```bash
kubectl get namespaces
```

Por defecto, Kubernetes crea los siguientes namespaces al iniciar el clúster:
- **default:** Namespace por defecto para objetos si no se especifica uno.
- **kube-node-lease:** Coordina el tiempo de vida de los nodos.
- **kube-public:** Generalmente accesible para todos, útil para datos públicos.
- **kube-system:** Contiene recursos del sistema de Kubernetes.

## Creación de Namespaces
Crear un nuevo namespace llamado `cloudnative`:
```bash
kubectl create namespace cloudnative
```

Utilizar la abreviatura `ns` para listar los namespaces:
```bash
kubectl get ns
```

### Crear un Namespace desde un YAML
Crear un archivo `ns.yaml` para definir el namespace `k8slab`:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: k8slab
```

Aplicar el archivo YAML y listar los namespaces:
```bash
kubectl apply -f ns.yaml
kubectl get ns
```

---

## Creación de Pods en Namespaces Específicos
Crear un pod en el namespace `cloudnative` usando la imagen `redis`. Crear el archivo `db.yaml` con el siguiente contenido:
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: db
  name: database
  namespace: cloudnative
spec:
  containers:
  - image: redis
    name: db
```

Aplicar el archivo YAML y verificar el estado del pod:
```bash
kubectl apply -f db.yaml
kubectl get pods -n cloudnative
```

### Crear el Mismo Pod en un Namespace Diferente
Crear el archivo `db2.yaml` para definir un pod idéntico en el namespace `k8slab`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: db
  name: database
  namespace: k8slab
spec:
  containers:
  - image: redis
    name: db
```

Aplicar el archivo YAML y verificar el estado del pod en `k8slab`:
```bash
kubectl apply -f db2.yaml
kubectl get pods -n k8slab
```

**Nota:** Si el namespace no está definido en el YAML, puede especificarlo con el parámetro `-n`:
```bash
kubectl apply -f file.yaml -n mynamespace
```

---

## Eliminación de Namespaces
Para eliminar un namespace y todos sus recursos:
```bash
kubectl delete ns k8slab
```

**Advertencia:** Al eliminar un namespace, se eliminan todos los recursos creados dentro de él (Pods, ReplicaSets, Deployments, etc.).
