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

Modules	use	data	files	to	load	their	configurations	into	the	database,	initial	data	and demonstration	data.	This	can	be	done	using	both	CSV	and	XML	files.	For	completeness, the	YAML	file	format	can	also	be	used,	but	this	is	rarely	used	for	data	loading,	so	we won’t	be	discussing	it. 

CSV	files	used	by	modules	are	exactly	the	same	as	those	we	have	seen	and	used	for	the import	feature.	When	using	them	in	modules,	the	only	additional	restriction	is	that	the	file name	must	match	the	name	of	the	model	to	which	the	data	will	be	loaded. 

A	common	example	is	security	access,	to	load	into	the	ir.model.acess	model.	This	is usually	done	using	CSV	files,	and	they	should	be	named	ir.model.acess.csv. 
 

**Demonstration	data**

Odoo	modules	may	install	demo	data.	This	is	useful	to	provide	usage	examples	for	a module	and	data	sets	to	be	used	in	tests.	It’s	considered	good	practice	for	modules	to provide	demonstration	data.	Demonstration	data	for	a	module	is	declared	using	the	demo attribute	of	the	`__openerp__.py`	manifest	file.	Just	like	the	data	attribute,	it	is	a	list	of	file names	with	the	corresponding	relative	paths	inside	the	module. 

We	will	be	adding	demonstration	data	to	our	todo_user	module.	We	can	start	by	exporting some	data	from	the	to-do	tasks,	as	explained	in	the	previous	section.	Next	we	should	save that	data	in	the	todo_user	directory	with	file	name	`todo.task.csv`.	Since	this	data will	be owned	by	our	module,	we	should	edit	the	id	values	to	replace	the	`__export__`	prefix	in the	identifiers	with	the	module	technical	name. 

As	an	example	our	`todo.task.csv`	data	file	might	look	like	this:
``` 
id,name,user_id/id,date_deadline todo_task_a,"Install	Odoo","base.user_root","2015-01-30" todo_task_b","Create	dev	database","base.user_root","" 
```
We	must	not	forget	to	add	this	data	file	to	the	`__openerp__.py`	manifest	demo	attribute: 
```				
'demo':	['todo.task.csv'], 
```
Next	time	we	update	the	module,	as	long	as	it	was	installed	with	demo	data	enabled,	the content	of	the	file	will	be	imported.	Note	that	this	data	will	be	rewritten	whenever	a module	upgrade	is	performed. 

XML	files	can	also	be	used	for	demonstration	data.	Their	file	names	are	not	required	to match	the	model	to	load,	because	the	XML	format	is	much	richer	and	that	information	is provided	by	the	XML	elements	inside	the	file. 

Let’s	learn	more	about	what	XML	data	files	allow	us	to	do	that	CSV	files	don’t. 
 

**XML	data	files**  

While	CSV	files	provide	a	simple	and	compact	format	to	serialize	data,	XML	files	are more	powerful	and	give	more	control	over	the	loading	process. 

We	have	already	used	XML	data	files	in	the	previous	chapters.	The	user	interface components,	such	as	views	and	menu	items,	are	in	fact	records	stored	in	system	models. The	XML	files	in	the	modules	are	a	means	used	to	load	those	records	into	the	server. 

To	showcase	this,	we	will	add	a	second	data	file	to	the	`todo_user`	module,	named `todo_data.xml`,	with	the	following	content: 
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
This	XML	is	equivalent	to	the	CSV	data	file	we	have	just	seen	in	the	previous	section. 

XML	data	files	have	a	`<openerp>`	element	containing	`<data>`	elements,	inside	of	which we	can	have	have	several	`<record>`	elements,	corresponding	to	the	CSV	data	rows. 

A	`<record>`	element	has	two	mandatory	attributes,	model	and	id	(the	external	identifier for	the	record),	and	contains	a	`<field>`	tag	for	each	field	to	write	on. 

Note	that	the	slash	notation	in	field	names	is	not	available	here:	we	can’t	use	`<field name="user_id/id">`.	Instead	the	ref	special	attribute	is	used	to	reference	External	IDs. We’ll	discuss	the	values	for	the	relational	“to	many”	fields	in	a	moment. 
 

**The	data	noupdate	attribute**

When	the	data	loading	is	repeated,	existing	records	from	the	previous	run	are	rewritten.
 
This	is	important	to	keep	in	mind:	it	means	that	upgrading	a	module	will	overwrite	any manual	changes	that	might	have	been	made	to	the	data.	Notably,	if	views	were	modified with	customizations,	those	changes	will	be	lost	with	the	next	module	upgrade.	The	correct procedure	is	to	instead	create	inherited	views	for	the	changes	we	need,	as	discussed	in	the  Chapter	3, 	*Inheritance	–	Extending	Existing	Applications*.
 
This	overwrite	behavior	is	the	default,	but	it	can	be	changed,	so	that	when	an	already created	record	is	loaded	again	no	change	is	made	to	it.	This	is	done	by	adding	to	the 
`<data>`	element	a	`noupdate="1"`	attribute.	With	this,	its	records	will	be	created	the	first time	they	are	loaded,	and	in	subsequent	module	upgrades	nothing	will	be	done	to	them. 

This	allows	for	manually	made	customizations	to	be	safe	from	module	upgrades.	It	is often	used	with	record	access	rules,	allowing	them	to	be	adapted	to	implementation specific	needs. 

