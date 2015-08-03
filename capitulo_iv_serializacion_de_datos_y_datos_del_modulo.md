Capítulo	4.	Serialización de Datos y Datos de Módulos
====

La mayoría de las configuraciones de Odoo, desde interfaces de usuario hasta reglas de seguridad, son en realidad registros de datos almacenados en tablas internas de Odoo. Los archivos XML y CSV que se encuentran en los módulos no son usados para ejecutar aplicaciones Odoo. Ellos solo son un medio para cargar esas configuraciones a las tablas de la base de datos.

Los módulos pueden también tener datos iniciales y de demostración (accesorios). La serialización de datos permite añadir eso a nuestros módulos. Adicionalmente, entendiendo los formatos de serialización de datos de Odoo es importante para exportar e importar datos en el contexto de la implementación de un proyecto.

Antes de entrar en casosprácticos, primero exploraremos el conceptop de identificador externo, el cual es la clave a la serialización de datos de Odoo 

![150_1](/images/Odoo Development Essentials - Daniel Reis-150_1.jpg)

Understanding	external	identifiers

All	records	in	the	Odoo	database	have	a	unique	identifier,	the	id	field. 

It	is	a	sequential	number	automatically	assigned	by	the	database.	However,	this	automatic identifier	can	be	a	challenge	when	loading	interrelated	data:	how	can	we	reference	a related	record if	we	can’t	know	beforehand	what	database	ID 	will	be	assigned	to	it? 

Odoo’s	answer	to	this	is	the	external	identifier.	External	identifiers	solve	this	problem	by assigning	named	identifiers	to	the	data	records	to	be	loaded.	A	named	identifier	can	be used	by	any	other	piece	of	record	data	to	reference	it	later	on.	Odoo	will	take	care	of translating	these	identifier	names	into	the	actual	database	IDs assigned	to	them. 

The	mechanism	behind	this	is	quite	simple:	Odoo	keeps	a	table	with	the	mapping	between the	named	External	IDs	and	their	corresponding	numeric	database	IDs.	That	is	the `ir.model.data`	model. 

To	inspect	the	existing	mappings,	go	to	the	Technical 	section	of	the	Settings 	menu,	and select	the	Sequences	&	Identifiers 	|	External	Identifiers 	menu	item. 

For	example,	if	we	visit	the	External	Identifiers 	list	and	filter	it	by	the	`todo_app`	module, we	will	see	the	external	identifiers	generated	by	the	module	created	previously. 

You	can	see	that	the	external	identifiers	have	a	Complete	ID 	label.	This	is	composed	of the	module	name	and	the	identifier	name	joined	by	a	dot,	for	example, `todo_app.action_todo_task`.
 
Since	only	the	Complete	ID 	is	required	to	be	unique,	the	module	name	ends	up	acting	as	a namespace	for	identifiers.	This	means	that	the	same	named	identifier	can	be	repeated	in different	modules,	and	we	don’t	need	to	worry	about	identifiers	in	our	module	colliding with	identifiers	in	other	modules. 
 
![151_1](/images/Odoo Development Essentials - Daniel Reis-151_1.jpg)

At	the	top	of	the	list,	you	can	see	the	`todo_app.action_todo_task`	ID.	This	is	the	menu action	we	created	for	the	module,	which	is	also	referenced	in	the	corresponding	menu item.	By	clicking	on	it,	you	can	open	a	form	with	its	details:	the	`action_todo_task`	in	the `todo_app`	module	maps	to	a	specific	record	ID	in	the	`ir.actions.act_window`	model.
 
Besides	providing	a	way	for	records	to	easily	reference	other	records,	External	IDs	also allow	avoiding	data	duplication	on	repeated	imports.	If	the	External	ID	is	already	present, the	existing	record	will	be	updated,	instead	of	creating	a	new	record.	This	is	why,	on subsequent	module	upgrades,	previously	loaded	records	are	updated	instead	of	being duplicated. 
 
![152_1](/images/Odoo Development Essentials - Daniel Reis-152_1.jpg)


**Finding	External	IDs**

When	preparing	configuration	and	demonstration	data	files	for	modules,	we	frequently need	to	look	up	existing	External	IDs	that	are needed	for	references. 

