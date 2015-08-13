Capítulo 4.	Serialización de Datos y Datos de Módulos
====

La mayoría de las configuraciones de Odoo, desde interfaces de usuario hasta reglas de seguridad, son en realidad registros de datos almacenados en tablas internas de Odoo. Los archivos XML y CSV que se encuentran en los módulos no son usados para ejecutar aplicaciones Odoo. Ellos solo son un medio para cargar esas configuraciones a las tablas de la base de datos.

Los módulos pueden también tener datos iniciales y de demostración (accesorios). La serialización de datos permite añadir eso a nuestros módulos. Adicionalmente, entendiendo los formatos de serialización de datos de Odoo es importante para exportar e importar datos en el contexto de la implementación de un proyecto.

Antes de entrar en casos prácticos, primero exploraremos el conceptop de identificador externo, el cual es la clave a la serialización de datos de Odoo 

![150_1](/images/Odoo Development Essentials - Daniel Reis-150_1.jpg)

**Entendiendo los Identificadores Externos**

Todos los registros en la base de datos de Odoo tienen un identificador único, el campo ```id```

Es un número sequencial asignado automáticamente por la base de datos. De cualquier forma, este identificador automático puede ser un desafío al momento de cargar datos interrelacionados: ¿cómo podemos hacer referencia a un registro relacionado si no podemos saber de antemano cual ID de base de datos le será asignado?

La respuesta de Odoo a esto es el identificador externo. Los identificadores externos solucionan este problema asignando indentificadores con nombre a los registros de datos a ser cargados. Un identificador con nombre puede ser usado por cualquier otra pieza de dato registrada para referenciarla luego. Odoo se encargará de traducir estos nombres de identificación a los IDs reales asignados a ellos.

El mecanismo detrás de esto es muy simple: Odoo mantiene una tabla con el mapeo entre los IDs externos con nombre y sus correspondiente IDs numéricos en la base de datos. Ese es el modelo `ir.model.data`.

Para Inpeccionar la existencia de mapeo,se dirige a la seccion tecnica en el menu ajuste, y selecciona secuencia & identificadores	| el item de menu  identificadores externo. 

Por ejemplo, si volvemos a visitar la lista de identificadores externos	y filtramos por el modulo	`todo_app`,veremos los identificadores externos creados previamente por el modulo.

Puedes visializar los identificadores externo debes completar la etiqueta ID. Esta compuesto por el nombre del modulo y el nombre de identificador unido por un punto, por ejemplo, `todo_app.action_todo_task`.

Debido a que el ID Completo es obligatorio que sea único, el nombre del módulo sirve como namespace para los identificadores. Esto significa que el mismo identificador puede repetirse en diferentes módulos, y no tenemos que preocuparnos por identificadores en nuestro módulo que colicionen con identificadores en otros módulos.

![151_1](/images/Odoo Development Essentials - Daniel Reis-151_1.jpg)

Al principio de la lista, puedes ver el ID de `todo_app.action_todo_task`. Esta es la accción del menú que creamos para el módulo, el cual también es referenciado en el elemento de menú correspondiente. Haciendo click en él, puedes abrir un formulario con sus detalles: el `action_todo_task` en el módulo `todo_app` mapea hacia un ID de un registro específico en el modelo `ir.actions.act_window`.

A la cabeza de la lista,Se pueden ver el ID `todo_app.action_todo_task`.	Esta es la accion de menu creada para el modulo,	que se hace referencia en el menu correspondiente.	Al hacer click sobre el,puedes abrir el formulario en detalles:	la	`action_todo_task`	en el `todo_app`	es asignado a un ID especifico para el modulo	`ir.actions.act_window`.
 
Ademas de proporcionar una forma de hacer referencia a un registro de una manera facil a otros registros,	ID externos	tambien permiten evitar la duplicidad de datos en las importaciones repetidas. Si el ID externo esta presente, el registro existente se actualiza,	en ves de crear un nuevo registro.	Esta razon es por,	la actualizacion de modulos subsiguientes,	los resgistros cargados previamente se actualiza en lugar de duplicarse. 
 