It	is	also	possible	to	have	more	than	one	`<data>`	section	in	the	same	XML	file.	We	can take	advantage	of	this	to	have	a	data	set	with	`noupdate="1"`	and	another	with `noupdate="0"`.
 
The	noupdate	flag	is	stored	in	the	External	Identifier	information	for	each	record.	It’s possible	to	edit	it	directly	using	the	External	Identifier	form	available	in	the	Technical menu,	with	the	Non	Updatable 	checkbox. 

*Tip*  
*The	noupdate	attribute	is	tricky	when	developing	modules,	because	changes	made	to	the data	later	will	be	ignored,	and	Odoo	won’t	pick	up	later	modifications.	A	solution	is	to keep	`noupdate="0”`	during	development	and	only	set	it	to	*1*	once	finished.* 
 

**Defining	Records	in	XML**

Each	`<record>`	element	has	two	basic	attributes,	id	and	model,	and	contains	`<field>` elements	assigning	values	to	each	column.	As	mentioned	before,	the	id	attribute corresponds	to	the	record’s	External	ID	and	the	model	to	the	target	model	where	the	record will	be	written.	The	`<field>`	elements	have	available	a	few	different	ways	to	assign values.	Let’s	look	at	them	in	detail. 
 

**Setting	field	values**

The	`<record>`	element	defines	a	data	record,	and	contains	<field>	elements	to	set	values on	each	field. 

The	name	attribute	of	the	field	element	identifies	the	field	to	be	written. 

The	value	to	write	is	the	element	content:	the	text	between	the	field’s	opening	and	closing tag.	In	general	this	is	also	suitable	to	set	non-text	values:	for	Booleans,	`"0"/	"1"`	or 
`"False"/"True"`	values	will	be	correctly	converted;	for	dates	and	datetimes,	strings	with `"YYYY-MM-DD"`	and	`"YYYY-MM-DD	HH:MI:SS"`	will	be	converted	properly. 
 

**Setting	values	using	expressions**

A	more	advanced	alternative	to	define	a	field	value	is	using	the	eval	attribute	instead.	This evaluates	a	Python	expression	and	assigns	the	resulting	value	to	the	field. 
The	expression	is	evaluated	in	a	context	that,	besides	Python	built-ins,	also	has	some additional	identifiers	available.	Let’s	have	a	look	at	them. 

To	handle	dates,	the	following	modules	are	available:	time,	datetime,	timedelta	and  relativedelta.	They	allow	calculating	date	values,	something	that	is	frequently	used	in demonstration	(and	test)	data.	For	example,	to	set	a	value	to	yesterday	we	would	use: 
```
<field	name="expiration_date" eval="(datetime.now()+timedelta(-1)).strftime('%Y-%m-%d')"/>
``` 
Also	available	in	the	evaluation	context	is	the	`ref()`	function,	used	to	translate	an	External ID	into	the	corresponding	database	ID.	This	can	be	used	to	set	values	for	relational	fields. As	an	example,	we	have	used	it	before	to	set	the	value	for	the	user_id: 
```
<field	name="user_id"	eval="ref('base.group_user')"	/> 
```
The	evaluation	context	also	has	a	reference	available	to	the	current	Model	being	written through	obj.	It	can	be	used	together	with	`ref()`	to	access	values	from	other	records.	Here is	an	example	from	the	Sales	module:
``` 
<value	model="sale.order" 		eval="obj(ref('test_order_1')).amount_total"	/> 
```

**Setting	values	for	relation	fields**

We	have	just	seen	how	to	set	a	value	on	a	many-to-one	relation	field,	such	as	user_id, using	the	eval	attribute	with	a	`ref()`	function.	But	there	is	a	simpler	way. 

The	`<field>`	element	also	has	a	ref	attribute	to	set	the	value	for	a	many-to-one	field	using an	External	ID.	Using	it,	we	can	set	the	value	for	user_id	using	just: 
```
<field	name="user_id"	ref="base.group_user"	/> 
```
For	one-to-many	and	many-to-many	fields,	a	list	of	related	IDs	is	expected,	so	a	different syntax	is	needed,	and	Odoo	provides	a	special	syntax	to	write	on	this	type	of	fields. 

The	following	example,	taken	from	the	Fleet	app,	replaces	the	list	of	related	records	of	a tag_ids	field: 
```
<field	name="tag_ids" eval="[(6,0,[ref('vehicle_tag_leasing'),ref('fleet.vehicle_tag_compact'),	ref('fleet.vehicle_tag_senior')] )]"	/> 
```
To	write	on	a	to	many-field	we	use	a	list	of	triples.	Each	triple	is	a	write	command	that does	different	things	according	to	the	code	used: 

- `(0,_	,{'field':	value})`: This	creates	a	new	record	and	links	it	to	this	one 
- `(1,id,{'field':	value})`: This	updates	values	on	an	already	linked	record 
- `(2,id,_)`: This	unlinks	and	deletes	a	related	record 
- `(3,id,_)`: This	unlinks	but	does	not	delete	a	related	record 
- `(4,id,_)`:	This	links	an	already	existing	record 
- `(5,_,_)`:	This	unlinks	but	does	not	delete	all	linked	records 
- `(6,_,[ids])`:	This	replaces	the	list	of	linked	records	with	the	provided list 

The	underscore	symbol	used	above	represents	irrelevant	values,	usually	filled	with	0	or False. 
 

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
