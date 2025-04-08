# Laboratorio N° 4.2 - Estrategias de Despliegue

## Objetivos
- Comprender las estrategias de despliegue en Kubernetes.
- Implementar la estrategia Blue-Green.

## Requerimientos
- Contar con un clúster de Kubernetes operativo.

---

### Canary Deployment
La estrategia **Canary** permite desplegar una nueva versión de la aplicación gradualmente para minimizar riesgos.

1. **Desplegar la versión inicial (v1)**  
   Crea un archivo llamado `canary-v1.yaml` con el siguiente contenido:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     labels:
       app: httpd
     name: myapp-v1
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: httpd
     template:
       metadata:
         labels:
           app: httpd
           class: canary
       spec:
         containers:
         - image: 040519/store:video
           name: httpd
           ports:
           - containerPort: 8080
   ```

   Aplica el archivo:
   ```bash
   kubectl apply -f canary-v1.yaml
   ```

2. **Crear un servicio NodePort**  
   Crea un archivo llamado `canary-svc.yaml` con el siguiente contenido:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     labels:
       class: canary
     name: myapp-svc
   spec:
     type: NodePort
     ports:
     - port: 80
       protocol: TCP
       targetPort: 8080
     selector:
       class: canary
   ```

   Aplica el servicio:
   ```bash
   kubectl apply -f canary-svc.yaml
   ```

3. **Desplegar la versión `v2` (nginx)**  
   Crea un archivo llamado `canary-v2.yaml` con el siguiente contenido:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     labels:
       app: nginx
     name: myapp-v2
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: nginx
     template:
       metadata:
         labels:
           app: nginx
           class: canary
       spec:
         containers:
         - image: 040519/store:apparels
           name: nginx
           ports:
           - containerPort: 8080
   ```

   Aplica el archivo:
   ```bash
   kubectl apply -f canary-v2.yaml
   ```

---

### Blue-Green Deployment

1. **Crear el namespace `lab4`**  
   Si el namespace no existe, créalo:
   ```bash
   kubectl create ns lab4
   ```

2. **Desplegar el entorno "Blue"**  
   Crea un archivo llamado `blue-deploy.yaml` con el siguiente contenido:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     labels:
       app: blue-app
     name: blue-deploy
     namespace: lab4
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: blue-app
     template:
       metadata:
         labels:
           app: blue-app
       spec:
         containers:
         - image: nginx
           name: blue-container
           ports:
           - containerPort: 80
   ```

   Aplica el archivo:
   ```bash
   kubectl apply -f blue-deploy.yaml
   ```

3. **Exponer el despliegue "Blue" con un servicio NodePort**  
   ```bash
   kubectl expose deployment blue-deploy --name blue-green-svc --port 80 --type NodePort -n lab4
   ```

4. **Desplegar el entorno "Green"**  
   Crea un archivo llamado `green-deploy.yaml` con el siguiente contenido:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     labels:
       app: green-app
     name: green-deploy
     namespace: lab4
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: green-app
     template:
       metadata:
         labels:
           app: green-app
       spec:
         containers:
         - image: httpd
           name: green-container
           ports:
           - containerPort: 80
   ```

   Aplica el archivo:
   ```bash
   kubectl apply -f green-deploy.yaml
   ```

5. **Modificar el servicio para redirigir al entorno "Green"**  
   Edita el servicio `blue-green-svc` para cambiar la etiqueta del selector:
   ```bash
   kubectl edit svc blue-green-svc -n lab4
   ```
   Cambia el valor del selector de `blue-app` a `green-app`.

---