![152_1](/images/Odoo Development Essentials - Daniel Reis-152_1.jpg)


**Encontrar identificadores externos**

En la demostracion y configuracion del archivo de datos para el modulo, tenemos que frecuentemente mirar la existencia de Ids Externos	que se necesitan para la referencia. 

Podemos utilizar identificadores externos en el menu mostrado anteriormente, pero el menu desarrollado puede proporcionar un metodo para  que sea mas conveniente conveniente.	como se recordara en el capitulo 1, 	*Getting Started	with	Odoo	Development*,	el menu desarrollo es activado sobre la opcion acerca Odoo, y entonces,	que esta disponible en la esquina izquierda de la vista cliente web.
 
Para buscar la ID externo para un registro de datos,	en el mismo formulario correspondiente,	seleccione la opcion metadatos desde el menu desarrollador.	Esto mostrara un cuadro de dialogo con el ID de la base de datos registrado y el ID externo (tambien conocido como ID HTML) 

Como un Ejemplo,buscamos el ID de usuario de demostracion,podemos navegar en la vista formulario	(configuracion	| usuarios )	y seleccione la opcion ver metadatos,	despues se mostrara lo siguiente: 
Para buscar el ID externo para ver los elementos,	en forma de,	arbol,	buscador,	y accion,	el menu de desarrollo es tambien una ayuda.	Para eso,utiliza su opcion vista de administrador	o abrir la informacion	para la vista deseada a utilizar como opcion de editor	<view	type> ,	y selecciona la opcion ver metadato. 
 

**Exportar y Importar datos**  

vamos a empezar a trabajar en la exportacion y importacion de datos en Odoo,	y desde alli,	vamos a pasar a los detalles tecnicos. 
 
![155_1](/images/Odoo Development Essentials - Daniel Reis-155_1.jpg)


**Exportando datos**

Exportar datos es una caracteristica estándar disponible en cualquier vista de lista.	para usarlo,	primero tenemos que seleccionar la fila de exportacion seleccionando las casillas de verificacion correspondiente en el extremo izquierdo,	y luego seleccionamos la opcion exportar en el boton "más". 

aqui esta un ejemplo,	utilizando las reciente tareas creadas a realizar: 
La opcion exportar nos lleva a un dialogo,	donde podemos elegir lo que se va a exportar.	La opcion exportar compatible se asegura de que el archivo exportado se pueda importar denuevo a Odoo. 

El formato de exportacion puede ser CSV	o	Excel.	Vamos a preferir archivos	CSV	para tener una mejor comprension del formato de exportacion.	comtinuamos,	eligiendo las columnas que queremos exportar y hacer click en el boton exportar archivo.	esto iniciara la descarga de un archivo con los datos exportados. 
 
![156_1](/images/Odoo Development Essentials - Daniel Reis-156_1.jpg)

si seguimos estas instrucciones y seleccionamos los campos que se demuestran en la imagen anterior,	debemos terminar con un archivo de texto CSV similar a este: 
```
"id","name","user_id/id","date_deadline","is_done" "__export__.todo_task_1","Install	Odoo","base.user_root","2015-01- 30","True" "__export__.todo_task_2","Create	dev	database","base.user_root","","False" 
```
Observe que Odoo exporta automaticamente una columna adicional identificada.	Este es un ID externo que se genera automaticamente para cada registro.	Estos identificadores externos generados utiliza `__export__` en lugar de un nombre real de módulo.	Nuevos identificadores solo se asignan a los que no poseen uno asignado,	y ya apartir de alli,	se mantienen unidos al mismos registro.	Esto significa que las exportaciones posteriores perserverán los mismos identificadores externos. 
 
![157_1](/images/Odoo Development Essentials - Daniel Reis-157_1.jpg)


**Importar datos**

Primero tenemos que asegurar de que la funcion de importar este habilitada.	Esto se hace en el menu de configuracion,	Configuracion	|	opcion de configuracion general.	bajar importacion	/	tema exportacion,	asegúrese de que la opcion permitir a los usuarios importar datos de archivos CSV este la casilla habilidata. 

