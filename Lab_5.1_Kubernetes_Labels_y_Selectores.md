
# Laboratorio: Uso de Labels y Selectores en Kubernetes

## Objetivo

Este laboratorio tiene como objetivo aprender a aplicar y gestionar labels (etiquetas) en objetos de Kubernetes y utilizar selectores para filtrar y organizar estos objetos. A través de ejercicios prácticos, entenderás cómo agrupar recursos y realizar operaciones de manera eficiente mediante labels y selectores.

## Requisitos

- Acceso a un clúster de Kubernetes en funcionamiento.
- Herramienta `kubectl` instalada y configurada para interactuar con el clúster.

---

## 1. Aplicación de Labels en Kubernetes

### Paso 1: Crear un Pod con Labels

Crea un archivo YAML llamado `labeled-pod.yaml` con el siguiente contenido. Este pod incluirá labels en su configuración para identificación y organización.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    env: production
spec:
  containers:
  - name: nginx
    image: nginx
```

Aplica el manifiesto YAML al clúster para crear el pod:

```bash
kubectl apply -f labeled-pod.yaml
```

Verifica que el pod tenga los labels aplicados:

```bash
kubectl get pods --show-labels
```

### Paso 2: Agregar un Label a un Pod Existente

Agrega un label nuevo al pod creado, indicando una versión para este recurso:

```bash
kubectl label pod myapp-pod version=1.0
```

Confirma que el label se ha añadido correctamente:

```bash
kubectl get pods myapp-pod --show-labels
```

### Paso 3: Eliminar un Label de un Pod

Para eliminar el label `env`, utiliza el siguiente comando:

```bash
kubectl label pod myapp-pod env-
```

Verifica que el label se haya eliminado:

```bash
kubectl get pod myapp-pod --show-labels
```

---

## 2. Uso de Selectores para Filtrar Objetos por Labels

### Paso 1: Crear Pods Adicionales con Labels

Crea dos archivos YAML adicionales, llamados `dev-pod.yaml` y `test-pod.yaml`, con diferentes labels para los entornos de desarrollo y prueba.

**Archivo dev-pod.yaml:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dev-pod
  labels:
    app: myapp
    env: development
spec:
  containers:
  - name: nginx
    image: nginx
```

**Archivo test-pod.yaml:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  labels:
    app: myapp
    env: testing
spec:
  containers:
  - name: nginx
    image: nginx
```

Aplica ambos archivos al clúster:

```bash
kubectl apply -f dev-pod.yaml
kubectl apply -f test-pod.yaml
```

### Paso 2: Listar Pods por Label

Usa selectores para listar todos los pods con el label `app=myapp`:

```bash
kubectl get pods --selector app=myapp
```

### Paso 3: Seleccionar Pods por Múltiples Labels

Filtra los pods que pertenecen al entorno `pruebas` y tienen la aplicación `myapp`:

```bash
kubectl get pods --selector app=myapp,env=testing
```

### Paso 4: Describir Recursos con un Selector

Obtén detalles de todos los pods en el entorno `development`:

```bash
kubectl describe pods --selector env=development
```

---

## 3. Uso de Labels en Servicios

### Paso 1: Crear un Servicio con Selectores

Crea un archivo YAML para un servicio llamado `myapp-service.yaml` que seleccionará los pods con el label `app=myapp`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

Aplica el manifiesto YAML al clúster:

```bash
kubectl apply -f myapp-service.yaml
```

### Paso 2: Verificar el Servicio

Lista el servicio para confirmar que se ha creado:

```bash
kubectl get services myapp-service
```

Confirma que el servicio está seleccionando los pods correctos:

```bash
kubectl describe service myapp-service
```
