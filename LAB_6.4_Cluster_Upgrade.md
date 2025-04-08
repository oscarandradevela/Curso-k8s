
# Laboratorio N° 6.4 - Clúster Upgrade

## Objetivos
- Actualizar el nodo controlplane.
- Actualizar los nodos worker.

## Requerimientos
- Tener configurado un clúster de Kubernetes.

---

### Verificar la Versión Actual del Clúster
Antes de iniciar con la actualización, es necesario verificar la versión actual del clúster. Utilice los siguientes comandos para obtener esta información:

```bash
kubectl get nodes
kubectl version
```

### Upgrade del Nodo Control Plane (nodo Master)
1. Verificar las posibles versiones a las que se puede actualizar:
   ```bash
   kubeadm upgrade plan
   ```

   La validación debe indicar si puede actualizarse a la versión `1.30.6`.

2. Verificar las versiones disponibles de `kubeadm`:
   ```bash
   apt-cache show kubeadm
   ```

3. Instalar la paquetería `kubeadm` versión `1.30.6-1.1`:
   ```bash
   apt-get install kubeadm=1.30.6-1.1
   ```

4. Verificar la nueva versión de `kubeadm`:
   ```bash
   kubeadm version
   ```

5. Actualizar el nodo master a la versión `1.30.6`:
   ```bash
   kubeadm upgrade apply v1.30.6
   ```

6. Actualizar `kubectl` y `kubelet` a la nueva versión:
   ```bash
   apt-get install kubectl=1.30.6-1.1 kubelet=1.30.6-1.1
   ```

7. Verificar la versión instalada:
   ```bash
   kubectl version
   ```

8. Reiniciar el servicio de `kubelet`. Es posible que necesite ejecutar `systemctl daemon-reload` antes de reiniciar:
   ```bash
   service kubelet restart
   service kubelet status
   ```

9. Verificar que los pods en el namespace `kube-system` estén en estado `running`:
   ```bash
   kubectl get pods -A
   ```

---

### Upgrade de Nodos Worker
1. Instalar la nueva versión de `kubeadm` en los nodos worker:
   ```bash
   apt-get install kubeadm=1.30.6-1.1
   ```

2. Actualizar el nodo a la versión `1.30.6`:
   ```bash
   kubeadm upgrade node
   ```

3. Instalar la nueva versión de `kubelet`:
   ```bash
   apt-get install kubelet=1.30.6-1.1
   ```

4. Reiniciar el servicio de `kubelet` en el nodo worker:
   ```bash
   service kubelet restart
   service kubelet status
   ```

**Nota:** Repita el mismo procedimiento en todos los nodos worker.

---

### Verificar la Actualización en el Nodo Master
En el nodo master, listar los nodos y verificar la versión instalada en cada uno:
```bash
kubectl get nodes
```
