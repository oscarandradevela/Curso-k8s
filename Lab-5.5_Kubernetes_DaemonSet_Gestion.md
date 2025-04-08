
# Laboratorio: Implementación y Gestión de DaemonSets en Kubernetes

## Objetivo

Este laboratorio permite aprender a crear, actualizar y gestionar DaemonSets en Kubernetes, usando como ejemplo un DaemonSet que ejecuta Fluentd en cada nodo del clúster para recolección de logs.

## Requisitos

- Clúster de Kubernetes en funcionamiento.
- Herramienta `kubectl` instalada y configurada.

---

## 1. Creación de un DaemonSet

### Paso 1: Crear el Archivo del DaemonSet

Crea un archivo llamado `fluentd-daemonset.yaml` con el siguiente contenido para implementar Fluentd en cada nodo:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-daemonset
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.17-1
```

### Paso 2: Aplicar el DaemonSet

Aplica el archivo para crear el DaemonSet en el clúster:

```bash
kubectl apply -f fluentd-daemonset.yaml
```

### Paso 3: Verificar los DaemonSets en Ejecución

Para ver todos los DaemonSets en el clúster, ejecuta el siguiente comando:

```bash
kubectl get daemonsets
```

---

## 2. Actualización de un DaemonSet

### Paso 1: Modificar el DaemonSet

Actualiza el archivo `fluentd-daemonset.yaml` para utilizar una versión específica de Fluentd, como la `v1.2.3`:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-daemonset
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.2.3
```

### Paso 2: Aplicar los Cambios

Aplica el archivo actualizado para realizar la actualización del DaemonSet:

```bash
kubectl apply -f fluentd-daemonset.yaml
```

### Paso 3: Verificar los Pods del DaemonSet

Para verificar que los pods se hayan actualizado, lista los pods con el label `app=fluentd`:

```bash
kubectl get pods -l app=fluentd
```

---

## 3. Visualización de Logs de los Pods del DaemonSet

Una vez que tengas el nombre de un pod de Fluentd, puedes ver los logs usando el siguiente comando:

```bash
kubectl logs <nombre-del-pod>
```

---

## 4. Realizar un Rollback de un DaemonSet

Modificamos el nombre del container del daemonset a fluentd-test

```bash
kubectl edit daemonset fluentd-daemonset
name: fluentd-test
```
Para restaurar el DaemonSet a una versión anterior, usa el comando `rollout undo`:

```bash
kubectl rollout undo daemonset fluentd-daemonset
```

---

## 5. Eliminación de un DaemonSet

Para eliminar el DaemonSet llamado `fluentd-daemonset`, usa el siguiente comando:

```bash
kubectl delete daemonset fluentd-daemonset
```
