
# Laboratorio N° 4.1 - Rolling Updates y Rollbacks

## Objetivos
- Crear deployment con estrategia de despliegue Rolling update.

## Requerimientos
- Tener configurado un clúster de Kubernetes.

---

## Creación de aplicación usando Rolling Update
1. Crear un namespace llamado `lab4`:
   ```bash
   kubectl create ns lab4
   ```

2. Crear un deployment llamado `ms-web.yaml` con el siguiente YAML:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     labels:
       app: ms-web
     name: ms-web
     namespace: lab4
   spec:
     replicas: 4
     selector:
       matchLabels:
         app: ms-web
     strategy: {}
     template:
       metadata:
         labels:
           app: ms-web
       spec:
         containers:
         - image: 040519/cna-webapp:red
           name: cna-webapp
   ```

3. Crear el recurso con el comando:
   ```bash
   kubectl create -f ms-web.yaml
   ```

4. Exponer el deployment mediante un servicio NodePort para validar cambios:
   ```bash
   kubectl expose deployment ms-web --name ms-web-svc --port 8080 --type NodePort -n lab4
   kubectl get svc -n lab4
   ```

5. Verificar el tipo de estrategia de despliegue aplicada por defecto:
   ```bash
   kubectl describe deployment ms-web -n lab4
   ```

## Actualización de la Imagen del Deployment
Actualizar la imagen del deployment `ms-web` a `040519/cna-webapp:green`:
```bash
kubectl edit deployment ms-web -n lab4
```
- O realizar el cambio desde línea de comandos:
  ```bash
  kubectl set image deployment/ms-web cna-webapp=040519/cna-webapp:green -n lab4
  ```

Validar en el navegador para ver el cambio en la aplicación (debe visualizarse en verde).

## Rollback de Actualizaciones
1. Verificar el historial de cambios del deployment:
   ```bash
   kubectl rollout history deployment ms-web -n lab4
   ```

2. Realizar el rollback a una versión anterior:
   ```bash
   kubectl rollout undo deployment ms-web -n lab4
   ```

3. Verificar en el navegador que la aplicación ha regresado al estado anterior (color rojo).

---
