Capítulo 6.	Vistas – Diseñar la Interfaz
====

Este capítulo le ayudara a construir la interfaz gráfica para sus aplicaciones. Hay varios tipos disponibles de vistas y widgets. Los conceptos de contexto y dominio también juegan un papel fundamental en la mejora de la experiencia del usuario y la usuaria, y aprenderá más sobre esto.

El módulo `todo_ui` tiene lista la capa de modelo, y ahora necesita la capa de vista con la interfaz. Agregaremos elementos nuevos a la IU y modificaremos las vistas existentes que fueron agregadas en capítulos anteriores.

La mejor manera de modificar vistas existentes es usar la herencia, como se explico en el Capítulo 3. Sin embargo, para mejorar la claridad en la explicación, sobre escribiremos las vistas existentes, y las reemplazaremos por unas vistas completamente nuevas. Esto hará que los temas sean más fáciles de entender y seguir.

Es necesario agregar un archivo XML nuevo al módulo, así que comenzaremos por editar el archivo manifiesto `__openerp__.py`. Necesitamos usar algunos campos del módulo `todo_user`, para que sea configurado como una dependencia:
``` 
{ 'name': 'User interface improvements to the To-Do app',
  'description': 'User friendly features.',
  'author': 'Daniel Reis',
  'depends': ['todo_user'],
  'data': ['todo_view.xml']
} 
```
Comencemos con las opciones de menú y las acciones de ventana.


**Acciones de ventana**

Las acciones de ventana dan instrucciones a la interfaz del lado del cliente. Cuando un usuario o una usuaria hace clic en una opción de menú o en un botón para abrir un formulario, es la acción subyacente la que da instrucciones a la interfaz sobre lo que debe hacer.

Comenzaremos por crear la acción de ventana que será usada en las opciones de manú, para abrir las vistas de las tareas por hacer y de los estados. Cree el archivo de datos `todo_view.xml` con el siguiente código: 
```
<?xml	version="1.0"?>
    <openerp>
        <data>
            <act_window	id="action_todo_stage" name="To-Do Task Stages" res_model="todo.task.stage" view_mode="tree,form"/>
            <act_window	id="todo_app.action_todo_task" name="To-Do Tasks" res_model="todo.task" view_mode="tree,form,calendar,gantt,graph" target="current "context="{'default_user_id':	uid}" domain="[]" limit="80"/>
            <act_window	id="action_todo_task_stage" name="To-Do Task Stages" res_model="todo.task.stage" src_model="todo.task" multi="False"/>	
        </data> 
     </openerp> 
```
Las acciones de ventana se almacenan en el modelo `ir.actions.act_window`, y pueden ser definidas en archivos XML usando el acceso directo `<act_window>` que recién usamos.

La primera acción abre el modelo de estados de la tarea, y solo usa los atributos básicos para una acción de ventana.

La segunda acción usa un ID en el espacio de nombre de `todo_app` para sobre escribir la acción original de tareas por hacer del módulo `todo_app`. Esta usa los atributos de aciones de ventana más relevantes.

- name: Este es el título mostrado en las vistas abiertas a través de esta acción.
- `res_model`: Es el identificador del modelo de destino.
- `view_mode`: Son los tipos de vista que estarán disponibles. El orden es relevenate y el primero de la lista será la vista que se abrirá de forma predeterminada.
- target: Si es fijado como “new”, la vista se abrirá en una ventana de dialogo.De forma predeterminada esta fijado a “current”, por lo que abre la vista en el área principal de contenido.
- context: Este fija información de contexto en las vistas de destino, la cual puede ser usada para establecer valores predeterminados en campos o filtros activos, entre otras cosas. Veremos más detalles sobre esto en este mismo capítulo.
- domain: Es una expresión de dominio que establece un filtro para los registros que estarán disponibles en las vistas abiertas.
- limit: Es el número de registros por cada página con vista de lista, 80 es el número predefinido.

La acción de ventana ya incluye los otros tipos de vista las cuales estaremos examinando en este capítulo: calendar, Gantt y gráfico. Una vez que estos cambios son instalados, los botones correspondientes serán mostrados en la esquina superior derecha, junto a los botones de lista y formulario. Note que esto no funcionará hasta crear las vistas correspondientes.

La tercera acción de ventana demuestra como agregar una opción bajo el botón “Mas”, en la parte superior de la vista. Estos son los atributos usados para realizar esto:

- multi: Si esta fijado a “True”, estará disponible en la vista de lista. De lo contrario, estará disponible en la vista de formulario.


**Opciones de menú**

Las opciones de menú se almacenan en el modelo `ir.ui.menu`, y pueden ser encontradas en el menú Configuraciones navegando a través de Técnico | Interfaz de Usuario | Opciones de Menú. Si buscamos Mensajería, veremos que tiene como submenú Organizador. Con la ayuda de las herramientas de desarrollo podemos encontrar el ID del XML para esa opción de menú: la cual es `mail.mail_my_stuff`.

Reemplazaremos la opción de menú existente en Tareas por Hacer con un submenú que puede encontrarse navegando a través de Mensajería | Organizador. En el `todo_view.xml`, despues de las acciones de ventana, agregue el siguiente código:
```
<menuitem id="menu_todo_task_main" name="To-Do" parent="mail.mail_my_stuff"/>
<menuitem id="todo_app.menu_todo_task" name="To-Do Tasks" parent="menu_todo_task_main" sequence="10" action="todo_app.action_todo_task"/>
<menuitem id="menu_todo_task_stage" name="To-Do Stages" parent="menu_todo_task_main" sequence="20" action="action_todo_stage"/> 
```
La opción de menú “data” para el modelo `ir.ui.menu` también puede cargarse usando el elemento de acceso directo `<menuitem>`, como se uso en el código anterior.

