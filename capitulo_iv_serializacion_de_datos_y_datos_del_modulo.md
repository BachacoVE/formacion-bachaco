Capítulo 4.	Serialización de Datos y Datos de Módulos
====

La mayoría de las configuraciones de Odoo, desde interfaces de usuario hasta reglas de seguridad, son en realidad registros de datos almacenados en tablas internas de Odoo. Los archivos XML y CSV que se encuentran en los módulos no son usados para ejecutar aplicaciones Odoo. Ellos solo son un medio para cargar esas configuraciones a las tablas de la base de datos.

Los módulos pueden también tener datos iniciales y de demostración (accesorios). La serialización de datos permite añadir eso a nuestros módulos. Adicionalmente, entendiendo los formatos de serialización de datos de Odoo es importante para exportar e importar datos en el contexto de la implementación de un proyecto.

Antes de entrar en casos prácticos, primero exploraremos el concepto de identificador externo, el cual es la clave a la serialización de datos de Odoo 

![150_1](/images/Odoo Development Essentials - Daniel Reis-150_1.jpg)

**Entendiendo los Identificadores Externos**

Todos los registros en la base de datos de Odoo tienen un identificador único, el campo `id`

Es un número sequencial asignado automáticamente por la base de datos. De cualquier forma, este identificador automático puede ser un desafío al momento de cargar datos interrelacionados: ¿cómo podemos hacer referencia a un registro relacionado si no podemos saber de antemano cual ID de base de datos le será asignado?

La respuesta de Odoo a esto es el identificador externo. Los identificadores externos solucionan este problema asignando indentificadores con nombre a los registros de datos que serán cargados. Un identificador con nombre puede ser usado por cualquier otra pieza de dato registrada para referenciarla luego. Odoo se encargará de traducir estos nombres de identificación a los IDs reales asignados a ellos.

El mecanismo detrás de esto es muy simple: Odoo mantiene una tabla con el mapeo entre los IDs externos con nombre y sus correspondiente IDs numéricos en la base de datos. Ese es el modelo `ir.model.data`.

Para inspeccionar la existencia de mapeo, dirijase a la sección **Tecnico** en el menu **Configuración**, y seleccione **Secuencias e identificadores**	| el item de menu **Identificadores externos**. 

Por ejemplo, si volvemos a visitar la lista de identificadores externos	y filtramos por el modulo	`todo_app`, veremos los identificadores externos creados previamente por el modulo.

Puede notar que los identificadores externos tienen una etiqueta ID completa. Esta compuesta por el nombre del modulo y el nombre de identificador unido por un punto, por ejemplo, `todo_app.action_todo_task`.

Debido a que solo es obligatorio que el ID completo sea único, el nombre del módulo sirve como namespace para los identificadores. Esto significa que el mismo identificador puede repetirse en diferentes módulos, y no tenemos que preocuparnos por identificadores en nuestro módulo que colicionen con identificadores en otros módulos.

![151_1](/images/Odoo Development Essentials - Daniel Reis-151_1.jpg)

Al principio de la lista, puedes ver el ID de `todo_app.action_todo_task`. Esta es la accción del menú que creamos para el módulo, el cual también es referenciado en el elemento de menú correspondiente. Haciendo clic en él, puede abrir un formulario con sus detalles: el `action_todo_task` en el módulo `todo_app` mapea hacia un ID de un registro específico en el modelo `ir.actions.act_window`.
 
Ademas de proporcionar una forma de hacer referencia a un registro de una manera facil a otros registros, los	ID externos	tambien permiten evitar la duplicidad de datos en las importaciones repetidas. Si el ID externo esta presente, el registro existente se actualiza,	en ves de crear un nuevo registro.	Esta es la razón de porque, en las siguientes actualizaciones del modulo,	los resgistros cargados previamente se actualizaran en lugar de duplicarse. 


**Encontrar identificadores externos**

Cuando preparamos archivos de datos para la demostracion y configuracion del modulo, tenemos que mirar frecuentemente la existencia de IDs Externos	que se necesitan para realizar las referencias. 

