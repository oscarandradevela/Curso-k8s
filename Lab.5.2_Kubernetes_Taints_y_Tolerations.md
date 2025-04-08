
# Laboratorio: Gestión de Taints y Tolerations en Kubernetes

## Objetivo

Este laboratorio está diseñado para dominar la configuración de `taints` y `tolerations` en Kubernetes, herramientas clave para gestionar y controlar en qué nodos pueden ejecutarse ciertos pods. Usaremos `taints` para marcar nodos de modo que solo ciertos pods, con las tolerancias adecuadas, puedan programarse en ellos.

## Requisitos

- Acceso a un clúster de Kubernetes en funcionamiento.
- Herramienta `kubectl` instalada y configurada para interactuar con el clúster.

---

## 1. Configuración de Taints

Los `taints` se aplican a los nodos para restringir la asignación de pods, a menos que estos tengan una `toleration` correspondiente. Existen tres efectos principales para los `taints`:

- **NoSchedule**: Impide que se programen pods en el nodo a menos que tengan la toleration adecuada.
- **PreferNoSchedule**: Kubernetes intentará evitar programar pods en el nodo, pero puede hacerlo si es necesario.
- **NoExecute**: Expulsa cualquier pod en el nodo que no tenga la toleration adecuada.

### Sintaxis para Agregar Taints

Para aplicar un `taint` en un nodo, usa:

```bash
kubectl taint nodes <nombre-del-nodo> key=value:<efecto-del-taint>
```

Un `taint` se compone de:

1. **Key**: Identificador del `taint`.
2. **Value**: Valor opcional del `taint`.
3. **Effect**: Efecto del `taint` (NoSchedule, PreferNoSchedule, o NoExecute).

### Ejemplo Práctico de un Taint

Para marcar un nodo y evitar que reciba pods sin la tolerancia necesaria:

```bash
kubectl taint nodes <nombre-del-nodo> role=backend:NoSchedule
```

### Remover un Taint

Para eliminar un `taint`, usa el mismo comando pero agrega un guion al final:

```bash
kubectl taint nodes <nombre-del-nodo> role=backend:NoSchedule-
```

---

## 2. Configuración de Tolerations en Pods

Las `tolerations` se usan para que ciertos pods toleren los `taints` de los nodos. Un pod con una `toleration` correspondiente puede programarse en un nodo marcado con un `taint`.

### Sintaxis de una Toleration

Una `toleration` se define en el manifiesto del pod y contiene:

1. **Key**: La clave del `taint` que el pod tolerará.
2. **Value**: Valor del `taint` (opcional).
3. **Effect**: Efecto del `taint` que el pod puede tolerar.

```yaml
tolerations:
- key: "role"
  operator: "Equal"
  value: "backend"
  effect: "NoSchedule"
```

### Operadores de Tolerations

- **Equal**: Coincide exactamente con el `taint`.
- **Exists**: Solo requiere la presencia de la clave sin importar el valor.

---

## 3. Implementación Paso a Paso

### Paso 1: Añadir un Taint a un Nodo

Ejecuta el siguiente comando para aplicar un `taint` `role=backend:NoSchedule` a un nodo en tu clúster:

```bash
kubectl taint nodes <nombre-del-nodo> role=backend:NoSchedule
```

### Paso 2: Crear un Pod sin Toleration

Crea el siguiente manifiesto `nginx-pod.yaml`, que genera un pod nginx sin `toleration`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

Aplica el manifiesto:

```bash
kubectl apply -f nginx-pod.yaml
```

### Verificación

Ejecuta el siguiente comando para revisar el estado del pod:

```bash
kubectl get pods
```

El pod debería aparecer en estado `Pending`, ya que no puede ejecutarse en el nodo con el `taint`.

### Paso 3: Agregar una Toleration al Pod

Edita el manifiesto `nginx-pod.yaml` para incluir una `toleration` que le permita ejecutarse en el nodo marcado:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
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

Aplica el manifiesto actualizado:

```bash
kubectl replace -f nginx-pod.yaml --force
```

### Paso 4: Verificar el Estado del Pod

Revisa el estado del pod para confirmar que ahora se ejecuta en el nodo con el `taint`:

```bash
kubectl get pod -o wide
```