El primer elemento del menú, “To-Do”, es hijo de la opción de menú Organizador `mail.mail_my_stuff`. No tiene ninguna acción asignada, debido a que será usada como padre para las próximas dos opciones.

El segundo elemento del menú re escribe la opción definida en el módulo `todo_app` para ser re ubicada bajo el elemento “To-Do” del menú principal.

El tercer elemento del menú agrega una nueva opción para acceder a los estados. Necesitaremos un orden para agregar algunos datos que permitan usar los estados en las tareas por hacer.


**Contexto y dominio**

Nos hemos referido varias veces al contexto y al dominio. También hemos visto que las acciones de ventana pueden fijar valores en estos, y que los campos relacionales pueden usarlos en sus atributos. Ambos conceptos son útiles para proveer interfaces mas sofisticadas. Veamos como.


**Contexto de sesión**

El contexto es un diccionario que contiene datos de sesión usados por las vistas en el lado del cliente y por los procesos del servidor. Puede transportar información desde una vista hasta otra, o hasta la lógica del lado del servidor. Es usado frecuentemente por las acciones de ventana y por los campos relacionales para enviar información a las vistas abiertas a través de ellos.


Odoo estable en el contexto alguna información básica sobre la sesión actual. La información inicial de sesión puede verse así:
```
{'lang': 'en_US',	'tz': 'Europe/Brussels', 'uid': 1} 
```
Tenemos información del ID de usuario actual, y las preferencias de idioma y zona horaria para la sesión de usuario.

Cuando se usa una acción en el cliente, como hacer clic en un botón, se agrega información al contexto sobre los registros seleccionados actualmente:

- `active_id` es el ID del registro seleccionado en el formulario,
- `active_model` es el modelo de los registros actuales,
- `active_ids` es la lista de los ID seleccionados en la vista de árbol/lista.

El contexto también puede usarse para proveed valores predeterminados en los campos o habilitar filtros en la vista de destino.

Para fijar el valor predeterminado en el campo `user_id`, que corresponda a la sesión actual de usuario, debemos usar:
```
{'default_user_id': uid} 
```
Y si la vista de destino tiene un filtro llamado `filter_my_task`, podemos habilitarlo usando:
```
{'search_default_filter_my_tasks':	True} 
``` 


**Expresiones de dominio**

Los dominios se usan para filtrar los datos de registro. Odoo los analiza detenidamente para formar la expresión WHERE SQL usada para consultar a la base de datos.

Cuando se usa en una acción de ventana para abrir una vista, el dominio fija un filtro en los registros que estarán disponibles en esa vista. Por ejemplo, para limitar solo a las Tareas del usuario actual:
```
domain=[('user_id', '=', uid)] 
```
El valor “uid” usado aquí es provisto por el contexto de sesión. Cuando se usa en un campo relacional, limitara las opciones disponibles de selección para ese campo. El filtro de dominio puede también usar valores de otros campos en la vista. Con esto podemos tener diferentes opciones disponibles dependiendo de lo seleccionado en otros campos. Por ejemplo, un campo de persona de contacto puede ser establecido para mostrar solo las personas de la compañía seleccionada previamente en otro campo.

Un dominio es una lista de condiciones, donde cada condición es una tupla `('field', 'operator', 'value')`.

El campo a la izquierda es al cual se aplicara el filtro, y puede ser usada la notación de punto en los campos relaciones.

Los operadores que pueden ser usados son:  

- `=`, “like” para coincidencias con el valor del patrón donde el símbolo de guión bajo (`_`) coincida con cualquier carácter único, y `%` coincida con cualquier secuencia de caracteres. “like” para hacer coincidir con el patrón SQL `%value%` sensible a mayúsculas, e “ilike” para coincidencias sin sensibilidad de mayúsculas. Los operadores “not like” y “not ilike” hacen la operación inversa.

- `child_of` encuentra los hijos directos e indirectos, si las relaciones padre/hijo están configuradas en el modelo de destino.

- “in” y “not” verifican la inclusión en una lista. En este caso, el valor de la derecha debe ser una lista Python. Estos son los únicos operadores que pueden ser usados con valores de una lista. Un caso especial es cuando el lado izquierdo es un campo “a-muchos”: aquí el operador “in” ejecuta una operación “contains”.

Están disponibles los operadores de comparación usuales: `<, >, <=, >=, =, y !=`.

El valor dela derecha puede puede ser una constante o una expresión Python a ser evaluada. Lo que puede ser usado en estas expresiones depende del contexto disponible (no debe ser confundido con el contexto de sesión, discutido en la sección anterior). Existen dos posibles contextos de evaluación para los dominios: del lado del cliente y del lado del servidor.

Para los dominios de campo y las acciones de ventana, la evaluación es realizada desde el lado del cliente. El contexto de evaluación incluye aquí los campos disponibles para la vista actual, y la notación de puntos no esta disponible. Puede ser usados los valores del contexto de sesión, como “uid” y “active_id”. Estan disponibles los módulo de Python “datetime” y “time” para ser usado en las operaciones de fecha y hora, y también esta disponible la función `context_today()` que devuelve la fecha actual del cliente.

