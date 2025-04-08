# Laboratorio: Kubernetes SecurityContext

## Objetivo

El objetivo de este laboratorio es aprender a aplicar configuraciones de `SecurityContext` en Kubernetes para gestionar los permisos de acceso y ejecutar procesos de forma segura en contenedores. Implementaremos configuraciones de seguridad avanzadas como usuarios no root, restricciones de capacidades y sistemas de archivos de solo lectura, lo cual es clave para proteger entornos en Kubernetes.

## Requisitos

- Acceso a un clúster de Kubernetes en funcionamiento.
- Herramienta `kubectl` instalada y configurada para interactuar con el clúster.
- Imágenes de contenedor necesarias: `redhat/ubi8` para contenedores de prueba y `demisto/iputils:1.0.0.62867` para ejecutar pods con usuarios no root.

---

## Configuración de un Pod Privilegiado Regular

### Paso 1: Crear un Pod Privilegiado Regular

Este pod utiliza una configuración sin restricciones de seguridad adicionales. Guarda el siguiente manifiesto en un archivo llamado `regular-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: regular-pod-demo
spec:
  volumes:
  - name: demo-volume
    emptyDir: {}
  containers:
  - name: regular-demo
    image: redhat/ubi8
    command: [ "tail", "-f", "/dev/null" ]
    volumeMounts:
    - name: demo-volume
      mountPath: /data/demo
```

### Paso 2: Desplegar el Pod y Acceder a Éste

Ejecuta los siguientes comandos para crear el pod y acceder a él en una sesión interactiva:

```bash
kubectl create -f regular-pod.yaml 
kubectl get pod/regular-pod-demo
kubectl exec -it regular-pod-demo -- sh
```

### Paso 3: Verificar Usuario y Capacidades Activas

Dentro del contenedor, verifica el UID, GID y capacidades activas del proceso:

```bash
stat -c "%u %g" /proc/1/
capsh --print
```

---

## Creación de un Pod No Privilegiado

### Paso 1: Crear un Pod con Restricciones de Privilegios

Este pod tiene todos los privilegios eliminados, mejorando la seguridad. Guarda el siguiente manifiesto en `non-privileged-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: non-privileged-pod-demo
spec:
  volumes:
  - name: demo-volume
    emptyDir: {}
  containers:
  - name: non-privileged-demo
    image: redhat/ubi8
    command: [ "tail", "-f", "/dev/null" ]
    volumeMounts:
    - name: demo-volume
      mountPath: /data/demo
    securityContext:
      privileged: false
      allowPrivilegeEscalation: false
      capabilities:
        drop:
          - ALL
```

### Paso 2: Desplegar y Verificar el Pod

```bash
kubectl create -f non-privileged-pod.yaml
kubectl exec -it non-privileged-pod-demo -- sh
capsh --print
```

---

## Pod No Privilegiado con Usuario No Root

### Paso 1: Crear un Pod con Usuario Específico (No Root)

Guarda el siguiente manifiesto en `non-root-user-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: non-root-user-pod-demo
spec:
  securityContext:
    runAsNonRoot: true
  volumes:
  - name: demo-volume
    emptyDir: {}
  containers:
  - name: non-root-user-demo
    image: demisto/iputils:1.0.0.62867
    command: [ "tail", "-f", "/dev/null" ]
    volumeMounts:
    - name: demo-volume
      mountPath: /data/demo
    securityContext:
      runAsUser: 1001
      privileged: false
      allowPrivilegeEscalation: false
      capabilities:
        drop:
          - ALL
```

### Paso 2: Desplegar y Verificar el Usuario No Root

```bash
kubectl create -f non-root-user-pod.yaml
kubectl exec -it non-root-user-pod-demo -- sh
stat -c "%u %g" /proc/1/
id
```

---

## Pod No Privilegiado con Capacidad Específica (Comando Ping)

### Paso 1: Crear Pod con Capacidad Específica

Guarda el siguiente manifiesto en `capability-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: capability-pod-demo
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
  - name: demo-volume
    emptyDir: {}
  containers:
  - name: capability-demo
    image: demisto/iputils:1.0.0.62867
    command: [ "tail", "-f", "/dev/null" ]
    volumeMounts:
    - name: demo-volume
      mountPath: /data/demo
    securityContext:
      privileged: false
      allowPrivilegeEscalation: true
      capabilities:
        drop:
          - ALL
        add:
         - NET_RAW
```

### Paso 2: Desplegar y Ejecutar Ping

```bash
kubectl create -f capability-pod.yaml
kubectl exec -it capability-pod-demo -- ping google.com
```

---

## Configurar el Sistema de Archivos como Solo Lectura

### Paso 1: Crear Pod con Sistema de Archivos de Solo Lectura

Guarda el siguiente manifiesto en `read-only-root-fs.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: read-only-root-fs-pod-demo
spec:
  containers:
  - name: read-only-root-fs-demo
    image: redhat/ubi8
    command: [ "tail", "-f", "/dev/null" ]
    securityContext:
      readOnlyRootFilesystem: true
```

### Paso 2: Desplegar y Probar

```bash
kubectl create -f read-only-root-fs.yaml
kubectl exec -it read-only-root-fs-pod-demo -- touch aa
```

---