Podemos utilizar identificadores externos en el menu mostrado anteriormente, pero el **menu de Desarrollo** puede proporcionar un metodo que sea mas conveniente.	Como recordara en el Capítulo 1, 	*Comenzando con el Desarrollo en Odoo*,	el **menu de Desarrollo** es activado en la opción **Acerca de Odoo**, y entonces, estara disponible en la esquina supeior izquierda de la vista del cliente web.
 
Para buscar el ID externo para un registro de datos,	en el mismo formulario correspondiente,	seleccione la opcion **Ver metadatos** desde el **menu de Desarrollador**.	Esto mostrara un cuadro de dialogo con el ID de la base de datos registrado y el ID externo (tambien conocido como ID XML) 

Como Ejemplo, para buscar el ID del usuario Demo, podemos navegar hasta la vista de formulario	(**Configuración**	| **Usuarios** )	y seleccionar la opcion **Ver metadatos**,	y se mostrara lo siguiente: 

![152_1](/images/Odoo Development Essentials - Daniel Reis-152_1.jpg)

Para buscar el ID externo de los elementos de vista,	como formulario,	árbol,	buscador,	y acción,	el **menu de Desarrollo** es tambien de ayuda.	Para esto, utilize la opción de **Obtener Campos de Vista**	o abra la información	para la vista deseada usando la opción Editar	<tipo de vista> ,	y seleccione la opcion Ver metadatos. 
 

**Exportar e importar datos**  

Vamos a empezar a trabajar en la exportacion e importacion de datos en Odoo,	y desde alli,	vamos a pasar a los detalles técnicos. 


**Exportando datos**

Exportar datos es una caracteristica estándar disponible en cualquier vista de lista.	Para usarlo,	primero tenemos que seleccionar la fila de exportacion seleccionando las casillas de verificacion correspondiente en el extremo izquierdo,	y luego seleccionamos la opcion exportar en el boton "más". 

Aqui esta un ejemplo,	utilizando las reciente tareas creadas a realizar: 

![155_1](/images/Odoo Development Essentials - Daniel Reis-155_1.jpg)

La opcion exportar nos lleva a un dialogo,	donde podemos elegir lo que se va a exportar.	La opcion exportar compatible se asegura de que el archivo exportado se pueda importar denuevo a Odoo. 

El formato de exportacion puede ser CSV	o	Excel.	Vamos a preferir archivos	CSV	para tener una mejor comprension del formato de exportacion.	Continuamos,	eligiendo las columnas que queremos exportar y hacer clic en el boton **Exportar a fichero**.	Esto iniciara la descarga de un archivo con los datos exportados. 
 
![156_1](/images/Odoo Development Essentials - Daniel Reis-156_1.jpg)

Si seguimos estas instrucciones y seleccionamos los campos que se demuestran en la imagen anterior,	debemos terminar con un archivo de texto CSV similar a este: 
```
"id","name","user_id/id","date_deadline","is_done" "__export__.todo_task_1","Install	Odoo","base.user_root","2015-01- 30","True" "__export__.todo_task_2","Create	dev	database","base.user_root","","False" 
```
Observe que Odoo exporta automaticamente una columna adicional identificada.	Este es un ID externo que se genera automaticamente para cada registro.	Estos identificadores externos generados utilizan `__export__` en lugar de un nombre real de módulo.	Nuevos identificadores solo se asignan a los que no poseen uno asignado,	y ya a partir de alli,	se mantienen unidos al mismos registro.	Esto significa que las exportaciones posteriores perserverán los mismos identificadores externos. 
 

**Importar datos**

Primero tenemos que asegurarnos que la funcion de importar este habilitada.	Esto se hace en el menu de **Configuración**,	**Configuración**	|	opcion de **Configuraciones Generales**.	En **Importar/Exportar**,	asegúrese que la opcion **Permitir a los usuarios importar datos desde archivos CSV** este habilidata. 

Con esta opcion habilitada,	los puntos de vista de la lista muestran la opcion de **Importar** junto al boton **Crear** en la parte superior de la lista. 

Vamos a realizar una edicion masiva en nuestros datos de tareas pendientes:	se abre en una hoja de calculo o en un editor de texto el archivo CSV que acabamos de descargar, a continuacion, cambie algunos valores y añada algunas nuevas filas. 

