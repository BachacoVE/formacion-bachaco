Chapter 3. Herencia - Extendiendo la Funcionalidad de las Aplicaciones Existentes
===

Una de las características más poderosas de Odoo es la capacidad para agregar características son modificar directamente los objetos subyacentes. Esto se logra a través de mecanismos de herencia, que funcionan como capas para la modificación por encima de los objetos existentes. 

Estas modificaciones puede suceder en todos los niveles: modelos, vistas, y lógica de negocio. En vez de modificar directamente un módulo existente, creamos un módulo nuevo para agregar las modificaciones previstas.

Aquí, aprenderá como escribir sus propios módulos de extensión, confiriendole facultades para aprovechar las aplicaciones base o comunitarias. Como ejemplo, aprenderá como agregar las características de mensajería y redes sociales de Odoo a sus propios módulos.
 
![122_1](/images/Odoo Development Essentials - Daniel Reis-122_1.jpg)


**Agregar la capacidad de compartir con otros a la aplicación To-Do**

Nuestra aplicación To-Do actualmente permite a los usuarios y las usuarias gestionar de forma privada sus tareas por hacer. ¿No sería grandioso llevar nuestra aplicación a otro nivel agregando características colaborativas y de redes sociales? Seriamos capaces de compartir las tareas y discutirlas con otras personas.

Haremos esto con un módulo nuevo para ampliar la funcionalidad de la aplicación To-Do creada anteriormente y agregar estas características nuevas. Esto es lo que esperamos lograr al final de este capítulo:

**Camino a seguir para las características colaborativas**

Aquí  esta nuestro plan de trabajo para la implementar la extensión de funcionalidades:

- Agregar campos al modelo Task, como el usuario quien posee la tarea.
- Modificar la lógica de negocio para operar solo en la tarea actual del usuario, en vez de todas las tareas disponibles para ser vistas por el usuario.
- Agregar los campos necesarios a las vistas.
- Agregar características de redes sociales: el muro de mensajes y los seguidores.

Comenzaremos creando la estructura básica para el módulo junto al módulo `todo_app`. Siguiendo el ejemplo de instalación del Capítulo 1, nuestros módulos estarán alojados en `~/odoo-dev/custom-addons/`: 
```
$ mkdir ~/odoo-dev/custom-addons/todo_user
$ touch ~/odoo-dev/custom-addons/todo_user/__init__.py
``` 
Ahora creamos el archivo `__openerp__.py`, con el siguiente código:
```
{
    'name': 'Multiuser To-Do',
    'description': 'Extend the To-Do app to multiuser.',
    'author': 'Daniel Reis',
    'depends': ['todo_app'],	
}
```
No hemos hecho esto, pero incluir las claves “summary” y “category” puede ser importante cuando se publican módulos en la tienda de aplicaciones en línea de Odoo.

Ahora, podemos instalarlo. Debe ser suficiente con solo actualizar el Lista de Módulos desde el menú Configuraciones, encuentre el módulo nuevo en la lista de Módulos Locales y haga clic en el botón Instalar. Para instrucciones más detalladas sobre como encontrar e instalar un módulo puede volver al Capítulo 1.

Ahora, comencemos a agregar le las nuevas características.


**Ampliando el modelo de tareas por hacer**

Los modelos nuevos son definidos a través de las clases Python. Ampliarlos también es hecho a través de las clases Python, pero usando un mecanismo específico de Odoo.

Para amplicar un modelo usamos una clase Python con un atributo `__inherit`. Este identifica el modelo que será ampliado. La clase nueva hereda todas las características del modelo padre, y solo necesitamos declarar las modificaciones que queremos introducir.

De hecho, los modelos de Odoo existen fuera de nuestro módulo particular, en un registro central. Podemos referirnos a este registro como la piscina, y puede ser accedido desde los métodos del modelo usando `self.env[<model name>]. Por ejemplo, para referirnos al modelo `res.partner` escribiremos `self.env['res.partner'].

Para modificar un modelo de Odoo obtenemos una referencia a la clase de registro y luego ejecutamos los cambios en ella. Esto significa que esas modificaciones también estarán disponibles en cualquier otro lado donde el modelo sea usado.

En la secuencia de carga del módulo, durante un reinicio del servidor, las modificaciones solo serán visibles en los modelos cargados después. Así que, la secuencia de carga es importante y debemos asegurarnos que las dependencias del módulo están fijadas correctamente.


