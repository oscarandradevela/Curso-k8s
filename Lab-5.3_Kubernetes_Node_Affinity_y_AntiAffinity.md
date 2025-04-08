
# Laboratorio: Implementación de Node Affinity y Anti-affinity en Kubernetes

## Objetivo

Este laboratorio permite configurar `node affinity` y `node anti-affinity` en Kubernetes para definir criterios de asignación de pods basados en etiquetas de nodos. Estas capacidades aseguran que los pods se programen en nodos que cumplan requisitos específicos, mejorando la disponibilidad y distribución de recursos.

## Requisitos

- Clúster de Kubernetes en funcionamiento.
- Herramienta `kubectl` instalada y configurada.

---

## 1. Configuración de Node Affinity en Kubernetes

### Paso 1: Etiquetar Nodos

Para usar reglas de afinidad, primero debes etiquetar los nodos de acuerdo con el criterio deseado. Usa el siguiente comando para etiquetar un nodo con `disktype=ssd`:

```bash
kubectl label nodes <nombre-del-nodo> disktype=ssd
```

### Paso 2: Crear Pod con Node Affinity

Define un pod que se ejecute solo en nodos etiquetados con `disktype=ssd`. Crea un archivo llamado `ssd-pod.yaml` con el siguiente contenido:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ssd-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
  tolerations:
  - key: "role"
    operator: "Equal"
    value: "backend"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx
```

Este manifiesto asegura que el pod se ejecute únicamente en un nodo con `disktype=ssd`.

### Paso 3: Desplegar el Pod

Aplica el archivo para desplegar el pod en el clúster:

```bash
kubectl apply -f ssd-pod.yaml
```

### Paso 4: Verificar Ubicación del Pod

Después de implementar el pod, verifica que esté programado en un nodo con la etiqueta `disktype=ssd`:

```bash
kubectl get pods -o wide
```

Busca la columna `NODE` para confirmar que `ssd-pod` se esté ejecutando en el nodo etiquetado.

---

## 2. Node Affinity Avanzada

### Asignación de Pods en Función de Múltiples Etiquetas

Para asignar un pod a un nodo con múltiples etiquetas, por ejemplo, `disktype=ssd` y `region=us-east`, agrega ambas etiquetas en `nodeSelectorTerms`:

```yaml
nodeSelectorTerms:
- matchExpressions:
  - key: disktype
    operator: In
    values:
    - ssd
  - key: region
    operator: In
    values:
    - us-east
```

### Node Affinity Preferencial

Para preferir que un pod se ejecute en nodos etiquetados, pero permitir otros si no hay disponibilidad, usa `preferredDuringSchedulingIgnoredDuringExecution`:

```yaml
affinity:
  nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      preference:
        matchExpressions:
        - key: disktype
          operator: In
          values:
          - ssd
```

---

## 3. Configuración de Node Anti-affinity en Kubernetes

La anti-affinity de nodo asegura que ciertos pods no se ubiquen juntos, útil para alta disponibilidad y distribución de carga.

### Paso 1: Crear una Implementación de Web Servers con Etiquetas

Primero, define una implementación con pods etiquetados. Crea un archivo `webserver-deployment.yaml` con el siguiente contenido:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webserver-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webserver
  template:
    metadata:
      labels:
        app: webserver
    spec:
      tolerations:
      - key: "role"
        operator: "Equal"
        value: "backend"
        effect: "NoSchedule"
      containers:
      - name: nginx
        image: nginx
```

Esto creará tres réplicas de un pod `webserver` etiquetado.

### Paso 2: Añadir Node Anti-affinity

Para distribuir los pods en distintos nodos, modifica el archivo para incluir una regla de `node anti-affinity`:

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - webserver
      topologyKey: "kubernetes.io/hostname"
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webserver-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webserver
  template:
    metadata:
      labels:
        app: webserver
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - webserver
            topologyKey: "kubernetes.io/hostname"
      tolerations:
      - key: "role"
        operator: "Equal"
        value: "backend"
        effect: "NoSchedule"
      containers:
      - name: nginx
        image: nginx
```
Este ajuste asegura que los pods `webserver` se ubiquen en distintos nodos, utilizando el `topologyKey` para mantenerlos separados.

### Paso 3: Aplicar la Implementación

Aplica el archivo modificado en el clúster:

```bash
kubectl apply -f webserver-deployment.yaml
```

### Paso 4: Verificar Distribución de Pods

Comprueba que los pods se distribuyan correctamente entre los nodos:

```bash
kubectl get pods -o wide -l app=webserver
```

Revisa la columna `NODE` para confirmar que cada pod esté en un nodo diferente.