Como se menciono antes,la primera columna de identificacion proporciona un identificador unico para cada fila permitiendo registros ya existentes que se actualizaran en ves de duplicarse cuando importamos los datos de nuevo a Odoo.	Para las nuevas filas que podemos añadir al archivo CSV,	el `id` se deben dejar en blanco, y se creara un nuevo registro para ellos.

Despues de guardar los cambios en el archivo CSV,	haga clic en la opcion **Importar**	(junto al boton crear)	y se presentara el asistente de importacion.	Hay que seleccionar la ubicacion del archivo CSV en el disco y hacer clic en **Validar**  para comprobar si el formato es correcto.	Debido a que en archivo a importar esta basdo en una importación de Odoo, es probable que es archivo sea correcto.

![157_1](/images/Odoo Development Essentials - Daniel Reis-157_1.jpg)

Ahora podemos hacer clic en **Importar** y allí va:	nuestras modificaciones y nuevos registros deberian haberse cargado en Odoo. 
 

**Registros relacionados en archivos de datos CSV**

En el ejemplo visto anteriormente,	el usuario responsable de cada tarea es un registro relacionado en el modelo de los usuarios,	con la relacion many	to	one - muchos a uno - 	(o	foreign	key - clave foranea).	El nombre de la columna para ello fue `usuario_id/id` y los valores de los campos eran identificadores externos para los registros relacionados, tales como `base.user_root`	para el usuario administrador. 

Las columnas de relacion deben tener	`/id`	anexo a su nombre,	si se usan IDs externos,	o	`/.id`, si se usan IDs (númericos) de base de datos.	Alternativamente,	dos puntos	`(:)`	se puede utilizar en lugar de la barra para el mismo efecto. 

Del mismo modo,	la relacion many	to	many - muchos a muchos - son soportables. Un ejemplo de relacion many	to	many	es la que existe entre usuarios y grupos:	cada usuario puede estar en muchos grupos, y cada grupo puede tener muchos usuarios.	La columna nombre	para este tipo de campo deberia haber añadido un	`/id`.	Los valores de los campos aceptan una lista separada por comas de Id externos, entre comillas dobles. 

Por ejemplo,	los Seguidores de las tareas a realizar es una relacion	many-to-many	entre Tareas por hacer	y Socios.	El nombre de la columna puede ser `follower_ids/id`	y un valor de campo con dos seguidores podria ser: 
`"__export__.res_partner_1,__export__.res_partner_2"`
 
Finalmente,	las relaciones one	to	many tambien se pueden importar a traves de CSV.	El ejemplo tipico de esta relacion es un documento “head”	con varias	“lines”. 

Podemos ver un ejemplo de tal relacion en el modelo de empresa	(la vista de formulario esta disponible en el menu configuracion):	una empresa puede tener varias cuentas bancarias,	cada una con sus propios detalles, y cada cuenta bancaria pertenece a (tiene una relacion many-to-one con)	solo una empresa. 

Es posible importar las empresa junto con sus cuentas bancarias en un solo archivo.	Para esto, algunas columnas corresponderan a empresas,	y otras columnas corresponderan a cuentas bancarias detalladas.	Los nombres de columnas de los detalles del banco deben ser precedidos de los campos con la relacion	one-to-many que vincula a la empresa con los bancos;	`bank_ids`	en este caso. 

Los primeros datos de la cuenta bancaria van en la misma fila de los datos vinculados de la empresa.	Los detalles de la proxima cuenta bancaria van en la siguiente fila,	pero solo los datos bancarios de la columna relacionada deben tener valores;	La columna de datos de la empresa debe tener esas lineas vacias. 

Aqui esta un ejemplo cargando una empresa con datos de tres bancos: 
```
id,name,bank_ids/id,bank_ids/acc_number,bank_ids/state base.main_company,YourCompany,__export__.res_partner_bank_4,123456789,bank ,,__export__.res_partner_bank_5,135792468,bank 
,,__export__.res_partner_bank_6,1122334455,bank
```