**Agregar campos a un modelo **

Ampliaremos el modelo `todo.task` para agregar un par de campos: el usuario responsable de la tarea, y la fecha de vencimiento.

Cree un archivo `todo_task.py` nuevo y declare una clase que extienda al modelo original:
```
#-*- coding: utf-8 -*- 
from openerp import models, fields, api 

class TodoTask(models.Model): 
    _inherit = 'todo.task'
    user_id	= fields.Many2one('res.users', 'Responsible')
    date_deadline	= fields.Date('Deadline')
```
El nombre de la clase es local para este archivo Python, y en general es irrelevante para los otros módulos. El atributo `_inherit` de la clase es la clave aquí: esta le dice a Odoo que esta clase hereda el modelo `todo.task`. Note la ausencia del atributo `_name`. Este no es necesario porque ya es heredado desde el modelo padre.
Las siguientes dos líneas son declaraciones de campos comúnes. El `user_id` representa un usuario desde el modelo Users, `res.users`. Es un campo de “Many2one” equivalente a una clave foránea en el argot de base de datos. El `date_deadline` es un simple campo de fecha. En el Capítulo 5, explicaremos con mas detalle los tipos de  campos disponibles en Odoo.

Aun nos falta agregar al archivo `__init__.py` la declaración “import” para incluir lo en el módulo:
```
from . import todo_task 
```
Para tener los campos nuevos agregados a la table soportada por el modelo, necesitamos ejecutar una actualización al módulo. Si todo sale como es esperado, debería poder ver los campos nuevos cuando revise el modelo `todo.task`, en el menú Técnico, Estructura de Base de Datos | opción Modelos.

**Modificar los campos existentes**

Como puede ver, agregar campos nuevos a un modelo existente es bastante directo. Desde Odoo 8, es posible modificar atributos en campos existentes. Esto es hecho agregando un campo con el mismo nombre, y configurando los valores solo para los atributos que serán modificados.

Por ejemplo, para agregar un comentario de ayuda a un campo “name”, podríamos agregar esta línea en el archivo `todo_task.py`:
```
name = fields.Char(help="What	needs	to be done?")
``` 
Si actualizamos el módulo, vamos a un formulario de tareas por hacer, y posicionamos el ratón sobre el campo Descripción, aparecerá el mensaje de texto escrito en el código anterior.


**Modificar los métodos del modelo**

La herencia también funciona en la lógica de negocio. Agregar métodos nuevos es simple: solo declare las funciones dentro de la clase heredada.

Para ampliar la lógica existente, un método puede ser sobreescrito declarando otro método con el mismo nombre, y el método nuevo reemplazará al anterior. Pero este puede extender el código de la clase heredada, usando la palabra clave de Python `super()` para llamar al método padre.

Es mejor evitar cambiar la función distintiva del metodo (esto es, mantener los mismos argumentos) para asegurarnos que las llamadas a el sigan funcionando adecuadamente. En caso que necesite agregar parámetros adicionales, hágalos opcionales (con un valor predeterminado).

La acción original de Borrar Todas las tareas Finalizadas no es apropiada para nuestro módulos de tareas compartidas, ya que borra todas las tareas sin importar a quien le pertenecen. Necesitamos modificarla para que borre solo las tareas del usuario actual.

Para esto, sobreescribiremos el método original con una nueva versión que primero encuentre las tareas completadas del usario actual, y luego las desactive:
```
@api.multi def do_clear_done(self):
    domain = [('is_done', '=', True),
             '|', ('user_id', '=', self.env.uid),
             ('user_id', '=', False)]
    done_recs = self.search(domain)
    done_recs.write({'active': False}) 
    return True
```
Primero listamos los registros finalizados sobre los cuales actuaremos usando el método “search” con un filtro de búsqueda. El filtro de búsqueda sigue una sintaxis especial de Odoo referida como dominio.

El filtro “domain” usado es definido en la primera instrucción: es una lista de condiciones, donde cada condición es una tupla.

Estas condiciones son unidas implícitamente con un operador AND (`&` en la sintaxis de dominio). Para agregar una operación OR se usa una “tubería” (`|`) en el lugar de la tupla, y afectara las siguientes dos condiciones. Ahondaremos más sobre este tema en el Capítulo 6.


El dominio usado aquí filtra todas las tareas finalizadas(`'is_done', '=', True`) que también tengan al usuario actual como responsable (`'user_id','=',self.env.uid`) o no tengan fijado un usuario (`'user_id', '=', False`).

