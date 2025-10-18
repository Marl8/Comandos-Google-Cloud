## Cloud Run 

Desplegar una aplicación sin servidores en un contenedor.

Pasos: 

- Crear la aplicación en el leguaje que elijas.
- Desplegar la aplicación en un contenedor.
- Subir el contenedor a Google Cloud y despelgarlo en Cloud Run.


### Habilita la API de Cloud Run y configura tu entorno de shell

1. En Cloud Shell, habilita la API de Cloud Run:

```shell

gcloud services enable run.googleapis.com

```

2. Establece la región de procesamiento:

```shell
gcloud config set compute/region "REGION"

```

3. Crea una variable de entorno LOCATION:

```shell

LOCATION="Region"

```

### Escribe la aplicación

1. En Cloud Shell, crea un nuevo directorio llamado helloworld y, luego, ve a él en la vista:

```shell

mkdir helloworld && cd helloworld

```

2. Crea un archivo package.json y agrégale el siguiente contenido:

```shell

nano package.json

```

```json
{
    "name": "helloworld",
    "description": "Simple hello world sample in Node",
    "version": "1.0.0",
    "main": "index.js",
    "scripts": {
        "start": "node index.js"
    },
    "author": "Google LLC",
    "license": "Apache-2.0",
    "dependencies": {
        "express": "^4.17.1"
    }
}

```

3. A continuación, en el mismo directorio, crea un archivo index.js y cópiale las siguientes líneas:

```shell

nano index.js

```

```javascript

const express = require('express');
const app = express();
const port = process.env.PORT || 8080;

app.get('/', (req, res) => {
  const name = process.env.NAME || 'World';
  res.send(`Hello ${name}!`);
});

app.listen(port, () => {
  console.log(`helloworld: listening on port ${port}`);
});

```


###  Aloja tu app en contenedores y súbela a Artifact Registry

1. Para alojar la app de ejemplo en contenedores, crea un nuevo archivo denominado Dockerfile en el mismo directorio que los archivos fuente y agrega el siguiente contenido:

```shell

nano Dockerfile

```

```shell

# Use the official lightweight Node.js 12 image.
# https://hub.docker.com/_/node
FROM node:12-slim

# Create and change to the app directory.
WORKDIR /usr/src/app

# Copy application dependency manifests to the container image.
# A wildcard is used to ensure copying both package.json AND package-lock.json (when available).
# Copying this first prevents re-running npm install on every code change.
COPY package*.json ./

# Install production dependencies.
# If you add a package-lock.json, speed your build by switching to 'npm ci'.
# RUN npm ci --only=production
RUN npm install --only=production

# Copy local code to the container image.
COPY . ./

# Run the web service on container startup.
CMD [ "npm", "start" ]

```

2. Usa Cloud Build para compilar la imagen de contenedor. Para ello, ejecuta el siguiente comando desde el directorio que contiene el Dockerfile (fíjate en la variable de entorno $GOOGLE_CLOUD_PROJECT del comando, que contiene el ID del proyecto del lab):

```shell

gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/helloworld

```

3. Obtén una lista de todas las imágenes de contenedores asociadas a tu proyecto actual con el siguiente comando:

```shell

gcloud container images list

```

4. Registra gcloud como auxiliar de credenciales para todos los registros de Docker compatibles con Google:


```shell

gcloud auth configure-docker

```

5. Para ejecutar y probar la aplicación de manera local desde Cloud Shell, iníciala con el siguiente comando estándar de docker:

```shell

docker run -d -p 8080:8080 gcr.io/$GOOGLE_CLOUD_PROJECT/helloworld

```
6. En la ventana de Cloud Shell, haz clic en Vista previa en la Web y selecciona Vista previa en el puerto 8080.


###  Implementa en Cloud Run


1. Para implementar tu aplicación alojada en contenedores en Cloud Run, usa el siguiente comando y agrega el ID de tu proyecto:

```shell

gcloud run deploy --image gcr.io/$GOOGLE_CLOUD_PROJECT/helloworld --allow-unauthenticated --region=$LOCATION

```