Observe que las dos ultimas lineas comienzan con comas:	Esto corresponde a valores en las dos primeras columnas,	`id`	y `name`,	con respecto a los datos del encabezado de empresa.	Pero las columnas restantes, con respecto a las cuentas bancarias, tienen valores para el segundo y tercer registro del banco. 

Estos son los elementos esenciales en el trabajo con la exportacion e importacion en la GUI.	Es util para establecer los datos en nuevas instancias Odoo, o para prepara nuevos archivos de datos que se incluiran en los módulos Odoo. 
 
A continuacion vamos aprender mas sobre el uso de los archivos de datos en los módulos. 
 
  
**Modulos de datos**  

Los módulos utilizan archivos de datos para cargar sus configuraciones en la base de datos, los datos iniciales y los datos de demostración. Esto se puede hacer utilizando tanto CSV y archivos XML. Para completar, el formato de archivo YAML también se puede utilizar, pero esto rara vez se utiliza para la carga de datos, por lo tanto no lo discutiremos.

Los archivos CSV utilizados por módulos son exactamente los mismos que los que hemos visto y utilizado para la función de importación. Cuando se usa en módulos, la única restricción adicional es que el nombre del archivo debe coincidir con el nombre del modelo a la que se cargan los datos.

Un ejemplo común es el acceso de seguridad, para cargar en el modelo `ir.model.acess`. Esto se hace generalmente con archivos CSV, y que debe ser nombrado `ir.model.acess.csv`.

**Datos de demostración**

Los módulos Odoo pueden instalar datos de demostración. Esto es útil para proporcionar ejemplos de uso para un módulo y conjuntos de datos para ser utilizados en pruebas. Se considera una buena práctica para los módulos proporcionar datos de demostración. Los datos de demostración para un módulo se declara con el atributo `demo` del archivo de manifiesto	`__openerp__.py`. Al igual que el atributo `data`, se trata de una lista de nombres de archivo con las rutas relativas correspondientes en el interior del módulo.

Estaremos agregando los datos de demostración en nuestro modulo	`todo_user`.	Podemos comenzar con la exportación de algunos datos de las tareas a realizar, como se explico en la sección anterior. Luego debemos guardar los datos en el directorio	`todo_user`	con el nombre del archivo	`todo.task.csv`.	Dado que esta información será propiedad de nuestro módulo, debemos editar los valores de `id` para reemplazar el prefijo	`__export__`	en los identificadores con el nombre técnico del módulo.

Como ejemplo nuestro archivo de datos	`todo.task.csv` podría tener este aspecto:
``` 
id,name,user_id/id,date_deadline todo_task_a,"Install	Odoo","base.user_root","2015-01-30" todo_task_b","Create	dev	database","base.user_root","" 
```
No hay que olvidar agregar este archivo de datos en el atributo `demo` del	`__openerp__.py`: 
```				
'demo':	['todo.task.csv'], 
```
La próxima vez que actualizamos el módulo, siempre y cuando se haya instalado con los datos de demostración habilitados, se importará el contenido del archivo. Tenga en cuenta que estos datos se reescribiran cada vez que se realiza una actualización del módulo.

Los archivos XML también pueden ser utilizados para los datos de demostración. Sus nombres de archivo no están obligados a coincidir con el modelo a cargar, porque el formato XML es mucho más rico y la información es proporcionada por los elementos XML dentro del archivo.

Vamos a aprender más sobre lo que los archivos de datos XML nos permiten hacer y que los archivos CSV no.

**Archivos de datos XML**  

Mientras que los archivos CSV proporcionan un formato simple y compacto para serializar los datos, los archivos XML son más potentes y dan un mayor control sobre el proceso de carga.

Ya hemos utilizado los archivos de datos XML en los capítulos anteriores. Los componentes de la interfaz de usuario, tales como vistas y elementos de menú, se encuentran en los registros de datos almacenados en los modelos de sistemas. Los archivos XML en los módulos son un medio utilizado para cargar los registros en el servidor.

Para mostrar esto, vamos a añadir un segundo archivo de datos para el	modulo `todo_user`, llamado `todo_data.xml`,	con el siguiente contenido: 
```
<?xml	version="1.0"?>
    <openerp>
        <data>
            <!--	Data	to	load	-->
            <record	model="todo.task"	id="todo_task_c">
                <field	name="name">Reinstall	Odoo</field>
                <field	name="user_id"	ref="base.user_root"	/>
                <field	name="date_deadline">2015-01-30</field>
            </record>
        </data>
    </openerp> 
```
Este XML es equivalente al archivo de datos CSV que acabamos de ver en la sección anterior. 

