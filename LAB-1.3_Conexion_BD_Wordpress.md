
# Administración de Docker & Kubernetes  

## “Conexión de base de datos - Wordpress”

### Objetivos:
- Comprender conexión mediante links y network de Docker.
- Utilizar variables de entorno en contenedores.

### Requerimientos:
- Servidor con Docker instalado.

---

## Despliegue de base de datos - Links

1. Inicie un contenedor de base de datos utilizando la imagen de MariaDB. Use las siguientes variables de entorno:

| Variable            | Descripción                               |
|---------------------|-------------------------------------------|
| `MYSQL_ROOT_PASSWORD` | Contraseña para el super usuario root    |
| `MYSQL_DATABASE`      | Nombre de la base de datos a crear       |
| `MYSQL_USER`          | Nombre de usuario a crear               |
| `MYSQL_PASSWORD`      | Contraseña para el usuario anterior      |

```bash
sudo docker run -d --name mariadb -e MYSQL_ROOT_PASSWORD=wordpress -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=wordpress mariadb
sudo docker ps
```

---

## Despliegue de Wordpress - Links

1. Despliegue un contenedor de Wordpress estableciendo una conexión con la base de datos mediante `link`:

```bash
sudo docker run -d --name wordpress --link mariadb:mysql -p 8085:80 wordpress
sudo docker ps
```

**Nota:** El uso de `links` entre contenedores está entrando en desuso. Se recomienda utilizar Networks para nuevos despliegues.

2. Configure Wordpress accediendo desde el navegador a la IP de su servidor Docker con el puerto 8085.

3. Complete la configuración de la base de datos con los siguientes valores:
   - Nombre de la base de datos: `wordpress`
   - Nombre de usuario: `wordpress`
   - Contraseña: `wordpress`
   - Host de la base de datos: `mysql`
   - Prefijo de la tabla: `wp_` (dejar el valor por defecto).

4. Configure los accesos para el administrador de Wordpress:
   - Título del sitio: `Cloud Native Academy`
   - Nombre de usuario: `admin`
   - Contraseña: `admin2024#$`
   - Correo electrónico: `admin@gmail.com`

5. Inicie sesión con las credenciales del usuario administrador y acceda al dashboard de Wordpress.

---

## Creación de Network personalizada

1. Cree una nueva red dedicada exclusivamente para los contenedores de base de datos y Wordpress:

```bash
sudo docker network create --driver bridge --subnet 174.18.0.0/16 redwordpress
sudo docker network ls
```

2. Verifique en detalle la red creada:

```bash
sudo docker network inspect redwordpress
```

---

## Despliegue de Base de datos y Wordpress - Network

1. Despliegue la base de datos en la red creada:

```bash
sudo docker run -d --name mariadbhost -e MYSQL_ROOT_PASSWORD=wordpress -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=wordpress --network=redwordpress mariadb
sudo docker ps
```

2. Despliegue Wordpress en la red `redwordpress`:

```bash
sudo docker run -d --name wordpress -p 8090:80 --network=redwordpress wordpress
```

3. Acceda desde el navegador a la IP del servidor con el puerto 8090 y configure Wordpress siguiendo los pasos anteriores.

**Nota:** En este caso, el Host de la base de datos será `mariadbhost`, ya que al usar Networks se puede referenciar directamente el nombre del contenedor de la base de datos.
