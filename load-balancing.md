## Comandos para crear un Load Balancing


Cuando configuras el servicio de balanceo de cargas, tus instancias de máquina virtual reciben paquetes destinados a la dirección IP externa estática que configures. Las instancias creadas con una imagen de Compute Engine se configuran automáticamente para administrar esta dirección IP.

1. Crea una dirección IP externa estática para tu balanceador de cargas:

```shell

    gcloud compute addresses create < network-name > \
  --region <Region>
```

2. Agrega un recurso de verificación de estado HTTP heredado:

```shell

    gcloud compute http-health-checks create basic-check
```

Un grupo de destino es un grupo de instancias de backend que reciben tráfico entrante de los NLB de transferencia externos. Todas las instancias de backend de un grupo de destino deben estar en la misma región de Google Cloud.

1. Ejecuta el siguiente comando para crear el grupo de destino y utilizar la verificación de estado requerida para que funcione el servicio:

```shell
    gcloud compute target-pools create < pool-name > \
  --region Region --http-health-check basic-check
```

2. Agrega al grupo las instancias que creaste anteriormente:

```shell
    gcloud compute target-pools add-instances www-pool \
    --instances <name-instance1>,<name-instance2>,<name-instance3>
```

3. A continuación, crearás la regla de reenvío. Una regla de reenvío especifica cómo enrutar el tráfico de red a los servicios de backend de un balanceador de cargas.

```shell

    gcloud compute forwarding-rules create www-rule \
    --region  Region \
    --ports 80 \
    --address <network-name> \
    --target-pool < pool-name >
```

Ahora que está configurado el servicio de balanceo de cargas, puedes comenzar a enviar tráfico a la regla de reenvío y ver cómo se dispersa el tráfico a las diferentes instancias.

1. Ingresa el comando siguiente para ver la dirección IP externa de la regla de reenvío www-rule que usa el balanceador de cargas:

```shell

    gcloud compute forwarding-rules describe www-rule --region Region
```

2. Accede a la dirección IP externa:

```shell

    IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region Region --format="json" | jq -r .IPAddress)
```

3. Muestra la dirección IP externa:


```shell

    echo $IPADDRESS
```

4. Utiliza el comando curl para acceder a la dirección IP externa (reemplaza IP_ADDRESS por la dirección IP externa del comando anterior):

```shell

    while true; do curl -m1 $IPADDRESS; done
```

La respuesta del comando curl se alterna de manera aleatoria entre las tres instancias. Si al principio la respuesta es incorrecta, espera aproximadamente 30 segundos para que la configuración se cargue por completo y las instancias estén en buen estado antes de volver a intentarlo.

5. Utiliza Ctrl + C para detener la ejecución del comando.