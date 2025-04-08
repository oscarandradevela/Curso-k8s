
# Laboratorio: Creación y Gestión de Pods en Kubernetes

## Objetivos
- Ejecutar y crear pods mediante un archivo YAML.
- Conocer el funcionamiento de un contenedor dentro de un pod.

## Requerimientos
- Tener configurado un clúster de Kubernetes.

## Visión General de Kubernetes
Verifique la versión de Kubernetes de su clúster:
```bash
kubectl version
```

**Kubectl** conecta al servidor API de Kubernetes utilizando la API RESTful. Por defecto, accede al host local mediante `.kube/config`; en caso contrario, especifique la dirección del servidor API con `--server`.

Kubectl admite muchos subcomandos; use `-h` para ver los detalles:
```bash
kubectl -h
kubectl get -h
kubectl run -h
```

**Sintaxis de `kubectl`:**
```bash
kubectl [command] [TYPE] [NAME] [flags]
```
- **command:** Operación a realizar en recursos.
- **TYPE:** Tipo de recurso (sensible a mayúsculas y minúsculas).
- **NAME:** Nombre del recurso; omitir para ver todos los recursos.
- **flags:** Etiquetas o banderas opcionales.

---

## Creación de Pods
Crear un archivo `webapp.yaml` con el siguiente contenido para desplegar un pod llamado `webapp` con la imagen `nginx:alpine`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - image: nginx:alpine
    name: webapp
```

**Explicación de parámetros:**
- **apiVersion:** Versión de API utilizada.
- **kind:** Tipo de recurso, en este caso, un Pod.
- **metadata:** Identificador único para el pod.
- **spec:** Detalles del pod.
- **containers:** Lista de contenedores dentro del pod.
- **image:** Imagen a ejecutar en el contenedor.
- **name:** Nombre del contenedor.

Para generar el YAML de ejemplo de un pod:
```bash
kubectl run podname --image name_image --dry-run=client -o yaml
```

Crear el pod desde el archivo YAML:
```bash
kubectl apply -f webapp.yaml
kubectl get pods
```

Ver los logs del pod:
```bash
kubectl logs webapp
```

---

## Crear un Pod con Más Parámetros
Crear el archivo `webapp-v2.yaml` para un nuevo pod con etiquetas y un puerto específico:
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: webapp-v2
  name: webapp-v2
spec:
  containers:
  - image: nginx:alpine
    name: webapp-v2
    ports:
    - containerPort: 80
```

**Explicación de parámetros adicionales:**
- **labels:** Etiquetas para categorizar pods.
- **containerPort:** Puerto por el cual escucha el contenedor.

Desplegar el pod:
```bash
kubectl apply -f webapp-v2.yaml
kubectl get pods
```

Para ver detalles sobre los pods creados, incluyendo el nodo en el que se están desplegando:
```bash
kubectl get pods -o wide
```

---

## Eliminar y Recrear Pods
Eliminar el pod `webapp` y recrearlo usando el archivo YAML:
```bash
kubectl delete pod webapp
kubectl apply -f webapp.yaml
```

Ingresar a un pod y verificar el funcionamiento de Nginx:
```bash
kubectl exec -it webapp-v2 -- sh
curl localhost:80
```

## Eliminar Todos los Pods
Para finalizar, eliminar todos los pods creados:
```bash
kubectl delete pod --all
```

**Nota:** Usar `--all` con precaución, ya que eliminará todos los pods en el namespace.
