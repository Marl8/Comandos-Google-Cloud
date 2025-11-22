## Terraform Estado

### Finalidad del estado de Terraform

El estado es un requisito necesario para el correcto funcionamiento de Terraform. En ocasiones, los usuarios preguntan si Terraform funciona sin estado o puede no usar uno y sencillamente inspeccionar recursos en la nube en cada ejecución. En los casos en los que Terraform podría ejecutarse sin un estado, hacerlo requeriría trasladar una cantidad muy significativa de la complejidad de un lugar (el estado) a otro (el concepto que lo sustituye). En esta sección, se explica por qué es necesario el estado de Terraform.

#### Asignación al mundo real

Terraform requiere algún tipo de base de datos para asignar la configuración de Terraform con recursos del mundo real. Cuando tu configuración contiene una declaración resource ``"google_compute_instance"`` ``"foo"``, Terraform la usa para saber que la instancia ``i-abcd1234`` está representada por ese recurso.

Terraform espera que cada objeto remoto se vincule a una sola instancia de recurso. Esto normalmente está garantizado porque Terraform es responsable de crear los objetos y registrar sus identidades en el estado. Si en su lugar importas objetos que se crearon fuera de Terraform, debes verificar que cada uno por separado se importe a una única instancia de recurso.

Si un objeto remoto se vincula a dos o más instancias de recurso, Terraform podría realizar acciones inesperadas con respecto a esos objetos, puesto que la asignación desde la configuración al estado del objeto remoto se volvió ambigua.

#### Metadatos

Además del seguimiento de las asignaciones entre recursos y objetos remotos, Terraform también debe hacer el seguimiento de metadatos, como las dependencias de recursos.

De manera habitual, Terraform usa la configuración para determinar el orden de la dependencia. Sin embargo, cuando quites un recurso de una de tus configuraciones, Terraform debe saber cómo eliminarlo. Terraform puede reconocer que existe una asignación para un recurso que no se encuentre en tu archivo de configuración y que planees destruir. Sin embargo, dado que el recurso ya no existe, no se puede determinar el orden solo a partir de la configuración.

Para garantizar una correcta operación, Terraform retiene una copia del conjunto de dependencias más reciente dentro del estado. Ahora bien, Terraform aún puede determinar el orden correcto para la destrucción a partir del estado cuando eliminas uno o más elementos de la configuración.

Esto se podría evitar si Terraform conociera el orden requerido entre los distintos tipos de recursos. Por ejemplo, Terraform podría saber que los servidores se deben eliminar antes que las subredes de las que forman parte. Sin embargo, la complejidad de este enfoque pronto se vuelve difícil de administrar: además de comprender la semántica de orden de cada recurso para cada nube, Terraform también debe entender el orden entre los distintos proveedores.

Por otra parte, Terraform almacena otros metadatos por motivos similares, por ejemplo, un puntero a la configuración del proveedor usada más recientemente con el recurso en situaciones en las que están presentes varios proveedores que utilizan un alias.

#### Rendimiento

Además de la asignación básica, Terraform almacena una caché de valores de atributos para todos los recursos en el estado. Esta es una función opcional del estado de Terraform, y se usa solo como una mejora de rendimiento.

Cuando se ejecuta el comando ``terraform plan``, Terraform debe conocer el estado actual de los recursos para determinar de manera efectiva los cambios necesarios para alcanzar la configuración que deseas.

Para infraestructuras pequeñas, Terraform puede hacer una búsqueda entre tus proveedores y sincronizar los atributos más recientes a partir de todos tus recursos. Este es el comportamiento predeterminado de Terraform: para cada comando ``plan`` y ``apply``, Terraform sincroniza todos los recursos en tu estado.

Para infraestructuras más grandes, la búsqueda de cada recurso es demasiado lenta. Muchos proveedores de servicios en la nube no proporcionan APIs para consultar múltiples recursos simultáneamente, y el tiempo de ida y vuelta para cada recurso es de cientos de milisegundos. Por otra parte, los proveedores de servicios en la nube suelen tener límites de frecuencia de API, por lo que Terraform solo puede solicitar una cantidad limitada de recursos en un período. Otros usuarios de Terraform con necesidades más grandes frecuentemente usan las marcas -refresh=false y -target para solucionar esta situación. En estos casos, el estado almacenado en caché se trata como un registro de certeza.


#### Sincronización

Según la configuración predeterminada, Terraform almacena el estado en un archivo en el directorio de trabajo actual en el que se ejecuta Terraform. Esto funciona cuando comienzas a trabajar, pero, cuando se usa Terraform en equipo, es importante que todos trabajen con el mismo estado de manera que las operaciones se apliquen a los mismos objetos remotos.

