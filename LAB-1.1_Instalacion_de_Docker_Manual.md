
# Administración de Docker & Kubernetes  

## “Instalación Docker” 

### Objetivos: 
- Instalar Docker Engine en la última versión.
- Manejo de comandos básicos para administración de imágenes. 

### Requerimientos: 
- Un servidor Ubuntu 24.04. 

**Nota:** Todos los comandos se estarán ejecutando con el usuario `Ubuntu`, por ello adicione `sudo` antes de cada comando.

---

## Instalando Docker 

### Paso 1: Configure el repositorio de Docker ejecutando los siguientes comandos. 

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

### Paso 2: Instale la última versión de Docker y verifique que el servicio esté activo. 

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
systemctl status docker
docker -v
```

### Paso 3: Deshabilite el firewall (opcional para este laboratorio).

```bash
sudo ufw disable
```

---

## Comandos básicos de Docker:

### 1. Descargar una imagen:

Para descargar una imagen, ejecute el siguiente comando:

```bash
sudo docker pull nginx
```

También puede descargar una imagen con una versión o tag específico:

```bash
sudo docker pull nginx:stable
```

### 2. Listar imágenes:

Para listar las imágenes descargadas en nuestro servidor de manera local:

```bash
sudo docker images
```

**Nota:** Es recomendable utilizar las imágenes `alpine`, ya que son imágenes mucho más pequeñas.

### 3. Eliminar una imagen:

Para eliminar una imagen, utilice el siguiente comando:

```bash
sudo docker rmi nginx:latest
```

### 4. Crear un contenedor:

Utilizando la imagen de `nginx:alpine`, cree su primer contenedor:

```bash
sudo docker run -d -p 80:80 nginx:alpine
```

- `run`: Indica a Docker que cree un nuevo contenedor.
- `-d`: Ejecuta el contenedor en segundo plano.
- `-p`: Publica puertos entre el contenedor y el host.
- `nginx:alpine`: Imagen de Docker que se utilizará para crear el contenedor.

### 5. Verificar contenedores:

Para verificar que el contenedor haya iniciado correctamente:

```bash
sudo docker ps
```

Si desea listar todos los contenedores, tanto detenidos como en ejecución, agregue el parámetro `-a`:

```bash
sudo docker ps -a
```
