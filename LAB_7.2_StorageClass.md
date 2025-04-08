
# Laboratorio N° 7.2 - Configurando StorageClass

## Objetivos
- Configurar un StorageClass - NFS.
- Crear volúmenes persistentes.

## Requerimientos
- Tener configurado un clúster de Kubernetes.
- Tener un servidor NFS para usar como almacenamiento.

---

### Configuración de StorageClass
1. Crear un directorio para la configuración del StorageClass:
   ```bash
   mkdir storage
   cd storage
   ```

2. Crear el archivo `sa-clusterrole.yaml` con la configuración de roles y cluster roles para el ServiceAccount encargado de aprovisionar recursos NFS:
   ```yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: nfs-client-provisioner
     namespace: default
   ---
   kind: ClusterRole
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: nfs-client-provisioner-runner
   rules:
     - apiGroups: [""]
       resources: ["persistentvolumes"]
       verbs: ["get", "list", "watch", "create", "delete"]
     - apiGroups: [""]
       resources: ["persistentvolumeclaims"]
       verbs: ["get", "list", "watch", "update"]
     - apiGroups: ["storage.k8s.io"]
       resources: ["storageclasses"]
       verbs: ["get", "list", "watch"]
     - apiGroups: [""]
       resources: ["events"]
       verbs: ["create", "update", "patch"]
   ---
   kind: ClusterRoleBinding
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: run-nfs-client-provisioner
   subjects:
     - kind: ServiceAccount
       name: nfs-client-provisioner
       namespace: default
   roleRef:
     kind: ClusterRole
     name: nfs-client-provisioner-runner
     apiGroup: rbac.authorization.k8s.io
   ---
   kind: Role
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: nfs-client-provisioner-locking
     namespace: default
   rules:
     - apiGroups: [""]
       resources: ["endpoints"]
       verbs: ["get", "list", "watch", "create", "update", "patch"]
   ---
   kind: RoleBinding
   apiVersion: rbac.authorization.k8s.io/v1
   metadata:
     name: nfs-client-provisioner-locking
     namespace: default
   subjects:
     - kind: ServiceAccount
       name: nfs-client-provisioner
       namespace: default
   roleRef:
     kind: Role
     name: nfs-client-provisioner-locking
     apiGroup: rbac.authorization.k8s.io
   ```

3. Crear el recurso:
   ```bash
   kubectl create -f sa-clusterrole.yaml
   ```

4. Crear el archivo `storageclass.yaml` con el siguiente contenido para definir el StorageClass:
   ```yaml
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: managed-nfs-storage
   provisioner: nfs-provisioner
   parameters:
     archiveOnDelete: "false"
   ```

5. Crear y listar el StorageClass:
   ```bash
   kubectl create -f storageclass.yaml
   kubectl get storageclass
   ```

---

### Deployment para NFS Provisioner
1. Crear un deployment para el NFS provisioner. El path debe reemplazarse según el asignado a cada estudiante; para este ejemplo, usaremos `/storage-cna/lab`.

   ```yaml
   kind: Deployment
   apiVersion: apps/v1
   metadata:
     labels:
       app: nfs-client-provisioner
     name: nfs-client-provisioner
     namespace: default
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: nfs-client-provisioner
     strategy:
       type: Recreate
     template:
       metadata:
         labels:
           app: nfs-client-provisioner
       spec:
         serviceAccountName: nfs-client-provisioner
         containers:
           - name: nfs-client-provisioner
             image: k8s.gcr.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2
             volumeMounts:
               - name: nfs-client-cna
                 mountPath: /persistentvolumes
             env:
               - name: PROVISIONER_NAME
                 value: nfs-provisioner
               - name: NFS_SERVER
                 value: 192.168.18.57
               - name: NFS_PATH
                 value: /storage-cna/lab
         volumes:
           - name: nfs-client-cna
             nfs:
               server: 192.168.18.57
               path: /storage-cna/lab
   ```

2. Crear y listar el deployment:
   ```bash
   kubectl create -f deployment.yaml
   kubectl get pods
   ```

3. Instalar `nfs-common` en los nodos worker si el pod no inicia:
   ```bash
   apt-get install nfs-common -y
   ```

---

### Creación de Persistent Volume Claim (PVC)
1. Crear un PVC `pvc.yaml`:
   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: cna-test-pvc
   spec:
     storageClassName: managed-nfs-storage
     accessModes:
       - ReadWriteMany
     resources:
       requests:
         storage: 10Mi
   ```

2. Crear y listar el PVC:
   ```bash
   kubectl create -f pvc.yaml
   kubectl get pvc
   ```

3. Verificar el PV creado automáticamente:
   ```bash
   kubectl get pv
   ```

4. Validar en el servidor NFS que se haya creado un nuevo directorio en `/storage-cna/lab`.

---
