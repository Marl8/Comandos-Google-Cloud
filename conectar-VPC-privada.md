## Conectar a una VPC privada

### Conectarse a vm-internal mediante SSH para probar el túnel IAP

1. Ejecuta el siguiente comando en Cloud Shell para configurar la autenticación sin abrir un navegador:

```shell

gcloud auth login --no-launch-browser

```

Aparecerá un vínculo para abrir en tu navegador. Ábrelo en el mismo navegador en el que accediste a la cuenta de Qwiklabs. Cuando accedas, obtendrás un código de verificación para copiar. Pega el código en Cloud Shell.

2. Para conectarte a vm-internal, ejecuta el siguiente comando:

```shell

gcloud compute ssh vm-internal --zone ZONE --tunnel-through-iap

```

### Habilita el Acceso privado a Google

Las instancias de VM que no tienen direcciones IP externas pueden utilizar el Acceso privado a Google para acceder a las direcciones IP externas de los servicios y las APIs de Google. De forma predeterminada, el Acceso privado a Google está inhabilitado en una red de VPC.

El Acceso privado a Google está habilitado a nivel de la subred. Cuando se encuentra habilitado, las instancias en la subred que solo tienen direcciones IP privadas pueden enviar tráfico a los servicios y las APIs de Google a través de la ruta predeterminada (0.0.0.0/0) con un próximo salto a la puerta de enlace de Internet predeterminada.


- En el menú de navegación de la consola de Cloud, haz clic en Red de VPC > Redes de VPC.
- Haz clic en privatenet para abrir la red.
- Haz clic en Subredes y, luego, en privatenet-us.- 
- Haz clic en Editar.
- En Acceso privado a Google, selecciona Activo.
- Haz clic en Guardar.

 Cuando las instancias no tienen direcciones IP externas, solo se puede acceder a ellas mediante otras instancias en la red a través de una puerta de enlace de VPN administrada o un túnel de Cloud IAP. Cloud IAP habilita el acceso a las VMs adaptado al contexto a través de SSH y RDP sin hosts de bastión. Para obtener más información sobre esto, consulta la entrada de blog Cloud IAP habilita el acceso a las VMs adaptado al contexto a través de SSH y RDP sin hosts de bastión.

### Configura una puerta de enlace de Cloud NAT

Aunque ahora vm-internal puede acceder a ciertos servicios y APIs de Google sin una dirección IP externa, la instancia no puede acceder a Internet para obtener actualizaciones ni parches. Configura una puerta de enlace de Cloud NAT que permita el acceso de vm-internal a Internet.


#### Configura una puerta de enlace de Cloud NAT

Cloud NAT es un recurso regional. Puedes configurarlo para permitir el tráfico de todos los rangos de todas las subredes de una región, de solo subredes específicas de la región o de solo rangos CIDR principales y secundarios específicos.

1. Escribe Servicios de red en el campo de búsqueda de la barra de título de la consola de Google Cloud y, luego, haz clic en Servicios de red en la sección Productos y páginas.

2. En la página Servicio de red, haz clic en Fijar junto a Servicios de red.

3. Haz clic en Cloud NAT.

4. Haz clic en Comenzar para configurar una puerta de enlace de NAT.

La puerta de enlace de Cloud NAT solo implementa operaciones de NAT de salida, no de entrada. Es decir, los hosts fuera de tu red de VPC solo pueden responder a las conexiones iniciadas por tus instancias; no pueden iniciar sus propias conexiones nuevas a las instancias a través de NAT.
