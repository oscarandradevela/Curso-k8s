# Laboratorio N° 4.3 - Creando Pods Multi-Contenedor

## Objetivos
- Crear un pod con varios contenedores.
- Crear contenedores tipo sidecar.

## Requerimientos
- Tener configurado un clúster de Kubernetes.

---

### Multi-Container Pods

1. **Crear un Pod con varios contenedores**  
   Crea un archivo llamado `ms-apps.yaml` con el siguiente contenido:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     labels:
       run: ms-apps
     name: ms-apps
   spec:
     containers:
     - image: nginx:alpine
       name: web
     - image: redis
       name: database
   ```

2. **Aplicar la configuración y verificar los logs**  
   Aplica el archivo YAML y verifica los logs de los contenedores dentro del Pod:
   ```bash
   kubectl apply -f ms-apps.yaml
   kubectl logs ms-apps database
   kubectl logs ms-apps web
   ```

3. **Ingresar a un contenedor específico dentro del Pod**  
   Para ejecutar comandos en un contenedor específico dentro del Pod:
   ```bash
   kubectl exec ms-apps -c web -it -- sh
   ```

---

### Sidecar Containers

1. **Crear un Pod con contenedor tipo Sidecar**  
   Crea un archivo llamado `initcont.yaml` con el siguiente contenido. Este Pod tiene un contenedor principal y un contenedor de inicialización (initContainer) que espera 20 segundos antes de iniciar el contenedor principal:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: cna
     namespace: default
   spec:
     containers:
     - command:
       - sh
       - -c
       - echo The app is running! && sleep 3600
       image: busybox:1.28
       name: cna-container
     initContainers:
     - image: busybox
       name: cna-initcontainer
       command:
       - "sleep"
       - "20"
   ```

2. **Aplicar la configuración y monitorear el despliegue**  
   Aplica el archivo YAML y observa los eventos del Pod:
   ```bash
   kubectl apply -f initcont.yaml
   kubectl get pods -w
   ```

   El flag `-w` permite monitorear los cambios en tiempo real.

---