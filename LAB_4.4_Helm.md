# Laboratorio N° 4.4 - Helm

## Objetivos
- Instalar Helm en el clúster.
- Desplegar aplicaciones utilizando Helm.

## Requerimientos
- Tener configurado un clúster de Kubernetes.

---

### Instalación de Helm

1. **Agregar la clave y repositorio de Helm**  
   Ejecuta los siguientes comandos para agregar la clave GPG y configurar el repositorio de Helm:
   ```bash
   curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
   sudo apt-get install apt-transport-https --yes
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
   ```

2. **Actualizar e instalar Helm**  
   Actualiza los repositorios e instala Helm:
   ```bash
   sudo apt-get update
   sudo apt-get install helm
   ```

3. **Validar la instalación**  
   Verifica que Helm esté instalado correctamente:
   ```bash
   helm -h
   ```

---

### Despliegue de Chart desde Repositorio Público

1. **Agregar un repositorio de charts**  
   Agrega el repositorio de `nginx`:
   ```bash
   helm repo add cloudnative https://charts.bitnami.com/bitnami
   ```

2. **Listar los repositorios agregados**  
   Comprueba que el repositorio se haya agregado:
   ```bash
   helm repo list
   ```

3. **Buscar un chart en el repositorio**  
   Busca `nginx` dentro del repositorio:
   ```bash
   helm search repo nginx
   ```

4. **Instalar un chart**  
   Instala `nginx` desde el repositorio con el nombre `mi-nginx`:
   ```bash
   helm install mi-nginx cloudnative/nginx
   ```

5. **Verificar el despliegue**  
   Revisa los recursos desplegados:
   ```bash
   helm list
   kubectl get pods
   kubectl get svc
   ```

6. **Validar el servicio**  
   Utiliza `curl` para validar el servicio:
   ```bash
   curl cluster-ip:80
   ```

7. **Desinstalar el chart**  
   Elimina el despliegue:
   ```bash
   helm uninstall mi-nginx
   helm list
   ```

---

### Creación de un Chart Local

1. **Crear un directorio para el chart**  
   Crea un directorio `labhelm` y navega a él:
   ```bash
   mkdir labhelm
   cd labhelm/
   ```

2. **Generar un chart nuevo**  
   Crea un chart llamado `cnachart`:
   ```bash
   helm create cnachart
   ```

3. **Examinar los archivos generados**  
   Revisa el archivo `values.yaml`:
   ```bash
   cd cnachart
   head -n 20 values.yaml
   ```

4. **Personalizar los valores del chart**  
   Modifica `values.yaml` para establecer los siguientes valores:
   ```yaml
   customport:
     port: 5000

   image:
     repository: 040500/app-python-2
     tag: "v2"

   service:
     type: ClusterIP
     port: 5000
   ```

5. **Actualizar el archivo `deployment.yaml`**  
   Edita el archivo para usar el puerto personalizado:
   ```yaml
   containerPort: {{ .Values.customport.port }}
   ```

6. **Desplegar el chart local**  
   Instala el chart en el clúster:
   ```bash
   helm install app-python .
   ```

7. **Verificar y probar el despliegue**  
   Revisa el pod y haz una petición al servicio:
   ```bash
   curl cluster-ip:5000
   ```

---
