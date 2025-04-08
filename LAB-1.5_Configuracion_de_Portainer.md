
# Administración de Docker & Kubernetes  

## “Configuración de Portainer”

### Objetivos:
- Desplegar y configurar Portainer.
- Crear nuevos contenedores desde Portainer.

### Requerimientos:
- Servidor con Docker instalado.

### Enlaces de referencia:
- [Documentación oficial de Portainer](https://docs.portainer.io/start/install-ce/server/docker/linux)
- [Imagen oficial de Portainer en Docker Hub](https://hub.docker.com/r/portainer/portainer-ce)

---

## Despliegue de Portainer

1. Descargamos la imagen de Portainer:

```bash
sudo docker pull portainer/portainer-ce
```

Al no especificar un tag, se descargará la última versión disponible.

2. Creamos un volumen para almacenar la información generada en Portainer. En este ejemplo, el volumen se llama `portainer_data`. Use el comando `volume ls` para listar los volúmenes creados:

```bash
sudo docker volume create portainer_data
sudo docker volume ls
```

3. Creamos un contenedor de Portainer y especificamos el volumen creado:

```bash
sudo docker run -d -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
sudo docker ps
```

4. Accedemos desde el navegador utilizando la IP del servidor con el puerto 9443. Ejemplo de URL:

```
https://192.168.18.104:9443
```

---

## Configuración del Portal de Portainer

1. Cree un nuevo usuario al acceder por primera vez. Complete los cuadros solicitados y seleccione "Create User".

2. Si se solicita reiniciar el contenedor de Portainer después de crear el usuario, ejecute:

```bash
sudo docker restart portainer
```

3. Acceda nuevamente a la URL de Portainer y realice el login con las credenciales creadas.

4. Seleccione "Get Started". El entorno por defecto mostrará un tipo `local`.

   **Nota:** Los valores en la sección de `local` pueden variar según su entorno.

5. Seleccione el entorno `local`. Automáticamente el estado pasará a `connected` y mostrará el dashboard de gestión para ese ambiente.

---

## Crear contenedores desde Portainer

1. En el dashboard, seleccione `Containers` y luego `Add Container`.

   **Nota:** La cantidad de contenedores mostrados puede variar según su entorno.

2. Ingrese los datos para el contenedor:
   - Nombre del contenedor.
   - Imagen y tag correspondiente.

   **Nota:** Este es un ejemplo básico. Puede personalizar configuraciones avanzadas según sus necesidades.

3. Valide que el contenedor esté en ejecución tanto por la interfaz web como por la CLI:

```bash
sudo docker ps
```
