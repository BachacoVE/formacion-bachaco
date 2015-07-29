Capítulo 1.	Comenzando con Odoo
====

**Desarrollo**

Antes de sumergirse en el desarrollo de Odoo, es necesario configurar el entorno de desarrollo, y para esto se debe aprender las tareas básicas de administración.

En este capítulo,	se aprenderá como configurar el entorno de desarrollo, donde luego se desarrollarán las aplicaciones Odoo. 

Se aprenderá a configurar sistemas Debian o Ubuntu para alojar las instancias del servidor de desarrollo, y como instalar Odoo desde el código fuente en GitHub. Luego aprenderá a configurar archivos compartidos con	Samba, permitiendo trabajar con archivos de Odoo desde una estación de trabajo con cualquier sistema operativo.

Odoo esta desarrollado usando el lenguaje de programación Python y usa PostgreSQL como base de datos para almacenar datos, estos son los requisitos principales para trabajar con Odoo. Para ejecutar Odoo desde el código fuente, es necesario instalar las librerías Python de las cuales depende. Luego el código fuente de Odoo debe descargarse desde GitHub y ejecutado desde el código fuente. Aunque es posible descargar un zip o tarball, es mejor obtener el código fuente usando GitHub,	así además tendremos Odoo instalado en nuestro equipo.


**Configurar un equipo como servidor Odoo**

Preferimos usar sistemas Debian/Ubuntu para el servidor Odoo, aunque puede trabajar desde el sistema operativo de su preferencia, sea Windows,	Macintosh, o Linux.

Odoo puede ser ejecutado en una gran variedad de sistemas operativos, entonces porque elegir Debian por encima de otros sistemas operativos? Debido a que Odoo es desarrollado pensado principalmente para sistemas Debian/Ubuntu, tienen mejor soporte para Odoo. Por lo tanto será más fácil encontrar ayuda y recursos adicionales si se trabaja con Debian/Ubuntu.

También son las plataformas más usadas por las personas que desarrollan aplicaciones, y donde se dan a conocer más implementaciones. Por lo tanto, inevitablemente, se espera que las desarrolladoras y los desarrolladores de Odoo se sientan a gusto con esta plataforma. Incluso quienes tiene una historial de trabajo con Windows, es importante que tengan algún conocimiento sobre estas plataformas.

En este capítulo, se aprenderá a configurar y trabajar con Odoo sobre un sistema Debian, usando unicamente la línea de comando. Para quienes están acostumbrados a sistemas Windows,	se describirá como configurar una máquina virtual para alojar un servidor Odoo. Adicionalmente, las técnicas aprendidas servirán para gestionar servidores Odoo en la nube donde el único acceso será a través de una Shell Segura (SSH ).

*Nota*

*Tenga en cuenta que estas instrucciones tienen como objetivo configurar un nuevo sistemas para el desarrollo. Si desea probarlas en un sistema existente, haga un respaldo a tiempo que le permita recuperar el sistema en caso de algún problema.*


**Disposiciones para un equipo Debian**

Como se explico antes, será necesario un equipo con Debian para alojar nuestro servidor Odoo versión 8.0.	Si estos son sus primeros pasos con Linux, le gustara saber que Ubuntu es una distribución Linux basada en Debian, por lo tanto son muy similares.

*Nota*

*Odoo asegura su funcionamiento con la versión estable de Debian o Ubuntu. Al momento de elegir este libro, las versión estable para Debian es la versión 7 “Wheezy” y para Ubuntu la versión 14.04 “Trusty	Tahr”. Ambas se distribuyen con Python 2.7, necesario para ejecutar	Odoo.*

Si ya está ejecutando Ubuntu u otra distribución basada en Debian, todo esta listo para comenzar; esta máquina también puede ser usada para alojar Odoo.

Para los sistemas operativos	Windows y Macintosh, es posible tener Python, PostgreSQL, y todas las dependencias instaladas, y luego ejecutar Odoo desde el código fuente de forma nativa.