We	can	use	the	External	Identifiers	menu	shown	earlier,	but	the	Developer	Menu 	can provide	a	more	convenient	method	for	that.	As	you	may	recall	from Chapter	1, 	*Getting Started	with	Odoo	Development*,	the	Developer	Menu 	is	activated	in	the	About	Odoo  option,	and	then,	it	is	available	at	the	top-left	corner	of	the	web	client	view.
 
To	find	the	External	ID	for	a	data	record,	on	the	corresponding	Form	view,	select	the	View Metadata 	option	from	the	Developer	Menu .	This	will	display	a	dialog	with	the	record’s database	ID	and	External	ID	(also	known	as	XML	ID). 

As	an	example,	to	look	up	the	Demo	user	ID,	we	can	navigate	to	its	Form	view 	(Settings 	| Users )	and	select	the	View	Metadata 	option,	after	which	we	will	be	shown	this: 
To	find	the	External	ID	for	view	elements,	such	as	form,	tree,	search,	and	action,	the Developer	Menu 	is	also	a	good	help.	For	that,	use	its	Manage	Views 	option	or	open	the information	for	the	desired	view	using	the	Edit	<view	type> 	options,	and	then	select	their View	Metadata 	option. 
 

**Exporting	and	importing	data**  

We	will	start	exploring	how	data	export	and	import	work	in	Odoo,	and	from	there,	we	will move	on	to	the	more	technical	details. 
 
![155_1](/images/Odoo Development Essentials - Daniel Reis-155_1.jpg)


**Exporting	data**

Data	export	is	a	standard	feature	available	in	any	List	view.	To	use	it,	we	must	first	select the	rows	to	export	by	selecting	the	corresponding	checkboxes	on	the	far	left,	and	then select	the	Export 	option	from	the	More 	button. 

Here	is	an	example,	using	the	recently	created	to-do	tasks: 
The	Export 	option	takes	us	to	a	dialog	form,	where	we	can	choose	what	to	export.	The Import	Compatible	Export 	option	makes	sure	that	the	exported	file	can	be	imported back	to	Odoo.	We	will	need	to	use	this. 

The	export	format	can	be	CSV	or	Excel.	We	will	prefer	CSV	file	to	get	a	better understanding	of	the	export	format.	Next,	we	should	pick	the	columns	we	want	to	export and	click	on	the	Export	To	File 	button.	This	will	start	the	download	of	a	file	with	the exported	data. 
 
![156_1](/images/Odoo Development Essentials - Daniel Reis-156_1.jpg)

If	we	follow	these	instructions	and	select	the	fields	shown	in	the	preceding	screenshot,	we should	end	up	with	a	CSV	text	file	similar	to	this: 
```
"id","name","user_id/id","date_deadline","is_done" "__export__.todo_task_1","Install	Odoo","base.user_root","2015-01- 30","True" "__export__.todo_task_2","Create	dev	database","base.user_root","","False" 
```
Notice	that	Odoo	automatically	exported	an	additional	id	column.	This	is	an	External	ID that	is	automatically	generated	for	each	record.	These	generated	External	IDs	use `__export__` in	place	of	an	actual	module	name.	New	identifiers	are	only	assigned	to records	that	don’t	already	have	one,	and	from	there	on,	they	are	kept	bound	to	the	same record.	This	means	that	subsequent	exports	will	preserve	the	same	External	IDs. 
 
![157_1](/images/Odoo Development Essentials - Daniel Reis-157_1.jpg)


**Importing	data**

First	we	have	to	make	sure	the	import	feature	is	enabled.	This	is	done	in	the	Settings  menu,	Configuration 	|	General	Settings 	option.	Under	the	Import	/	Export 	topic,	make sure	the	Allow	users	to	import	data	from	CSV	files 	checkbox	is	enabled. 

With	this	option	enabled,	the	List	views	show	an	Import 	option	next	to	the	Create 	button at	the	top	of	the	list. 
Let’s	perform	a	mass	edit	on	our	to-do	data:	open	in	a	spreadsheet	or	a	text	editor	the	CSV file	we	just	downloaded,	then	change	a	few	values	and	add	some	new	rows. 

