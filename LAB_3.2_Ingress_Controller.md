# Laboratorio N° 3.2 - Configuración de Ingress Controller

## Objetivos
- Instalar y configurar un **Ingress Controller** en un clúster de Kubernetes.
- Publicar aplicaciones utilizando **Ingress** para gestionar el tráfico HTTP/S.

## Requisitos
- Un clúster de Kubernetes configurado y operativo.

---

### Despliegue del Ingress Controller

1. **Desplegar el Ingress Controller**  
   Ejecuta el siguiente comando para instalar el Ingress Controller:
   ```bash
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.3/deploy/static/provider/baremetal/deploy.yaml
   ```

2. **Verificar los recursos creados**  
   Comprueba que los recursos del Ingress Controller se hayan desplegado correctamente:
   ```bash
   kubectl get all -n ingress-nginx
   ```

---

### Configuración de Ingress para Aplicaciones

1. **Crear Deployments para las aplicaciones**  
   Crea los deployments `video` y `wear`:
   ```bash
   kubectl create deployment video --image=040519/store:video
   kubectl create deployment wear --image=040519/store:apparels
   ```

2. **Exponer los Deployments con Services**  
   Expón ambos deployments utilizando servicios en el puerto 8080:
   ```bash
   kubectl expose deployment video --port=8080
   kubectl expose deployment wear --port=8080
   ```

3. **Crear el recurso Ingress**  
   Configura un recurso Ingress para gestionar el tráfico HTTP hacia las aplicaciones y guárdalo como `ingress-store.yaml`:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     annotations:
       nginx.ingress.kubernetes.io/rewrite-target: /
     name: store
   spec:
     ingressClassName: nginx
     rules:
     - host: "cloudnative.apps.com"
       http:
         paths:
         - path: /wear
           pathType: Prefix
           backend:
             service:
               name: wear
               port:
                 number: 8080
         - path: /watch
           pathType: Prefix
           backend:
             service:
               name: video
               port:
                 number: 8080
   ```

   Aplica la configuración del archivo `ingress-store.yaml`:
   ```bash
   kubectl apply -f ingress-store.yaml
   ```

4. **Verificar el recurso Ingress creado**  
   Comprueba que el recurso Ingress se haya creado correctamente:
   ```bash
   kubectl get ingress
   ```

5. **Actualizar el archivo de hosts en tu equipo local**  
   Añade las siguientes líneas al archivo de hosts para acceder a las aplicaciones:
   ```
   192.168.18.121 cloudnative.apps.com
   192.168.18.122 cloudnative.apps.com
   ```

---