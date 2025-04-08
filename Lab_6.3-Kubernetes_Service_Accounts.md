
# Laboratorio: Kubernetes Service Accounts

## Objetivos

- Entender el uso de Service Accounts en Kubernetes para controlar acceso y permisos.
- Crear y usar Service Accounts en Pods con configuraciones de seguridad específicas.
- Implementar autenticación y autorización mediante RBAC (Role-Based Access Control).

## Requisitos

- Acceso a un clúster de Kubernetes funcional.
- Herramienta `kubectl` configurada para interactuar con el clúster.

---

## Implementación y Verificación de Service Accounts

### Paso 1: Crear una Service Account
Crea una Service Account para usarla en Pods:
```bash
kubectl create sa mysa
kubectl get sa mysa
```

### Paso 2: Crear un Token para la Service Account
Genera un token asociado a la Service Account:
```bash
kubectl create token mysa
```

---

## Definir Permisos con Roles y RoleBindings

### Paso 1: Crear un Role
Define un Role que permita acceso a Pods:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups:
  - ''
  resources:
  - pods
  verbs:
  - get
  - watch
  - list
```
Aplica el Role:
```bash
kubectl apply -f role.yaml
```

### Paso 2: Asociar el Role con la Service Account
Usa un RoleBinding para asociar el Role con la Service Account:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: mysa
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
Aplica el RoleBinding:
```bash
kubectl apply -f rolebinding.yaml
```

---

## Usar la Service Account en un Pod

### Paso 1: Crear un Pod que Use la Service Account
Define un Pod que utilice la Service Account `mysa`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  serviceAccountName: mysa
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```
Aplica el Pod:
```bash
kubectl apply -f pod.yaml
```

### Paso 2: Verificar el Uso de la Service Account
Comprueba la configuración del Pod:
```bash
kubectl describe pod nginx
```
Asegúrate de que la Service Account sea `mysa`.

---

## Verificar Permisos de la Service Account

Usa los siguientes comandos para verificar los permisos asignados:
```bash
kubectl auth can-i get pods --as=system:serviceaccount:default:mysa
kubectl auth can-i watch pods --as=system:serviceaccount:default:mysa
kubectl auth can-i list pods --as=system:serviceaccount:default:mysa
kubectl auth can-i create pods --as=system:serviceaccount:default:mysa
kubectl auth can-i delete pods --as=system:serviceaccount:default:mysa
```

---

## Configurar un Pod para No Montar Automáticamente el Token

Si no necesitas que el token de la Service Account sea accesible desde el Pod, usa la configuración `automountServiceAccountToken: false`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  automountServiceAccountToken: false
  containers:
  - name: my-container
    image: nginx:latest
    ports:
    - containerPort: 80
```
Aplica el Pod:
```bash
kubectl apply -f pod.yaml
```

---

## Resumen

En este laboratorio:
- Creaste y configuraste una Service Account.
- Implementaste permisos con Roles y RoleBindings.
- Usaste una Service Account en un Pod para controlar su acceso.
- Aprendiste a verificar y restringir permisos mediante RBAC.

Este enfoque mejora la seguridad, el control de acceso y el aislamiento de aplicaciones en Kubernetes.