Lo que acabamos de hacer fue sobreescribir completamente el método padre, reemplazandolo con una implementación nueva. 

Pero esto no es lo que usualmente querremos hacer. En vez de esto, ampliaremos la lógica actual y agregaremos operaciones adicionales. De lo contrario podemos dañar operaciones existentes. La lógica existente es insertada dentro de un método sobreescrito usando el comando `super()` de Python para llamar a la versión padre del método.

Veamos un ejemplo de esto: podemos escribir una versión mejor de `do_toggle_done()` que solo ejecute la acción sobre las Tareas asignadas a nuestro usuario:
```
@api.one def do_toggle_done(self):
    if self.user_id != self.env.user:
        raise Exception('Only	the responsible can do this!')
    else:
        return super(TodoTask, self).do_toggle_done()
```
Estas son las técnicas básicas para sobreescribir y ampliar la lógica de negocio definida en las clases del modelo. Veremos ahora como extender las vistas de la interfaz con las usuarias y usuarios.


**Ampliar las vistas**

Forms,	lists,	and	search	views	are	defined	using	the	arch	XML	structures.	To	extend views	we	need	a	way	to	modify	this	XML.	This	means	locating	XML	elements	and	then introducing	modifications	on	those	points. 

Inherited	views	allow	just	that.	An	inherited	view	looks	like	this: 
```
<record	id="view_form_todo_task_inherited" model="ir.ui.view">
    <field name="name">Todo Task form – User extension</field>
    <field name="model">todo.task</field>
    <field name="inherit_id" ref="todo_app.view_form_todo_task"/>
    <field name="arch" type="xml">
        <!-- ...match and extend elements here! ... -->
    </field> 
</record>
```
The "inherit_id" field identifies the view to be extended,	by	referring	to	its	external identifier	using	the	special	ref	attribute.	External	identifiers	will	be	discussed	in	more detail	in  Chapter	4 ,	`Data	Serialization	and	Module	Data`.
 