Con esta opcion habilitada,	los puntos de vista de la lista muestran una opcion de importacion junto al boton crear en la parte superior de la lista. 
Vamos a realizar una edicion masiva en nuestros datos de tareas pendientes:	se abre en una hoja de calculo o en un editor de texto el archivo CSV que acabamos de descargar, a continuacion, cambiar algunos valores y añadir algunas nuevas filas. 

Como se menciono antes,la primera columna de identificacion proporciona un identificador unico para cada fila permitiendo registros ya existentes que se actualizaran en ves de duplicarse cuando importamos los datos de nuevo a Odoo.	Para las nuevas filas podemos añadir el archivo CSV,	se deben dejar en blanco, y se crea un nuevo record para ellos.

Despues de guardar los cambios en el archivo CSV,	haga click en la opcion importar	(seguido del boton crear)	y se presentara el asistente de importacion.	hay que seleccionar la ubicacion del archivo CSV sobre el disco y hasle click en validar  para conprobar su formato para la correccion.	desde el archivo de importacion basandose como la exportacion de Odoo,	hay buena probabilidad que sea validado. 

Ahora podemos hacer click en importar y se va:	nuevas modificaciones y nuevos registros deberian haber cargado en Odoo. 
 

**Registrado relacionados en los archivos de datos CSV**

en el ejemplo visto anteriormente,	el usuario responsable de cada tarea es un registro relacionado en el modelo de los usuarios,	con la relacion many	to	one 	(o	foreign	key).	El nombre de la columna para ello fue de usuario _id/id y los valores de los campos eran identificadores externos para los registros relacionados,tales como  `base.user_root`	para el usuario administrador. 

Columnas de relacion deben tener	`/id`	anexo a su nombre,	si el uso IDs externos,	o	`/.id`, si el uso de base de datos	(númerico)	IDs.	Alternativamente,	dos puntos	`(:)`	se puede utilizar en lugar de la barra para el mismo efecto. 

Del mismo modo,	la relacion many	to	many son soportable. Un ejemplo de relacion many	to	many	es la que existe entre usuarios y grupos:	cada usuario puede estar en muchos grupos, y cada grupo puede tener muchos usuarios.	La columna nombre	por este typo de campo deberia haber añadido un	`/id`.	Los valores de los campos acepta una lista separada por comas de Id externo, entre comillas dobles. 

Por ejemplo,	Seguir tareas a realizar con la relacion	many-to-many	entre hacer los Tasks	y Partners.	El nombre de la columna podia ser follower_ids/id	y un valor de campo con dos seguidores podria ser: 
`"__export__.res_partner_1,__export__.res_partner_2"`
 
Finalmente,	la relaciones one	to	many 	tambien se pueden importar a traves de CSV.	El ejemplo tipico de esta relacion es un documento “head”	con varias	“lines”. 

Podemos ver un ejemplo de tal relacion en el modelo de empresa	(la vista de formulario esta disponible en el menu configuracion):	una empresa puede tener varias cuentas bancarias,	cada una con sus propios detalles, y cada cuenta bancaria pertenece a (tiene una relacion con many-to-one)	solo una empresa. 

Es posible importar las empresa juntos con sus cuentas bancarias en un solo archivo.	Para esto,algunas columnas corresponderan a empresas,	 y otras columnas corresponderan a cuentas bancarias detalladas.	Los nombres de columnas de los detalles del banco 	debe ser precedido de la relacion	one-to-many	campos que vincula a la empresa con los bancos;	bank_ids	en este caso. 

Los primeros datos de la cuenta bancaria van en la misma fila de los datos vinculados de la empresa.	Los detalles de la proxima cuenta bancaria van en la siguiente fila,	pero solo los datos bancarios de la columna relacionada debe tener valores;	La columna de datos de la empresa debe tener esas lineas vacias. 
Aqui esta un ejemplo cargando una empresa con datos de tres bancos: 
```
id,name,bank_ids/id,bank_ids/acc_number,bank_ids/state base.main_company,YourCompany,__export__.res_partner_bank_4,123456789,bank ,,__export__.res_partner_bank_5,135792468,bank ,,__export__.res_partner_bank_6,1122334455,bank
```