Los dominios usados en las reglas de registro de seguridad y en el código Pyhton del servidor son evaluados del lado el servidor. El contexto de evaluación tiene los campos los registros actuales disponibles, y se permite la notación de puntos. También están disponibles los registros de la sesión de usuario actual. Al usar `user.id` es equivalente a usar “uid” en el contexto de evaluación del lado del cliente.

Las condiciones de dominio pueden ser combinadas usando los operadores lógicos: `&` para “AND” (el predeterminado), `|` para “OR” y `!` para la negación.

La negación es usada antes de la condición que será negada. Por ejemplo, para encontrar todas las tareas que no pertenezca al usuario actual: `['!', ('user_id','=', uid)]`.

El “AND” y “OR” operan en las dos condiciones siguientes. Por ejemplo: para filtrar las tareas del usuario actual o sin un responsable asignado:
```
['|', ('user_id', '=', uid), ('user_id', '=', False)] 
```
Un ejemplo más  complejo, usado en las reglas de registro del lado del servidor:
```
['|', ('message_follower_ids', 'in', [user.partner_id.id]), '|', ('user_id', '=', user.id), ('user_id', '=', False)]
``` 
El dominio filtra todos los registro donde los seguidores (un campo de muchos a muchos) contienen al usuario actual además del resultado de la siguiente condición. La siguiente condición es, nuevamente, la unión de otras dos condiciones: los registros donde el “user_id” es el usuario de la sesión actual o no esta fijado.
 

**Vistas de Formulario**

As	we	have	seen	in	previous	chapters,	form	views	can	follow	a	simple	layout	or	a	business document	layout,	similar	to	a	paper	document. 

We	will	now	see	how	to	design	business	views	and	to	use	the	elements	and	widgets available.	Usually	this	would	be	done	by	inheriting	the	base	view.	But	to	make	the	code simpler,	we	will	instead	create	a	completely	new	view	for	to-do	tasks	that	will	override	the previously	defined	one. 

In	fact,	the	same	model	can	have	several	views	of	the	same	type.	When	an	action	asks	to open	a	view	type	for	a	model,	the	one	with	the	lowest	priority	is	picked.	Or	as	an alternative,	the	action	can	specify	the	exact	identifier	of	the	view	to	use.	The	action	we defined	at	the	beginning	of	this	chapter	does	just	that;	the	view_id	tells	the	action	to specifically	use	the	form	with	ID } `view_form_todo_task_ui`.	This	is	the	view	we	will create	next. 
 

**Business	views**

In	a	business	application	we	can	differentiate	auxiliary	data	from	main	business	data.	For example,	in	our	app	the	main	data	is	the	to-do	tasks,	and	the	tags	and	stages	are	auxiliary tables	used	by	it. 

These	business	models	can	use	improved	business	view	layouts	for	a	better	user experience.	If	you	recall	the	to-do	task	form	view	added	in	 Odoo Development Essentials - Daniel Reiss.html#79 Chapter	2 ,	*Building	Your	First Odoo	Application*,	it	was	already	following	the	business	view	structure. 

The	corresponding	form	view	should	be	added	after	the	actions	and	menu	items	we	added before,	and	its	generic	structure	is	this: 
```
<record	id="view_form_todo_task_ui"	model="ir.ui.view"> 		<field	name="name">view_form_todo_task_ui</field> 		<field	name="model">todo.task</field> 		<field	name="arch"	type="xml"> 				<form>	<header>	<!--	Buttons	and	status	widget	-->	</header> 						<sheet>		<!--	Form	content	-->	</sheet> 						<!--	History	and	communication:	--> 						<div	class="oe_chatter"> 								<field	name="message_follower_ids" 															widget="mail_followers"	/> 								<field	name="message_ids" 															widget="mail_thread"	/> 						</div>	</form> 
		</field> </record> 
```
Business	views	are	composed	of	three	visual	areas: 

- A	top	header 
- A	sheet	for	the	content 
- A	bottom	history	and	communication	section 

The	history	and	communication	section,	with	the	social	network	widgets	at	the	lower	end is	added	by	inheriting	our	model	from	mail.thread	(from	the	mail	module),	and	adding at	the	end	of	the	form	view	the	elements	in	the	XML	sample	as	previously	mentioned. We’ve	also	seen	this	in Chapter	3 ,	*Inheritance	-	Extending	Existing	Applications*. 
 

**The	header	status	bar**

The	status	bar	on	top	usually	features	the	business	flow	pipeline	and	action	buttons. 

The	action	buttons	are	regular	form	buttons,	and	the	most	common	next	steps	should	be highlighted,	using	class="oe_highlight".	In	todo_ui/todo_view.xml	we	can	now expand	the	empty	header	to	add	a	status	bar	to	it: 
```
<header> 		<field	name="stage_state"	invisible="True"	/> 		<button	name="do_toggle_done"	type="object" 										attrs="{'invisible': 																	[('stage_state','in',['done','cancel'])]}" 										string="Toggle	Done"	class="oe_highlight"	/> 		<!--	Add	stage	statusbar:		...	--> </header> 
```
Depending	on	where	in	the	process	the	current	document	is,	the	action	buttons	available could	differ.	For	example,	a	Set	as	Done 	button	doesn’t	make	sense	if	we	are	already	in the	“Done”	state. 

This	can	be	done	using	the	states	attribute,	listing	the	states	where	the	button	should	be visible,	like	this:	`states="draft,open"`.
 
For	more	flexibility	we	can	use	the	attrs	attribute,	forming conditions	where	the	button should	be	made	invisible:	`attrs="{'invisible' [('stage_state','in', ['done','cancel'])]`.
 
