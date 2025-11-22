## Terraform Modules

#### ¿Para qué sirven los módulos?

Estas son algunas de las maneras en que los módulos te permiten solucionar los problemas enumerados anteriormente:

- **Organiza la configuración**: Los módulos facilitan la navegación, comprensión y actualización de tu configuración, ya que agrupan las partes relacionadas. Incluso una infraestructura con una complejidad moderada puede requerir cientos o miles de líneas de configuración para implementarse. Sin embargo, si usas módulos, puedes organizar tu configuración en componentes lógicos.

- **Encapsula la configuración:** Otro beneficio de usar módulos es que puedes encapsular la configuración en componentes lógicos separados. El encapsulamiento permite evitar consecuencias inesperadas (por ejemplo, que un cambio en una parte de tu configuración modifique otra infraestructura por accidente) y reduce las probabilidades de que se produzcan errores sencillos, como usar el mismo nombre para dos recursos diferentes.

- **Reutiliza la configuración:** Escribir tu configuración completa sin usar el código existente puede tardar mucho y provocar errores. En cambio, si usas módulos puedes ahorrar tiempo y reducir los errores costosos, ya que se reutiliza la configuración escrita por ti, otros miembros de tu equipo, o bien otros profesionales de Terraform que publicaron módulos para que los utilices. También puedes compartir los módulos que escribas con tu equipo o el público general para que se beneficien de tu arduo trabajo.

- **Proporciona coherencia y asegúrate de que se apliquen las prácticas recomendadas:** Los módulos también te ayudan a proporcionar coherencia a tus configuraciones. La coherencia permite que los parámetros de configuración complejos sean más fáciles de comprender, y esto garantiza que se apliquen las prácticas recomendadas en toda tu configuración. Por ejemplo, los proveedores de servicios en la nube ofrecen varias opciones para configurar servicios de almacenamiento de objetos, como Amazon S3 (Simple Storage Service) o los buckets de Google Cloud Storage. Muchos incidentes de seguridad de alto perfil han implicado objetos almacenados sin una correcta protección. Debido a la cantidad de opciones complejas de configuración involucradas, es fácil configurar incorrectamente estos servicios de manera accidental.

Usar módulos ayuda a reducir estos errores. Por ejemplo, podrías crear un módulo para describir cómo se configurarán todos los buckets del sitio web público de tu organización, y otro módulo para los buckets privados que se usan en aplicaciones de registro. Además, si se debe actualizar una configuración para un tipo de recurso, usar módulos te permite hacerlo en un solo lugar y aplicar la actualización a todos los casos en los que uses ese módulo.


### ¿Qué es un módulo de Terraform?

Un módulo de Terraform es un conjunto de archivos de configuración de Terraform en un solo directorio. Incluso una configuración simple que consista en un único directorio con uno o más archivos .tf es un módulo. Cuando ejecutas los comandos de Terraform directamente desde un directorio de este tipo, este se considera el módulo raíz. En ese sentido, cada configuración de Terraform es parte de un módulo. Podrías tener un conjunto sencillo de archivos de configuración de Terraform como el siguiente:

````text
├── LICENSE
├── README.md
├── main.tf
├── variables.tf
├── outputs.tf

````


En este caso, cuando ejecutas comandos de Terraform desde el directorio minimal-module, el contenido de dicho directorio se considera como módulo raíz.

#### Llamadas a módulos

Los comandos de Terraform solo usan directamente los archivos de configuración en un directorio que, por lo general, es el directorio de trabajo actual. Sin embargo, tu configuración puede usar bloques de módulos para llamar módulos de otros directorios. Cuando Terraform encuentra un bloque de módulo, lo carga y procesa sus archivos de configuración.

En ocasiones, cuando otra configuración llama a un módulo, este último se denomina “módulo secundario” de la configuración.

#### Módulos locales y remotos

Puedes cargar módulos desde el sistema de archivos local o una fuente remota. Terraform es compatible con una variedad de fuentes remotas, entre las que se incluyen Terraform Registry, la mayoría de los sistemas de control de versiones, URLs HTTP y los registros de módulos privados de Terraform Cloud o Terraform Enterprise.

#### Prácticas recomendadas de módulos

En muchos sentidos, los módulos de Terraform son similares a los conceptos de bibliotecas, paquetes o módulos presentes en la mayor parte de los lenguajes de programación, y proporcionan muchos de sus beneficios. De la misma manera que casi cualquier programa de cómputo no trivial, las configuraciones del mundo real de Terraform deberían usar módulos en la medida de lo posible para ofrecer los beneficios que se mencionan arriba.

Se sugiere que todos los usuarios de Terraform sigan estas prácticas recomendadas cuando usen módulos:

- Comienza a escribir tu configuración con un plan para los módulos. Incluso para configuraciones de Terraform poco complejas y administradas por una sola persona, los beneficios de usar los módulos superan el tiempo que toma usarlos de manera adecuada.

- Usa módulos locales para organizar y encapsular tu código. Incluso si no usas o publicas módulos remotos, organizar tu configuración en términos de módulos desde un inicio reduce significativamente la carga de mantener y actualizar tu configuración a medida que aumenta la complejidad de tu infraestructura.

- Usa Terraform Registry, un servicio público que te permite buscar módulos útiles. De esta manera, puedes implementar tu configuración rápidamente y con confianza, pues se vale del trabajo de otros usuarios.