Sin embargo, esto puede ser un gran reto, por lo que nuestra recomendación es usar una máquina virtual ejecutando Debian o Ubuntu	Server. Puede usar su software de virtualización preferido para hacer funcionar Debian en una máquina virtual.	Si necesita alguna ayuda, aqui hay algunos consejos: en lo que se refiere a software de vistualización, tiene muchas opciones, como Microsoft Hyper-V (disponible para algunas versiones de Windows), Oracle VirtualBox, o VMWare	Player (o VMWare Fusion para	Macintosh).	VMWare Player es probablemente el más fácil de usar, y puede descargarse gratuitamente	en [https://my.vmware.com/web/vmware/downloads](https://my.vmware.com/web/vmware/downloads)

Con relación a la imagen Linux a usar, Ubuntu Server es más amigable para las usuarias y usuarios para instalar que Debian. Si esta comenzando con Linux, es recomendable que use una distribución lista para usar.

TurnKey de Linux provee imágenes fáciles de usar, preinstaladas en distintos formatos, incluyendo ISO. El formato ISO funcionara con cualquier software de vistualización de su preferencia, o incluso en cualquier equipo actual. Una buena opción sería una imagen	LAPP,	que puede hallarse en [http://www.turnkeylinux.org/lap](http://www.turnkeylinux.org/lap)

Una vez instalado el sistema e iniciado, debería ser capaz de ingresar en la línea de comando.

Si ingresa usado root, su primera tarea sera crear un usuario para ser usado en el trabajo cotidiano, ya que es considerado mala práctica trabajar como root. Particularmente, el servidor Odoo se rehusará a ejecutarse su esta usando root.

Si esta usando Ubuntu, probablemente no necesite esto ya que el proceso de instalación le habrá guiado en la creación de un usuario personal. 


**Creando una cuenta de usuario para Odoo**

Primero, asegúrese que Odoo este instalado. Su usuario de trabajo lo necesitara. Si ha accedido como root:

```
# apt-get update & apt-get upgrade 	# Instalar actualizaciones del sistema
# apt-get install	sudo 	# Asegurarse que 'sudo'	esta instalada 
```
Los siguientes comandos creará un usuario odoo:
```
# useradd -m -g sudo -s /bin/bash odoo # Crea un usuario 'Odoo' con poderes sudo
# passwd odoo # Solicita y configura una contraseña para el nuevo usuario 
```
Puede cambiar odoo por cualquier nombre que desee. 

La opción -m crea el directorio home. El `-g sudo` agrega al nuevo usuario a la lista de usuarios sudo, por lo tanto podrá ejecutar comandos como root, y `-s /bin/bash` configura la línea de comando predeterminada a bash, la cual es mas amigable de usar que la fijada por omisión estándar sh.

Ahora puede acceder con el nuevo usuario y configurar Odoo.


**Instalar Odoo desde el código fuente**

Los paquetes de Odoo listos para instalar pueden ser encontrados en [nightly.odoo.com]	(http://nightly.odoo.com), disponibles para Windows (.exe), Debian (.deb), CentOS (.rpm),	y código fuente (.tar.gz).

Como desarrolladoras y desarrolladores, preferimos hacer la instalación directamente desde el repositorio GitHub. Esto nos permitirá tener más control sobre las sucesivas versiones y actualizaciones.

Para mantener el orden de las cosas, se trabaja en el directorio `/odoo-dev` que se encuentra en su diretorio `/home`. A lo largo del libro, se asume que este es el lugar donde esta instalado el servidor Odoo.

Primero, asegúrese que ha accedido con el usuario creado anteriormente, o durante el proceso de instalación, y no como root. Asumiento que su usuario es odoo, puede confirmar esto con el siguiente comando:
```
$ whoami odoo
$ echo $HOME/home/odoo 
```
Ahora es posible usar este script. Muestra como instalar Odoo desde el código fuente en un sistema Debian:
```
$ sudo apt-get update &	sudo apt-get upgrade # Instala las actualizaciones del sistema
$ sudo apt-get install git # Instala Git
$ mkdir	~/odoo-dev # Crear el diretorio de trabajo
$ cd	~/odoo-dev  # Ingresar en el directorio de trabajo
$ git clone	https://github.com/odoo/odoo.git -b 8.0 # Obtiene el código fuente de Odoo
$ ./odoo/odoo.py setup_deps # Instala las dependencias del sistema Odoo
$ ./odoo/odoo.py setup_pg # Instala PostgreSQL y el usuario administrador para un usuario unix 
```
Al finalizar, Odoo estará listo para ser usado.	

El símbolo `~` es un atajo para su directorio raíz (por ejemplo, `/home/odoo`). La opción `git	-b 8.0` explicitamente solicita descargar la rama 8.0 de Odoo. En el momento de escribir este libro, esto es redundante, ya que 8.0 es la rama  predeterminada, pero esto puede cambiar, lo que hará más flexible los aquí descrito.

Para iniciar una instancia del servidor Odoo, simplemente ejecute `odoo.py`:
```
$ ~/odoo-dev/odoo/odoo.py 
```
De forma predeterminada, las instancias de Odoo escuchan a través del puerto 8069, si en un navegador web se apunta a `http://<server-address>:8069` se llegará a esa instancia.	Cuando se acceso por primera vez, se mostrará un asistente para crear una nueva base de datos, como se muestra en la siguiente imagen:

![52_1](/images/Odoo Development Essentials - Daniel Reis-52_1.jpg)

Pero aprenderá como inicializar bases de datos nuevas desde la línea de comando,  ahora presione *Ctrl* + *C* para detener el servidor y volver a la línea de comandos.


**Inicializar una base de datos nueva en Odoo**

Para poder crear una base de datos nueva, su usuario debe ser un super usuario de PostgreSQL.	Lo siguiente hace esto por usted `./odoo.py setup_pg`; de lo contrario use el siguiente comando para crear un super usuario PostgreSQL para el usuario Unix actual:
```
$ sudo createuser --superuser	$(whoami) 
```
Para crear una base de datos nueva use este el comando createdb.	Cree la base de datos	v8dev:
```
$ createdb v8dev  
```
Para inicializar esta base de datos con el esquema de datos de Odoo debe ejecutar Odoo en la base de datos vacía usando la opción	 `-d`:
```
$	~/odoo-dev/odoo/odoo.py	-d	v8dev 
```
Tomará unos minutos inicializar la base de datos v8dev, y terminará con un mensaje de registro INFO	Módulos cargados . Luego el servidor estará listo para atender las peticiones del cliente.

Este método inicializará la base de datos con los datos de demostración, lo cual frecuentemente es útil en el desarrollo de base de datos. Para inicializar una base de datos sin los datos de demostración, agregue la siguiente opción al comando anterior: `--without-demo-data=all`.

Para mostrar la pantalla de acceso abra en un navegador web `http://<server-name>:8069`.

Si no conoce el nombre de su servidor, escriba el comando hostname en la terminal para averiguarlo, o el comando ifconfig para conocer la dirección IP.

Si esta alojando Odoo en una máquina virtual probablemente necesite hacer algunas configuraciones de red para poder usarlo como servidor. La solución más simple es cambiar el tipo de red de la VM de NAT a	Bridged. Con esto, en vez de compartir la dirección IP del equipo, la VM huésped tendrá su propia dirección IP. También es posible usar NAT, pero esto requiere que configure el enrutamiento de puerto, así su sistema sabrá que algunos puertos, como el	8069, deben ser controlados por la VM. En caso de algún problema, con suerte estos detalles puedan ayudarle a encontrar ayuda en la documentación del software de virtualización de su preferencia.

La cuenta de usuario predeterminada es admin con la contraseña admin. Una vez acceda se mostrará el menú	Configuración , revelando los módulos instalados. Elimine el filtro de	Instalado y podrá ver e instalar cualquiera de los módulos oficiales.

En cualquier momento que desee detener la instancia del servidor Odoo y volver a la línea de comando, presione *Ctrl* +	*C*. En consola, presiona la tecla de flecha Arriba para mostrar el comando anterior ejecutado,	esta es una forma rápida de iniciar Odoo con las mismas opciones. Notará que *Ctrl* + *C* seguido de la flecha Arriba y *Enter* es una combinación frecuentemente usada para re-iniciar el servidor Odoo durante el desarrollo.


**Gestionar la base de datos**

Ha visto como crear e inicializar bases de datos nuevas en Odoo desde la línea de comando. Existen mas comandos que valen la pena conocer para gestionar bases de datos.

Ya sabe como usar el comando	createdb para crear una base de datos vacía, pero también puede crear una base de datos copiando una existente, usando la opción `--template`.

Asegúrese que su instancia de Odoo este detenida y no tenga otra conexión abierta con la base de datos v8dev creada anteriormente, y ejecute:
```
$ createdb –template=v8dev v8test 
```
De hecho, cada vez que se crea una base de datos, es usada una plantilla. Si no se especifica ninguna, se usa una predefinida llamada template1.

Para listas las bases de datos existentes en su sistema use la utilidad psql de PostgreSQL	con la opcion `-l`:
```
$ psql -l 
```
Al ejecutar esto se debe listar las dos bases de datos creadas hasta los momentos: v8dev	y v8test. La lista también mostrará la codificación usada en cada base de datos. La codificación predeterminada es UTF8, la cual es necesaria para las bases de datos Odoo.

Para eliminar una base de datos que ya no necesite (o necesita crear nuevamente), use el comando dropdb:
```
$ dropdb v8test 
```
Ahora ya conoce lo básico para trabajar con varias bases de datos. Para aprender más sobre PostgresSQL, puede encontrar la documentación oficial en [http://www.postgresql.org/docs/](http://www.postgresql.org/docs/)

*Nota*

*ADVERTENCIA: Eliminar una base de datos destruirá los datos de forma irrevocable. Tenga cuidado al ejecutar esta acción y mantenga siempre respaldos de sus bases de datos.*


**Unas palabras sobre las versiones de Odoo**

A la fecha de publicación, la última versión estable de Odoo es la 8, marcada en GitHub como branch 8.0. Esta es la versión con la que se trabajará a lo largo de este libro.

Es importante saber que las bases de datos de Odoo son incompatibles ente versiones principales de Odoo. Esto significa que si ejecuta un servidor Odoo 8 contra una base de datos Odoo/OpenERP 7, no funcionará. Es necesario un trabajo de migración significativo para que una base de datos pueda ser usada con una versión más reciente del producto.

Lo mismo pasa con los módulos: como regla general un módulo desarrollado para una versión más reciente de Odoo no funcionará con otras versiones.	Cuando descargue módulos desde la Web desarrollados por la comunidad, asegúrese que estén dirigidos a la versión de Odoo que este usando.

Por otro lado, los lanzamientos principales (7.0, 8.0) reciben actualizaciones frecuentes, pero en su mayoría deberán ser correcciones de fallos. Tiene la garantía de ser “estables para la API”, lo que significa que las estructuras del modelo de datos y los identificadores de vista de los elementos se mantendrán estables. Esto es importante porque significa que no habrá riesgo de estropear los módulos personalizados por causa de cambios incompatibles en los módulos base.

Y sea consciente que la versión en la rama master se convertirá en la próxima versión principal estable, pero hasta entonces no será “estable para la API” y no debe usarla para construir módulos personalizados. Hacer esto es como moverse en arena movediza: no hay formar de asegurar cuando algún cambio será introducido el cual hará que su módulo falle.


**Más opciones de configuración del servidor**

El servidor Odoo soporta unas pocas opciones más. Es posible verificar todas las opciones disponibles con la opción `--help`:
```
$ ./odoo.py --help 
```
Vale la pena tener una idea general de las más importantes.


**Archivos de configuración del servidor Odoo**

La mayoría de las opciones pueden ser guardadas en un archivo de configuración. De forma predeterminada, Odoo usará el archivo `.openerp-serverrc` en su directorio home. Convenientemente, existe una opción `--save` para guardar la instancia actual de configuración dentro de ese archivo:
```
$ ~/odoo-dev/odoo/odoo.py --save –stop-after-init # guarda la configuración en	 archivo 
```
Aquí también se usa la opción	`--stop-after-init`, para que el servidor se detenga al finalizar las acciones. Esta opción es usada frecuentemente para ejecutar pruebas y solicitar la ejecución de actualización de un módulo para verificar que se instala correctamente.

Ahora se puede inspeccionar lo que se guardo en este archivo de configuración:
```
$ more ~/.openerp_serverrc # mostrar el archivo de configuración 
```
Esto mostrara todas las opciones de configuración disponibles con sus valores predeterminados.	La edición se hará efectiva la próxima vez que inicie una instancia de Odoo.	Escriba	q	para salir y retornar a la línea de comandos.

También es posible seleccionar un archivo específico de configuración,	usando la opción `--conf=<filepath>`. Los archivos de configuración no necesitan tener todas las opciones de configuración que ha visto hasta ahora. Solo es necesario que estén aquellas opciones que cambian los valores predeterminados.


**Cambiar el puerto de escucha**

El comando `--xmlrpc-server=<port>` permite cambiar el puerto de escucha predeterminado 8069 desde donde escuchan las instancias del servidor. Esto puede ser usado para ejecutar más de una instancia al mismo tiempo, en el mismo servidor.

Intentemos esto. Abra dos ventanas de la terminal. En la primera ejecute:
```
$ ~/odoo-dev/odoo.py --xmlrpc-port=8070
```
y en la otra ejecute:
```
$ ~/odoo-dev/odoo.py --xmlrpc-port=8071 
```
Y allí lo tiene: dos instancias de Odoo en el mismo servidor escuchando a través de diferentes puertos. Las dos instancias pueden ser usadas en la misma o en diferentes base de datos. Y ambas pueden ejecutar versiones iguales o diferentes de Odoo.


**Registro**

La opción `--log-level` permite configurar el nivel de detalle del registro.	Esto puede ser muy útil para entender lo que esta pasando en el servidor. Por ejemplo, para habilitar el nivel de registro de depuración utilice: `--log-level=debug` 

Los siguientes niveles de registro pueden ser particularmente interesantes: 
- debug_sql para inspeccionar el SQL generado por el servidor 
- debug_rpc para detallar las solicitudes recividas por el servidor 
- debug_rpc para detallar las respuestas enviadas por el servidor 

La salida del registro es enviada de forma predeterminada a la salida estándar (la terminal),	pero puede ser dirigida a un archivo de registro con la opción `--logfile=<filepath>`. 

Finalmente, la opción `--debug` le enviará al depurador Python (pdb) cuando aparezca una excepción.	Es útil hacer un análisis post-mortem de un error del servidor. Note que esto no tiene ningún efecto en el nivel de detalle del registro. Se pueden encontrar más detalles sobre los comandos del depurador de Python aquí: [https://docs.python.org/2/library/pdb.html#debugger-commands (https://docs.python.org/2/library/pdb.html#debugger-commands). 


**Desarrollar desde la estación de trabajo**

Puede ejecutar Odoo con un sistema Debian/Ubuntu, en una máquina virtual local o en un servidor remoto. Pero posiblemente prefiera hacer el trabajo de desarrollo en su estación de trabajo personal, usando su editor de texto o IDE favorito. 

Este puede ser el caso para las personas que desarrollan en estaciones de trabajo con	Windows. Pero puede también ser el caso para las personas que usan Linux que necesiten trabajar en un servidor Odoo desde una red local. 

Una solución para esto es habilitar el uso compartido de archivos en el servidor Odoo, así los archivos son fáciles de editar desde su estación de trabajo. Para las operaciones del servidor Odoo, como reiniciar el servidor, es posible usar un intérprete de comando SSH (como PUTTY en Windows) junto a su editor favorito. 


**Usar un editor de texto Linux**

Tarde o temprano, será necesario editar archivos desde la línea de comandos. En muchos sistemas Debian el editor de texto predeterminado es vi. Si no se siente a gusto con este, puede usar una alternativa más amigable. En sistemas Ubuntu el editor de texto predeterminado es nano. Puede que prefiera usar este ya que es más fácil de usar. En caso que no este disponible en su servidor, puede instalarlo con: 
```
$ sudo apt-get install nano  
```
En las siguientes secciones se asumirá como el editor de preferencia. Si prefiere cualquier otro editor, siéntase libre de adaptar los comandos de acuerdo a su elección. 


**Instalar y configurar Samba**

El proyecto Samba proporciona a Linux servicios para compartir archivos compatibles con sistemas Microsoft Windows. Se puede instalar en el servidor Debian/Ubuntu con: 
```
$ sudo apt-get install samba samba-common-bin  
```
El paquete samba instala el servicio para compartir archivos y el paquete `samba-common-bin` es necesario para la herramienta smbpasswd. De forma predeterminada los usuarios autorizados para acceder a los archivos compartido necesitan ser registrados. Es necesario registrar el usuario odoo y asignarle una contraseña para su acceso a los archivos compartidos: 
```
$ sudo smbpasswd -a odoo  
```
Después de esto el usuario odoo podrá acceder a un recurso compartido de archivos para su directorio home, pero será de solo lectura. Se requiere el acceso a escritura, así que es necesario editar los archivos de configuración de Samba para cambiar eso: 
```
$ sudo nano /etc/samba/smb.conf  
```
En el archivo de configuración, busque la sección `[homes]`. Edite las líneas de configuración para que sean iguales a los siguientes ajustes: 
```
[homes] 			
comment	= Home Directories	
browseable	= yes 			
read	only	= no 			
create mask = 0640 			
directory mask = 0750 
```
Para que estos cambios en la configuración tengan efecto, reinicie el servicio: 
```
$ sudo /etc/init.d/smbd restart  
```


**Habilitar las herramientas técnicas**

Odoo incluye algunas herramientas que son muy útiles para las personas que desarrollan, y se usaran a lo largo de este libro. Estas son las Características Técnicas y el Modo de Desarrollo. 

Estas están deshabilitadas de forma predeterminada, así que este es algún momento para aprender como habilitarlas. 

![70_1](/images/Odoo Development Essentials - Daniel Reis-70_1.jpg)


**Activar las Características Técnicas**

Las Características Técnicas proporcionan herramientas avanzadas de configuración del servidos. 

Estas están deshabilitadas de forma predeterminada, y para habilitarlas, es necesario acceder con el usuario administrador. En el menú Configuraciones, seleccione Usuarios y edite el usuario Administrador. En la pestaña Derechos de Acceso, encontrará una casilla de selección de Características Técnicas . Seleccionala y guarde los cambios. 

Ahora es necesario recargar la página en el navegador web. Deberá poder ver en el menú	Configuraciones  una nueva sección	Técnico  que da acceso a lo interno del servidor Odoo. 

La opción del menú Técnico permite inspeccionar y editar todas las configuraciones de Odoo almacenadas in la base de datos, desde la interfaz de usuario, la seguridad y otros parámetros del sistema. Aprenderá mas sobre esto a lo largo del libro. 

![71_1](/images/Odoo Development Essentials - Daniel Reis-71_1.jpg)


**Activar el modo de Desarrollo**

El modo de Desarrollo habilita una caja de selección cerca de la parte superior de la ventana Odoo, haciendo accesible algunas opciones de configuración avanzadas en toda la aplicación. También deshabilita la modificación del código JavaScript y CSS usado por el cliente web, haciendo mas fácil la depuración del comportamiento del lado del cliente. 

Para habilitarlo, abra el menú desplegable en la esquina superior derecha de la ventana del navegador, al lado del nombre de usuario, y seleccione la opción	Acerca de Odoo. En la ventana de dialogo Acerca de, haga clic sobre el botón Activar el modo de desarrollo en la esquina superior derecha. 

Luego de esto, verá una caja de selección Vista de Depuración  en la parte superior izquierda del área actual del formulario. 


**Instalar módulos de terceras partes**

Hacer que nuevos módulos estén disponibles en una instancia de Odoo para que puedan ser instalados es algo que puede resultar confuso para las personas nuevas. Pero no necesariamente tiene que ser así, así que a continuación se desmitificará esta suposición. 


**Encontrar módulos de la comunidad**

Existen muchos módulos para Odoo disponibles en Internet. El sitio web [http://apps.odoo.com](	apps.odoo.com) es un catalogo de módulos que pueden ser descargados e instalados. La Odoo Community Association (OCA) coordina las contribuciones de la comunidad y mantiene unos pocos repositorios en GitHub,	en [https://github.com/OCA/] (https://github.com/OCA)

Para agregar un módulo a la instalación de Odoo puede simplemente copiarlo dentro de el directorio de complementos, junto a los módulos oficiales. En este caso, el directorio de complementos esta en	`~/odoo-dev/odoo/addons/`. Esta puede que no sea la mejor opción para Ud., debido a que su instalación esta basada en una versión controlada por el repositorio, y querrá tenerla sincronizada con el repositorio de GitHub. 

Afortunadamente, es posible usar ubicaciones adicionales para los módulos, por lo que se puede tener los módulos personalizados en un directorio diferente, sin mezclarlos con los complementos adicionales. 

Como ejemplo, se descargará el proyecto de departamento OCA y sus módulos se harán disponibles en la instalación de Odoo. Este proyecto es un conjunto de módulos muy simples que agregan un campo Departamento en muchos formularios, como en el de Proyectos u Oportunidades de CRM. 

Para obtener el código fuente desde	GitHub: 
```
$ cd ~/odoo-dev
$ git clone https://github.com/OCA/department.git -b 8.0  
```
Se uso la opción `-b` para asegurar que se descargan los módulos de la versión 8.0. 

Pero debido a que en el momento de escribir esto la versión 8.0 en la rama predeterminada del proyecto la opción `-b` podría haber sido omitida. 

Luego, se tendrá un directorio `/department` nuevo junto al directorio `/odoo`, que contendrá los módulos. Ahora es necesario hacer saber a Odoo sobre este nuevo directorio. 


**Configurar la ruta de complementos**

El servidor Odoo tiene una opción llamada ruta de complementos que define donde buscar los módulo. De forma predeterminada este apunta al directorio `/addons` del servidor Odoo que se esta ejecutando. 

Afortunadamente, es posible asignar no uno, si no una lista de directorios donde se pueden encontrar los módulos. Esto permite mantener los módulos personalizados en un directorio diferente, sin mezclarlos con los complementos oficiales. 

Se ejecutará el servidor con una ruta de complemento incluyendo el nuevo directorio de módulos: 
```
$ cd ~/odoo-dev/odoo
$ ./odoo.py	-d v8dev --addons-path=”/department,./addons”  
```
Si se observa en profundidad el registro del servidor notará una línea reportando la ruta de los complementos en uso: 
```
INFO ? Openerp: addons paths: (…). Confirm that it contains	 our department directory. 
```
![76_1](/images/Odoo Development Essentials - Daniel Reis-76_1.jpg)


**Actualizar la lista de módulos**

Es necesario pedirle a Odoo que actualice su lista de módulos antes que estos módulos nuevos estén disponibles para ser instalados. 

Para esto es necesario habilitar el menú Técnico , debido a que esta provee la opción de menú Actualizar Lista de Módulos . Esta puede ser encontrada en la sección Módulos en el menú Configuraciones . 

Luego de ejecutar la actualización de la lista de módulos se puede confirmar que los módulos nuevos están disponibles para ser instalados. En la lista de Módulos Locales , quite el filtro de Aplicaciones y busque por departamento. Debería poder ver los módulos nuevos disponibles. 


**Resumen**

En este capítulo, aprendió como configurar un sistema Debian para alojar Odoo e instalarlo desde GitHub. También aprendió como crear bases de datos en Odoo y ejecutar instancias Odoo. Para permitir que las personas que desarrollan usen sus herramientas favoritas en sus estaciones de trabajo, se explicó como configurar archivos compartidos en el servidor Odoo. 

En estos momentos debería tener un ambiente Odoo completamente funcional para trabajar con y cómodamente con el manejo de bases de datos e instancias. 

Con esto claro, es momento de ir directo a la acción. En el próximo capítulo se creará el primer modulo Odoo y entenderá los elementos principales involucrados. 

Comencemos! 