Observe que las dos ultimas lineas comienzan con comas:	Esto corresponde a valores en las dos p´rimeras columnas,	id	y nombre,	con respecto a los datos del encabezado de empresa.	pero las columnas restantes,con respecto a las cuentas bancarias,tienen valores para la segunda y tercera record del banco. 

estos son los elementos esenciales en el trabajo con la exportacion y importacion de la guia.	Es util para establecer los datos en nuevas instancias Odoo, o para prepara nuevos archivos de datos que se incluira en los módulos Odoo. 
 
A continuacion vamos aprender mas sobre el uso de los archivos de datos en los módulos. 
 
  
**Modulos de datos**  

Los módulos utilizan archivos de datos para cargar sus configuraciones en la base de datos, los datos iniciales y los datos de demostración. Esto se puede hacer utilizando tanto CSV y archivos XML. Para completar, el formato de archivo YAML también se puede utilizar, pero esto rara vez se utiliza para la carga de datos, asi no se discute en este.

Archivos CSV utilizados por módulos son exactamente las mismas que las que hemos visto y utilizado para la función de importación. Cuando se usa en módulos, la única restricción adicional es que el nombre del archivo debe coincidir con el nombre del modelo a la que se cargan los datos.

Un ejemplo común es el acceso de seguridad, para cargar en el modelo ir.model.acess. Esto se hace generalmente con archivos CSV, y que debe ser nombrado ir.model.acess.csv.

**Demostracion de datos**

Módulos Odoo pueden instalar datos de demostración. Esto es útil para proporcionar ejemplos de uso para un módulo y conjuntos de datos para ser utilizado en pruebas. Se considera una buena práctica para los módulos para proporcionar datos de demostración. Los datos de demostración para un módulo se declara con el atributo de demostración de la	`__openerp__.py`	archivo de manifiesto. Al igual que el atributo de datos, se trata de una lista de nombres de archivo con las rutas relativas correspondientes en el interior del módulo.

Estaremos agregando los datos de demostración en nuestro	todo_user	module.	Podemos comenzar con la exportación de algunos datos de las tareas a realizar, como se explica en la sección anterior. Siguiente debemos guardar los datos en el	todo_user	directorio con el nombre del archivo	`todo.task.csv`.	Dado que esta información será propiedad de nuestro módulo, debemos editar los valores de id para reemplazar el	`__export__`	prefijo en los identificadores con el nombre técnico del módulo.

Como ejemplo nuestra	`todo.task.csv`	archivo de datos podría tener este aspecto:
``` 
id,name,user_id/id,date_deadline todo_task_a,"Install	Odoo","base.user_root","2015-01-30" todo_task_b","Create	dev	database","base.user_root","" 
```
No hay que olvidar agregar este archivo de datos para el	`__openerp__.py`	atributo de demostración manifiesta: 
```				
'demo':	['todo.task.csv'], 
```
La próxima vez que actualizamos el módulo, siempre y cuando se instaló con datos de demostración habilitadas, se importará el contenido del archivo. Tenga en cuenta que estos datos se reescribe cada vez que se realiza una actualización del módulo.

Archivos XML también pueden ser utilizados para los datos de demostración. Sus nombres de archivo no están obligados a coincidir con el modelo para cargar, porque el formato XML es mucho más rico y que la información es proporcionada por los elementos XML dentro del archivo.

Vamos a aprender más sobre lo que los archivos de datos XML nos permiten hacer que los archivos CSV no.

**Archivos de datos XML**  

Mientras que los archivos CSV proporcionan un formato simple y compacto para serializar los datos, archivos XML son más potentes y dan un mayor control sobre el proceso de carga.

Ya hemos utilizado los archivos de datos XML en los capítulos anteriores. Los componentes de la interfaz de usuario, tales como vistas y elementos de menú, se encuentran en los registros de datos almacenados en los modelos de sistemas. Los archivos XML en los módulos son un medio utilizado para cargar los registros en el servidor.

