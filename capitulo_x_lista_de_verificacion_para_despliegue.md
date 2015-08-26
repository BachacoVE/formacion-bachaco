Capítulo 10. Lista de Verificación para Despliegue – En Vivo!
===

En este capítulo, aprenderá como preparar su servidor Odoo para usarlo en un entorno de producción.

Existen varias estrategias y herramientas posibles que pueden usarse para el despliegue y gestión de un servidor de producción Odoo. Le guiaremos a través de una de estas formas de hacerlo.

Esta es la lista de verificación para la configuración que seguiremos:

- Instalar Odoo desde la fuente.
- Crear archivo de configuración de Odoo.
- Configuración de multiproceso de trabajos
- Configuración del servicio del sistema Odoo
- Configuración de un proxy inverso (reverse proxy) con soporte SSL

Comencemos. 
 

**Instalar Odoo**

Odoo tiene paquetes disponibles para su instalación en sistemas Debian/Ubuntu. Así, se obtiene un servidor que se comienza a funcionar automáticamente cuando el sistema se inicia. Este proceso de instalación es bastante directo, y puede encontrar todo lo que necesita en [http://nightly.odoo.com](http://nightly.odoo.com).

Aunque esta es una forma fácil y conveniente de instalar Odoo, aquí preferimos ejecutar una versión controlada desde el código fuente, debido a que proporciona mejor control sobre lo que se esta desplegando.


**Instalación desde el código fuente**

Tarde o temprano, su servidor necesitará actualizaciones y parches. Una versión controlada por repositorio puede ser de gran ayuda cuando estos momentos lleguen.

Usamos git para obtener el código desde un repositorio, como hicimos para instalar nuestro entorno de desarrollo. Por ejemplo:
```
$ git clone https://github.com/odoo/odoo.git -b 8.0 --depth=1  
```
Este comando obtiene el código fuente de la rama 8.0 desde GitHub dentro del subdirectorio `odoo/`. En el momento de escribir esto, la 8.0 es la rama predeterminada, por la tanto la opción `-b 8.0` es opcional. La opción `--depth=1` se usa para obtener una copia superficial del repositorio, sin toda la historia de la versión. Esto reduce el espacio en disco usado y hace que el proceso de clonación sea más rápido.

Puede valer la pena tener una configuración un poco más sofisticada, con un entorno de prueba junto al entorno de producción.

Con esto, podríamos traernos la última versión de código fuente y probarlo en el entorno de prueba, sin perturbar el entorno de producción. Cuando la nueva versión este lista, podemos desplegarla desde el entorno de pruebas a producción.

Consideremos que el repositorio en `~/odoo-dev/odoo` será nuestro entorno de pruebas. Fue clonado desde GitHub, por lo tanto un `git pull` dentro de este traerá y mezclara los últimos cambios. Pero el mismo es un repositorio, y podemos clonarlo desde nuestro entorno de producción, como se muestra en el siguiente ejemplo: 
```
$ mkdir ~/odoo-prd && cd ~/odoo-prd 
$ git clone ~/odoo-dev/odoo ~/odoo-prd/odoo/  
```
Esto creará el repositorio de producción en `~/odoo-prd/odoo` clonado desde en entorno de prueba `~/odoo-dev/odoo`. Se configurara para que monitorear este repositorio, por lo que un `git pull` dentro de producción traerá y mezclara las últimas versiones desde pruebas. Git es lo suficientemente inteligente para saber que esto es una clonación local y usar enlaces duros al repositorio padre para ahorrar espacio en disco, por lo tanto la opción `--depth` no es necesaria.

Cuando sea necesario resolver un problema en el entorno de producción, podemos verificar en el entorno de prueba la versión del código de producción, y depurar para diagnosticar y resolver el problema sin tocar el código de producción. Luego, el parche de la solución puede ser entregado al historial Git de prueba, y desplegado al repositorio de producción usando un comando `git pull`.

*Nota*
*Git sera una herramienta invaluables para gestionar las versiones de los despliegues de Odoo. Solo hemos visto la superficie de lo que puede hacerse para gestionar las versiones de código. Si aun no conoce Git, vale la pena aprender más sobre este. Un buen sitio para comenzar es [http://git-scm.com/doc](http://git-scm.com/doc).* 
 

**Crear el archivo de configuración**

Al agregar la opción `--save` cuando se inicia un servidor Odoo almacena la configuración usada en el archivo `~/.openerp_serverrc`. Podemos usar este archivo como punto de partida para nuestra configuración del servidor, la cual será almacenada en `/etc/odoo`, como se muestra a continuación:
```
$ sudo mkdir /etc/odoo
$ sudo chown $(whoami) /etc/odoo 
$ cp ~/.openerp_serverrc /etc/odoo/openerp-server.conf  
```
Este tendrá los parámetros de configuración para ser usados en nuestra instancia del servidor. Los siguientes son los parámetros para que el servidor trabaje correctamente:

- addons_path: Es una lista separada por coma de las rutas de directorios donde se buscarán los módulos, usando los directorios de izquierda a derecha. Esto significa que los directorios mas a la izquierda tienen mayor prioridad.

- xmlrpc_port: Es el número de puerto en el cual escuchara el servidor. De forma predeterminada es el puerto 8069.

- log_level: Este es la cantidad de información en el regirstro. De forma predeterminada es el nivel “info”, pero al usar el nivel “debug_rpc”, más descriptivo, agrega información importante para el monitoreo del desempaño del servidor.


Las configuraciones siguientes también son importantes para una instancia de producción:

- admin_passwd: Es la contraseña maestra para acceder a las funciones de gestión de base de datos del cliente web. Es importante fijarlo con una contraseña segura o con un valor vacío para desactivar la función.

- dbfilter: Es una expresión regular interpretada por Python para filtrar la lista de base de datos. Para que no sea requerido que el usuario o la usuaria seleccione una base de datos, debe fijarse con `^dbname$`, por ejemplo, `dbfilter = ^v8dev$`.

- `logrotate = True`: Divide el registro en archivos diarios y mantendrá solo un historias de registro mensual.

- data_dir: Es la ruta donde son almacenados los archivos adjuntos. Recuerde tener respaldo de estos.

- `withput_demo = True`: Se fija en los entornos de producción para que las bases de datos nuevas no tengan datos de demostración.

Cuando se usa un proxy inverso (reverse proxy), se deben considerar las siguientes configuraciones:

- `proxy_mode = True`: Es importante fijarlo cuando se usa un proxy inverso.

- xmlrpc-interface: Este fija las direcciones que serán escuchadas. De forma predeterminada escucha todo 0.0.0.0, pero cuando se usa un proxy inverso, puede configurarse a 127.0.0.1 para responder solo a solicitudes locales.

Se espera que una instancia de producción gestione una carga de trabajo significativa. De forma predeterminada, el servidor ejecuta un proceso y es capaz de gestionar solo una solicitud al mismo tiempo. De todas maneras, el modo multiproceso esta disponible para que puedan gestionarse solicitudes concurrentes. 

La opción `workers=N` fija el número de procesos de trabajo que serán usados. Como guía puede intentar fijarlo a `1+2*P` donde P es el número de procesos. Es necesario afinar la mejor configuración para cada caso, debido a que depende de la carga del servidor y que otros servicios son ejecutados en el servidor (como PostgreSQL).

Podemos verificar el efecto de las configuraciones ejecutando el servidor con la opción `-c` o `--config` como se muestra a continuación:
```
$ ./odoo.py -c /etc/odoo/openerp-server.conf 
```


**Configurar como un servicio del sistema**

Ahora, queremos configurar Odoo como un servicio del sistema y que sea ejecutado automáticamente cuando el sistema sea iniciado.

El código fuente de Odoo incluye un script de inicio, usado para las distribuciones Debian. Podemos usarlo como nuestro script de inicio con algunas modificaciones menores, como se muestra a continuación:
```
$ sudo cp ~/odoo-prd/odoo/debian/init /etc/init.d/odoo 
$ sudo chmo +x /etc/init.d/odoo  
```
En este momento, quizás quiera verificar el contenido del script de inicio. Los parámetros claves son a variables al inicio del archivo. A continuación se muestra un ejemplo:
```
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin 
DAEMON=/usr/bin/openerp-server 
NAME=odoo 
DESC=odoo 
CONFIG=/etc/odoo/openerp-server.conf 
LOGFILE=/var/log/odoo/odoo-server.log 
PIDFILE=/var/run/${NAME}.pid 
USER=odoo 
```
La variable USER es el usuario del sistema bajo el cual se ejecutara el servidor, y probablemente quiera cambiarlo. Las otras variables deberían ser las correctas y prepararemos el resto de la configuración teniendo en mente estos valores predeterminados. DEAMON es la ruta a el ejecutable del servidor, CONFIG es el archivo de configuración que será usado, y LOGFILE es la ubicación del archivo de registro.

Los ejecutables en DEAMON pueden ser un enlace simbólico a nuestra ubicación actual de Odoo, como se muestra a continuación:
```
$ sudo ln -s ~/odoo-prd/odoo/odoo.py /usr/bin/openerp-server 
$ sudo chown $(whoami) /usr/bin/openerp-server  
```
Luego debemos crear el directorio LOGFILE como sigue:
```
$ sudo mkdir /var/log/odoo
$ sudo chown $(whoami) /etc/odoo  
```
Ahora deberíamos poder iniciar y parar el servicio de Odoo:
```
$ sudo /etc/init.d/odoo start 
Starting odoo: ok  
```
Deberíamos ser capaces de obtener una respuesta del servidor sin ningún error en la archivo de registro, como se muestra a continuación:
```
$ curl http://localhost:8069 <html><head><script>window.location	= '/web' + location.hash;</script> </head></html> 
$ less /var/log/odoo/odoo-server.log     # show the log file  
```
La parada del servicio se hace de forma similar:
```
$ sudo /etc/init.d/odoo stop 
Stopping odoo: ok  
```

*Tip*
*Ubuntu proporciona el comando más fácil de recordar para gestionar los servicios, si lo prefiere puede usar `sudo service odoo start` y `sudo service odoo stop`. *

Ahora solo necesitamos que el servicio se ejecute automáticamente cuando se inicia el sistema:
```
$ sudo update-rc.d odoo defaults  
```
Luego de esto, al reiniciar el servidor, el servicio de Odoo debería comenzar a ejecutarse automáticamente son errores. Es un buen momento para verificar que todo este funcionando como se espera. 


**Usar un proxy inverso**

Mientras que Odoo puede entregar páginas web por si mismo, es recomendable usar un proxy inverso delante de Odoo. Un proxy inverso actúa como un intermediario que gestiona el tráfico entre los clientes que envían solicitudes y el servidor Odoo que responde a esas solicitudes. Usar un proxy inverso tiene múltiples beneficios.

De cara a la seguridad, puede hacer lo  siguiente:

- Gestionar (y reforzar) los protocolos HTTPS para cifrar el tráfico.
- Esconder las características internas de la red.
- Actuar como un “aplicación firewall” limitando el número de URLs aceptados para su procesamiento.

Y del lado del desempeño, puede proveer mejoras significativas:

- Contenido estático cache, por lo tanto reduce la carga en los servidores Odoo.
- Comprime el contenido para acelerar el tiempo de carga.
- Balancea la carga distribuyendo la entre varios servidores.

Apache es una opción popular que se usa como proxy inverso. Nginx es una alternativa reciente con buenos argumentos técnicos. Aquí usaremos nginx como proxy inverso y mostraremos como puede usarse para ejecutar las funciones mencionadas anteriormente.


**Configurar nginx como proxy inverso**

Primero, debemos instalar nginx. Queremos que escuche en los puertos HTTP predeterminados, así que debemos asegurarnos que no estén siendo usados por otro servicio. Ejecutar el siguiente comando debe arrojar un error, como se muestra a continuación:
```
$ curl http://localhost 
curl:	(7) Failed to connect to localhost port 80  
```
De lo contrario, deberá deshabilitar o eliminar ese servicio para permitir que nginx use esos puertos. Por ejemplo, para parar un servidor Apache existente, deberá hacer lo siguiente:
```
$ sudo /etc/init.d/apache2 stop  
```
Ahora podemos instalar nginx, lo cual es realizado de la forma esperada:
```
$ sudo apt-get install nginx  
```
Para conformar que este funcionando correctamente, deberíamos ver una página que diga “Welcome to nginx” cuando se ingrese la dirección del servidor en la navegador o usarndo `curl http://localhost`

Los archivos de configuración de nginx siguen el mismo enfoque que los de Apache: son almacenados en `/etc/nginx/available-sites/` y se activan agregando un enlace simbólico en `/etc/nginx/enabled-sites/`. Deberíamos deshabilitar la configuración predeterminada que provee la instalación de nginx, como se muestra a continuación:
```
$ sudo rm /etc/nginx/sites-enabled/default 
$ sudo touch /etc/nginx/sites-available/odoo 
$ sudo ln -s /etc/nginx/sites-available/odoo /etc/nginx/sites-enabled/odoo  
```
Usando un editor, como nano o vi, editamos nuestros archivo de configuración nginx como sigue:
```
$ sudo nano /etc/nginx/sites-available/odoo 
```
Primero agregamos los “upstreams”, los servidores traseros hacia los cuales nginx redireccionara el tráfico, en nuestro caso el servidor Odoo, el cual escucha en el puerto 8069, como se muestra a continuación:
```
upstream backend-odoo {
    server 127.0.0.1:8069; 
} 

server {
    location / {
        proxy_pass http://backend-odoo;
    } 
} 
```
Para probar que la configuración es correcta, use lo siguiente:
```
$ sudo nginx -t  
```
En caso que se encuentren errores, verifique que el archivo de configuración esta bien escrito. Además, un problema común es que el HTTP este tomado de forma predeterminada por otro servicio, como Apache o la página web predeterminada de nginx. Realice una doble revisión de las instrucciones dadas anteriormente para asegurarse que este no sea el caso, luego reinicio nginx. Luego de esto, podremos hacer que nginx cargue la nueva configuración: 
``` 
$ sudo /etc/init.d/nginx reload  
```
Ahora podemos verificar que nginx este redirigiendo el tráfico al servidor de Odoo, como se muestra a continuación:
```
$ curl http://localhost <html><head><script>window.location = '/web' + location.hash;</script> </head></html>  
``` 


**Reforzar el HTTPS**

Ahora, deberíamos instalar un certificado para poder usar SSL. Para crear un certificado auto-firmado, siga los pasos a continuación:
```
$ sudo mkdir /etc/nginx/ssl && cd /etc/nginx/ssl 
$ sudo openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem – days 365 -nodes 
$ sudo chmod a-wx *                     # make files read only 
$ sudo chown www-data:root *            # access only to www-data group  
```
Esto crea un directorio `ssl/` dentro del directorio `/etc/nginx/` y un certificado auto-firmado sin contraseña. Cuando se ejecute el comando openssl, se solicitara más información, y se generaran un certificado y archivos llave. Finalmente, estos archivos serán propiedad del usuario www-data, usado para ejecutar el servidor web.  

*Nota*
*Usar un certificado auto-firmado puede plantear algunos riesgos de seguridad, como ataques “man-in-the-middle”, y pueden no ser permitidos por algunos navegadores. Para una solución más robusta, debe usar un certificado firmado por una autoridad de certificación reconocida. Esto es particularmente importante si se esta ejecutando un sitio web comercial o de e-commerce. *

Ahora que tenemos un certificado SSL, podemos configurar nginx para usarlo. 

Para reforzar HTTPS, redireccionaremos todo el tráfico HTTP. Reemplace la directiva “server” que definimos anteriormente con lo siguiente:
```
server {
    listen 80; 
    add_header Strict-Transport-Security max-age=2592000;
    rewrite ^/.*$ https://$host$request_uri? permanent; 
} 
```
Si recargamos la configuración de nginx y accedemos al servidor con el navegador web, veremos que la dirección `http://` se convierte en `https://`.

Pero no devolverá ningún contenido antes que configuremos el servicio HTTPS apropiadamente, agregando la siguiente configuración a “server”:
```
server {
    listen 443 default;              
    # ssl settings
    ssl on;
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    keepalive_timeout 60;
    # proxy header and settings
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forward-For $proxy_add_x_forwarded_for; 		
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_redirect off; 	 
 
    location / {
        proxy_pass http://backend-odoo;
    } 
} 
```
Esto escuchara al puerto HTTPS y usará los archivos del certificado `/etc/nginx/ssl/` para cifrar el tráfico. También agregamos alguna información al encabezado de solicitud para hacer que el servicio de Odoo sepa que esta pasando a través de un proxy. Por razones de seguridad, es importante para Odoo asegurarse que el parámetro `proxy_mode` este fijado a True. Al final, la directiva “location” define que todas las solicitudes sean pasadas al upstream “backend-oddo”.

Recargue la configuración, y deberíamos poder tener nuestro servicio Odoo trabajando a través de HTTPS, como se muestra a continuación:
```
$ sudo nginx -t 
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok 
nginx: configuration file /etc/nginx/nginx.conf	test is successful 
$ sudo service nginx reload * 
Reloading nginx configuration nginx ...done. 
$ curl -k https://localhost  
<html><head><script>window.location	= '/web' + location.hash;</script></head></html>
```
La última salida confirma que el cliente Odoo esta siendo servido sobre HTTPS.


**Optimización de Nginx**

Es hora para algunas mejoras en las configuraciones de nginx. Estas son recomendadas para habilitar el búfer de respuesta y compresión de datos que debería mejorar la velocidad del sitio web. También fijamos una localización específica para los registros.

Las siguientes configuraciones deberían ser agregadas dentro de “server” que escucha en el puerto 443, por ejemplo, justo despues de las definiciones del proxy:
```
# odoo log files access_log /var/log/nginx/odoo-access.log;
error_log /var/log/nginx/odoo-error.log; 
# increase proxy buffer size 
proxy_buffers 16 64k;
proxy_buffer_size 128k; 
# force timeouts if the backend dies
proxy_next_upstream error timeout invalid_header http_500 http_502 http_503; 
# enable data compression 
gzip on; 
gzip_min_length 1100; 
gzip_buffers 4 32k;
gzip_types text/plain application/x-javascript text/xml text/css;
gzip_vary on; 
```
También podemos activar el caché de contenido para respuestas más rápidas para los tipos de solicitudes mencionados en el código anterior y para impedir su carga en el servidor Odoo. Después de la sección `location /`, agregue una segunda sección “location”:
```
location ~* /web/static/ {
    # cache static data
    proxy_cache_valid 200 60m;
    proxy_buffering on;
    expires 864000;
    proxy_pass http://backend-odoo;
} 
```
Con esto, se hace caché de los datos estáticos por 60 minutos. Las solicitudes siguientes de esas solicitudes en este intervalo de tiempo serán respondidas directamente por nginx desde el caché.


**Long polling**

“Long polling” es usada para soportar la aplicación de mensajería instantánea, y cuando se usan trabajos multiproceso, esta es gestionada en un puerto separado, el cual de forma predeterminada es el puerto 8072.

Para nuestro proxy inverso, esto significa que las solicitudes “longpolling” deberían ser pasadas por este puerto. Para soportar esto, necesitamos agregar un nuevo “upstream” a nuestra configuración nginx, como se muestra en el siguiente código:
```
upstream backend-odoo-im { server 127.0.0.1:8072; } 
```
Luego, deberíamos agregar otra “location” al “server” que gestiona las solicitudes HTTPS, como se muestra a continuación:
```
location /longpolling { proxy_pass http://backend-odoo-im; } 
```
Con estas configuraciones, nginx debería pasar estas solicitudes al puerto apropiado del servidor Odoo.
 

**Actualización del servidor y módulos**

Una vez que el servidor Odoo este listo y ejecutándose, llegara el momento en que necesite instalar actualizaciones. Lo cual involucra dos pasos: primero, obtener las nuevas versiones del código fuente (servidor o módulos), y segundo, instalar las. 

Si ha seguido el enfoque descrito en la sección * Instalación desde el código fuente *, podemos buscar y probar las nuevas versiones dentro del repositorio de preparación. Es altamente recomendable hacer una copia de la base de datos de producción y probar la actualización en ella. Si `v8dev` es nuestra base de datos de producción, esto podría ser realizado con los siguientes comandos:
```
$ dropdb v8test ; createdb v8test 
$ pg_dump v8dev | psqlpsql -d v8test 
$ cd ~/odoo-dev/odoo/ 
$ ./odoo.py -d v8test –xmlrpc-port=8080 -c /etc/odoo/openerp-server.conf –u all  
```
Si todo resulta bien, debería ser seguro realizar la actualización en el servicio en producción. Recuerde colocar una nota de la versión actual de referencia Git, con el fin de poder regresar, revisando esta versión otra vez. Hacer un respaldo de la base de datos antes de realizar la actualización es también recomendable.

Luego de esto, podemos hacer un “pull” de las nuevas versiones al repositorio de producción usando Git y completando la actualización, como se muestra aquí:
```
$ cd ~/odoo-prd/odoo/
$ git pull 
$ ./odoo.py -c /etc/odoo/openerp-server.conf –stop-after-init -d v8dev -u all 
$ sudo /etc/init.d/odoo restart 
``` 


**Resumen**

En este capítulo, aprendió sobre los pasos adicionales para configurar y ejecutar Odoo en un servidor de producción basado en Debian. Fueron vistas las configuraciones más importantes del archivo de configuración, y aprendió como aprovechar el modo multiproceso.

También aprendió como usar nginx como un proxy inverso frente a nuestro servidor Odoo, para mejorar la seguridad y la escalabilidad.

Esperamos que esto cubra lo esencial de lo que es necesario para ejecutar un servidor Odoo y proveer un servicio estable y seguro a sus usuarios y usuarias.
