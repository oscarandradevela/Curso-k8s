# Simulación del Examen CKA
---

## Pregunta 1: Cluster Architecture, Installation & Configuration
**Configura un ClusterRole y un RoleBinding:**
1. Crea un `ClusterRole` llamado `read-nodes-role` que permita ver información de nodos.
2. Crea un `RoleBinding` llamado `read-nodes-binding` en el namespace `default` que:
   - Asigne el ClusterRole `read-nodes-role` al usuario `dev-user`.

---

## Pregunta 2: Workloads & Scheduling 
**Configura un DaemonSet llamado `log-collector`:**
1. Despliega un DaemonSet llamado `log-collector` que:
   - Use la imagen `fluentd:latest`.
   - Se ejecute en todos los nodos, incluyendo los del plano de control.
   - Monte el directorio `/var/log` del host en `/logs` dentro del contenedor.

---

## Pregunta 3: Workloads & Scheduling 
**Configura nodos y pods con taints y tolerancias:**
1. Aplica un taint al nodo `worker-node-1` con:
   - Clave: `dedicated`.
   - Valor: `database`.
   - Efecto: `NoSchedule`.
2. Crea un Deployment llamado `db-deployment` en el namespace `default` que:
   - Use la imagen `mysql:5.7`.
   - Configure una tolerancia para el taint `dedicated=database:NoSchedule`.
   - Escale a 3 réplicas.

---

## Pregunta 4: Workloads & Scheduling 
**Crea un pod con múltiples contenedores llamado `multi-container-pod`:**
1. Configura el pod `multi-container-pod` que:
   - Contenga un contenedor llamado `web-container` usando la imagen `nginx` que exponga el puerto 80.
   - Contenga un segundo contenedor llamado `health-checker` usando la imagen `busybox`, que ejecute un comando que verifique cada 10 segundos si el puerto 80 está disponible:
     ```
     wget -q --spider http://localhost:80 && echo "OK"
     ```
2. Usa un volumen de tipo `emptyDir` llamado `shared-volume` para compartir datos entre los contenedores.

---

## Pregunta 5: Services & Networking 
**Configura un servicio ClusterIP llamado `backend-service`:**
1. Crea un Deployment llamado `backend-deployment` en el namespace `default` que:
   - Use la imagen `httpd:2.4`.
   - Tenga 2 réplicas.
2. Crea un servicio de tipo `ClusterIP` llamado `backend-service` que:
   - Apunte al Deployment `backend-deployment`.
   - Exponga el servicio en el puerto 80.

---

## Pregunta 6: Services & Networking 
**Configura un servicio LoadBalancer llamado `web-app-service`:**
1. Crea un Deployment llamado `web-app-deployment` en el namespace `default` que:
   - Use la imagen `nginx`.
   - Tenga 2 réplicas.
2. Crea un servicio de tipo `LoadBalancer` llamado `web-app-service` que:
   - Exponga el Deployment `web-app-deployment` en el puerto 8080.

---

## Pregunta 7: Services & Networking 
**Configura una NetworkPolicy llamada `allow-frontend`:**
1. Crea un pod llamado `backend-pod` en el namespace `default` usando la imagen `nginx`.
2. Crea una NetworkPolicy llamada `allow-frontend` en el namespace `default` que:
   - Permita tráfico entrante al pod `backend-pod` solo desde pods etiquetados con `app=frontend`.
   - Bloquee todo el tráfico entrante desde otros pods.

---

## Pregunta 8: Storage 
**Usa Dynamic Provisioning con StorageClass llamada `fast-storage`:**
1. Crea una `StorageClass` llamada `fast-storage` que soporte provisión dinámica.
2. Crea un PersistentVolumeClaim llamado `dynamic-pvc` que:
   - Use la StorageClass `fast-storage`.
   - Solicite 1Gi de almacenamiento.
3. Crea un pod llamado `storage-pod` en el namespace `default` que:
   - Monte el PVC `dynamic-pvc` en `/data`.
   - Use la imagen `nginx`.

---

## Pregunta 9: Storage 
**Configura un pod llamado `web-logs` con subPaths:**
1. Crea un PersistentVolume llamado `pv-logs` y un PersistentVolumeClaim llamado `pvc-logs`.
2. Crea un pod llamado `web-logs` en el namespace `default` que:
   - Monte el PVC `pvc-logs` en `/mnt`.
   - Configure dos subpaths:
     - `/mnt/html` en `/usr/share/nginx/html`.
     - `/mnt/logs` en `/var/log/nginx`.
   - Use la imagen `nginx`.

---

## Pregunta 10: Troubleshooting 
**Repara un pod llamado `broken-pod`:**
1. Identifica por qué el pod `broken-pod` en el namespace `default` está en estado `CrashLoopBackOff` usando:
   - `kubectl describe pod broken-pod`.
   - `kubectl logs broken-pod`.
2. Corrige los errores para que el pod funcione correctamente.

---

## Pregunta 11: Troubleshooting 
**Corrige problemas con el servicio `api-service`:**
1. Un servicio llamado `api-service` en el namespace `default` no está resolviendo correctamente.
2. Inspecciona y corrige la configuración del servicio para que funcione.

---

## Pregunta 12: Troubleshooting 
**Repara la conectividad entre nodos en el clúster:**
1. El tráfico entre nodos no está funcionando.
2. Inspecciona la configuración del complemento de red aplicado al clúster.
3. Realiza las correcciones necesarias para restaurar la conectividad.

---

## Pregunta 13: Backup y Restauración de etcd
**Configura un respaldo y restauración de etcd:**
1. Realiza un backup de etcd guardando el archivo en `/var/lib/etcd-backup/snapshot.db`.
2. Simula una falla eliminando los datos de un pod.
3. Restaura el backup para recuperar el estado del clúster.

---

## Pregunta 14: Troubleshooting 
**Diagnostica y corrige problemas en el nodo `worker-node`:**
1. El nodo `worker-node` está en estado `NotReady`.
2. Diagnostica problemas relacionados con:
   - Kubelet.
   - Recursos insuficientes.
   - Red.
3. Corrige los problemas y valida que el nodo esté listo.
