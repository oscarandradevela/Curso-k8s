
# Laboratorio N° 3.1 - Creación y Gestión de Services en Kubernetes

## Objetivos
- Configurar y exponer aplicaciones mediante **Services** para habilitar la comunicación interna entre Pods.
- Publicar aplicaciones utilizando **Services** de tipo `NodePort` para acceso externo.

## Requisitos
- Contar con un clúster de Kubernetes previamente configurado y funcional.

---

### Introducción a los Services
En Kubernetes, los **Services** son objetos que abstraen y exponen aplicaciones ejecutándose en un conjunto de Pods. Su propósito es mantener una comunicación estable y consistente entre Pods, independientemente de las direcciones IP efímeras de estos.

---

### Configuración de un Service de tipo `ClusterIP`
Un `ClusterIP` es el tipo de Service por defecto, diseñado para permitir la comunicación interna dentro del clúster.

1. **Listar los Services existentes**  
   Para verificar los servicios configurados en tu clúster:
   ```bash
   kubectl get svc -A
   ```

2. **Crear un Deployment**  
   Crea un Deployment llamado `webapp-deploy.yaml` con dos réplicas:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: webapp
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: webapp
     template:
       metadata:
         labels:
           app: webapp
       spec:
         containers:
         - image: kodekloud/simple-webapp:red
           name: webapp
           ports:
           - containerPort: 8080
             protocol: TCP
   ```

   Aplica el Deployment:
   ```bash
   kubectl create ns cloudnative
   kubectl apply -f webapp-deploy.yaml -n cloudnative
   ```

3. **Crear el Service ClusterIP**  
   Crea un Service llamado `webapp-svc.yaml`:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: webapp-svc
   spec:
     selector:
       app: webapp
     ports:
     - port: 8080
       targetPort: 8080
       protocol: TCP
   ```

   Aplica el archivo y verifica:
   ```bash
   kubectl apply -f webapp-svc.yaml
   kubectl get svc 
   ```

---

### Configuración de un Service de tipo `NodePort`
El tipo `NodePort` permite exponer un Service fuera del clúster utilizando un puerto específico en cada nodo.

1. **Crear un Deployment llamado `frontend`**  
   Ejecuta el siguiente comando:
   ```bash
   kubectl create deployment frontend --image kodekloud/simple-webapp:red --replicas 2 -n cloudnative
   ```

2. **Crear el Service NodePort**  
   Crea un Service llamado `frontend-svc.yaml` con un puerto NodePort específico:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: frontend-svc
     namespace: cloudnative
     labels:
       app: frontend
   spec:
     type: NodePort
     selector:
       app: frontend
     ports:
     - port: 8080
       targetPort: 8080
       protocol: TCP
       nodePort: 32661
   ```

3. **Aplicar el Service y verificar**  
   ```bash
   kubectl apply -f frontend-svc.yaml
   kubectl get svc frontend-svc -n cloudnative
   ```

4. **Eliminar el Service al finalizar**  
   ```bash
   kubectl delete svc frontend-svc.yaml -n cloudnative
   ```

---