The	natural	way	to	locate	elements	in	XML	is	to	use	XPath	expressions.	For	example, taking	the	form	view	defined	in	the	previous	chapter,	the	XPath	expression	to	locate	the 
`<field	name="is_done">	element	is: //field[@name]='is_done'`.	This	expression	finds a	field	element	with	a	name	attribute	equal	to	"is_done".	You	can	find	more	information	on XPath	at:[https://docs.python.org/2/library/xml.etree.elementtree.html#xpath-support](https://docs.python.org/2/library/xml.etree.elementtree.html#xpath-support).
 
Having	name	attributes	on	elements	is	important	because	it	makes	it	a	lot	easier	to	select them	for	extension	points.	Once	the	extension	point	is	located,	it	can	be	modified	or	have XML	elements	added	near	it. 

As	a	practical	example,	to	add	the	date_deadline	field	before	the	is_done	field,	we would	write	in	the	arch: 
```
<xpath expr="//field[@name]='is_done'" position="before">
    <field name="date_deadline" /> 
</xpath> 
```
Fortunately	Odoo	provides	shortcut	notation	for	this,	so	most	of	the	time	we	can	avoid	the XPath	syntax	entirely.	Instead	of	the	xpath	element	above	we	can	use	the	element	type	we want	to	locate	and	its	distinctive	attributes.	The	above	could	also	be	written	as: `<field	name="is_done"	position="before"> 		<field	name="date_deadline"	/> </field>`
 
Adding	new	fields	next	to	existing	fields	is	done	often,	so	the	`<field>`	tag	is	frequently used	as	the	locator.	But	any	other	tag	can	be	used:	`<sheet>`,	`<group>`,	`<div>`,	and	so	on. The	name	attribute	is	usually	the	best	choice	for	matching	elements,	but	sometimes,	we may	need	to	use	string	(the	displayed	label	text)	or	the	CSS	class	element. 

The	position	attribute	used	with	the	locator	element	is	optional,	and	can	have	the following	values:
 
- after:	This	is	added	to	the	parent	element,	after	the	matched	node. 
- before:	This	is	added	to	the	parent	element,	before	the	matched	node. 
- inside	(the	default	value):	This	is	appended	to	the	content	of	the	matched	node. 
- replace:	This	replaces	the	matched	node.	If	used	with	empty	content,	it	deletes	an element. 
- attributes:	This	modifies	the	XML	attributes	of	the	matched	element	(there	are more	details	described	following	this	list). 

The	attribute	position	allows	us	to	modify	the	matched	element’s	attributes.	This	is	done using	`<attribute	name="attr-name">`	elements	with	the	new	attribute	values. 

In	the	Task	form,	we	have	the	Active 	field,	but	having	it	visible	is	not	that	useful.	Maybe, we	can	hide	it	from	the	user.	This	can	be	done	setting	its	invisible	attribute: 
```
<field	name="active"	position="attributes">
    <attribute	name="invisible">1<attribute/> 
</field> 
```
Setting	the	invisible	attribute	to	hide	an	element	is	a	good	alternative	to	using	the  replace	locator	to	remove	nodes.	Removing	should	be	avoided,	since	it	can	break extension	models	that	may	depend	on	the	deleted	node. 

Finally,	we	can	put	all	of	this	together,	add	the	new	fields,	and	get	the	following	complete inheritance	view	to	extend	the	to-do	tasks	form: 
```
<record	id="view_form_todo_task_inherited" model="ir.ui.view">
    <field	name="name">Todo	Task	form	–	User	extension</field>
    <field	name="model">todo.task</field>
    <field	name="inherit_id"	ref="todo_app.view_form_todo_task"/>
    <field	name="arch"	type="xml">
        <field	name="name"	position="after">
            <field	name="user_id"	/>
        </field>
        <field	name="is_done"	position="before">
            <field	name="date_deadline"	/> 
        </field>
        <field	name="name"	position="attributes">
            <attribute	name="string">I	have	to…<attribute/>
        </field>
    </field> 
</record>
``` 
This	should	be	added	to	a	todo_view.xml	file	in	our	module,	inside	the	`<openerp>`	and `<data>`	tags,	as	shown	in	the	previous	chapter. 

*Note* 
*Inherited	views	can	also	be	inherited,	but	since	this	creates	more	intricate	dependencies,	it should	be	avoided. *

Also,	we	should	not	forget	to	add	the	data	attribute	to	the	`__openerp__.py`	descriptor	file: 
``` 
'data':	['todo_view.xml'], 
``` 

**Extending	tree	and	search	views**
 
Tree	and	search	view	extensions	are	also	defined	using	the	arch	XML	structure,	and	they can	be	extended	in	the	same	way	as	form	views.	We	will	follow	example	of	a	extending the	list	and	search	views. 

For	the	list	view,	we	want	to	add	the	user	field	to	it: 
```
<record	id="view_tree_todo_task_inherited"	model="ir.ui.view">
    <field	name="name">Todo	Task	tree	–	User	extension</field>
    <field	name="model">todo.task</field>
    <field	name="inherit_id"	ref="todo_app.view_tree_todo_task"/>
    <field	name="arch"	type="xml">
        <field	name="name"	position="after">
            <field	name="user_id"	/>
        </field>
    </field> 
</record> 
```
For	the	search	view,	we	will	add	search	by	user,	and	predefined	filters	for	the	user’s	own tasks	and	tasks	not	assigned	to	anyone: 
```
<record	id="view_filter_todo_task_inherited"	model="ir.ui.view">
    <field	name="name">Todo	Task	tree	–	User	extension</field>
    <field	name="model">todo.task</field>
    <field	name="inherit_id"	ref="todo_app.view_filter_todo_task"/>
    <field	name="arch"	type="xml">
        <field	name="name"	position="after">
            <field	name="user_id"	/>
            <filter	name="filter_my_tasks"	string="My	Tasks" 	domain="[('user_id','in',[uid,False])]"	/>
            <filter	name="filter_not_assigned"	string="Not	Assigned" domain="[('user_id','=',False)]" />
        </field>
    </field> 
</record> 
```
Don’t	worry	too	much	about	the	views-specific	syntax.	We’ll	cover	that	in	more	detail	in Chapter	6, 	*Views	-	Designing	the	User	Interface*. 
 

**More	on	using	inheritance	to	extend models** 
We	have	seen	the	basic	in	place	extension	of	models,	which	is	also	the	most	frequent	use of	inheritance.	But	inheritance	using	the	_inherit	attribute	has	more	powerful capabilities,	such	as	mixin 	classes. 
We	also	have	available	the	delegation	inheritance	method,	using	the	_inherits	attribute. It	allows	for	a	model	to	contain	other	models	in	a	transparent	way	for	the	observer,	while behind	the	scenes	each	model	is	handling	its	own	data. 

Let’s	explore	these	possibilities	in	more	detail. 
 

**Copying	features	using	prototype	inheritance**
 
The	method	we	used	before	to	extend	a	model	used	just	the	`_inherit`	attribute.	We defined	a	class	inheriting	the	todo.task	model,	and	added	some	features	to	it.	The	class `_name`	was	not	explicitly	set;	implicitly	it	was	todo.task	also. 

But	using	the	`_name`	attribute	allows	us	to	create	mixin	classes,	by	setting	it	to	the	model we	want	to	extend.	Here	is	an	example: 
```
from	openerp	import	models class	TodoTask(models.Model): 
    _name	=	'todo.task'
    _inherit	=	'mail.thread' 
```
This	extends	the	`todo.task`	model	by	copying	to	it	the	features	of	the	`mail.thread model`.	The	mail.thread	model	implements	the	Odoo	messages	and	followers	features, and	is	reusable,	so	that	it’s	easy	to	add	those	features	to	any	model. 

Copying	means	that	the	inherited	methods	and	fields	will	also	be	available	in	the inheriting	model.	For	fields	this	means	that	they	will	be	also	created	and	stored	in	the target	model’s	database	tables.	The	data	records	of	the	original	(inherited)	and	the	new (inheriting)	models	are	kept	unrelated.	Only	the	definitions	are	shared. 

These	mixins	are	mostly	used	with	abstract	models,	such	as	the	`mail.thread`	used	in	the example.	Abstract	models	are	just	like	regular	models	except	that	no	database representation	is	created	for	them.	They	act	like	templates,	describing	fields	and	logic	to be	reused	in	regular	models.	The	fields	they	define	will	only	be	created	on	those	regular models	inheriting	from	them.	In	a	moment	we	will	discuss	in	detail	how	to	use	this	to	add  `mail.thread`	and	its	social	networking	features	to	our	module.	In	practice	when	using mixins	we	rarely	inherit	from	regular	models,	because	this	causes	duplication	of	the	same data	structures. 

Odoo	provides	the	delegation	inheritance	mechanism,	which	avoids	data	structure duplication,	so	it	is	usually	preferred	when	inheriting	from	regular	models.	Let’s	look	at	it in	more	detail. 
 

**Embedding	models	using	delegation	inheritance**

Delegation	inheritance	is	the	less	frequently	used	model	extension	method,	but	it	can provide	very	convenient	solutions.	It	is	used	through	the	_inherits	attribute	(note	the additional	-s)	with	a	dictionary	mapping	inherited	models	with	fields	linking	to	them. 
A	good	example	of	this	is	the	standard	Users	model,	res.users,	that	has	a	Partner	model embedded	in	it: 
```
from	openerp	import	models,	fields class	User(models.Model):
    _name	=	'res.users'
    _inherits	=	{'res.partner':	'partner_id'}
    partner_id	=	fields.Many2one('res.partner') 
```
With	delegation	inheritance	the	model	res.users	embeds	the	inherited	model `res.partner`,	so	that	when	a	new	User	is	created,	a	partner	is	also	created	and	a	reference to	it	is	kept	in	the	partner_id	field	of	the	User.	It	is	similar	to	the	polymorphism	concept in	object	oriented	programming. 

All	fields	of	the	inherited	model,	Partner,	are	available	as	if	they	were	User	fields,	through the	delegation	mechanism.	For	example,	the	partner	name	and	address	fields	are	exposed as	User	fields,	but	in	fact	they	are	being	stored	in	the	linked	Partner	model,	and	no	data structure	duplication	occurs. 

The	advantage	of	this,	compared	to	prototype	inheritance,	is	that	there	is	no	need	to	repeat data	structures	in	many	tables,	such	as	addresses.	Any	new	model	that	needs	to	include	an address	can	delegate	that	to	an	embedded	Partner	model.	And	if	modifications	are introduced	in	the	partner	address	fields	or	validations,	these	are	immediately	available	to all	the	models	embedding	it! 

*Note* 
*Note	that	with	delegation	inheritance,	fields	are	inherited,	but	methods	are	not.* 
 

**Using	inheritance	to	add	social	network features** 

The	social	network	module	(technical	name	mail)	provides	the	message	board	found	at the	bottom	of	many	forms,	also	called	Open	Chatter,	the followers	are	featured	along	with the	logic	regarding	messages	and	notifications.	This	is	something	we	will	often	want	to add	to	our	models,	so	let’s	learn	how	to	do	it. 

The	social	network	messaging	features	are	provided	by	the `mail.thread`	model	of	the mail	module.	To	add	it	to	a	custom	model	we	need	to: 

- Have	the	module	depend	on	mail. 
- Have	the	class	inherit	from	`mail.thread`. 
- Have	the	Followers	and	Thread	widgets	added	to	the	form	view. 

Optionally,	set	up	record	rules	for	followers. 

Let’s	follow	this	checklist: 

Regarding	*#1*,	since	our	extension	module	depends	on	`todo_app`,	which	in	turn	depends on	mail,	the	dependency	on	mail	is	already	implicit,	so	no	action	is	needed. 

Regarding	*#2*,	the	inheritance	on	mail.thread	is	done	using	the	`_inherit`	attribute	we used	before.	But	our	to-do	task	extension	class	is	already	using	the	`_inherit`	attribute. 

Fortunately	it	can	also	accept	a	list	of	models	to	inherit	from,	so	we	can	use	that	to	make	it also	include	the	inheritance	on	`mail.thread`: 
```
_name	=	'todo.task'
_inherit	=	['todo.task',	'mail.thread'] 
```
The	`mail.thread`	model	is	an	abstract	model.	Abstract	models	are	just	like	regular	models except	that	they	don’t	have	a	database	representation;	no	actual	tables	are	created	for them.	Abstract	models	are	not	meant	to	be	used	directly.	Instead	they	are	expected	to	be used	in	mixin	classes,	as	we	just	did.	We	can	think	of	them	as	templates	with	ready-to-use features.	To	create	an	abstract	class	we	just	need	it	to	use	models. AbstractModel	instead of	`models.Model`. 

For	*#3*,	we	want	to	add	the	social	network	widgets	at	the	bottom	of	the	form.	We	can	reuse the	inherited	view	we	already	created,	`view_form_todo_task_inherited`,	and	add	this into	its	arch	data: 
```
<sheet	position="after">
    <div	class="oe_chatter">
        <field	name="message_follower_ids"	widget="mail_followers"	/>
        <field	name="message_ids"	widget="mail_thread"	/>
    </div>
</sheet> 
```
The	two	fields	we	add	here	haven’t	been	explicitly	declared	by	us,	but	they	are	provided by	the	`mail.thread`	model. 
 
The	final	step	is	to	set	up	record	rules	for	followers.	This	is	only	needed	if	our	model	has record	rules	implemented	that	limit	other	users’	access	to	its	records.	In	this	case,	we	need to	make	sure	that	the	followers	for	each	record	have	at	least	read	access	to	it. 

We	do	have	record	rules	on	the	to-do	task	model	so	we	need	to	address	this,	and	that’s what	we	will	do	in	the	next	section. 
 

**Modifying	data**

Unlike	views,	regular	data	records	don’t	have	an	XML	arch	structure	and	can’t	be extended	using	XPath	expressions.	But	they	can	still	be	modified	to	replace	values	in	their fields. 

The	`<record	id="x"	model="y">`	element	is	actually	performing	an	insert	or	update operation	on	the	model:	if	x	does	not	exist,	it	is	created;	otherwise,	it	is	updated/written over. 

Since	records	in	other	modules	can	be	accessed	using	a	`<model>.<identifier>`	identifier, it’s	perfectly	legal	for	our	module	to	overwrite	something	that	was	written	before	by another	module. 

*Note* 
*Note	that	the	dot	is	reserved	to	separate	the	module	name	from	the	object	identifier,	so they	shouldn’t	be	used	in	identifiers.	Instead	use	the	underscore. *

As	an	example,	let’s	change	the	menu	option	created	by	the	todo_app	module	to	into	My To	Do .	For	that	we	could	add	the	following	to	the	todo_user/todo_view.xml	file: 
```
<!--	Modify	menu	item	-->
<record	id="todo_app.menu_todo_task"	model="ir.ui.menu">
    <field	name="name">My	To-Do</field>
</record> 
<!--	Action	to	open	To-Do	Task	list	-->
<record	model="ir.actions.act_window"	id="todo_app.action_todo_task">
    <field	name="context">
        {'search_default_filter_my_tasks':	True}
    </field>
</record> 
```
 
**Extending	the	record	rules**

The	To-Do	application	included	a	record	rule	to	ensure	that	each	task	would	only	be visible	to	the	user	that	created	it.	But	now,	with	the	addition	of	the	social	features,	we	need the	task	followers	to	also	have	access	to	them.	The	social	network	module	does	not	handle this	by	itself. 

Also,	now	tasks	can	have	users	assigned	to	them,	so	it	makes	more	sense	to	have	the access	rules	to	work	on	the	responsible	user	instead	of	the	user	who	created	the	task. 
The	plan	would	be	the	same	as	we	did	for	the	menu	item: overwrite	the `todo_app.todo_task_user_rule`	to	modify	the	domain_force	field	to	a	new	value.
 
Unfortunately	this	won’t	work	this	time.	Remember	the	`<data	no_update="1">`	we	used in	the	security	rules	XML	file:	it	prevents	later	write	operations	on	it. 

Since	updates	on	that	record	are	being	prevented,	we	need	a	workaround.	That	will	be	to delete	that	record	and	add	a	replacement	for	it	in	our	module. 

To	keep	things	organized,	we	will	create	a	security/todo_access_rules.xml	file	and add	the	following	content	to	it: 
```
<?xml	version="1.0"	encoding="utf-8"?> 
    <openerp>
        <data	noupdate="1"> 
	    <delete	model="ir.rule"	search="[('id',	'=',ref('todo_app.todo_task_user_rule'))]" /> 
	    <record	id="todo_task_per_user_rule"	model="ir.rule">
                <field	name="name">ToDo	Tasks	only	for	owner</field>
                <field	name="model_id"	ref="model_todo_task"/>
                <field	name="groups"	eval="[(4,	ref('base.group_user'))]"/>
                <field	name="domain_force">
                    ['|',('user_id','in',	[user.id,False]),
                    ('message_follower_ids','in',[user.partner_id.id])]
                </field>
            </record> 
        </data>
    </openerp> 
```
This	finds	and	deletes	the	`todo_task_user_rule`	record	rule	from	the	todo_app	module, and	then	creates	a	new	`todo_task_per_user_rule`	record	rule.	The	domain	filter	we	will now	use	makes	a	task	visible	to	the	responsible	user	`user_id`,	to	everyone	if	the responsible	user	is	not	set	(equals	False),	and	to	all	followers.	The	rule	will	run	in	a context	where	user	is	available	and	represents	the	current	session	user.	The	followers	are partners,	not	User	objects,	so	instead	of	`user.id`,	we	need	to	use	`user.partner_id.id`. 

*Tip* 
*Working	on	data	files	with	`<data	noupdate="1">`	is	tricky	because	any	later	edit	won’t	be updated	on	Odoo.	To	avoid	that,	temporarily	use	`<data	noupdate="0">`	during development,	and	change	it	back	only	when	you’re	done	with	the	module.*
 
As	usual,	we	must	not	forget	to	add	the	new	file	to	the	`__openerp__.py`	descriptor	file	in the	data	attribute: 
```
'data':	['todo_view.xml', 'security/todo_access_rules.xml'], 
```
Notice	that	on	module	upgrade,	the	`<delete>`	element	will	produce	an	ugly	warning message,	because	the	record	to	delete	does	not	exist	anymore.	It	is	not	an	error	and	the upgrade	will	be	successful,	so	we	don’t	need	to	worry	about	it.
 
 
**Summary** 

You	should	now	be	able	to	create	new	modules	to	extend	existing	modules.	We	saw	how to	extend	the	To-Do	module	created	in	the	previous	chapter. 

New	features	were	added	onto	the	several	layers	that	make	up	an	application.	We	extended the	Odoo	model	to	add	new	fields,	and	extended	the	methods	with	its	business	logic.	Next, we	modified	the	views	to	make	the	new	fields	available	on	them.	Finally,	you	learned	how to	extend	a	model	by	inheriting	from	other	models,	and	we	used	that	to	add	the	social network	features	to	our	To-Do	app. 

With	these	three	chapters,	we	had	an	overview	of	the	common	activities	involved	in	Odoo development,	from	Odoo	installation	and	setup	to	module	creation	and	extension.	The next	chapters	will	focus	on	specific	areas,	most	of	which	we	visited	briefly	in	these overviews.	In	the	following	chapter,	we	will	address	data	serialization	and	the	usage	of XML	and	CSV	files	in	more	detail. 