These	visibility	features	are	also	available	for	other	view	elements,	and	not	only	for buttons.	We	will	be	exploring	that	in	more	detail	later	in	this	chapter. 


**The	business	flow	pipeline**

The	business	flow	pipeline	is	a	status-bar	widget	on	a	field	that	represents	the	point	in	the flow	where	the	record	is.	This	is	usually	a	State 	selection	field,	or	a	Stage 	many	to	one field.	Both	cases	can	be	found	across	several	Odoo	modules. 

The	Stage 	is	a	many	to	one	field	using	a	model	where	the	process	steps	are	defined. Because	of	this	they	can	be	easily	configured	by	end	users	to	fit	their	specific	business process,	and	are	perfect	to	support	kanban	boards. 

The	State 	is	a	selection	list	featuring	rather	stable	major	steps	in	a	process,	such	as	New , In	Progress ,	or	Done .	They	are	not	configurable	by	end	users	but	on	the	other	hand	are easier	to	use	in	business	logic.	States	also	have	special	support	for	views:	the	state attribute	allows	for	an	element	to	be	selectively	available	to	the	user	depending	on	the state	of	the	record. 

*Tip*  
*It	is	possible	to	benefit	from	the	best	of	both	worlds,	by	using	stages	that	are	also	mapped into	states.	This	was	what	we	did	in	the	previous	chapter,	by	making	the	State 	available	in to-do	task	documents	through	a	computed	field.*
 
To	add	a	stage	pipeline	to	our	form	header: 
```
<!--	Add	stage	statusbar:	...	--> <field	name="stage_id"	widget="statusbar" 							clickable="True"	options="{'fold_field':	'fold'}"	/> 
```
The	clickable	attribute	enables	clicking	on	the	widget,	to	change	the	document’s	stage	or state.	We	may	not	want	this	if	the	progress	through	process	steps	should	be	done	only through	action	buttons. 
In	the	options	attribute	we	can	use	some	specific	settings: 
fold_field,	when	using	stages,	is	the	name	of	the	field	that	the	stage	model	uses	to indicate	which	stages	should	be	shown	folded. 
statusbar_visible,	when	using	states,	lists	the	states	that	should	be	always	made visible,	to	keep	hidden	the	other	exception	states	used	for	less	common	cases. Example:	`statusbar_visible="draft,open,done"`.
 
The	sheet	canvas	is	the	area	of	the	form	containing	the	main	form	elements.	It	is	designed to	look	like	an	actual	paper	document,	and	its	data	records	are	sometimes	referred	to	as documents. 

The	document	structure	in	general	has	these	components: 

- Title	and	subtitle	information 
- A	smart	button	area,	on	the	top	right Document	header	fields 
- A	notebook	with	tab	pages,	with	document	lines	or	other	details 
Title	and	subtitle  

When	using	the	sheet	layout,	fields	outside	a	`<group>`	block	won’t	have	automatic	labels displayed.	It’s	up	to	the	developer	to	control	if	and	where	to	display	the	labels. 

HTML	tags	can	also	be	used	to	make	the	title	shine.	For	best	results,	the	document	title should	be	in	a	div	with	the	oe_title	class: 
```
<div	class="oe_title"> 		<label	for="name"	class="oe_edit_only"/> 		<h1><field	name="name"/></h1> 		<h3> 				<span	class="oe_read_only">By</span> 				<label	for="user_id"	class="oe_edit_only"/> 				<field	name="user_id"	class="oe_inline"	/> 		</h3> </div> 
```
Here	we	can	see	the	use	of	regular	HTML	elements	such	as	div,	span,	h1	and	h3. 


**Labels	for	fields**

Outside	`<group>`	sections	the	field	labels	are	not	automatically	displayed,	but	we	can display	them	using	the	`<label>`	element: 
The	for	attribute	identify	the	field	to	get	the	label	text	from 
 
The	string	attribute	to	override	the	field’s	original	label	text 
With	the	class	attribute	to	we	can	also	use	CSS	classes	to	control	their	presentation.	Some useful	classes	are: 

- oe_edit_only	to	display	only	when	the	form	is	in	edit	mode 
- oe_read_only	to	display	only	when	the	form	is	in	read	mode 

