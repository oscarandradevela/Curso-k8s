
# Administración de Docker & Kubernetes  

## “Límites de recursos en contenedores”

### Objetivos:
- Creación de imagen personalizada para pruebas de estrés.
- Pruebas de configuración de políticas de reinicio.

### Requerimientos:
- Servidor con Docker instalado.

---

## Límites por defecto

1. Inicialice un contenedor usando la imagen `httpd`:

```bash
sudo docker run -d httpd
```

2. Verifique la cantidad de recursos usados y los límites por defecto del contenedor de Apache:

```bash
sudo docker stats
```

**Nota:** El límite por defecto es toda la memoria disponible en el servidor host.

---

## Creación de imagen personalizada

1. Cree un nuevo directorio para la nueva imagen:

```bash
mkdir limit
cd limit
```

2. Cree el archivo `Dockerfile` con el siguiente contenido:

```dockerfile
FROM fedora:latest
RUN yum -y install stress procps-ng
ENV CPU_LOAD=1 MEM_LOAD=1 MEM_SIZE=256M
CMD stress --cpu $CPU_LOAD --vm $MEM_LOAD --vm-bytes $MEM_SIZE
```

3. Construya la nueva imagen con el nombre `stress`:

```bash
sudo docker build -t stress .
```

---

## Ejecución de contenedores con límites

1. Ejecute el contenedor `stress` con las variables de entorno necesarias:

```bash
sudo docker run -d -e CPU_LOAD=2 -e MEM_LOAD=1 -e MEM_SIZE=900M stress
sudo docker stats
```

2. Ejecute el mismo contenedor, pero saturado de memoria con límites:

```bash
sudo docker run -d --memory="200m" --cpus="0.5" --name=containerLimit --restart always -e CPU_LOAD=2 -e MEM_LOAD=1 -e MEM_SIZE=500M stress
```

   - `--memory`: Establece una cantidad límite de memoria para los contenedores.
   - `--cpus`: Establece una cantidad límite de CPU para correr el contenedor.

**Nota:** El contenedor creado estará en constante reinicio debido a los límites de recursos.

3. Ejecute un nuevo contenedor con recursos equivalentes a las variables de entorno:

```bash
sudo docker run -d --memory="700m" --cpus="1.5" --name=containerLimit-v2 --restart always -e CPU_LOAD=1 -e MEM_LOAD=1 -e MEM_SIZE=500M stress
```

**Resultado:** El contenedor `containerLimit-v2` estará ejecutándose sin errores.