Para mostrar esto, vamos a añadir un segundo archivo de datos para el	`todo_user`	módulo, llamado `todo_data.xml`,	con el siguiente contenido: 
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

Archivos de datos XML tienen una `<openerp>`	elemento que contiene	`<data>`	elementos, dentro de los cuales podemos tener tienen varios`<record>`	elementos, correspondientes a las filas de datos CSV.

Un	`<record>`	elemento tiene dos atributos obligatorios, de modelo y de Identificación (el identificador externo para el registro), y contiene una	`<field>`	etiqueta para cada campo para escribir. 

Tenga en cuenta que la notación con barras en los nombres de campo no está disponible aquí: no podemos usar	`<field name="user_id/id">`.	En cambio, el atributo especial ref se utiliza para hacer referencia a los identificadores externos. Hablaremos de los valores para el relacional "para muchos" campos en un momento.
 

**El atributo noupdate datos**

Cuando se repite la carga de datos, los registros existentes de la ejecución anterior se reescriben.
 
Esto es importante a tener en cuenta: significa que la actualización de un módulo se sobreponen a los cambios manuales que podrían haber sido realizados en los datos. Cabe destacar que, si visitas fueron modificados con personalizaciones, esos cambios se perderán con la próxima actualización del módulo. El procedimiento correcto es crear en vez vistas heredadas de los cambios que necesitamos, como se explica en el Capítulo 3, 	


*Herencia - Ampliación de aplicaciones existentes*.
 
Este comportamiento de sobrescritura es el valor por defecto, pero se puede cambiar, por lo que cuando un registro ya creado se carga de nuevo se realiza ningún cambio a la misma. Esto se hace añadiendo a la `<data>`	un elemento atributo	`noupdate="1"`.Con esto, sus registros se crearán la primera vez que se cargan, y en mejoras de módulos subsiguientes no se hará nada para ellos.

Esto permite personalizaciones realizadas manualmente para estar a salvo de las actualizaciones de módulos. Se utiliza a menudo con las reglas de acceso de grabación, lo que les permite adaptarse a las necesidades específicas de aplicación.

También es posible tener más de un `<datos> sección` en el mismo archivo XML. Podemos tomar ventaja de esto para tener un conjunto de datos con	`noupdate="1"`	y otro con `noupdate="0"`.
 
La bandera noupdate se almacena en la información Identificador externo para cada registro. Es posible editar directamente utilizando el formulario Identificador externo disponible en el menú de Técnico, con la casilla de verificación no se pueden actualizar.

*Tip*  
*El atributo noupdate es difícil cuando se desarrolla el módulos, ya que los cambios hechos a los datos más tarde será ignorado y Odoo no recogerá más tarde modificaciones. Una solución es mantener `noupdate =" 0 "` durante el desarrollo y sólo ponerlo a * 1 * una vez terminado.* 
 

**Definición de registros en XML**

Cada	`<record>`	elemento tiene dos atributos básicos, id y el modelo, y contiene	`<field>` elementos de la asignación de valores a cada columna. Como se mencionó antes, el atributo id corresponde al del disco externo ID y el modelo para el modelo de destino donde se escribirá el registro. Los	`<field>`	elementos tienen disponibles algunas maneras diferentes para asignar valores. Veamos en detalle.
 

**Configuración de los valores de campo**

El	`<record>`	elemento define un registro de datos, y contiene	<field>	elementos para establecer los valores de cada campo.

El atributo name del elemento de campo identifica el campo para ser escrito.

El valor de escribir es el contenido del elemento: el texto entre la apertura del campo y la etiqueta de cierre. En general, esto también es adecuado para establecer los valores no son de texto: para Booleans,	`"0"/	"1"`	o 
`"False"/"True"`	valores serán convertidos correctamente; para fechas y datetimes, cuerdas con `"YYYY-MM-DD"`	y	`"YYYY-MM-DD	HH:MI:SS"`	se convertirán correctamente. 
 

**Ajuste de valores utilizando expresiones**

