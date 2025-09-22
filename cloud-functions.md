## Cloud Functions Run


Una función de Cloud Run es un fragmento de código que se ejecuta en respuesta a un evento, como una solicitud HTTP, un mensaje desde un servicio de mensajería o la carga de un archivo. Los eventos de la nube representan todo lo que ocurre en tu entorno de nube. Pueden ser cambios en la información de una base de datos, la adición de archivos a un sistema de almacenamiento o la creación de una nueva instancia de máquina virtual.

Debido a que las Cloud Run Functions se basan en eventos, solo se ejecutan cuando ocurre una acción. Esto las convierte en una buena opción para las tareas que necesitan realizarse rápido o que no es necesario que se ejecuten todo el tiempo.

Por ejemplo, puedes usar una función de Cloud Run para realizar las siguientes tareas:

- generar automáticamente miniaturas para imágenes que se suben a Cloud Storage
- enviar una notificación al teléfono de un usuario cuando se recibe un nuevo mensaje en Pub/Sub
- procesar datos de una base de datos de Cloud Firestore y generar un informe

Puedes escribir tu código en cualquier lenguaje que admita Node.js y puedes implementarlo en la nube con pocos clics. Cuando se implemente tu función de Cloud Run, se comenzará a ejecutar automáticamente en respuesta a eventos.


### Crea una función

Primero, crearás una función simple llamada helloWorld. Con esta función, se escribirá un mensaje en los registros de Cloud Run Functions. Se activa con los eventos de la función de Cloud Run y acepta una función de devolución de llamada que indica el final de la función.

En este lab, el evento de la función de Cloud Run es un evento de tema de Pub/Sub, un servicio de mensajería en el que los remitentes están separados de los destinatarios. Cuando se envía o publica un mensaje, se requiere una suscripción para que el receptor reciba una alerta y obtenga el mensaje. Para obtener más información acerca de Pub/Sub, en las guías de Pub/Sub, consulta el artículo sobre Pub/Sub, un servicio de mensajería a escala de Google.

Para obtener más información sobre el parámetro de evento y el de devolución de llamada, en la documentación de Cloud Run Functions, consulta funciones en segundo plano.

Para crear una función de Cloud Run, sigue estos pasos:

1. En Cloud Shell, ejecuta el siguiente comando para establecer la región predeterminada:

```shell

    gcloud config set run/region < REGION >

```

2. Crea un directorio para el código de la función:

```shell

    mkdir < gcf_hello_world && cd $_ >


```

3. Crea el archivo index.js y ábrelo para editarlo:

```shell

    nano index.js

```

4. Copia lo siguiente en el archivo index.js:

```javascript

const functions = require('@google-cloud/functions-framework');

// Registrar una devolución de llamada de CloudEvent con el framework de Functions que se
// ejecutará cuando el tema activador de Pub/Sub reciba un mensaje.
functions.cloudEvent('helloPubSub', cloudEvent => {
  // El mensaje de Pub/Sub se pasa como la carga útil de CloudEvent.
  const base64name = cloudEvent.data.message.data;

  const name = base64name
    ? Buffer.from(base64name, 'base64').toString()
    : 'World';

  console.log(`Hello, ${name}!`);
});

```

5. Para guardar el archivo y salir de nano, presiona Ctrl + X, luego Y y, por último, Intro.

6. Crea el archivo package.json y ábrelo para editarlo:

7. Copia lo siguiente en el archivo package.json:

```json

    {
  "name": "gcf_hello_world",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "dependencies": {
    "@google-cloud/functions-framework": "^3.0.0"
  }
}

```

8. Para guardar el archivo y salir de nano, presiona Ctrl + X, luego Y y, por último, Intro.

9. npm install



### Implementa tu función


Para este lab, establecerás --trigger-topic como cf_demo.

***Nota:***
Cloud Run Functions se basa en eventos, lo que significa que se debe especificar un tipo de activador. Cuando se implementa una función nueva, --trigger-topic, --trigger-bucket o --trigger-http son eventos activadores comunes. Cuando se implementa una actualización de una función existente, se mantiene el activador actual de la función, a menos que se indique lo contrario. 


1. Implementa la función nodejs-pubsub-function en un tema de Pub/Sub denominado cf-demo.

```shell

    gcloud functions deploy < nodejs-pubsub-function > \
  --gen2 \
  --runtime=nodejs22 \
  --region=<REGION> \
  --source=. \
  --entry-point=helloPubSub \
  --trigger-topic cf-demo \
  --stage-bucket < PROJECT_ID-bucket > \
  --service-account < cloudfunctionsa@PROJECT_ID.iam.gserviceaccount.com > \
  --allow-unauthenticated

```

***Nota:***
Si recibes una notificación de la cuenta de servicio serviceAccountTokenCreator, selecciona "n". 


2. Verifica el estado de la función:

```shell

    gcloud functions describe nodejs-pubsub-function \
  --region=<REGION> 

```

El estado ACTIVE indica que se implementó la función.
Todos los mensajes publicados en el tema activan la ejecución de la función; el contenido de los mensajes pasa como datos de entrada.


### Prueba la función

Luego de implementar la función y comprobar que se encuentra activa, prueba que esta pueda escribir un mensaje en el registro de la nube después de detectar un evento.

1. Invoca Pub/Sub con algunos datos.

```shell

    gcloud pubsub topics publish cf-demo --message="Cloud Function Gen2"

```
Consulta los registros para confirmar que haya mensajes de registro con ese ID de ejecución.


### Consulta registros


1. Revisa los registros para ver tus mensajes en el historial:

```shell

    gcloud functions logs read nodejs-pubsub-function \
  --region=<REGION> 

```
***Nota:***
Los registros pueden tardar alrededor de 10 min en aparecer. Además, la forma alternativa de ver los registros es en **Logging > Explorador de registros.**


***NOTA:***

#### Solucióm rpblema de permisos

Ver todos los permisos:

```shell

gcloud projects describe < Proyecto id >

```

Otorgar permiso:

```shell

gcloud storage buckets add-iam-policy-binding < Nombre del Bucket > \
--member='serviceAccount:service-< Número del Proyecto >@gcp-sa-eventarc.iam.gserviceaccount.com' \
--role='roles/storage.objectViewer'

```