Los archivos de datos XML tienen un elemento `<openerp>`	que contiene elementos	`<data>`, dentro de los cuales podemos tener tienen varios elementos `<record>`, correspondientes a las filas de datos CSV.

Un elemento	`<record>` tiene dos atributos obligatorios, `model` y `id` (el identificador externo para el registro), y contiene una etiqueta	`<field>` para cada campo de texto. 

Tenga en cuenta que la notación con barras en los nombres de campo no está disponible aquí: no podemos usar	`<field name="user_id/id">`.	En cambio, el atributo especial `ref` se utiliza para hacer referencia a los identificadores externos. Hablaremos de los valores para el campo relacional "a muchos" en un momento.
 

**El atributo de datos noupdate**

Cuando se repite la carga de datos, los registros existentes de la ejecución anterior se reescriben.
 
Esto es importante a tener en cuenta: significa que la actualización de un módulo se sobrepone a los cambios manuales que podrían haber sido realizados en los datos. Cabe destacar que, si las vistas fueron modificadas con personalizaciones, esos cambios se perderán con la próxima actualización del módulo. El procedimiento correcto es crear vistas heredadas de los cambios que necesitamos, como se explica en el Capítulo 3. 	

Este comportamiento de sobrescritura es el valor predeterminado, pero se puede cambiar, por lo que cuando un registro ya creado se carga de nuevo no se realiza ningún cambio al mismo. Esto se hace añadiendo al elemento `<data>`	un atributo	`noupdate="1"`. Con esto, sus registros se crearán la primera vez que se cargan, y en mejoras de módulos subsiguientes no se hará nada para ellos.

Esto permite que las personalizaciones realizadas manualmente esten a salvo de las actualizaciones del módulo. Se utiliza a menudo con las reglas de acceso de registro, lo que les permite adaptarse a las necesidades específicas de aplicación.

También es posible tener más de una sección `<data>` en el mismo archivo XML. Podemos tomar ventaja de esto para tener un conjunto de datos con	`noupdate="1"`	y otro con `noupdate="0"`.
 
La etiqueta `noupdate` se almacena en la información de Identificador Externo para cada registro. Es posible editar la directamente utilizando el formulario de Identificador Externo disponible en el menú Técnico | opción Secuencias e identificadores | Identificadores externos, con la casilla de verificación **No actualizable**.

*Tip*  
*El atributo noupdate es difícilde manejar cuando se esta desarrollando el módulo, ya que los cambios hechos a los datos más tarde serán ignorados y Odoo no recogerá las modificaciones. Una solución es mantener `noupdate =" 0 "` durante el desarrollo y sólo ponerlo a * 1 * una vez terminado.* 
 

**Definición de registros en XML**

Cada elemento	`<record>` tiene dos atributos básicos, `id` y `model`, y contiene elementos	`<field>` de la asignación de valores a cada columna. Como se mencionó antes, el atributo `id` corresponde ID Externo del registro y el `model` al el modelo de destino donde se escribirá el registro. Los elementos	`<field>` tienen disponibles algunas maneras diferentes para asignar valores. Veamos en detalle.
 

**Configuración de los valores de campo**

El elemento	`<record>` define un registro de datos, y contiene elementos	<field> para establecer los valores de cada campo.

El atributo `name` del elemento `field` identifica el campo a ser escrito.

El valor a escribir es el contenido del elemento: el texto entre la etiqueta de apertura y la etiqueta de cierre del elemento `field`. En general, esto también es adecuado para establecer los valores que no son texto: para Booleanos,	`"0"/	"1"`	o valores `"False"/"True"`; para fechas, fechas y horas, cadenas de texto como `"YYYY-MM-DD"`	y	`"YYYY-MM-DD	HH:MI:SS"`, se realizará una correcta conversión.
 

**Ajuste de valores utilizando expresiones**

