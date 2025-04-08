# Laboratorio N° 3.3 - Configuración de Network Policies

## Objetivos
- Comprender el funcionamiento de las **Network Policies** en Kubernetes.
- Implementar políticas de red de tipo `Ingress` y `Egress` para controlar el tráfico.

## Requerimientos
- Tener un clúster de Kubernetes configurado y operativo.

---

### Creación de Network Policy - Ingress

1. **Crear Pods para probar restricciones de red**  
   Ejecuta los siguientes comandos para desplegar dos Pods en el namespace `cloudnative`:
   ```bash
   kubectl run nginx --image nginx --port 80 -n cloudnative
   kubectl run apache --image httpd --port 80 -n cloudnative
   ```

2. **Crear una política para denegar todo el tráfico hacia el pod `apache`**  
   Guarda la siguiente configuración como `deny-ingress.yaml`:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: deny-all
     namespace: cloudnative
   spec:
     podSelector:
       matchLabels:
         run: apache
     policyTypes:
     - Ingress
   ```

   Aplica la configuración:
   ```bash
   kubectl apply -f deny-ingress.yaml
   ```

3. **Permitir tráfico de entrada desde el pod `nginx`**  
   Guarda esta configuración como `allow-ingress.yaml`:
   ```yaml
   kind: NetworkPolicy
   apiVersion: networking.k8s.io/v1
   metadata:
     name: allow
     namespace: cloudnative
   spec:
     podSelector:
       matchLabels:
         run: apache
     policyTypes:
     - Ingress
     ingress:
     - from:
       - podSelector:
           matchLabels:
             run: nginx
   ```

   Aplica la configuración:
   ```bash
   kubectl apply -f allow-ingress.yaml
   ```

4. **Validar las políticas aplicadas**  
   Lista las políticas configuradas en el namespace:
   ```bash
   kubectl get networkpolicy -n cloudnative
   ```

---

### Configuración de Network Policy - Egress

1. **Bloquear el tráfico de salida desde el pod `nginx`**  
   Guarda esta configuración como `deny-egress.yaml`:
   ```yaml
   kind: NetworkPolicy
   apiVersion: networking.k8s.io/v1
   metadata:
     name: deny-all
     namespace: cloudnative
   spec:
     podSelector:
       matchLabels:
         run: nginx
     policyTypes:
     - Egress
   ```

   Aplica la configuración:
   ```bash
   kubectl apply -f deny-egress.yaml
   ```

2. **Permitir tráfico de salida hacia el pod `apache`**  
   Guarda esta configuración como `allow-egress.yaml`:
   ```yaml
   kind: NetworkPolicy
   apiVersion: networking.k8s.io/v1
   metadata:
     name: allow-egress
     namespace: cloudnative
   spec:
     podSelector:
       matchLabels:
         run: nginx
     policyTypes:
     - Egress
     egress:
     - to:
       - podSelector:
           matchLabels:
             run: apache
       ports:
       - protocol: TCP
         port: 80
   ```

   Aplica la configuración:
   ```bash
   kubectl apply -f allow-egress.yaml
   ```

3. **Validar la conexión entre Pods**  
   Verifica que el pod `nginx` puede comunicarse con el pod `apache` en el puerto 80 tras aplicar las políticas.

---