- Publica y comparte módulos con tu equipo. La mayoría de las infraestructuras son administradas por un equipo, y los módulos son una herramienta importante que pueden usar para crear y mantener tus infraestructuras. Como se mencionó antes, puedes publicar módulos de manera pública o privada. Explorarás cómo hacerlo en un lab posterior de esta serie.

#### Cómo establecer valores para las variables de entrada de los módulos

Algunas variables de entrada son obligatorias, lo que significa que el módulo no proporciona un valor predeterminado. Por el contrario, se debe proporcionar un valor explícito para que Terraform se ejecute correctamente.

Para usar la mayoría de los módulos, deberás pasar variables de entrada a la configuración del módulo. La configuración que llama a un módulo es responsable de establecer sus valores de entrada, los cuales se pasan como argumentos al bloque de módulo. Además de source y version, la mayoría de los argumentos para un bloque de módulo establecen valores de variables.

En la página de Terraform Registry para el módulo de red de Google Cloud, la pestaña Entradas describe todas las [variables de entrada](https://developer.hashicorp.com/terraform/language/values/variables) que son compatibles con el módulo.

#### Cómo definir variables de entrada raíz

El uso de variables de entrada con módulos es muy similar a la manera en que las usarías en cualquier configuración de Terraform. Un patrón común es identificar las variables de entrada del módulo que quizá debas modificar en el futuro y, luego, crear variables coincidentes en el archivo variables.tf de tu configuración con valores predeterminados sensatos. Luego, esas variables se pueden pasar al bloque de módulo como argumentos.


#### Cómo definir valores de salida de raíz

Los módulos también tienen valores de salida, que se definen dentro del módulo con la palabra clave output. Puedes acceder a ellos consultando module.``<MODULE NAME>`` ``<OUTPUT NAME>`` Al igual que las variables de entrada, las salidas del módulo se enumeran en la pestaña outputs de Terraform Registry.

Las salidas de los módulos usualmente se pasan a otras partes de tu configuración o se definen como salidas en tu módulo raíz. Este lab incluye ejemplos de ambos casos.

#### Cómo funcionan los módulos

Cuando uses un módulo nuevo por primera vez, debes ejecutar ``terraform init`` o ``terraform get`` para instalarlo. Con cualquiera de estos dos comandos, Terraform instala los módulos nuevos en el directorio ``.terraform/modules``, que se encuentra en el directorio de trabajo de tu configuración. Para los módulos locales, Terraform crea un symlink al directorio del módulo. Por ello, cualquier cambio a los módulos locales se verá reflejado de inmediato, sin que debas volver a ejecutar ``terraform get``.

#### Estructura de módulos

Terraform trata todos los directorios locales a los que se hace referencia en el argumento source de un bloque module como un módulo. Esta es la estructura de archivos convencional de un módulo nuevo:

```text
├── LICENSE
├── README.md
├── main.tf
├── variables.tf
├── outputs.tf
```

    tura de archivos que desees.

Cada uno de estos archivos cumple un propósito:

- ``LICENSE ``contiene la licencia bajo la cual se distribuirá tu módulo. Cuando lo compartes, el archivo LICENSE les informará a los usuarios los términos bajo los cuales está disponible. Terraform no usa este archivo.
- ``README.md`` contiene documentación en formato markdown, que describe cómo usar tu módulo. Terraform no usa este archivo, pero otros servicios como Terraform Registry y GitHub mostrarán el contenido de este archivo a los visitantes de las páginas de Terraform Registry o GitHub correspondientes a tu módulo.
 - ``main.tf`` contiene el conjunto principal de parámetros de configuración para tu módulo. También puedes crear otros archivos de configuración y organizarlos de manera que tengan sentido para tu proyecto.
- ``variables.tf`` contiene las definiciones de variables para tu módulo. Cuando otras personas usan tu módulo, las variables se configurarán como argumentos en el bloque de módulo. Dado que debes definir todos los valores de Terraform, cualquier variable que no tenga un valor predeterminado se convertirá en un argumento obligatorio. También puedes proporcionar una variable con un valor predeterminado como si fuera un argumento del módulo, con lo que se anula el valor predeterminado.
- ``outputs.tf`` contiene las definiciones de salida para tu módulo. Estas salidas están disponibles para la configuración que usa el módulo, por lo que a menudo se usan para pasar información sobre las partes de tu infraestructura que define el módulo a otras partes de tu configuración.

Ten cuidado con estos archivos y asegúrate de no distribuirlos como parte de tu módulo.

- Los archivos`` terraform.tfstate`` y ``terraform.tfstate``.backup contienen el estado y son la manera en que Terraform hace un seguimiento de la relación entre tu configuración y la infraestructura que esta aprovisiona.
 - El directorio ``.terraform`` contiene los módulos y complementos que se usan para aprovisionar tu infraestructura. Estos archivos son específicos para cada instancia individual de Terraform cuando aprovisionas la infraestructura, no para la configuración de la infraestructura que se define en los archivos ``.tf``.
- No es necesario distribuir los archivos ``*.tfvars`` con tu módulo, a menos que también lo uses como una configuración independiente de Terraform. Esto se debe a que las variables de entrada del módulo se establecen mediante argumentos para el bloque de módulo en tu configuración.

#### Cómo instalar el módulo local

Cuando agregues un módulo nuevo a una configuración, Terraform debe instalarlo para que se pueda usar. Los comandos ``terraform get`` y ``terraform init`` instalan y actualizan los módulos. El comando ``terraform init ``también inicializa los backends y los complementos de instalación.