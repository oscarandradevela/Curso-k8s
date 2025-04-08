
# Laboratorio N° 4.8: Configmaps y Secrets

## Objetivos:
- Creación de Configmaps.
- Creación de Secrets.
- Despliegue de una aplicación.

## Requerimientos:
- Tener configurado un clúster de Kubernetes.

---

## Configmaps

### 1. Creación de un Pod con una variable de entorno:
Crea un archivo `webapp.yaml` con el siguiente contenido:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: webapp
  name: webapp
spec:
  containers:
  - image: 040519/cna-webapp
    name: webapp
    env:
    - name: APP_COLOR
      value: blue
```

Aplica el archivo:
```bash
kubectl apply -f webapp.yaml
```

Expón el Pod con un Service de tipo NodePort y valida la variable de entorno:
```bash
kubectl expose pod webapp --name=webapp --type=NodePort --port=8080
kubectl get pods webapp -o wide
kubectl get svc webapp
```

### 2. Creación de un Configmap:
```bash
kubectl create configmap webapp-cm --from-literal=APP_COLOR=darkblue --from-literal=APP_OTHER=disregard
kubectl get configmap
```

Modifica el archivo `webapp.yaml` para usar el Configmap:
```yaml
env:
- name: APP_COLOR
  valueFrom:
    configMapKeyRef:
      name: webapp-cm
      key: APP_COLOR
```

Aplica los cambios:
```bash
kubectl replace -f webapp.yaml --force
kubectl get pods -o wide
```

### 3. Despliegue de un Deployment con Configmap:
Crea un archivo `index.html`:
```html
<html>
  <head><title>Cloud Native Academy Labs</title></head>
  <body>
    <h1>¡Bienvenido al Laboratorio 13 de CNA!</h1>
    <p>Estamos aprendiendo a crear Configmaps.</p>
  </body>
</html>
```

Crea un Configmap con el archivo:
```bash
kubectl create configmap custome-index --from-file=index.html -n cloudnative
kubectl get configmap -n cloudnative
```

Crea el archivo `nginx.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
  namespace: cloudnative
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
      - name: nginx-content
        configMap:
          name: custome-index
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-content
          mountPath: /usr/share/nginx/html/
```

Despliega el archivo:
```bash
kubectl apply -f nginx.yaml
kubectl expose deployment nginx --name=nginx-svc --port=80 -n cloudnative
kubectl get svc -n cloudnative
```

---

## Secrets

### 1. Uso de Secrets como variable de entorno:
Crea un Secret:
```bash
kubectl create secret generic secreto-uno --from-literal=username=admin --from-literal=password=@passwd$ -n cloudnative
kubectl get secret -n cloudnative
kubectl describe secret secreto-uno -n cloudnative
kubectl get secret secreto-uno -n cloudnative -o yaml
```

Crea un Pod usando el Secret:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-env-secret
  namespace: cloudnative
spec:
  containers:
  - name: demo-app
    image: busybox
    command: ["sh", "-c", "echo $USERNAME && echo $PASSWORD && sleep 3600"]
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: secreto-uno
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: secreto-uno
          key: password
```

Despliega el Pod:
```bash
kubectl create -f app-sample.yaml
kubectl get pods -n cloudnative
kubectl exec -it pod-env-secret -n cloudnative -- sh
echo $USERNAME
echo $PASSWORD
```

### 2. Uso de Secrets como un volumen:
Crea los archivos:
```bash
echo -n '[user1]' > username.txt
echo -n '[cna1234]' > password.txt
kubectl create secret generic secreto-dos --from-file=username.txt --from-file=password.txt -n cloudnative
kubectl get secret secreto-dos -o yaml -n cloudnative
```

Crea un Pod con el Secret como volumen:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-secret
  namespace: cloudnative
spec:
  containers:
  - name: demo-app
    image: redis
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: secreto-dos
```

Despliega el Pod y valida el volumen:
```bash
kubectl create -f app-sample-dos.yaml
kubectl get pods -n cloudnative
kubectl exec -it pod-volume-secret -n cloudnative -- /bin/bash
ls -l /etc/secrets/
```