Una alternativa más avanzada para definir un valor de `field` es utilizar el atributo `eval`. Este evalúa una expresión Python y asigna el valor resultante al campo.

La expresión se evalúa en un contexto que, además de Python empotrado, también tiene algunos identificadores adicionales disponibles. Vamos a echar un vistazo a ellos.

Para manejar fechas, los siguientes módulos están disponibles: `time, datetime, timedelta` y `relativedelta`. Ellos permiten el cálculo de los valores de fecha, algo que se utiliza con frecuencia en los datos de demostración (y prueba). Por ejemplo, para establecer un valor de ayer usaríamos:
```
<field	name="expiration_date" eval="(datetime.now()+timedelta(-1)).strftime('%Y-%m-%d')"/>
``` 
También esta disponible en el contexto de evaluación la función `ref()`, que se utiliza para traducir un ID Externo al ID de base de datos correspondiente. Esto puede ser usado para establecer los valores para los campos relacionales. A modo de ejemplo, lo hemos usado antes para ajustar el valor para el `user_id`:
```
<field	name="user_id"	eval="ref('base.group_user')"	/> 
```
El contexto de evaluación también tiene una referencia, disponible para el Modelo actual, escrita a través de `obj`. Se puede utilizar junto con	`ref()`	para acceder a los valores de otros registros. He aquí un ejemplo del módulo de venta:
``` 
<value	model="sale.order" eval="obj(ref('test_order_1')).amount_total"	/> 
```

**Configuración de los valores de los campos de relación**

Acabamos de ver cómo establecer un valor en un campo de relación muchos-a-uno, como `user_id`, usando el atributo `eval` con una función	`ref()`. Pero hay una manera más sencilla. 

El elemento	`<field>` también tiene un atributo `ref` para establecer el valor de campo many-to-one utilizando un ID Externo. Usándolo, podemos establecer el valor de `user_id` con solo: 
```
<field	name="user_id"	ref="base.group_user"	/> 
```
Para campos	one-to-many	y	many-to-many,	se espera una lista de ID relacionados, por lo que es necesaria una sintaxis diferente, y Odoo proporciona una sintaxis especial para escribir sobre este tipo de campos.

El siguiente ejemplo, tomado de la aplicación de Flota, sustituye a la lista de registros relacionados de un campo `tag_ids`:
```
<field	name="tag_ids" eval="[(6,0,[ref('vehicle_tag_leasing'),ref('fleet.vehicle_tag_compact'),	ref('fleet.vehicle_tag_senior')] )]"	/> 
```
Para escribir sobre un campo a-muchos se utiliza una lista de tripletas. Cada tripleta es un comando de escritura que hace cosas diferentes según el código utilizado:

- `(0,_	,{'field':	value})`: Esto crea un nuevo registro y lo vincula a ésta 
- `(1,id,{'field':	value})`: Esto actualiza los valores en un registro ya vinculados
- `(2,id,_)`: Esto desvincula y elimina un registro relacionado 
- `(3,id,_)`: Esto desvincula pero no elimina un registro relacionado
- `(4,id,_)`:	Esto vincula un registro ya existente 
- `(5,_,_)`:	Esto desvincula pero no elimina todos los registros vinculados 
- `(6,_,[ids])`:	Esto reemplaza la lista de registros vinculados con la lista proporcionada 

El símbolo guión bajo utilizado anteriormente representa valores irrelevantes, por lo general lleno de 0 o False. 
 

**Atajos para modelos de uso frecuente**

Si nos remontamos al Capítulo 2,	*La construcción de su primera aplicación Odoo*, podemos encontrar en los archivos XML otros elementos además de	`<record>` ,	como	`<act_window>`	y	`<menuitem>`.
 
Estos son los atajos convenientes para los modelos de uso frecuente, que también se pueden cargar utilizando elemento `<record>`	regulares. Estos cargan datos en los modelos base y dan apoyo a la interfaz de usuario, se estudiarán con detalle más adelante, en el capítulo 6, *Vistas - Diseño de la interfaz de usuario*.
 
Como referencia, de manera que podamos comprender mejor los archivos XML que podemos encontrar en los módulos existentes, los siguientes elementos de acceso directo están disponibles con los modelos correspondientes donde cargan los datos:

