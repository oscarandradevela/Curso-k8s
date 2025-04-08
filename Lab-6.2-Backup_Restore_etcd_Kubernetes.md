
# Laboratorio: Backup y Restore de etcd en Kubernetes

## Objetivo
Configurar un backup y restore de etcd en un clúster de Kubernetes utilizando métodos imperativos y declarativos. Aprenderás a realizar una copia de seguridad manual y a restaurarla en el mismo clúster, además de automatizar el proceso de backup usando CronJobs de Kubernetes.

## Requisitos
- Acceso a un clúster de Kubernetes en ejecución.
- `kubectl` instalado y configurado.
- Acceso SSH al nodo de control plane del clúster.
- Volúmenes persistentes en el clúster para almacenar backups y datos de etcd.

---

## Parte 1: Método Imperativo

### Paso 1: Preparar el Entorno
1. **Verificar la Versión de etcd en el Clúster**:

   ```bash
   kubectl -n kube-system logs etcd-controlplane | grep -i 'etcd-version'
   kubectl -n kube-system describe pod etcd-controlplane | grep Image:
   ```


2. **Instalar `etcdctl` en el nodo donde se ejecuta etcd**:

   ```bash
   ETCD_VER=v3.5.15
   DOWNLOAD_URL=https://github.com/etcd-io/etcd/releases/download
   curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o etcd-${ETCD_VER}-linux-amd64.tar.gz
   tar xzvf etcd-${ETCD_VER}-linux-amd64.tar.gz
   sudo mv etcd-${ETCD_VER}-linux-amd64/etcdctl /usr/local/bin/
   sudo mv etcd-${ETCD_VER}-linux-amd64/etcdutl /usr/local/bin/
      sudo mv etcd-${ETCD_VER}-linux-amd64/etcd /usr/local/bin/
   etcdctl version
   ```



### Paso 2: Realizar un Backup de etcd
Crea el directorio backup.
```bash
mkdir /opt/backup
```

Ejecuta el siguiente comando directamente desde el nodo de control plane donde está corriendo etcd:

```bash
export ETCDCTL_API=3
etcdctl --endpoints=https://127.0.0.1:2379   --cacert=/etc/kubernetes/pki/etcd/ca.crt   --cert=/etc/kubernetes/pki/etcd/server.crt   --key=/etc/kubernetes/pki/etcd/server.key   snapshot save /opt/backup/etcd.db
```

Si la ejecución es exitosa, recibirás un mensaje como: *Snapshot saved at /opt/backup/etcd.db*.

### Verificar el Snapshot
Para verificar que el snapshot se haya creado correctamente, ejecuta:

```bash
ETCDCTL_API=3 etcdctl --write-out=table snapshot status /opt/backup/etcd.db
```

#### Ejemplo de salida:

|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
|----------|----------|------------|------------|
| b7147656 |    51465 |       1099 |     5.1 MB |

### Restauración de etcd usando el Snapshot
Ejecutar el Comando de Restauración: Para restaurar el snapshot, utiliza el siguiente comando, reemplazando `/opt/backup/etcd.db` con la ubicación de tu archivo de backup:

```bash
ETCDCTL_API=3 etcdctl snapshot restore /opt/backup/etcd.db
```

Si deseas especificar un directorio de datos diferente para la restauración, utiliza la opción `--data-dir`:

```bash
ETCDCTL_API=3 etcdctl --data-dir /opt/etcd snapshot restore /opt/backup/etcd.db
```

Después de restaurar el snapshot en la nueva ubicación, actualiza el archivo `/etc/kubernetes/manifests/etcd.yaml` para que apunte al nuevo directorio:

```yaml
volumes:
  - hostPath:
      path: /opt/etcd
      type: DirectoryOrCreate
    name: etcd-data
```

El pod de etcd se reiniciará automáticamente con la nueva configuración y debería mostrar los datos previos almacenados en el directorio especificado.

---

## Parte 2: Método Declarativo (Automatización con CronJobs)

### Paso 1: Automatizar el Backup usando un CronJob de Kubernetes
Crear un archivo `etcd-backup-cronjob.yaml` con el siguiente contenido para automatizar el backup diario a las 2 AM:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: etcd-backup
  namespace: kube-system
spec:
  schedule: "0 2 * * *"  # Corre diariamente a las 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: etcd-backup
            image: bitnami/etcd:3.5.0
            command:
            - /bin/sh
            - -c
            - |
              export ETCDCTL_API=3
              etcdctl --endpoints=https://127.0.0.1:2379               --cacert=/certs/ca.crt               --cert=/certs/etcd.crt               --key=/certs/etcd.key               snapshot save /backups/snapshot.db
            volumeMounts:
            - name: etcd-certs
              mountPath: /certs
              readOnly: true
            - name: etcd-backup
              mountPath: /backups
          restartPolicy: OnFailure
          volumes:
          - name: etcd-certs
            secret:
              secretName: etcd-certs
          - name: etcd-backup
            persistentVolumeClaim:
              claimName: etcd-backup-pvc
```

Aplicar el CronJob:

```bash
kubectl apply -f etcd-backup-cronjob.yaml
```

Verificar el CronJob:

```bash
kubectl get cronjob -n kube-system
kubectl get jobs -n kube-system
```

### Paso 2: Restaurar el Backup usando un Pod
Crear un archivo `etcd-restore-job.yaml` para restaurar el backup:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: etcd-restore
  namespace: kube-system
spec:
  template:
    spec:
      containers:
      - name: etcd-restore
        image: bitnami/etcd:3.5.0
        command:
        - /bin/sh
        - -c
        - |
          export ETCDCTL_API=3
          etcdctl snapshot restore /backups/snapshot.db --data-dir=/var/lib/etcd/new.etcd
        volumeMounts:
        - name: etcd-backup
          mountPath: /backups
        - name: etcd-data
          mountPath: /var/lib/etcd/new.etcd
      restartPolicy: OnFailure
      volumes:
      - name: etcd-backup
        persistentVolumeClaim:
          claimName: etcd-backup-pvc
      - name: etcd-data
        persistentVolumeClaim:
          claimName: etcd-data-pvc
```

Aplicar el Job de Restauración:

```bash
kubectl apply -f etcd-restore-job.yaml
```

Verificar el Estado del Job de Restauración:

```bash
kubectl get jobs -n kube-system
```
