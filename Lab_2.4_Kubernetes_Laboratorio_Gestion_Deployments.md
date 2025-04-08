
# Laboratorio: Creación y Gestión de Deployments en Kubernetes

## Objetivos
- Crear deployments.
- Desplegar una instancia de Wildfly.

## Requerimientos
- Tener configurado un clúster de Kubernetes.

---

## Creación de Deployments
Crear un archivo llamado `webserver.yaml` con el siguiente contenido para definir un Deployment llamado `webserver` que utilice la imagen `nginx:alpine`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: webserver
  name: webserver
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webserver
  template:
    metadata:
      labels:
        app: webserver
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
        ports:
        - containerPort: 80
```

**Explicación de Parámetros:**
- **apiVersion:** Versión de la API de Kubernetes utilizada.
- **kind:** Tipo de recurso (Deployment).
- **metadata:** Nombre del Deployment y etiquetas asociadas.
- **replicas:** Cantidad de pods para el Deployment.
- **selector:** Etiqueta para agrupar y gestionar los pods.
- **template:** Define las etiquetas de los pods y especifica el contenedor.
- **containers:** Lista de contenedores dentro del pod.
  - **image:** Imagen a utilizar.
  - **name:** Nombre del contenedor.
  - **containerPort:** Puerto de escucha del contenedor.

Crear el Deployment y listar los recursos:
```bash
kubectl apply -f webserver.yaml
kubectl get deploy
kubectl get pods
```

---

## Desplegando Wildfly en Kubernetes
Crear un Deployment para **Wildfly** usando la imagen `docker.io/dineshverma877/wildfly:latest` con credenciales `admin` y `Redhat`. El servicio se expondrá como NodePort.

Crear un archivo `wildfly.yaml` con el siguiente contenido:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: wildfly
  name: wildfly
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wildfly
  strategy: {}
  template:
    metadata:
      labels:
        app: wildfly
    spec:
      containers:
      - image: docker.io/dineshverma877/wildfly:latest
        name: wildfly
        env:
        - name: WILDFLY_USER
          value: admin
        - name: WILDFLY_PASSWORD
          value: Redhat
```

Aplicar el archivo YAML para crear el Deployment:
```bash
kubectl apply -f wildfly.yaml
kubectl get pods
```

### Exponer el Deployment Wildfly
Crear un servicio NodePort para acceder al servidor Wildfly en el puerto 8080 y la consola en el puerto 9990:
```bash
kubectl expose deployment wildfly --name wildfly-server --port 8080 --target-port 8080 --type NodePort
kubectl expose deployment wildfly --name wildfly-console --port 9990 --target-port 9990 --type NodePort
```

La aplicación Wildfly se ha desplegado correctamente y está accesible mediante NodePort.
