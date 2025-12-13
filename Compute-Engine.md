## Compute Engine - Google Virtual Machine

#### Guía de comparación y recursos de familias de máquinas

https://cloud.google.com/compute/docs/machine-resource?hl=es-419


#### Conectarse a máquinas virtuales de Windows mediante RDP

**1.** Descargar la extención para Chrome:

https://chromewebstore.google.com/detail/chrome-rdp-for-google-clo/mpbbnannobiobpnfblimoapbephgifkm?hl=es-419

**2.** Generar contraseña para Windows:

Establece una contraseña para la VM

- Haz clic en el nombre de tu VM de Windows para acceder a Detalles de instancia de VM.
- Como no tienes una contraseña válida para esta VM de Windows, no podrás acceder a ella. Haz clic en Configurar contraseña de Windows.
- Haz clic en Configurar.
- Copia la contraseña proporcionada y haz clic en CERRAR.

**3.** Conectarse a máquinas virtuales de Windows mediante RDP

https://docs.cloud.google.com/compute/docs/instances/connecting-to-windows?hl=es



#### Cómo trabajar con máquinas virtuales 

Levantar un servidor de Minicraft:

https://www.skills.google/paths/621/course_templates/50/labs/594661?locale=es

[Revisión del laborratorio](https://www.youtube.com/watch?v=7c_3m5t53dA)

**Utilizar en las instancias de SSH**

El servidor está vinculado a la duración de tu sesión SSH, es decir, si cierras tu terminal SSH, también se cerrará el servidor. Para evitar este problema, utiliza ***screen***, una aplicación que te permite crear una terminal virtual que se puede “desconectar” y ejecutar como proceso en segundo plano, o bien "volver a conectar" y ejecutar como proceso en primer plano. Cuando una terminal virtual se desconecta para iniciar el proceso en segundo plano, se ejecutará ya sea que hayas accedido al sistema o no.

1. Para instalar screen, ejecuta el siguiente comando:

```shell

sudo apt-get install -y screen

```