La solución recomendada para este problema es el [estado remoto](https://developer.hashicorp.com/terraform/language/state/remote). Gracias a su backend de estado con todas las funciones, Terraform puede usar el bloqueo remoto como medida para evitar que distintos usuarios ejecuten Terraform accidentalmente al mismo tiempo, lo cual garantiza que cada instancia de ejecución de Terraform se inicie con el estado actualizado más reciente.

#### Bloqueo de estado

Si tu backend lo admite, Terraform bloqueará tu estado para todas las operaciones que podrían escribirlo. Esto evita que otros usuarios adquieran la información del bloqueo y corrompan tu estado.

El bloqueo de estado sucede automáticamente en todas las operaciones que podrían escribirlo. No verás ningún mensaje de que esto sucede. Si falla el bloqueo de estado, Terraform no continuará. Puedes inhabilitar el bloqueo de estado para la mayoría de los comandos con la marca ``-lock``, pero no se recomienda hacerlo.

Si la adquisición de la información de bloqueo tarda más de lo esperado, Terraform mostrará un mensaje de estado. Si Terraform no muestra un mensaje, el bloqueo de estado todavía está en efecto.

No todos los backends admiten el bloqueo. Consulta la lista de tipos de backend para obtener detalles al respecto.


#### Espacios de trabajo

Cada configuración de Terraform tiene un backend asociado que define la manera en que se ejecutan las operaciones y donde se almacenan los datos persistentes, como el estado de Terraform.

Los datos persistentes que se almacenan en el backend pertenecen a un espacio de trabajo. Inicialmente, el backend tiene un solo espacio de trabajo, llamado default y, por lo tanto, solo un estado de Terraform se asocia con esa configuración.

Algunos backends admiten múltiples lugares de trabajo con nombre, lo que permite asociar varios estados con una sola configuración. En todo caso, la configuración tiene un solo backend, aunque se pueden implementar varias instancias distintas de esa configuración sin tener que configurar un nuevo backend o cambiar las credenciales de autenticación.

Un ***backend*** en Terraform determina la manera en que se carga el estado y cómo se ejecuta una operación como apply. Esta abstracción permite el almacenamiento de estado de archivos no locales, la ejecución remota, etcétera.

De forma predeterminada, Terraform usa el backend “local”, que es el comportamiento habitual que conoces. Este es el backend que se invocaba en los labs anteriores.

Estos son algunos de los beneficios de los backends:

- **Trabajo en equipo:** Los backends pueden almacenar su estado de manera remota y protegerlo con bloqueos para evitar su corrupción. Algunos backends, como Terraform Cloud, pueden almacenar automáticamente un historial de todas las revisiones del estado.
- **Mantener información sensible fuera del disco:** el estado se recupera de los backends según demanda y solo se almacena en la memoria.
- **Realizar operaciones remotas:** para infraestructuras de mayor tamaño o ciertos cambios, la instrucción terraform apply puede llevar mucho tiempo. Algunos backends admiten operaciones remotas, lo que permite que la operación se ejecute de este modo. Puedes apagar la computadora, y tu operación se completará de cualquier manera. Cuando se combina con el almacenamiento de estado remoto y el bloqueo (que se describió más arriba), esto también es útil en entornos de trabajo en equipo.

**Los backends son completamente opcionales:** puedes utilizar Terraform de forma correcta sin tener que aprender a usar los backends. Sin embargo, resuelven puntos débiles que afectan a los equipos a determinada escala. Si trabajas de forma independiente, probablemente no tengas necesidad de usar un backend.

Incluso si tu intención es usar solo el backend “local”, podría ser útil aprender sobre el tema porque también puedes cambiar el comportamiento del backend local.

#### Agregar un backend local

Cuando configures un backend por primera vez (con lo que pasas de no tener un backend definido a configurar uno explícitamente), Terraform te da la opción de migrar tu estado al backend nuevo. Esto te permite adoptar backends sin perder los estados existentes.

Para mayor precaución, siempre se recomienda que también crees manualmente una copia de seguridad de tu estado. Para hacerlo, simplemente copia tu archivo ``terraform.tfstate`` en otra ubicación. El proceso de inicialización debería crear una copia de seguridad, pero nunca sobran las medidas adicionales.

Configurar un backend por primera vez no es diferente de hacer cambios en el futuro: se crea la nueva configuración y se ejecuta ``terraform init``. Terraform te guiará durante el resto del proceso.

El backend local almacena el estado en el sistema de archivos local, bloquea ese estado mediante las APIs del sistema y realiza operaciones de manera local.

Terraform debe inicializar cualquier backend que se haya configurado antes de usarlo. Para hacerlo, debes ejecutar ``terraform init``. Como primer paso, cualquier miembro de tu equipo debe ejecutar el comando ``terraform init`` en cualquier configuración de Terraform. Es seguro ejecutarlo varias veces, además de que realiza todas las acciones de configuración que requiere un entorno de Terraform, incluida la inicialización del backend.

El comando ``init`` debe llamarse en los siguientes casos:

- En un entorno nuevo que configura un backend
- Ante cualquier cambio a la configuración del backend (esto incluye su tipo)
- Cuando se elimina por completo la configuración del backend

No es necesario que recuerdes estos casos exactos. Terraform detectará si es necesaria la inicialización y presentará un mensaje de error en tal situación. Terraform no se inicializa automáticamente porque podría requerir información adicional por parte del usuario o realizar migraciones de estado, entre otras situaciones.


#### Agregar un backend de Cloud Storage

Un backend de Cloud Storage almacena el estado como un objeto en un prefijo configurable en un bucket determinado de Cloud Storage. Este backend admite además el [bloqueo de estado](https://developer.hashicorp.com/terraform/language/state/locking) Esto bloquea tu estado para todas las operaciones que podrían escribirlo. Esto evita que otros usuarios adquieran la información del bloqueo y corrompan tu estado.

El bloqueo de estado sucede automáticamente en todas las operaciones que podrían escribirlo. No verás ningún mensaje de que esto sucede. Si falla el bloqueo de estado, Terraform no continuará. Puedes inhabilitar el bloqueo de estado para la mayoría de los comandos con la marca ``-lock``, pero no se recomienda hacerlo.