Una alternativa más avanzada para definir un valor de campo utiliza el atributo eval lugar. Este evalúa una expresión Python y asigna el valor resultante al campo.
La expresión se evalúa en un contexto que, además de Python empotrados, también tiene algunos identificadores adicionales disponibles. Vamos a echar un vistazo a ellos.

Para manejar fechas, los siguientes módulos están disponibles: Tiempo, fecha y hora, y timedelta relativedelta. Ellos permiten el cálculo de los valores de fecha, algo que se utiliza con frecuencia en los datos de demostración (y prueba). Por ejemplo, para establecer un valor de ayer usaríamos:
```
<field	name="expiration_date" eval="(datetime.now()+timedelta(-1)).strftime('%Y-%m-%d')"/>
``` 
También disponible en el contexto de evaluación es el `ref función ()`, que se utiliza para traducir un documento de identidad externa en el ID de la base de datos correspondiente. Esto puede ser usado para establecer los valores para los campos relacionales. A modo de ejemplo, hemos usado antes para ajustar el valor para el user_id:
```
<field	name="user_id"	eval="ref('base.group_user')"	/> 
```
El contexto de evaluación también tiene una referencia disponible para el modelo actual que se está escrita a través de objeto. Se puede utilizar junto con	`ref()`	para acceder a los valores de otros registros. He aquí un ejemplo del módulo de venta:
``` 
<value	model="sale.order" 		eval="obj(ref('test_order_1')).amount_total"	/> 
```

**Configuración de los valores de los campos de relación**

Acabamos de ver cómo establecer un valor en un campo de relación muchos-a-uno, como user_id, usando el atributo eval con una función	`ref()`. Pero hay una manera más sencilla. 

El	`<field>`	elemento también tiene un atributo ref para establecer el valor de campo many-to-one utilizando una identificación externa. Usándolo, podemos establecer el valor de user_id usando sólo: 
```
<field	name="user_id"	ref="base.group_user"	/> 
```
Para	one-to-many	y	many-to-many	fields,	Se espera que una lista de identificadores relacionados, por lo que es necesaria una sintaxis diferente, y Odoo proporciona una sintaxis especial para escribir sobre este tipo de campos.

El siguiente ejemplo, tomado de la aplicación de la flota, sustituye a la lista de registros relacionados de un campo tag_ids:
```
<field	name="tag_ids" eval="[(6,0,[ref('vehicle_tag_leasing'),ref('fleet.vehicle_tag_compact'),	ref('fleet.vehicle_tag_senior')] )]"	/> 
```
Para escribir sobre una de muchas de campo se utiliza una lista de triples. Cada triple es un comando de escritura que hace cosas diferentes según el código utilizado:

- `(0,_	,{'field':	value})`: Esto crea un nuevo registro y lo vincula a ésta 
- `(1,id,{'field':	value})`: Esto actualiza los valores en un registro ya vinculados
- `(2,id,_)`: Esto desvincula y elimina un registro relacionado 
- `(3,id,_)`: Esto desvincula pero no elimina un registro relacionado
- `(4,id,_)`:	Esto vincula un registro ya existente 
- `(5,_,_)`:	Esto desvincula pero no elimina todos los registros vinculados 
- `(6,_,[ids])`:	Esto reemplaza la lista de registros vinculados con la lista proporcionada 

El símbolo de subrayado utilizado anteriormente representa valores irrelevantes, por lo general llenas de 0 o Falso. 
 

**Shortcuts	for	frequently	used	Models**