- `<act_window>`:	Este es el modelo de acciones de ventana	`ir.actions.act_window` 
- `<menuitem>`:	Este es el modelo de elementos de menú	`ir.ui.menu` 
- `<report>`:	Este es el modelo de acciones de reporte	`ir.actions.report.xml`
- `<template>`:	Esto es el modelo de plantillas de vistas QWeb almacenadas en `ir.ui.view` 
- `<url>`:	Este es el modelo de acciones de URL	`ir.actions.act_url`  
 

**Otras acciones en archivos de datos XML**
  
Hasta ahora hemos visto cómo añadir o actualizar datos mediante archivos XML. Pero los archivos XML también permiten realizar otro tipo de acciones, a veces necesarios para configurar los datos. En particular, son capaces de eliminar los datos, ejecutar métodos arbitrarios del modelo, e iniciar la ejecución de eventos de flujo de trabajo.


**Eliminación de registros**

Para borrar un registro de datos se utiliza el elemento `<delete>`, siempre que sea con un `id` o un dominio de búsqueda para encontrar el registro de destino.

En el capítulo 3, 	*Herencia - Ampliación de aplicaciones existentes*,	tuvimos la necesidad de eliminar una regla de registro añadida por la aplicación de tareas pendientes.	En el archivo	`todo_user/security/todo_access_rules.xml` se utilizó un elemento	`<delete>`, con un dominio de búsqueda para encontrar el registro a eliminar: 
```
<delete	model="ir.rule"	search="[('id','=',ref('todo_app.todo_task_user_rule'))]" /> 
```
En este caso, el mismo efecto se puede lograr mediante el atributo `id` para identificar el registro a eliminar: 
```
<delete	model="ir.rule"	id="todo_app.todo_task_user_rule"	/> 
```

**Activación de las funciones y flujos de trabajo**

Un archivo XML también puede ejecutar métodos durante su proceso de carga a través del elemento	`<function>`. Esto puede ser usado para establecer datos de demostración y de prueba. Por ejemplo, en el módulo de miembros se utiliza para crear facturas de demostración de membresía:
```
<function model="res.partner" name="create_membership_invoice"	eval="(ref('base.res_partner_2'), ref('membership_0'), {'amount':180})" /> 
```
Esto llama al método	`create_membership_invoice()` del modelo	`res.partner`. Los argumentos se pasan como una tupla en el atributo `eval`. En este caso tenemos una tupla con tres argumentos: el ID de socio, la identificación de membresía y un diccionario que contiene el importe de la factura.

Otra forma en que los archivos de datos XML pueden realizar acciones es mediante la activación de los flujos de trabajo Odoo, a través del elemento	`<workflow>`.
 
Los flujos de trabajo pueden, por ejemplo, cambiar el estado de un pedido de cliente o convertirlo en una factura. He aquí un ejemplo tomado del módulo de venta, la conversión de un proyecto de orden de ventas para el estado confirmado:
```
<workflow	model="sale.order" ref="sale_order_4" action="order_confirm"	/>
``` 
A estas alturas, `model` se explica por sí mismo, y `ref` identifica la instancia de flujo de trabajo sobre la cual estamos actuando. `action` es la señal del flujo de trabajo enviada a la instancia de flujo de trabajo.
 

**Resumen**

Hemos aprendido todo lo necesario sobre la serialización de datos, y ganado una mejor comprensión de los aspectos de XML que vimos en los capítulos anteriores.

También pasamos algún tiempo comprendiendo los identificadores externos, un concepto central para el manejo de datos en general, y para las configuraciones de módulo en particular.

Los archivos de datos XML se explicaron en detalle. Aprendimos sobre las distintas opciones disponibles para establecer los valores de los campos y también para realizar acciones como eliminar registros y llamar a métodos de modelo.

Los archivos CSV y las características de importación / exportación de datos también fueron explicadas. Estas son herramientas valiosas para la configuración inicial de Odoo o para la edición masiva de datos.

En el siguiente capítulo se estudiará con detalle cómo construir modelos Odoo y posteriormente obtener más información sobre la construcción de sus interfaces de usuario.
