## Comandos para crear un Load Balancing - Cargas Internas

El balanceador de cargas de aplicaciones interno es esencial para crear aplicaciones internas sólidas, seguras y fáciles de administrar que potencien las operaciones comerciales. En este lab, aprenderás a distribuir el tráfico de red dentro de tu red de nube privada sin exponer tus máquinas virtuales (VMs) directamente al Internet público, lo que mantiene tus servicios seguros y eficientes.

Compilarás un patrón de arquitectura simplificado, pero muy común:

- Un "nivel web" (sitio web público) que necesita pedir ayuda a otro servicio interno
- Un "nivel de servicio interno" (una calculadora de números primos) que realiza tareas específicas y se distribuye en varias máquinas

Esta configuración garantiza que, incluso si una parte de tu servicio interno se ocupa o deja de funcionar, el sistema general siga funcionando sin problemas, ya que el balanceador de cargas dirige automáticamente las solicitudes a las máquinas en buen estado.


### Crea un entorno virtual

Un entorno virtual mantiene el software de tu proyecto ordenado y garantiza que tu código siempre se ejecute con las versiones específicas de las herramientas que necesita.

Se utilizan entornos virtuales de Python para aislar del sistema la instalación de paquetes.

1. Instala el entorno virtualenv:

```shell

    sudo apt-get install -y virtualenv

```

2. Crea el entorno virtual:

```shell

    python3 -m venv venv

```

3. Activa el entorno virtual:

```shell

    source venv/bin/activate

```

### Configura el balanceador de cargas interno

Configura el balanceador de cargas interno

Estás creando esa única entrada VIP privada para tu servicio interno que permite que otras aplicaciones internas accedan a tu "calculadora de números primos" de forma confiable, sin necesidad de saber qué VM de backend específica está activa o disponible. Ahora, configuremos el balanceador de cargas interno y conectémoslo al grupo de instancias que acabas de crear.

Un balanceador de cargas interno consta de tres partes principales:

- **Regla de reenvío:** Es la dirección IP privada real a la que otros servicios internos enviarán solicitudes. "Reenvía" el tráfico a tu servicio de backend.
- **Servicio de backend:** Define cómo el balanceador de cargas distribuye el tráfico a tus instancias de VM. También incluye la verificación de estado.
- **Verificación de estado:** Es una verificación continua que supervisa el "estado" de tus VMs de backend. El balanceador de cargas solo envía tráfico a las máquinas con verificaciones de estado aprobadas, lo que garantiza que tu servicio esté siempre disponible.

En el siguiente diagrama, se muestra cómo se balancean las cargas de las instancias con varias instancias en varios grupos de backend en diferentes zonas.


![bolad-balancing](img/T65P3C60.png)


#### Crea una verificación de estado

1. Se necesita una verificación de estado para asegurarse de que el balanceador de cargas solo envíe tráfico a instancias en buen estado. Tu servicio de backend es un servidor HTTP, por lo que debes verificar si responde con un "200 OK" en una ruta de URL específica.

```shell

    gcloud compute health-checks create http ilb-health 
    --request-path < request >

```

Dado que se proporciona el servicio HTTP, verifica si se propaga una respuesta 200 en una ruta de URL específica.


#### Crea un servicio de backend

2. Ahora, crea el servicio de backend llamado prime-service:

```shell

    gcloud compute backend-services create prime-service \
--load-balancing-scheme internal --region=$REGION \
--protocol tcp --health-checks ilb-health

```

Este servicio vinculará la verificación de estado al grupo de instancias.


#### Agrega el grupo de instancias al servicio de backend

3. Conecta tu grupo de instancias de backend al servicio de backend de prime-service. Esto le indica al balanceador de cargas qué máquinas debe administrar:

```shell

    gcloud compute backend-services add-backend prime-service \
--instance-group backend --instance-group-zone=$ZONE \
--region=$REGION

```

#### Crea la regla de reenvío

4. Por último, crea la regla de reenvío llamada prime-lb con una IP estática de IP:

```shell

    gcloud compute forwarding-rules create prime-lb \
--load-balancing-scheme internal \
--ports 80 --network default \
--region=$REGION --address IP \
--backend-service prime-service

```

Tu servicio interno de "cálculo de números primos" ahora está completamente configurado y listo para recibir consultas a través de su dirección IP interna.


### Prueba el balanceador de cargas

Este paso es fundamental para confirmar que tu balanceador de cargas de aplicaciones interno dirija el tráfico correctamente a los servicios de backend. Esto demuestra que otras aplicaciones internas ahora pueden acceder de manera confiable a tu servicio a través de una sola dirección IP estable, lo que garantiza un funcionamiento continuo.

Para probar el balanceador de cargas, debes crear una instancia de VM nueva en la misma red que tu balanceador de cargas de aplicaciones interno. Solo se puede acceder a ella desde tu red de nube privada, no directamente desde Cloud Shell (que se encuentra fuera de esta red específica).

1. Con gcloud en Cloud Shell, crea una instancia de prueba simple:

```shell

    gcloud compute instances create testinstance \
--machine-type=e2-standard-2 --zone $ZONE

```

2. Luego, establece una conexión SSH a ella:

```shell

    gcloud compute ssh testinstance --zone $ZONE

```

Si se te solicita, escribe Y y presiona Intro dos veces para continuar.