If	we	go	back	to Chapter	2 ,	*Building	Your	First	Odoo	Application*,	we	can	find	in	the XML	files	elements	other	than	`<record>} ,	such	as	`<act_window>`	and	`<menuitem>`.
 
These	are	convenient	shortcuts	for	frequently	used	Models	that	can	also	be	loaded	using regular	`<record>`	elements.	They	load	data	into	base	Models	supporting	the	user	interface and	will	be	explored	in	more	detail	later,	in Chapter	6 ,	*Views	-	Designing	the	User Interface*.
 
For	reference,	so	that	we	can	better	understand	XML	files	we	may	encounter	in	existing modules,	the	following	shortcut	elements	are	available	with	the	corresponding	Models they	load	data	into: 

- `<act_window>`:	This	is	the	Window	Actions	model	`ir.actions.act_window` 
- `<menuitem>`:	This	is	the	Menu	Items	model	`ir.ui.menu` 
- `<report>`:	This	is	the	Report	Actions	model	`ir.actions.report.xml`
- `<template>`:	This	is	View	QWeb	Templates	stored	in	model	`ir.ui.view` 
- `<url>`:	This	is	the	URL	Actions	model	`ir.actions.act_url` 
 

**Other	actions	in	XML	data	files**
  
Until	now	we	have	seen	how	to	add	or	update	data	using	XML	files.	But	XML	files	also allow	performing	other	types	of	actions,	sometimes	needed	to	set	up	data.	In	particular, they	are	capable	in	deleting	the	data,	execute	arbitrary	model	methods,	and	trigger workflow	events. 


**Deleting	records**

To	delete	a	data	record	we	use	the	<delete>	element,	providing	it	with	either	an	id	or	a search	domain	to	find	the	target	record. 

In Chapter	3, 	*Inheritance	–	Extending	Existing	Applications*,	we	had	the	need	to	remove	a record	rule	added	by	the	to-do	app.	In	the	`todo_user/security/todo_access_rules.xml` file	a	`<delete>`	element	was	used,	with	a	search	domain	to	find	the	record	to	delete: 
```
<delete	model="ir.rule"	search="[('id','=',ref('todo_app.todo_task_user_rule'))]" /> 
```
In	this	case	the	same	exact	effect	could	be	achieved	using	the	id	attribute	to	identify	the record	to	delete: 
```
<delete	model="ir.rule"	id="todo_app.todo_task_user_rule"	/> 
```

**Triggering	functions	and	workflows**

An	XML	file	can	also	execute	methods	during	its	load	process	through	the	`<function>` element.	This	can	be	used	to	set	up	demo	and	test	data.	For	example,	in	the	membership module	it	is	used	to	create	demonstration	membership	invoices: 
```
<function model="res.partner" name="create_membership_invoice"	eval="(ref('base.res_partner_2'), ref('membership_0'), {'amount':180})" /> 
```
This	is	calling	the	`create_membership_invoice()`	method	of	the	`res.partner`	model. The	arguments	are	passed	as	a	tuple	in	the	eval	attribute.	In	this	case	we	have	a	tuple	with three	arguments:	the	Partner	ID,	the	Membership	ID	and	a	dictionary	containing	the invoice	amount. 

Another	way	XML	data	files	can	perform	actions	is	by	triggering	Odoo	workflows, through	the	`<workflow>`	element.
 
Workflows	can,	for	example,	change	the	state	of	a	sales	order	or	convert	it	into	an	invoice. Here	is	an	example	taken	from	the	sale	module,	converting	a	draft	sales	order	to	the confirmed	state: 
```
<workflow	model="sale.order" ref="sale_order_4" action="order_confirm"	/>
``` 
The	model	is	self-explanatory	by	now,	and	ref	identifies	the	workflow	instance	we	are acting	upon.	The	action	is	the	workflow	signal	sent	to	that	workflow	instance. 
 

**Summary**

We	have	learned	all	the	essentials	about	data	serialization,	and	gained	a	better understanding	of	the	XML	aspects	we	saw	in	the	previous	chapters. 

We	also	spent	some	time	understanding	External	Identifiers,	a	central	concept	for	data handling	in	general,	and	for	module	configurations	in	particular. 

XML	data	files	were	explained	in	detail.	We	learned	about	the	several	options	available	to set	values	on	fields	and	also	to	perform	actions	such	as	deleting	records	and	calling	model methods. 
CSV	files	and	the	data	import/export	features	were	also	explained.	These	are	valuable tools	for	Odoo	initial	setup	or	for	mass	editing	of	data. 

In	the	next	chapter	are	will	explore	in	detail	how	to	build	Odoo	models	and	later	learn more	about	building	their	user	interfaces. 
