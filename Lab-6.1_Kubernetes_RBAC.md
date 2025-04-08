
# Laboratorio: Configuración de Acceso a Kubernetes con Usuario

## Objetivo

Configurar un usuario llamado `operador1` en Kubernetes con acceso restringido a un solo namespace (`dev`). El usuario debe autenticar usando un certificado aprobado, y el acceso será a través del cliente `kubectl` en un servidor Ubuntu.

## Requerimientos

- Servidores proporcionados para el laboratorio (servidor de Kubernetes y un servidor cliente con Ubuntu).
- Cliente `kubectl` en el servidor de Kubernetes.
- Acceso SSH entre el servidor de Kubernetes y el servidor cliente Ubuntu.

---

## 1. Crear el Namespace Restringido

En el nodo maestro de Kubernetes, crea un namespace específico para el usuario `operador1`:

```bash
kubectl create namespace dev
```

> **Nota**: Los namespaces en Kubernetes permiten agrupar recursos lógicamente, ayudando a gestionar permisos de acceso específicos para cada conjunto de recursos.

---

## 2. Generar Clave Privada y CSR para el Usuario

1. **Genera la clave privada RSA** para `operador1`:

    ```bash
    openssl genrsa -out operador1.key 2048
    ```

2. **Crea una solicitud de firma de certificado (CSR)** usando la clave generada:

    ```bash
    openssl req -new -key operador1.key -out operador1.csr -subj "/CN=operador1/O=ose.pe"
    ```

> **Explicación**:
> - `/CN=operador1` se refiere al nombre común (Common Name) del certificado, que aquí estamos usando como el nombre del usuario.
> - `/O=ose.pe` representa la organización y es opcional, pero ayuda a identificar a qué grupo o compañía pertenece el usuario.

3. Ejecuta el siguiente comando en tu terminal para codificar el archivo operador1.csr.

    ```bash
    base64 < operador1.csr | tr -d '\n'
    ```

4. **Crear una CSR en Kubernetes**: Para gestionar el CSR en Kubernetes y aprobarlo posteriormente, crea un archivo `operador1-csr.yaml` con el siguiente contenido:

**Nota: Reemplaza el valor de request con el output del comando anterior**
```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: operador1-csr
spec:
  groups:
  - system:authenticated
  request: <reemplazar_cadena_base64_generada>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - digital signature
  - key encipherment
  - client auth

```
4. **Aplicar el CSR** en Kubernetes:

    ```bash
    kubectl apply -f operador1-csr.yaml
    ```

> **Nota**: Esta CSR está configurada para autenticar a `operador1` usando certificados en el clúster.

---

## 3. Aprobar el CSR en Kubernetes

1. **Aprobar el CSR** usando el siguiente comando:

    ```bash
    kubectl certificate approve operador1-csr
    ```

2. **Recuperar el certificado aprobado** y guardar la respuesta en `operador1.crt`:

    ```bash
    kubectl get csr operador1-csr -o jsonpath='{.status.certificate}' | base64 --decode > operador1.crt
    ```

> **Explicación**: Esta aprobación permite que Kubernetes confíe en el certificado generado para `operador1`. Es importante almacenar este archivo `.crt` ya que lo utilizaremos para configurar el acceso del usuario.

---

## 4. Configuración del Rol de Usuario en el Namespace

1. Crea un archivo de rol (`operador1-role.yaml`) con permisos específicos para el namespace `dev`:

    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      namespace: dev
      name: restricted-role
    rules:
    - apiGroups: ["", "extensions", "apps"]
      resources: ["deployments", "replicasets", "pods"]
      verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
    ```

2. Aplica la configuración:

    ```bash
    kubectl apply -f operador1-role.yaml
    ```

> **Nota**: Este rol le da acceso a `operador1` solo a los recursos específicos dentro del namespace `dev` (no en otros namespaces).

---

## 5. Creación de un RoleBinding para Acceso en el Namespace

1. Crea un archivo de RoleBinding (`operador1-role-binding.yaml`) para asociar el rol al usuario `operador1` en el namespace `dev`:

    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: operador1-role-binding
      namespace: dev
    subjects:
    - kind: User
      name: operador1
      apiGroup: ""
    roleRef:
      kind: Role
      name: restricted-role
      apiGroup: ""
    ```

2. Aplica el RoleBinding:

    ```bash
    kubectl apply -f operador1-role-binding.yaml
    ```

> **Explicación**: Este RoleBinding permite que `operador1` acceda solo al namespace `dev` y solo a los recursos permitidos por el rol `restricted-role`.

---

## 6. Creación del Usuario `operador1` en el Servidor Cliente (Ubuntu)

1. En el servidor cliente, crea el usuario `operador1`:

    ```bash
    sudo adduser operador1
    ```

2. **Transferencia del Certificado y Clave**: Copia `operador1.crt` y `operador1.key` al servidor cliente Ubuntu:

    ```bash
    scp operador1.crt root@<IP-Cliente>:/home/operador1/
    scp operador1.key root@<IP-Cliente>:/home/operador1/
    ```

> **Nota**: Asegúrate de que los archivos se encuentren en `/home/operador1/` y con permisos adecuados para evitar problemas de seguridad.

---

## 7. Instalación de `kubectl` en el Servidor Cliente Ubuntu

1. **Configura el repositorio de Kubernetes** en el cliente Ubuntu:

    ```bash
    sudo apt update
    sudo apt install -y apt-transport-https ca-certificates curl
    sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
    echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    ```

2. **Instala `kubectl`**:

    ```bash
    sudo apt update
    sudo apt install -y kubectl
    ```

3. **Mueve los archivos `key` y `crt`** a la carpeta `.kube` del usuario `operador1`:

    ```bash
    sudo su - operador1
    mkdir -p ~/.kube
    mv /home/operador1/operador1.key ~/.kube/
    mv /home/operador1/operador1.crt ~/.kube/
    ```

---

## 8. Configuración del Archivo kubeconfig en el Servidor Cliente

1. Edita el archivo `~/.kube/config` para incluir los detalles del clúster y el usuario `operador1`:

    ```yaml
    apiVersion: v1
    clusters:
    - cluster:
        insecure-skip-tls-verify: true
        server: https://<CLUSTER_SERVER_IP>:6443
      name: kubernetes
    contexts:
    - context:
        cluster: kubernetes
        namespace: dev
        user: operador1
      name: operador1
    current-context: operador1
    kind: Config
    preferences: {}
    users:
    - name: operador1
      user:
        client-certificate: /home/operador1/.kube/operador1.crt
        client-key: /home/operador1/.kube/operador1.key
    ```

2. **Establecer el contexto del usuario**:

    ```bash
    kubectl config set-context operador1
    ```

> **Nota**: Asegúrate de que los caminos a los certificados (`client-certificate` y `client-key`) coincidan con la ubicación de los archivos `.crt` y `.key` del usuario.

---

## 9. Verificar Acceso y Crear un Deployment

1. En el servidor cliente, verifica el acceso de `operador1` creando un deployment de prueba en el namespace `dev`:

    ```bash
    kubectl create deployment nginx --image=nginx -n dev
    ```

2. Comprueba el estado de los pods en el namespace `dev`:

    ```bash
    kubectl get pods -n dev -w
    ```

> **Nota**: Si el usuario `operador1` tiene permisos configurados correctamente, debería poder crear y gestionar recursos dentro del namespace `dev` sin acceso a otros namespaces.