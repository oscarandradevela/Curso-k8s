
# Administración de Docker & Kubernetes  

## “Gestión de imágenes con Docker”

### Objetivos:
- Creación de imagen usando Dockerfile.
- Subir imagen a Registry de Docker Hub.

### Requerimientos:
- Un servidor Ubuntu 24.04.

**Nota:** Todos los comandos se estarán ejecutando con el usuario `Ubuntu`, por ello adicione la palabra `sudo` antes de cada comando.

---

## Creación de una imagen

1. En un directorio llamado `image`, cree los siguientes archivos:

```bash
mkdir image
cd image
```

### Archivo `package.json`:

```json
{
  "name": "docker_web_app",
  "version": "1.0.0",
  "description": "Node.js on Docker",
  "author": "lab cna <cna@example.com>",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.16.1"
  }
}
```

### Archivo `server.js`:

```javascript
'use strict';
const express = require('express');
// Constants
const PORT = 8080;
const HOST = '0.0.0.0';
// App
const app = express();
app.get('/', (req, res) => {
  res.send(`Bienvenido a Cloud Native Academy`);
});
app.listen(PORT, HOST);
console.log(`Running on http://${HOST}:${PORT}`);
```

### Archivo `Dockerfile`:

```dockerfile
FROM node:16
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 8080
CMD [ "node", "server.js" ]
```

2. Construya la nueva imagen con el siguiente comando:

```bash
sudo docker build -t cna-nodejs .
```

- `-t`: Especifica nombre y versión de la imagen.
- `.`: Indica la ruta donde se encuentra el Dockerfile.

3. Liste las imágenes existentes y verifique que tenga la nueva imagen creada:

```bash
sudo docker images
```

4. Cree un nuevo contenedor utilizando la imagen `cna-nodejs`:

```bash
sudo docker run -d -p 8080:8080 cna-nodejs
```

5. Desde su navegador, verifique que el contenedor haya iniciado correctamente.

---

## Creación de una cuenta en Docker Hub

1. Ingrese a la siguiente URL: [https://app.docker.com/signup](https://app.docker.com/signup).
2. Complete los datos solicitados (email, username y password). También puede iniciar sesión usando una cuenta de Gmail o GitHub.
3. Inicie sesión con la cuenta que acaba de crear.

---

## Publicación de imágenes

1. Suba a Docker Hub la imagen creada en la primera sección. Inicie sesión de Docker en el servidor (use su cuenta creada en el paso anterior):

```bash
sudo docker login -u user -p password
```

2. Agregue un tag a su imagen y envíela a Docker Hub:

```bash
sudo docker tag cna-nodejs <usuario_docker>/cna-nodejs:v1
sudo docker push <usuario_docker>/cna-nodejs:v1
```

3. En Docker Hub, verifique que se haya subido la imagen.
