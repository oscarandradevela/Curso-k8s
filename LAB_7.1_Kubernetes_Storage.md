
# Laboratorio N° 7.1 - Kubernetes Storage

## Objetivos
- Crear volúmenes de tipo EmptyDir.
- Crear volúmenes persistentes (PVC).

## Requerimientos
- Tener configurado un clúster de Kubernetes.

---

### Creación de Volumen EmptyDir
1. Crear un directorio para almacenar los archivos YAML:
   ```bash
   mkdir storage
   cd storage
   ```

2. Crear el archivo `empty_dir.yaml` con el siguiente contenido:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-database
     labels:
       app: db
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: db
     template:
       metadata:
         labels:
           app: db
       spec:
         containers:
         - name: db
           image: mysql
           imagePullPolicy: IfNotPresent
           env:
           - name: MYSQL_ROOT_PASSWORD
             value: cna2024$
           volumeMounts:
           - mountPath: /var/lib/mysql
             name: test-volume
         volumes:
         - name: test-volume
           emptyDir: {}
   ```

3. Crear el recurso:
   ```bash
   kubectl create -f empty_dir.yaml
   kubectl get pods
   ```

4. Ingresar al pod y crear una instancia de base de datos:
   ```bash
   kubectl exec -it my-database-xxxxx -- bash
   mysql -u root -p
   create database cna;
   show databases;
   ```

   **Nota:** Los volúmenes `emptyDir` no guardan datos de forma permanente.

5. Verificación de no persistencia de datos al eliminar el pod y crear uno nuevo:
   ```bash
   kubectl delete pod my-database-xxxxx
   kubectl get pods
   kubectl exec -it my-database-aaaa -- bash
   ```

---

### Creación de Persistent Volume (PV) y Persistent Volume Claim (PVC)
1. Crear un archivo `pv.yaml` para el PV:
   ```yaml
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: db-pv
     labels:
       type: local
   spec:
     storageClassName: manual
     capacity:
       storage: 1Gi
     accessModes:
       - ReadWriteOnce
     hostPath:
       path: "/mnt/data"
   ```

2. Crear el PV:
   ```bash
   kubectl create -f pv.yaml
   kubectl get persistentvolume
   ```

3. Crear el PVC `pvc.yaml`:
   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: db-pvc
   spec:
     storageClassName: manual
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 1Gi
   ```

4. Crear el PVC:
   ```bash
   kubectl create -f pvc.yaml
   kubectl get pvc
   ```

---

### Creación de una Aplicación que Utilice el PVC
1. Crear un deployment de base de datos `app.yaml`:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: db-deploy
     labels:
       app: database
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: database
     template:
       metadata:
         labels:
           app: database
       spec:
         volumes:
         - name: volumen-persistente
           persistentVolumeClaim:
             claimName: db-pvc
         containers:
         - name: database
           image: mysql
           imagePullPolicy: IfNotPresent
           env:
           - name: MYSQL_ROOT_PASSWORD
             value: redhat
           volumeMounts:
           - mountPath: /var/lib/mysql
             name: volumen-persistente
   ```

2. Crear la aplicación:
   ```bash
   kubectl create -f app.yaml
   kubectl get pods
   ```

3. Ingresar al pod y verificar la persistencia de datos:
   ```bash
   kubectl exec -it db-deploy-xxxx -- bash
   mysql -u root -p
   show databases;
   ```

4. Validar la persistencia de datos al eliminar y recrear el pod.

---
