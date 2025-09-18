## Plantilla para crear Virtual Machines

Configuar una variable de entorno para establecer la Zona 

```shell
    gcloud config set compute/region Region 

```

Configuar una variable de entorno para establecer la Zona 

```shell
    gcloud config set compute/zone Zone 

```


Para esta situación de balanceo de cargas, crea instancias de VM de Compute Engine e instala Apache en ellas. Luego, agrega una regla de firewall que permita que el tráfico HTTP llegue a las instancias.

El código proporcionado establece la zona en Zone. Si configuras el campo tags, podrás hacer referencia a estas instancias de una sola vez, por ejemplo, con una regla de firewall. Con estos comandos, también se instala Apache en cada instancia y se les otorga una página principal única:


```shell
  gcloud compute instances create www1 \
    --zone=Zone \
    --tags=<network-name> \
    --machine-type=e2-small \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "
<h3>Web Server: www1</h3>" | tee /var/www/html/index.html'

```

Sin instalar Apache server

```shell
cloud compute instances create www1 \
    --zone=Zone \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-11 \
    --image-project=debian-cloud \

```

Regla de firewall para permitir la entrada del tráfico externo a las instancias de VM

```shell
gcloud compute firewall-rules create www-firewall-network-lb \
    --target-tags network-lb-tag --allow tcp:80

```

Ver la lista de instancias

```shell
    gcloud compute instances list

```
