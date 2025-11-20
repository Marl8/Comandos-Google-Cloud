## Terraform

### ¿Qué es terraform?

Terraform es una herramienta para compilar y modificar infraestructuras, así como controlar sus versiones, de forma segura y eficiente. Permite administrar proveedores de servicios existentes y populares, además de soluciones internas personalizadas.

Los archivos de configuración le brindan a Terraform una descripción de los componentes necesarios para ejecutar desde una sola aplicación hasta un centro de datos completo. Terraform genera un plan de ejecución que describe lo que hará para alcanzar el estado deseado y, luego, lo ejecuta para compilar la infraestructura descrita. Cuando se producen cambios en la configuración, Terraform puede determinar qué se modificó y crear planes de ejecución incrementales que se puedan aplicar.

La infraestructura que puede administrar Terraform incluye componentes de niveles bajos (como instancias de procesamiento, almacenamientos y redes) y altos (como entradas de DNS y funciones de SaaS).

### Características clave

#### Infraestructura como código

La infraestructura se describe a través de una sintaxis de configuración de alto nivel. Esto permite que se controlen las versiones de un plano de tu centro de datos y que se lo trate de la misma forma que a cualquier otro fragmento de código. Además, la infraestructura se puede compartir y reutilizar.

#### Planes de ejecución

Terraform tiene un paso de planificación en el que se genera un plan de ejecución. El plan de ejecución muestra qué hará Terraform cuando se ejecute el comando apply. De esa forma, no habrá sorpresas cuando Terraform manipule infraestructura.

#### Gráfico de recursos

Terraform compila un gráfico de todos tus recursos y paraleliza la creación y modificación de todos los recursos no dependientes. Debido a esto, Terraform compila infraestructura de la forma más eficiente posible, y los operadores obtienen una mayor comprensión de las dependencias de sus infraestructuras.

#### Automatización de cambios

Se requiere muy poca interacción humana para poder aplicar conjuntos complejos de cambios en tu infraestructura. Con el plan de ejecución y el gráfico de recursos mencionados anteriormente, sabrás con exactitud qué cambios realizará Terraform y en qué orden, lo que ayudará a evitar una gran cantidad de posibles errores humanos.

1. En Cloud Shell, habilita la API de Gemini for Google Cloud con el siguiente comando:

```shell

gcloud services enable cloudaicompanion.googleapis.com

```
En la barra de herramientas de Cloud Shell, haz clic en Abrir editor.

Haz clic en ``Cloud Code - Sin proyecto en la barra de estado``, en la parte inferior de la pantalla.

Autoriza el complemento según las instrucciones. Si no se selecciona un proyecto automáticamente, haz clic en Seleccionar un proyecto de Google Cloud y elige ``Project ID``.


#### Inicialización

El primer comando que se debe ejecutar para obtener un archivo de configuración nuevo (o después de verificar uno existente desde el control de versiones) es terraform init, que inicializará diversos parámetros de configuración y datos locales que se usarán en comandos posteriores.

Terraform usa una arquitectura basada en complementos para admitir la gran cantidad de proveedores de servicios y de infraestructura disponibles. Cada “proveedor” es su propio objeto binario encapsulado distribuido independientemente de Terraform. El comando terraform init descargará e instalará automáticamente cualquier objeto binario de proveedor para que los proveedores lo usen en la configuración, que en este caso es solo el de Google.

- Descarga el objeto binario y, luego, instálalo con el siguiente comando:

```shell
terraform init
```

Se copió correctamente

Se descargará y se instalará el complemento de proveedor de Google en un subdirectorio del directorio de trabajo actual, junto con varios archivos de contabilidad. Verás el mensaje "Initializing provider plugins". Terraform sabrá que estás realizando la ejecución desde un proyecto de Google y obtendrá los recursos correspondientes.

En el resultado, se especifica qué versión del complemento se instalará y se sugiere especificar esa versión en archivos de configuración futuros para asegurarse de que terraform init instale una versión compatible.

- Crea un plan de ejecución con el siguiente comando:

````shell
terraform plan
````

Se copió correctamente

Terraform realiza una actualización (salvo que esté explícitamente inhabilitada) y, luego, determina las acciones necesarias para lograr el estado deseado que se especifica en los archivos de configuración. Este comando es un modo conveniente de comprobar si un plan de ejecución de un conjunto de cambios coincide con tus expectativas sin que debas modificar el estado o los recursos reales. Por ejemplo, puedes ejecutar el comando antes de confirmar un cambio en el control de versiones para garantizar que se comportará según lo esperado.
Nota: Se puede usar el argumento opcional -out para guardar el plan generado en un archivo y ejecutarlo más tarde con terraform apply. 

Aplica cambios

- Ejecuta este comando en el mismo directorio que el archivo instance.tf que creaste:

````shell
terraform apply
````

Se copió correctamente

En este resultado, se muestra el plan de ejecución, que describe las acciones que Terraform realizará para cambiar la infraestructura real de modo que coincida con la configuración. El formato del resultado es similar al formato diff que generan herramientas como Git.

El signo + junto a google_compute_instance.terraform significa que Terraform creará ese recurso. A continuación, se encuentran los atributos que se configurarán. Cuando el valor que se muestra es ``<computed>``, no se sabrá el valor hasta que se cree el recurso.

Si se creó correctamente el plan, Terraform se pausará y esperará a obtener la aprobación antes de continuar. En un entorno de producción, si alguna parte del plan de ejecución parece incorrecta o peligrosa, lo más seguro es cancelar la operación en esta instancia, ya que aún no se realizaron cambios en su infraestructura.

- En este caso, el plan parece aceptable, así que escribe yes cuando recibas la solicitud de confirmación para continuar y presiona Intro.
- Ejecutar el plan tardará algunos minutos porque Terraform espera a que esté disponible la instancia de VM.

Después de eso, Terraform habrá terminado.

Para ver el estado actual y observar los cambios:

````shell
terraforn show
````