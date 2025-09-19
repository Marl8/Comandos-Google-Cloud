## Comandos para crear un Load Balancing Capa 7 - Cargas de HTTP

Como configurar un balanceador de cargas de aplicaciones de capa 7 (L7) en máquinas virtuales (VMs) de Compute Engine. Los balanceadores de cargas L7 pueden comprender los protocolos HTTP(S), lo que les permite tomar decisiones de enrutamiento basadas en parámetros como la URL, los encabezados, las cookies y el contenido de la solicitud. Esto permite mejorar el rendimiento y la capacidad de respuesta de las aplicaciones.


### Configura la región y la zona predeterminadas para todos los recursos

1. Configura la región predeterminada:

```shell

    gcloud config set compute/region Region
```

2. Configura la zona predeterminada:

```shell

    gcloud config set compute/zone Zone
```


### Crea un balanceador de cargas de aplicaciones

 El balanceo de cargas de aplicaciones se implementa en Google Front End (GFE). Los GFE se distribuyen globalmente y operan juntos con el plano de control y la red global de Google. Puedes configurar reglas de URL que enruten algunas URLs a un conjunto de instancias y otras URLs a otras instancias.

Las solicitudes siempre se enrutan al grupo de instancias más cercano al usuario si el grupo tiene la capacidad suficiente y es apropiado para la solicitud. Si el grupo más cercano no tiene suficiente capacidad, la solicitud se envía al grupo más cercano que sí la tenga.

Para configurar un balanceador de cargas con un backend de Compute Engine, tus VMs deben estar en un grupo de instancias. El grupo de instancias administrado proporciona las VMs que ejecutan los servidores de backend de un balanceador de cargas de aplicaciones externo. En este lab, los backends entregan sus propios nombres de host.


1. Primero crea la plantilla del balanceador de cargas:

```shell

    gcloud compute instance-templates create < template-name > \
   --region=Region \
   --network=default \
   --subnet=default \
   --tags=allow-health-check \
   --machine-type=e2-medium \
   --image-family=debian-11 \
   --image-project=debian-cloud \
   
```
Servidor Apache

```shell

    --metadata=startup-script='#!/bin/bash
     apt-get update
     apt-get install apache2 -y
     a2ensite default-ssl
     a2enmod ssl
     vm_hostname="$(curl -H "Metadata-Flavor:Google" \
     http://169.254.169.254/computeMetadata/v1/instance/name)"
     echo "Page served from: $vm_hostname" | \
     tee /var/www/html/index.html
     systemctl restart apache2'
```

Los grupos de instancias administrados (MIG) te permiten operar apps en varias VMs idénticas. Puedes hacer que tus cargas de trabajo sean escalables y tengan alta disponibilidad gracias a los servicios de MIG automatizados, que incluyen el escalado automático, la reparación automática, la implementación regional (en varias zonas) y la actualización automática.

2. Crea un grupo de instancias administrado basado en la plantilla:

```shell

    gcloud compute instance-groups managed create < backend-group-name > \
   --template=< template-name > --size=2 --zone=Zone
```

3. Crea la regla de firewall fw-allow-health-check:

```shell

    gcloud compute firewall-rules create fw-allow-health-check \
  --network=default \
  --action=allow \
  --direction=ingress \
  --source-ranges=130.211.0.0/22,35.191.0.0/16 \
  --target-tags=allow-health-check \
  --rules=tcp:80
```

4. Ahora que las instancias están en funcionamiento, configura una dirección IP externa, estática y global que usarán tus clientes para llegar al balanceador de cargas:

```shell

    gcloud compute addresses create < lb-ipv4-1 > \
  --ip-version=IPV4 \
  --global
```

Ten en cuenta la dirección IPv4 que estaba reservada:

```shell

    gcloud compute addresses describe lb-ipv4-1 \
  --format="get(address)" \
  --global
```

5. Crea una verificación de estado para el balanceador de cargas (con esto garantizarás que solo se envíe tráfico a los backends en buen estado):

```shell

    gcloud compute health-checks create http http-basic-check \
  --port 80

```

6. Crea un servicio de backend:

```shell

    gcloud compute backend-services create < backend-name > \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=http-basic-check \
  --global

```

7. Agrega tu grupo de instancias como backend al servicio de backend:

```shell

    gcloud compute backend-services add-backend < backend-name > \
  --instance-group=< backend-group-name > \
  --instance-group-zone=Zone \
  --global

```

8. Crea un mapa de URLs para enrutar las solicitudes entrantes al servicio de backend predeterminado:

```shell

    gcloud compute url-maps create < map-http-name > \
    --default-service < web-backend-name >

```

***Nota:*** Un mapa de URLs es un recurso de configuración de Google Cloud que se usa para enrutar las solicitudes a servicios de backend o buckets de backend. Por ejemplo, con un balanceador de cargas de aplicaciones externo, puedes usar un solo mapa de URLs para enrutar solicitudes a diferentes destinos según las reglas configuradas en aquel mapa:

    Las solicitudes de https://example.com/video se enrutan a un solo servicio de backend.
    Las solicitudes de https://example.com/audio se envían a un servicio de backend diferente.
    Las solicitudes de https://example.com/images se enrutan a un bucket de backend de Cloud Storage.
    Las solicitudes de cualquier otra combinación de host y ruta de acceso se envían a un servicio de backend predeterminado.


9. Crea un Proxy HTTP de destino para enrutar las solicitudes a tu mapa de URLs.

```shell

    gcloud compute target-http-proxies create < http-proxy-name > \
    --url-map < map-http-name > 

```

10. Crea una regla de reenvío global para enrutar las solicitudes entrantes al proxy:

```shell

    gcloud compute forwarding-rules create < rule-name > \
   --address=lb-ipv4-1\
   --global \
   --target-http-proxy=< http-proxy-name > \
   --ports=80

```
***Nota:*** Una regla de reenvío y su dirección IP correspondiente representan la configuración del frontend de un balanceador de cargas de Google Cloud. Consulta la guía de Descripción general sobre las reglas de reenvío para obtener más información acerca de los conceptos básicos. 


### Pruebar el tráfico enviado a las instancias

**1.** En el campo Buscar de la consola de Google Cloud, escribe Balanceo de cargas y, luego, elige Balanceo de cargas en los resultados de la búsqueda.

**2.** Haz clic en el balanceador de cargas que acabas de crear.

**3.** En la sección Backend, haz clic en el nombre del backend y confirma que las VMs estén En buen estado. De lo contrario, espera unos minutos y vuelve a cargar la página.

**4.** Cuando las VMs estén en buen estado, prueba el balanceador de cargas en un navegador web. Ve a http://IP_ADDRESS/ (reemplaza IP_ADDRESS por la dirección IP del balanceador de cargas que copiaste anteriormente).


***NOTA:*** El navegador debe mostrar una página con contenido que indique el nombre de la instancia que entregó la página, junto con su zona (por ejemplo, Page served from: lb-backend-group-xxxx).