An	interesting	example	is	to	replace	the	text	with	an	icon: 
```
<label	for="name"	string="	"	class="fa	fa-wrench"	/> 
```
Odoo	bundles	the	Font	Awesome	icons,	being	used	here.	The	available	icons	can	be browsed	at	 [http://fontawesome.org](http://fontawesome.org).
  

**Smart	buttons**
  
The	top	right	area	can	have	an	invisible	box	to	place	smart	buttons.	These	work	like regular	buttons	but	can	include	statistical	information.	As	an	example	we	will	add	a	button displaying	the	total	number	of	to-dos	for	the	owner	of	the	current	to-do. 
First	we	need	to	add	the	corresponding	computed	field	to	todo_ui/todo_model.py.	Add the	following	to	the	TodoTask	class: 
```
@api.one def	compute_user_todo_count(self): 				self.user_todo_count	=	self.search_count( 								[('user_id',	'=',	self.user_id.id)]) 
user_todo_count	=	fields.Integer( 			'User	To-Do	Count', 				compute='compute_user_todo_count') 
```
Now	we	will	add	the	button	box	with	one	button	inside	it.	Right	after	the	end	of	the 
oe_title	div	block,	add	the	following: 
```
<div	name="buttons"	class="oe_right	oe_button_box"> 		<button	class="oe_stat_button" 										type="action"	icon="fa-tasks" 										name="%(todo_app.action_todo_task)d" 										string="" 										context="{'search_default_user_id':	user_id, 																				'default_user_id':	user_id}" 										help="Other	to-dos	for	this	user"	> 
						<field	string="To-dos"	name="user_todo_count" 													widget="statinfo"/> 		</button> </div> 
```
The	container	for	the	buttons	is	a	div	with	the	oe_button_box	class	and	also	oe_right,	to have	it	aligned	to	the	right	hand	side	of	the	form. 

In	the	example	the	button	displays	the	total	number	of	to-do	tasks	the	document responsible	has.	Clicking	on	it	will	browse	them,	and	if	creating	new	tasks	the	original responsible	will	be	used	as	default. 

The	button	attributes	used	are: 

- class="oe_stat_button"	is	to	use	a	rectangle	style	instead	of	a	button. 
- icon	is	the	icon	to	use,	chosen	from	the	Font	Awesome	icon	set. 
- type	will	be	usually	action,	for	a	window	action,	and	name	will	be	the	ID	of	the action	to	execute.	It	can	be	inserted using	the	formula	`%(action-external-id)d`,	to translate	the	external	ID	into	the	actual	ID	number.	This	action	is	expected	to	open	a view	with	related	records. 
- string	can	be	used	to	add	text	to	the	button.	It	is	not	used	here	because	the	contained field	already	provides	the	text	for	it. 
- context	will	set	defaults	on	the	target	view,	when	clicking	through	the	button,	to filter	data	and	set	default	values	for	new	records	created. 
- help	is	the	tooltip	to	display. 

The	button	itself	is	a	container	and	can	have	inside	it’s	fields	to	display	statistics.	These are	regular	fields	using	the	widget	statinfo.	

The	field	should	be	a	computed	field, defined	in	the	underlying	module.	We	can	also	use	static	text	instead	or	alongside	the 
statinfo	fields,	such	as:	`<div>User's	To-dos</div>`

 
**Organizing	content	in	a	form**

The	main	content	of	the	form	should	be	organized	using	<group>	tags.	A	group	is	a	grid with	two	columns.	A	field	and	its	label	take	two	columns,	so	adding	fields	inside	a	group will	have	them	stacked	vertically. 

If	we	nest	two	`<group>`	elements	inside	a	top	group,	we	will	be	able	to	get	two	columns	of fields	with	labels,	side	by	side. 
```
<group	name="group_top"> 		<group	name="group_left"> 				<field	name="date_deadline"	/> 				<separator	string="Reference"	/> 				<field	name="refers_to"	/> 		</group> 		<group	name="group_right"> 				<field	name="tag_ids"	widget="many2many_tags"/> 		</group> </group> 
```
Groups	can	have	a	string	attribute,	used	as	a	title	for	the	section.	Inside	a	group	section, titles	can	also	be	added	using	a	separator	element. 

*Tip*  
*Try	the	Toggle	Form	Layout	Outline 	option	of	the	Developer 	menu:	it	draws	lines around	each	form	section,	allowing	for	a	better	understanding	of	how	the	current	view	is organized.*


**Tabbed	notebooks**

Another	way	to	organize	content	is	the	notebook,	containing	multiple	tabbed	sections called	pages.	These	can	be	used	to	keep	less	used	data	out	of	sight	until	needed	or	to organize	a	large	number	of	fields	by	topic. 

We	won’t	need	this	on	our	to-do	task	form,	but	here	is	an	example	that	could	be	added	in the	task	stages	form: 
```
<notebook> 		<page	string="Whiteboard"	name="whiteboard"> 				<field	name="docs"	/> 		</page> 		<page	name="second_page"> 				<!--	Second	page	content	--> 		</page> </notebook> 
```
It	is	good	practice	to	have	names	on	pages,	to	make	it	more	reliable	for	other	modules	to extend	them. 
   

**View	elements**

We	have	seen	how	to	organize	the	content	in	a	form,	using	elements	such	as	header,  group,	and	notebook.	Now,	we	can	take	a	closer	look	at	the	field	and	button	elements,	and what	we	can	do	with	them. 
 

**Buttons**
  
Buttons	support	these	attributes: 

- icon	to	display.	Unlike	smart	buttons,	icons	available	for	regular	buttons	are	those found	in	`addons/web/static/src/img/icons`. 
- string	is	the	button	text	description. 
- type	can	be	workflow,	object	or	action,	to	either	trigger	a	workflow	signal,	call	a Python	method,	or	run	a	window	action. 
- name	is	the	workflow	trigger,	model	method,	or	window	action	to	run,	depending	on the	button	type. 
- args	can	be	used	to	pass	additional	parameters	to	the	method,	if	the	type	is	object. 
- context	sets	values	on	the	session	context,	which	can	have	an	effect	after	the windows	action	is	run,	or	when	a	Python	method	is	called.	In	the	latter	case,	it	can sometimes	be	used	as	an	alternative	to	args. 
- confirm	adds	a	dialog	with	this	message	text	asking	for	a	confirmation. 
- `special="cancel"`	is	used	on	wizards,	to	cancel	and	close	the	form.	It	should	not	be used	with	type. 
 

**Fields**
  
Fields	have	these	attributes	available	for	them.	Most	are	taken	from	what	was	defined	in the	model,	but	can	be	overridden	in	the	view. 
General	attributes: 

- name:	identifies	the	field	technical	name. 
- string:	provides	label	text	description	to	override	the	one	provided	by	the	model. 
- help:	tooltip	text	to	use	replace	the	one	provided	by	the	model. 
- placeholder:	provides	suggestion	text	to	display	inside	the	field. 
- widget:	overrides	the	default	widget	used	for	the	field’s	type.	We	will	explore	the available	widgets	a	bit	later	in	the	chapter. 
- options:	holds	additional	options	to	be	used	by	the	widget. 
- class:	provides	CSS	classes	to	use	for	the	field’s	HTML. 
- `invisible="1"`:	makes	the	field	invisible. 
- `nolabel="1"`:	does	not	display	the	field’s	label,	it	is	only	meaningful	for	fields	inside a `<group>`	element. 
- `readonly="1"`:	makes	the	field	non	editable. 
- `required="1"`:	makes	the	field	mandatory. 

Attributes	specific	for	some	field	types: 

- sum,	avg:	for	numeric	fields,	and	in	list/tree	views,	add	a	summary	at	the	end	with	the total	or	the	average	of	the	values. 
- `password="True"`:	for	text	fields,	displays	the	field	as	a password	field. 
- filename:	for	binary	fields,	is	the	field	for	the	name	of	the	file. 
- `mode="tree"`:	for	One2many	fields,	is	the	view	type	to	use	to	display	the	records.	By default	it	is	tree,	but	can	also	be	form,	kanban	or	graph. 

For	the	Boolean	attributes	in	general,	we	can	use	True	or	1	to	enable	and	False	or	0	to disable	them.	For	example,	`readonly="1"`	and	`readonly="True"`	are	equivalent. 


**Relational	fields**

On	relational	fields,	we	can	have	some	additional	control	on	what	the	user	is	allowed	to do.	By	default,	the	user	can	create	new	records	from	these	fields	(also	known	as	quick create)	and	open	the	related	record	form.	This	can	be	disabled	using	the	options	field attribute: 
```
options={'no_open':	True,	'no_create':	True} 
```
The	context	and	domain	are	also	particularly	useful	on	relational	fields.	The	context	can define	default	values	for	the	related	records,	and	the	domain	can	limit	the	selectable records,	for	example,	based	on	another	field	of	the	current	record.	Both	context	and  domain	can	be	defined	in	the	model,	but	they	are	only	used	on	the	view. 


**Field	widgets**

Each	field	type	is	displayed	in	the	form	with	the	appropriate	default	widget.	But	other additional	widgets	are	available	and	can	be	used	as	well: 

Widgets	for	text	fields: 

- email:	makes	the	e-mail	text	an	actionable	mail-to	address. 
- url:	formats	the	text	as	a	clickable	URL. 
- html:	expects	HTML	content	and	renders	it;	in	edit	mode	it	uses	a	WYSIWYG	editor to	format	the	content	without	the	need	to	know	HTML. 

Widgets	for	numeric	fields: 

- handle:	specifically	designed	for	sequence	fields,	this	displays	a	handle	to	drag	lines in	a	list	view	and	manually	reorder	them. 
- float_time:	formats	a	float	value	as	time	in	hours	and	minutes. 
- monetary:	displays	a	float	field	as	a	currency	amount.	The	currency	to	use	can	be taken	from	a	field,	such	as `options="{'currency_field':	'currency_id'}"`. 
- progressbar:	presents	a	float	as	a	progress	percentage,	usually	it	is	used	on	a computed	field	calculating	a	completion	rate. 

Some	widgets	for	relational	and	selection	fields: 

- many2many_tags:	displays	a	many	to	many	field	as	a	list	of	tags. 
- selection:	uses	the	Selection	field	widget	for	a	many	to	one	field. 
- radio:	allows	picking	a	value	for	a	selection	field	option	using	radio	buttons. 
- `kanban_state_selection`:	shows	a	semaphore	light	for	the	kanban	state	selection list. 
- priority:	represents	a	selection	as	a	list	of	clickable	stars. 


**On-change	events**

Sometimes	we	need	the	value	for	a	field	to	be	automatically	calculated	when	another	field is	changed.	The	mechanism	for	this	is	called	on-change. 

Since	version	8,	the	on-change	events	are	defined	on	the	model	layer,	without	the	need	for any	specific	markup	on	the	views.	This	is	done	by	creating	the	methods	to	perform	the calculations	and	binding	them	to	the	triggering	field(s)	using	a	decorator `@api.onchange('field1',	'field2')`.
 
In	previous	versions,	this	binding	was	done	in	the	view	layer,	using	the	onchange	field attribute	to	set	the	class	method	called	when	that	field	was	changed.	This	is	still	supported, but	is	deprecated.	Be	aware	that	the	old-style	on-change	methods	can’t	be	extended	using the	new	API.	If	you	need	to	do	that,	you	should	use	the	old	API. 
 

**Dynamic	views**

The	elements	visible	as	a	form	can	also	be	changed	dynamically,	depending,	for	example, on	the	user’s	permissions	or	the	process	stage	the	document	is	in. 

These	two	attributes	allow	us	to	control	the	visibility	of	user	interface	elements: 

- groups:	makes	the	element	visible	only	for	members	of	the	specified	security	groups. It	expects	a	comma	separated	list	of	group’s	XML	IDs. 
- states:	makes	the	element	visible	only	when	the	document	is	in	the	specified	state.	It expects	a	comma-separated	list	of	State	codes,	and	the	document	model	must	have	a 
state	field. 

For	more	flexibility,	we	can	instead	set	an	element’s	visibility	using	client-side	evaluated expressions.	This	is	done	using	the	attrs	attribute	with	a	dictionary	mapping	the  invisible	attribute	to	the	result	of	a	domain	expression. 

For	example,	to	have	the	refers_to 	field	visible	in	all	states	except	draft: 
```
<field	name="refers_to"	 							attrs="{'invisible':	[('state','=','draft')]}"	/> 
```
The	invisible	attribute	is	available	in	any	element,	not	only	fields.	We	can	use	it	on notebook	pages	or	groups,	for	example. 
The	attrs	can	also	set	values	for	two	other	attributes:	readonly	and	required,	but	these only	make	sense	for	data	fields,	making	them	not	editable	or	mandatory.	With	this	we	can add	some	client	logic	such	as	making	a	field	mandatory,	depending	on	the	value	from another	field,	or	only	from	a	certain	state	onward. 
 

**List	views**

Compared	to	form	views,	list	views	are	much	simpler.	A	list	view	can	contain	fields	and buttons,	and	most	of	their	attributes	for	forms	are	also	valid	here. 

Here	is	an	example	of	a	list	view	for	our	To-do	Tasks: 
```
<record	id="todo_app.view_tree_todo_task"	model="ir.ui.view"> 		<field	name="name">To-do	Task	Tree</field> 		<field	name="model">todo.task</field> 		<field	name="arch"	type="xml"> 				<tree	editable="bottom" 										colors="gray:is_done==True" 										fonts="italic:	state!='open'"	delete="false"> 						<field	name="name"/> 						<field	name="user_id"/> 				</tree> 			</field> </record> 
```
The	attributes	for	the	tree	top	element	are: 
editable:	makes	the	records	editable	directly	on	the	list	view.	The	possible	values are	top	and	bottom,	the	location	where	new	records	will	be	added. 

- colors:	dynamically	sets	the	text	color	for	the	records,	based	on	their	content.	It	is	a semicolon-separated	list	of	color:condition	values.	The	color	is	a	CSS	valid	color (see [http://www.w3.org/TR/css3-color/#html4](http://www.w3.org/TR/css3-color/#html4) ),	and	the	condition	is	a	Python expression	to	evaluate	on	the	context	of	the	current	record. 
fonts:	dynamically	modifies	the	font	for	the	records	based	on	their	content.	Similar to	the	colors	attribute,	but	instead	sets	a	font	style	to	bold,	italic	or	underline.
 
create,	delete,	edit:	if	set	to	false	(in	lowercase),	these	disable	the	corresponding action	on	the	list	view. 
 

**Search	views**

The	search	options	available	on	views	are	defined	with	a	search	view.	It	defines	the	fields to	be	searched	when	typing	in	the	search	box	It	also	provides	predefined	filters	that	can	be activated	with	a	click,	and	data	grouping	options	for	the	records	on	list	and	kanban	views. 

Here	is	a	search	view	for	the	to-do	tasks: 
```
<record	id="todo_app.view_filter_todo_task" 								model="ir.ui.view"> 		<field	name="name">To-do	Task	Filter</field> 		<field	name="model">todo.task</field> 		<field	name="arch"	type="xml"> 				<search> 						<field	name="name"	domain_filter="['|', 										('name','ilike',self),('user_id','ilike',self)]"/> 						<field	name="user_id"/> 						<filter	name="filter_not_done"	string="Not	Done" 														domain="[('is_done','=',False)]"/> 						<filter	name="filter_done"	string="Done" 														domain="[('is_done','!=',False)]"/> 						<separator/> 						<filter	name="group_user"	string="By	User" 														context="{'group_by':	'user_id'}"/> 				</search> 		</field> </record>
``` 
We	can	see	two	fields	to	be	searched	for:	name	and	user_id.	On	name	we	have	a	custom filter	rule	that	makes	the	“if	search”	both	on	the	description	and	on	the	responsible	user. Then	we	have	two	predefined	filters,	filtering	the	not	done	and	done	tasks.	These	filters can	be	activated	independently,	and	will	be	joined	with	an	“OR”	operator	if	both	are enabled.	Blocks	of	filters	separated	with	a	`<separator/>`	element	will	be	joined	with	an “AND”	operator. 

The	third	filter	only	sets	a	group-by	context.	This	tells	the	view	to	group	the	records	by that	field,	user_id	in	this	case. 
The	field	elements	can	use	these	attributes:
 
- name:	identifies	the	field	to	use. 
- string:	provides	a	label	text	to	use	instead	of	the	default. 
- operator:	allows	us	to	use	a	different	operator	other	than	the	default	–	`=`	for numeric	fields	and	ilike	for	the	other	field	types. 
- filter_domain:	can	be	used	to	set	a	specific	domain	expression	to	use	for	the	search, providing	much	more	flexibility	than	the	operator	attribute.	The	text	being	searched for	is	referenced	in	the	expression	using	self. 
- groups:	makes	the	search	on	the	field	available	only	for	a	list	of	security	groups (identified	by	XML	IDs). 

For	the	filter	elements	these	are	the	attributes	available: 
 
- name:	is	an	identifier	to	use	for	inheritance	or	for	enabling	it	through  `search_default_`	keys	in	the	context	of	window	actions. 

- string:	provides	label	text	to	display	for	the	filter	(required). 
- domain:	provides	the	filter	domain	expression	to	be	added	to	the	active	domain. 
- context:	is	a	context	dictionary	to	add	to	the	current	context.	Usually	this	sets	a  group_by	key	with	the	name	of	the	field	to	group	the	records. 
- groups:	makes	the	search	filter	available	only	for	a	list	of	groups. 
 

** Other	types	of	views**

The	most	frequent	view	types	used	are	the	form	and	list	views,	discussed	until	now.	Other than	these,	a	few	other	view	types	are	available,	and	we	will	give	a	brief	overview	on	each of	them.	Kanban	views	won’t	be	addressed	now,	since	we	will	cover	them	in Chapter	8,  *QWeb	–	Creating	Kanban	Views	and	Reports*. 

Remember	that	the	view	types	available	are	defined	in	the	view_mode	attribute	of	the corresponding	window	action. 
 

**Calendar	views**
  
As	the	name	suggests,	this	view	presents	the	records	in	a	calendar.	A	calendar	view	for	the to-do	tasks	could	look	like	this: 
```
<record	id="view_calendar_todo_task"	model="ir.ui.view"> 		<field	name="name">view_calendar_todo_task</field> 		<field	name="model">todo.task</field> 		<field	name="arch"	type="xml"> 				<calendar	date_start="date_deadline"	color="user_id" 														display="[name],	Stage	[stage_id]"> 						<!--	Fields	used	for	the	text	of	display	attribute	--> 						<field	name="name"	/> 						<field	name="stage_id"	/> 				</calendar> 		</field> </record> 
```
The	calendar	attributes	are	these: 

- date_start:	This	is	the	field	for	the	start	date	(mandatory). 
- date_end:	This	is	the	field	for	the	end	date	(optional). 
- date_delay:	This	is	the	field	with	the	duration	in	days.	This	is	to	be	used	instead	of  date_end. 
- color:	This	is	the	field	used	to	color	the	calendar	entries.	Each	value	in	the	field	will be	assigned	a	color,	and	all	its	entries	will	have	the	same	color. 
- display:	This	is	the	text	to	be	displayed	in	the	calendar	entries.	Fields	can	be	inserted using	[<field>].	These	fields	must	be	declared	inside	the	calendar	element. 
 

**Gantt	views**

This	view	presents	the	data	in	a	Gantt	chart,	which	is	useful	for	scheduling.	The	to-do tasks	only	have	a	date	field	for	the	deadline,	but	we	can	use	it	to	have	a	basic	Gantt	view working: 
```
<record	id="view_gantt_todo_task"	model="ir.ui.view"> 		<field	name="name">view_gantt_todo_task</field> 		<field	name="model">todo.task</field> 		<field	name="arch"	type="xml"> 				<gantt	date_start="date_deadline" 											default_group_by="user_id"	/> 		</field> </record> 
```
Attributes	that	can	be	used	for	Gantt	views	are	as	follows: 

- date_start:	This	is	the	field	for	the	start	date	(mandatory). 
- date_stop:	This	is	the	field	for	the	end	date.	It	can	be	replaced	by	the	date_delay. 
- date_delay:	This	is	the	field	with	the	duration	in	days.	It	can	be	used	instead	of 
date_stop. 
- progress:	This	is	the	field	that	provides	completion	percentage	(between	0	and	100). 
- default_group_by:	This	is	the	field	used	to	group	the	Gantt	tasks. 
 

**Graph	views**

The	graph	view	type	provides	a	data	analysis	of	the	data,	in	the	form	of	a	chart	or	an interactive	pivot	table. 
We	will	add	a	pivot	table	to	the	to-do	tasks.	First,	we	need	a	field	to	be	aggregated.	In	the 
TodoTask	class,	in	the	`todo_ui/todo_model.py`	file,	add	this	line:
``` 
				effort_estimate	=	fields.Integer('Effort	Estimate') 
```
This	should	also	be	added	to	the	to-do	task	form	so	that	we	can	set	some	data	on	it.	Now, let’s	add	the	graph	view	with	a	pivot	table: 
```
<record	id="view_graph_todo_task"	model="ir.ui.view"> 		<field	name="name">view_graph_todo_task</field> 		<field	name="model">todo.task</field> 		<field	name="arch"	type="xml"> 				<graph	type="pivot"> 						<field	name="stage"	type="col"	/> 						<field	name="user_id"	/> 						<field	name="date_deadline"	interval="week"	/> 						<field	name="effort_estimate"	type="measure"	/> 				</graph> 		</field> </record> 
```
The	graph	element	has	a	type	attribute	set	to	pivot.	It	can	also	be	bar	(default),	pie,	or line.	In	the	case	of	bar,	an	additional	stacked="True"	can	be	used	to	make	it	a	stacked bar	chart. 

The	graph	should	contain	fields	that	have	these	possible	attributes: 

- name:	This	identifies	the	field	to	use	in	the	graph,	as	in	other	views. 
- type:	This	describes	how	the	field	will	be	used,	as	a	row	group	(default),	as	a	col group	(column),	or	as	a	measure. 
- interval:	only	meaningful	for	date	fields,	this	is	the	time	interval	used	to	group	time data	by	day,	week,	month,	quarter	or	year. 
 

**Summary**

You	learned	more	about	Odoo	views	used	to	build	the	user	interface.	We	started	by	adding menu	options	and	the	window	actions	used	by	them	to	open	views.	The	concepts	of context	and	domain	were	explained	in	more	detail	in	following	sections. 

You	also	learned	about	designing	list	views	and	configuring	search	options	using	search views.	Next,	we	had	an	overview	of	the	other	view	types	available:	calendar,	Gantt,	and graph.	Kanban	views	will	be	explored	later,	when	you	learn	how	to	use	QWeb. 

We	have	already	seen	models	and	views.	In	the	next	chapter,	you	will	learn	how	to implement	server-side	business	logic. 