
# Guía de Instalación y Configuración de Kubernetes con Containerd y Configuraciones Previas

## Configuraciones Previas (Todos los Servidores)
1. **Clonar la Infraestructura**  
   Usar la herramienta de VMware para clonar la infraestructura del paso anterior.
2. **Asignar Hostname**  
   Asignar el nombre de hostname a cada servidor:
   - Nodo Master: `hostnamectl set-hostname master.ose.pe`
   - Nodo Worker1: `hostnamectl set-hostname worker1.ose.pe`
3. **Configuración DNS en /etc/hosts**
   ```bash
   cat <<EOF >> /etc/hosts
   192.168.18.X master.ose.pe master
   192.168.18.X worker1.ose.pe worker1
   EOF
   ```

## 1. Deshabilitar Firewall y SElinux
```bash
sudo ufw disable
```

## 2. Desactivar la Memoria Swap
```bash
swapoff -a
```
Editar el archivo `fstab` para desactivar swap permanentemente:
```bash
vim /etc/fstab
```
Comentar la línea de swap en `fstab`:
```plaintext
# /swap.img      none    swap    sw      0       0
```

## 3. Instalación de Containerd (en todos los servidores)
### Habilitar Módulos de Kernel
```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

### Configuración Persistente para IP Forwarding y Netfilter
```bash
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward=1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

### Agregar el Repositorio de Containerd e Instalar
```bash
apt-get install apt-transport-https ca-certificates curl gpg -y
mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update
apt-get install containerd.io -y
```

### Configurar Containerd
```bash
containerd config default | sudo tee /etc/containerd/config.toml
```
Modificar el archivo de configuración `/etc/containerd/config.toml`:
- Cambiar `snapshotter` a `"overlayfs"`.
- Asegurarse de que `SystemdCgroup` esté configurado en `true` en la sección `plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options`.

### Inicializar el Servicio de Containerd
```bash
systemctl enable --now containerd
systemctl status containerd
```

## 4. Instalación de Kubernetes (en todos los servidores)
### Agregar el Repositorio de Kubernetes
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
```

### Instalar Kubernetes y Fijar la Versión
```bash
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
systemctl enable --now kubelet
```

## 5. Configuración de Kubernetes en el Nodo Master
Ejecutar el comando para inicializar el clúster:
```bash
kubeadm init
```

### Configurar el Entorno del Administrador
```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Instalar el Plugin de Red (Calico)
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

## 6. Configuración de Nodos Worker en Kubernetes
Ejecutar el comando `kubeadm join` proporcionado al finalizar la configuración en el nodo master para unir los nodos worker.

```bash
# Ejemplo:
kubeadm join 192.168.18.X:6443 --token 4f9u0b.qdsbbz81yv7sukm5 --discovery-token-ca-cert-hash sha256:<hash>
```
### (Opcional) En caso de expiración del token

Si el token ha expirado y necesitas generar uno nuevo, vuelve a ejecutar el siguiente comando. Esto creará un token actualizado y te proporcionará el comando de unión necesario para conectar los nodos al clúster sin problemas.

```bash
kubeadm token create --print-join-command
```


## 7. Validación del Clúster de Kubernetes
En el nodo master, validar el estado de los nodos:
```bash
kubectl get nodes
```

## 8. Configuración de Métricas
Instalar el servidor de métricas para monitoreo:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
Editar el despliegue del servidor de métricas para habilitar `--kubelet-insecure-tls`:
```bash
kubectl edit deploy metrics-server -n kube-system
```
```yaml
- --kubelet-insecure-tls
```
