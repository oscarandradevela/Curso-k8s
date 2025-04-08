
# Laboratorio: Implementación de Resource Quotas y Limit Ranges en Kubernetes

## Objetivo

Este laboratorio guía la configuración de `Resource Quotas` y `Limit Ranges` en Kubernetes para gestionar los recursos dentro de un namespace. Estas configuraciones permiten controlar el uso de CPU, memoria y almacenamiento, asegurando que cada pod, contenedor y PersistentVolumeClaim (PVC) respete los límites establecidos.

## Requisitos

- Clúster de Kubernetes en funcionamiento.
- Herramienta `kubectl` instalada y configurada.

---

## 1. Configuración de Resource Quotas en Kubernetes

### Paso 1: Crear el Archivo de Resource Quota

Creamos un nuevo namespace llamado example-namespace.
```bash
kubectl create ns example-namespace
```

Crea un archivo llamado `resource-quota.yaml` con el siguiente contenido. Esto define una Resource Quota que limita el número de pods, CPU y memoria en el namespace `example-namespace`:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: resource-quota-example
  namespace: example-namespace
spec:
  hard:
    pods: "1"
    requests.cpu: "2"
    requests.memory: "5Gi"
    limits.cpu: "4"
    limits.memory: "10Gi"
```

### Paso 2: Aplicar la Resource Quota

Aplica la Resource Quota creada al namespace deseado:

```bash
kubectl apply -f resource-quota.yaml
```

### Paso 3: Verificar la Resource Quota

Para verificar los límites establecidos y el uso actual, ejecuta:

```bash
kubectl get resourcequota resource-quota-example -n example-namespace
```

También puedes revisar el estado de todo el namespace:

```bash
kubectl describe namespace example-namespace
```

---

## 2. Pruebas de Resource Quota

### Paso 1: Intentar Crear Múltiples Pods

La Resource Quota anterior permite solo un pod en el namespace. Crea una implementación `quota-test` con 2 réplicas y observa el comportamiento.

Crea un archivo `deploy.yaml` con el siguiente contenido:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: quota-test
  namespace: example-namespace
  labels:
    app: deployment-label
spec:
  selector:
    matchLabels:
      app: deployment-label
  replicas: 2
  template:
    metadata:
      labels:
        app: deployment-label
    spec:
      containers:
      - name: nginx-deploy
        image: nginx:latest
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
          limits:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 80
          name: nginx-deploy
```

Aplica el archivo:

```bash
kubectl apply -f deploy.yaml
```

### Paso 2: Verificar la Implementación

Verifica que uno de los pods esté en estado `Running` mientras el otro permanece `Pending` debido a la restricción de `Resource Quota`:

```bash
kubectl get all -n example-namespace
```

Ecalamos los pods de quota-test a cero antes de ocntinuar con la siguiente sección de ejercicios.

```bash
kubectl scale deploy quota-test  --replicas=0 -n example-namespace
```

---

## 3. Configuración de Limit Ranges en Kubernetes

`Limit Ranges` asegura que cada pod o contenedor respete ciertos valores de recursos mínimos y máximos.

### Paso 1: Crear el Archivo de Limit Range

Crea un archivo llamado `limit-range.yaml` con el siguiente contenido para definir los valores de recursos predeterminados en el namespace:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: limit-range-example
  namespace: example-namespace
spec:
  limits:
  - type: Pod
    max:
      cpu: "2"
      memory: "4Gi"
    min:
      cpu: "200m"
      memory: "256Mi"
    maxLimitRequestRatio:
      cpu: "4"
      memory: "8"
  - type: Container
    default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "250m"
      memory: "256Mi"
    max:
      cpu: "1"
      memory: "1Gi"
    min:
      cpu: "100m"
      memory: "128Mi"
    maxLimitRequestRatio:
      cpu: "2"
      memory: "4"
  - type: PersistentVolumeClaim
    max:
      storage: "10Gi"
    min:
      storage: "1Gi"
    default:
      storage: "5Gi"
    defaultRequest:
      storage: "2Gi"
    maxLimitRequestRatio:
      storage: "2"
```

### Paso 2: Aplicar el Limit Range

Aplica el Limit Range en el namespace correspondiente:

```bash
kubectl apply -f limit-range.yaml
```

### Paso 3: Verificar el Limit Range

Revisa los límites aplicados con:

```bash
kubectl describe limitrange limit-range-example -n example-namespace
kubectl describe namespace example-namespace
```

---

## 4. Pruebas de Limit Range

### Paso 1: Crear una Implementación sin Especificar Recursos

Para ver los valores predeterminados del Limit Range en acción, crea una implementación sin definir límites específicos en su archivo YAML.

Crea un archivo `limit-test-deploy.yaml` con el siguiente contenido:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: limit-test
  namespace: example-namespace
  labels:
    app: deployment-label
spec:
  selector:
    matchLabels:
      app: deployment-label
  replicas: 1
  template:
    metadata:
      labels:
        app: deployment-label
    spec:
      containers:
      - name: nginx-deploy
        image: nginx:latest
```

Aplica el manifiesto:

```bash
kubectl apply -f limit-test-deploy.yaml
```

### Paso 2: Verificar los Recursos del Pod

Verifica que el pod tiene los valores de recursos establecidos por el Limit Range:

```bash
kubectl describe pod -n example-namespace
```

Esto concluye la implementación de `Resource Quotas` y `Limit Ranges` en un namespace de Kubernetes.