As	mentioned	before,	the	first	id	column	provides	a	unique	identifier	for	each	row allowing	already	existing	records	to	be	updated	instead	of	duplicated	when	we	import	the data	back	to	Odoo.	For	new	rows	we	may	add	to	the	CSV	file,	the	id	should	be	left	blank, and	a	new	record	will	be	created	for	them. 

After	saving	the	changes	on	the	CSV	file,	click	on	the	Import 	option	(next	to	the	Create  button)	and	we	will	be	presented	with	the	import	assistant.	There	we	should	select	the CSV	file	location	on	disk	and	click	on	Validate 	to	check	its	format	for	correctness.	Since the	file	to	import	is	based	on	an	Odoo	export,	there	is	a	good	chance	it	will	be	valid. 

Now	we	can	click	on	Import 	and	there	you	go:	our	modifications	and	new	records	should have	been	loaded	into	Odoo. 
 

**Related	records	in	CSV	data	files**

In	the	example	seen	above,	the	user	responsible	for	each	task	is	a	related	record	in	the users	model,	with	a	many	to	one 	(or	foreign	key)	relation.	The	column	name	used	for	it was	user_id/id	and	the	field	values	were	External	IDs	for	the	related	records,	such	as  `base.user_root`	for	the	administrator	user. 

Relation	columns	should	have	`/id`	appended	to	their	name,	if	using	External	IDs,	or	`/.id`, if	using	database	(numeric)	IDs.	Alternatively,	a	colon	`(:)`	can	be	used	in	place	of	the	slash for	the	same	effect. 

Similarly,	many	to	many 	relations	are	also	supported.	An	example	of	a	many	to	many relations	is	the	one	between	Users	and	Groups:	each	User	can	be	in	many	Groups,	and each	Group	can	have	many	Users.	The	column	name	for	this	type	of	field	should	have appended	a	`/id`.	The	field	values	accept	a	comma-separated	list	of	External	IDs, surrounded	by	double	quotes. 

For	example,	the	to-do	task	follower	is	a	many-to-many	relation	between	To-do	Tasks	and Partners.	It’s	column	name	could	be	follower_ids/id	and	a	field	value	with	two followers	could	be: 
`"__export__.res_partner_1,__export__.res_partner_2"`
 
Finally,	one	to	many 	relations	can	also	be	imported	through	a	CSV.	The	typical	example of	this	type	of	relations	is	a	document	“head”	with	several	“lines”. 

We	can	see	an	example	for	such	a	relation	in	the	company	model	(form	view	available	in the	Settings 	menu):	a	company	can	have	several	bank	accounts,	each	with	its	own	details, and	each	bank	account	belongs	to	(has	a	many-to-one	relation	with)	only	one	company. 

It’s	possible	to	import	companies	along	with	their	bank	accounts	in	a	single	file.	For	this, some	columns	will	correspond	to	the	company,	and	other	columns	will	correspond	to	the bank	account	details.	The	bank	details	column	names	should	be	prefixed	with	the	one-to- many	fields	linking	the	company	to	the	banks;	bank_ids	in	this	case. 

The	first	bank	account	details	goes	in	the	same	row	as	its	related	company	data.	The	next bank	account’s	details	go	in	the	next	rows,	but	only	the	bank	details	related	columns should	have	values;	the	company	data	columns	should	be	empty	in	those	lines. 
Here	is	an	example	loading	a	company	with	three	banks: 
```
id,name,bank_ids/id,bank_ids/acc_number,bank_ids/state base.main_company,YourCompany,__export__.res_partner_bank_4,123456789,bank ,,__export__.res_partner_bank_5,135792468,bank ,,__export__.res_partner_bank_6,1122334455,bank
```

Notice	that	the	two	last	lines	begin	with	two	commas:	this	corresponds	to	empty	values	in the	first	two	columns,	id	and	name,	regarding	the	head	company	data.	But	the	remaining columns,	regarding	bank	accounts,	have	the	values	for	the	second	and	third	bank	records. 

These	are	the	essentials	on	working	with	export	and	import	from	the	GUI.	It’s	useful	to	set up	data	in	new	Odoo	instances,	or	to	prepare	data	files	to	be	included	in	Odoo	modules. 
 
Next	we	will	learn	more	about	using	data	files	in	modules. 
 
  
**Module	data**  

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