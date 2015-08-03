Chapter	5.	Models	–	Structuring	the Application	Data
===

In	the	previous	chapters,	we	had	an	end-to-end	overview	of	creating	new	modules	for Odoo.	In Chapter	2, 	*Building	Your	First	Odoo	Application*,	a	completely	new	application was	built,	and	in Chapter	3, 	*Inheritance	–	Extending	Existing	Applications*,	we	explored  Odoo Development Essentials - Daniel Reiss.html#148 inheritance	and	how	to	use	it	to	create	an	extension	module	for	our	application.	In	Chapter 4,	*Data	Serialization	and	Module	Data*,	we	discussed	how	to	add	initial	and	demonstration  data	to	our	modules.
 
In	these	overviews,	we	touched	all	the	layers	involved	in	building	a	backend	application for	Odoo.	Now,	in	the	following	chapters,	it’s	time	to	explain	in	more	detail	these	several layers	making	up	an	application:	models,	views,	and	business	logic. 
In	this	chapter,	you	will	learn	how	to	design	the	data	structures	supporting	an	application, and	how	to	represent	the	relations	between	them. 
 

**Organizing	application	features	into modules**

As	before,	we	will	use	an	example	to	help	explain	the	concepts.	One	of	the	great	things about	Odoo	is	being	able	to	pick	any	existing	application	or	module	and	add,	on	top	of	it, those	extra	features	you	need.	So	we	are	going	to	continue	improving	our	to-do	modules, and	in	no	time	they	will	form	a	fully	featured	application! 

It	is	a	good	practice	to	split	Odoo	applications	into	several	smaller	modules,	each	of	them responsible	for	specific	features.	This	reduces	overall	complexity	and	makes	them	easier to	maintain	and	upgrade	to	later	Odoo	versions.	The	problem	of	having	to	install	all	these individual	modules	can	be	solved	by	providing	an	app	module	packaging	all	those features,	through	its	dependencies.	To	illustrate	this	approach	we	will	be	implementing	the additional	features	using	new	to-do	modules. 
 

**Introducing	the	todo_ui	module**

In	the	previous	chapter,	we	first	created	an	app	for	personal	to-dos,	and	then	extended	it	so that	the	to-dos	could	be	shared	with	other	people. 

Now	we	want	to	take	our	app	to	a	new	level	by	adding	to	it	a	kanban	board	and	a	few other	nice	user	interface	improvements.	The	kanban	board	will	let	us	organize	our	tasks	in columns,	according	to	their	stages,	such	as	Waiting,	Ready,	Started	or	Done. 

We	will	start	by	adding	the	data	structures	to	enable	that	vision.	We	need	to	add	stages	and it	will	be	nice	if	we	add	support	for	tags	as	well,	allowing	the	tasks	to	be	categorized	by subject. 

The	first	thing	to	figure	out	is	how	our	data	will	be	structured	so	that	we	can	design	the supporting	Models.	We	already	have	the	central	entity:	the	to-do	task.	Each	task	will	be	in a	stage,	and	tasks	can	also	have	one	or	more	tags	on	them.	This	means	we	will	need	to	add these	two	additional	models,	and	they	will	have	these	relations: 

- Each	task	has	a	stage,	and	there	can	be	many	tasks	in	each	stage. 
- Each	task	can	have	many	tags,	and	each	tag	can	be	in	many	tasks. 

This	means	that	tasks	have	many	to	one	relation	with	stages,	and	many	to	many	relations with	tags.	On	the	other	hand,	the	inverse	relations	are:	stages	have	a	one	to	many relationship	with	tasks	and	tags	have	a	many	to	many	relation	with	tasks. 
We	will	start	by	creating	the	new	todo_ui	module	and	add	the	to-do	stages	and	to-do	tags models	to	it.
 
We’ve	been	using	the	`~/odoo-dev/custom-addons/`	directory	to	host	our	modules.	To create	the	new	module	alongside	the	existing	ones,	we	can	use	these	shell	commands: 
```
$ cd ~/odoo-dev/custom-addons 
$ mkdir	todo_ui 
$ cd	todo_ui 
$ touch	todo_model.py 
$ echo	"from . import	todo_model" > __init__.py
```  
Next,	we	should	add	the	`__openerp__.py`	manifest	file	with	this	content: 
```
{ 'name': 'User	interface improvements	to	the	To-Do	app',
  'description': 'User	friendly features.',
  'author':	'Daniel	Reis',
  'depends':	['todo_app']	
}
``` 
Note	that	we	are	depending	on	todo_app	and	not	on	todo_user.	In	general,	it	is	a	good idea	to	keep	modules	as	independent	as	possible.	When	an	upstream	module	is	changed,	it can	impact	all	other	modules	that	directly	or	indirectly	depend	on	it.	It’s	best	if	we	can keep	the	number	of	dependencies	low,	and	also	avoid	long	dependency	stacks,	such	as 
todo_ui	→	todo_user	→	todo_app	in	this	case. 

Now	we	can	install	the	module	in	our	Odoo	work	database	and	get	started	with	the models. 
 

**Creating	models**

For	the	to-do	tasks	to	have	a	kanban	board,	we	need	stages.	Stages	are	the	board	columns, and	each	task	will	fit	into	one	of	these	columns. 

Let’s	add	the	following	code	to	the	todo_ui/todo_model.py	file: 
```
#-*- coding:	utf-8	-*- 
from openerp import	models,	fields,	api 
class	Tag(models.Model):
    _name	=	'todo.task.tag'
    name	=	fields.Char('Name',	40,	translate=True) 

class	Stage(models.Model):
    _name	=	'todo.task.stage'
    _order	=	'sequence,name'
    _rec_name	=	'name'		# the	default
    _table	=	'todo_task_stage' #the	default
    name	=	fields.Char('Name',	40,	translate=True)
    sequence	=	fields.Integer('Sequence') 
```
Here,	we	created	the	two	new	Models	we	will	be	referencing	in	the	to-do	tasks. 

Focusing	on	the	task	stages,	we	have	a	Python	class,	Stage,	based	on	the	class `models.Model`,	defining	a	new	Odoo	model,	todo.task.stage.	We	also	defined	two fields,	name	and	sequence.	We	can	see	some	model	attributes,	(prefixed	with	an underscore)	that	are	new	to	us.	Let’s	have	a	closer	look	at	them. 
 

**Model	attributes**

Model	classes	can	have	additional	attributes	used	to	control	some	of	their	behaviors: 

- `_name`:	This	is	the	internal	identifier	for	the	Odoo	model	we	are	creating. 
- `_order`:	This	sets	the	order	to	use	when	the	model’s	records	are	browsed.	It	is	a	text string	to	be	used	as	the	SQL	order	by	clause,	so	it	can	be	anything	you	could	use there. 
- `_rec_name`:	This	indicates	the	field	to	use	as	the	record	description	when	referenced from	related	fields,	such	as	a	many	to	one	relation.	By	default,	it	uses	the	name	field, which	is	a	commonly	found	field	in	models.	But	this	attribute	allows	us	to	use	any other	field	for	that	purpose. 
- `_table`:	This	is	the	name	of	the	database	table	supporting	the	model.	Usually,	it	is	left to	be	calculated	automatically,	and	is	the	model	name	with	the	dots	replaced	by underscores.	But	it	can	be	set	to	indicate	a	specific	table	name.
 
For	completeness,	we	can	also	have	the	_inherit	and	_inherits	attributes,	as	explained in Chapter	3, 	*Inheritance	-	Extending	Existing	Applications*. 
 

**Models	and	Python	classes**

Odoo	models	are	represented	by	Python	classes.	In	the	preceding	code,	we	have	a	Python class	Stage,	based	on	the	models.Model	class,	used	to	define	a	new	Odoo	model `todo.task.stage`.
 
Odoo	models	are	kept	in	a	central	registry,	also	referred	to	as	pool	in	the	previous versions.	It	is	a	dictionary	keeping	references	of	all	the	model	classes	available	in	the instance,	and	can	be	referenced	by	model	name.	Specifically,	the	code	in	a	model	method can	use	`self.envl['x']`	or	`self.env.get('x')`	to	get	a	reference	to	a	class	representing model	x.
 
You	can	see	that	model	names	are	important	since	they	are	the	key	used	to	access	the registry.	The	convention	for	model	names	is	to	use	a	list	of	lowercase	words	joined	with dots,	like	`todo.task.stage`.	Other	examples	from	the	core	modules	are	`project.project`, `project.task`	or	`project.task.type`.	We	should	use	the	singular	form:	`todo.task` instead	of	`todo.tasks`.	For	historical	reasons	it’s	possible	to	find	some	core	models	not following	this,	such	as	`res.users`,	but	that	is	not	the	rule. 

Model	names	must	be	globally	unique.	Because	of	this,	the	first	word	should	correspond to	the	main	application	the	module	relates	to.	In	our	example,	it	is	todo.	Other	examples from	the	core	modules	are	project,	crm,	or	sale. 

Python	classes,	on	the	other	hand,	are	local	to	the	Python	file	where	they	are	declared.	The identifier	used	for	them	is	only	significant	for	the	code	in	that	file. 

Because	of	this,	class	identifiers	are	not	required	to	be	prefixed	by	the	main	application they	relate	to.	For	example,	there	is	no	problem	to	call	just	Stage	to	our	class	for	the 
todo.task.stage	model.	There	is	no	risk	of	collision	with	possible	classes	with	the	same name	on	other	modules. 

Two	different	conventions	for	class	identifiers	can	be	used:	snake_case	or	CamelCase. Historically,	Odoo	code	used	snake	case,	and	it	is	still	very	frequent	to	find	classes	using that	convention.	But	the	recent	trend	is	to	use	camel	case,	since	it	is	the	Python	standard defined	by	the	PEP8	coding	conventions.	You	may	have	noticed	that	we	are	using	the latter	form. 
 

**Transient	and	Abstract	models**

In	the	preceding	code,	and	in	the	vast	majority	of	Odoo	models,	classes	are	based	on	the `models.Model`	class.	This	type	of	models	have	database	persistence:	database	tables	are created	for	them	and	their	records	are	stored	until	explicitly	deleted. 

But	Odoo	also	provides	two	other	model	types	to	be	used:	Transient	and	Abstract	models. 

Transient	models 	are	based	on	the	models.TransientModel	class	and	are	used	for wizard-style	user	interaction.	Their	data	is	still	stored	in	the	database,	but	it	is	expected	to be	temporary.	A	vacuum	job	periodically	clears	old	data	from	these	tables. 
Abstract	models 	are	based	on	the	models.AbstractModel	class	and	have	no	data	storage attached	to	them.	They	act	as	reusable	feature	sets	to	be	mixed	in	with	other	models.	This is	done	using	the	Odoo	inheritance	capabilities. 
 

![185_1](/images/Odoo Development Essentials - Daniel Reis-185_1.jpg)


**Inspecting	existing	models**

The	information	about	models	and	fields	created	with	Python	classes	is	available	through the	user	interface.	In	the	Settings 	top	menu,	select	the	Technical	|	Database	Structure	| Models 	menu	item.	Here,	you	will	find	the	list	of	all	models	available	in	the	database. Clicking	on	a	model	in	the	list	will	open	a	form	with	its	details. 

This	is	a	good	tool	to	inspect	the	structure	of	a	Model,	since	you	have	in	one	place	the result	of	all	additions	that	may	come	from	several	different	modules.	In	this	case,	as	you can	see	at	the	In	Modules 	field,	on	the	top	right,	the	todo.task	definitions	are	coming from	the	todo_app	and	todo_user	modules. 

In	the	lower	area,	we	have	some	information	tabs	available:	a	quick	reference	for	the model	Fields ,	the	Access	Rights 	granted,	and	also	list	the	Views 	available	for	this	model. 
We	can	find	the	model’s	External	Identifier ,	by	activating	the	Developer	Menu 	and accessing	its	View	Metadata 	option.	These	are	automatically	generated	but	fairly predictable:	for	the	todo.task	model,	the	External	Identifier 	is	model_todo_task. 

*Tip*
*The	Models 	form	is	editable!	It’s	possible	to	create	and	modify	models,	fields,	and	views from	here.	You	can	use	this	to	build	prototypes	before	carving	them	into	proper	modules.*
 

**Creating	fields**

After	creating	a	new	model,	the	next	step	is	to	add	fields	to	it.	Let’s	explore	the	several types	of	fields	available	in	Odoo. 
 

**Basic	field	types**

We	now	have	a	Stage	model	and	will	expand	it	to	add	some	additional	fields.	We	should edit	the	todo_ui/todo_model.py	file,	by	removing	some	unnecessary	attributes	included before	for	the	purpose	of	explanation,	making	it	look	like	this: 
```
class	Stage(models.Model):
    _name	=	'todo.task.stage'
    _order	=	'sequence,name'	
    # String fields:
    name	=	fields.Char('Name',	40)
    desc	=	fields.Text('Description')
    state	=	fields.Selection([('draft','New'),('open','Started'),('done','Closed')],'State')
    docs	=	fields.Html('Documentation')
    #	Numeric	fields:
    sequence	=	fields.Integer('Sequence')
    perc_complete =	fields.Float('%	Complete', (3,	2))
    # Date fields:
    date_effective	= fields.Date('Effective Date')
    date_changed	= fields.Datetime('Last	Changed')
    # Other	fields:
    fold	=	fields.Boolean('Folded?')
    image	=	fields.Binary('Image')
``` 
Here,	we	have	a	sample	of	the	non-relational	field	types	available	in	Odoo,	with	the	basic arguments	expected	by	each	function.	For	most,	the	first	argument	is	the	field	title, corresponding	to	the	string	keyword	attribute.	It’s	an	optional	argument,	but	it	is recommended	to	be	provided.	If	not,	a	title	will	be	automatically	generated	from	the	field name. 

There	is	a	convention	for	date	fields	to	use	date	as	a	prefix	in	their	name.	For	example,	we should	use	date_effective	instead	of	effective_date.	This	can	also	apply	to	other fields,	such	as	`amount_`,	`price_`	or	`qty_`. 

A	few	more	arguments	are	available	for	most	field	types: 
Char	accepts	a	second,	optional	argument,	size,	corresponding	to	the	maximum	text size.	It’s	recommended	to	use	it	only	if	you	have	a	good	reason	to. 

Text	differs	from	Char	in	that	it	can	hold	multiline	text	content,	but	expects	the	same arguments. 

Selection	is	a	drop-down	selection	list.	The	first	argument	is	the	list	of	selectable options	and	the	second	is	the	title	string.	The	selection	list	items	are	`('value', 'Title')`	tuples	for	the	value	stored	in	the	database	and	the	corresponding description	string.	When	extending	through	inheritance,	the	selection_add	argument can	be	used	to	append	items	to	an	existing	selection	list.
 
Html	is	stored	as	a	text	field,	but	has	specific	handling	to	present	HTML	content	on the	user	interface. 

Integer	just	expects	a	string	argument	for	the	field	title. 
Float	has	a	second	optional	argument,	an	`(x,y)`	tuple	with	the	field’s	precision:	x	is the	total	number	of	digits;	of	those,	y	are	decimal	digits. 

Date	and	Datetime	data	is	stored	in	UTC	time.	There	are	automatic	conversions made,	based	on	the	user	time	zone	preferences,	made	available	through	the	user session	context.	This	is	discussed	in	more	detail	in Chapter	6, 	*Views	–	Designing	the User	Interface*. 

Boolean	only	expects	the	field	title	to	be	set,	even	if	it	is	optional.  Binary	also	expects	only	a	title	argument. 
O
ther	than	these,	we	also	have	the	relational	fields,	which	will	be	introduced	later	in	this chapter.	But	now,	there	is	still	more	to	learn	about	these	field	types	and	their	attributes. 
 

**Common	field	attributes**

Fields	also	have	a	set	of	attributes	we	can	use,	and	we’ll	explain	these	in	more	detail: 

- string	is	the	field	title,	used	as	its	label	in	the	UI.	Most	of	the	time	it	is	not	used	as	a keyword	argument,	since	it	can	be	set	as	a	positional	argument. 
default	sets	a	default	value	for	the	field.	It	can	be	a	static	value	or	a	callable,	either	a function	reference	or	a	lambda	expression. 

- size	applies	only	to	Char	fields,	and	can	set	a	maximum	size	allowed. 
translate	applies	to	text	fields,	Char,	Text	and	Html,	and	makes	the	field translatable:	it	can	have	different	values	for	different	languages. 

- help	provides	the	text	for	tooltips	displayed	to	the	users. 

- `readonly=True`	makes	the	field	not	editable	on	the	user	interface. 

- `required=True`	makes	the	field	mandatory. 

- `index=True`	will	create	a	database	index	on	the	field. 

- `copy=False`	has	the	field	ignored	when	using	the	copy	function.	The	non-relational fields	are	copyable	by	default. 

- groups	allows	limiting	the	field’s	access	and	visibility	to	only	some	groups.	It	is	a comma-separated	list	of	strings	for	security	group	XML	IDs. 

- states	expects	a	dictionary	mapping	values	for	UI	attributes	depending	on	values	of the	state	field.	For	example:	`states={'done':[('readonly',True)]}`.	Attributes that	can	be	used	are	readonly,	required,	and	invisible. 

For	completeness,	two	other	attributes	are	sometimes	used	when	upgrading	between	Odoo major	versions: 

- `deprecated=True`	logs	a	warning	whenever	the	field	is	being	used. 
- `oldname='field'`	is	used	when	a	field	is	renamed	in	a	newer	version,	enabling	the data	in	the	old	field	to	be	automatically	copied	into	the	new	field. 
 

**Reserved	field	names**

A	few	field	names	are	reserved	to	be	used	by	the	ORM: 
id	is	an	automatic	number	uniquely	identifying	each	record,	and	used	as	the	database primary	key.	It’s	automatically	added	to	every	model. 

The	following	fields	are	automatically	created	on	new	models,	unless	the 
_log_access=False	model	attribute	is	set: 

- `create_uid` 	for	the	user	that	created	the	record 
- `create_date`	for	the	date	and	time	when	the	record	is	created 
- `write_uid`	for	the	last	user	to	modify	the	record 
- `write_date`	for	the	last	date	and	time	when	the	record	was	modified 

This	information	is	available	from	the	web	client,	using	the	Developer	Mode 	menu	and selecting	the	View	Metadata 	option. 

There	some	built-in	effects	that	expect	specific	field	names.	We	should	avoid	using	them for	purposes	other	than	the	intended	ones.	Some	of	them	are	even	reserved	and	can’t	be used	for	other	purposes	at	all: 

- name	is	used	by	default	as	the	display	name	for	the	record.	Usually	it	is	a	Char,	but other	field	types	are	also	allowed.	It	can	be	overridden	by	setting	the	_rec_name model	attribute. 
- active	(type	Boolean)	allows	inactivating	records.	Records	with	`active==False`	will automatically	be	excluded	from	queries.	To	access	them	an	('active','=',False) condition	must	be	added	to	the	search	domain,	or	`'active_test':	False`	should	be added	to	the	current	context. 
- sequence	(type	Integer)	if	present	in	a	list	view,	allows	to	manually	define	the	order of	the	records.	To	work	properly	it	should	also	be	in	the	model’s	_order. 
- state	(type	Selection)	represents	basic	states	of	the	record’s	life	cycle,	and	can	be used	by	the	state’s	field	attribute	to	dynamically	modify	the	view:	some	form	fields can	be	made	read	only,	required	or	invisible	in	specific	record	states. 
- `parent_id`,	`parent_left`,	and	`parent_right`	have	special	meaning	for	parent/child hierarchical	relations.	We	will	shortly	discuss	them	in	detail. 

So	far	we’ve	discussed	scalar	value	fields.	But	a	good	part	of	an	application	data	structure is	about	describing	the	relationships	between	entities.	Let’s	look	at	that	now. 
 

**Relations	between	models**

Looking	again	at	our	module	design,	we	have	these	relations: 

- Each	task	has	a	stage	–	that’s	a	many	to	one	relation,	also	known	as	a	foreign	key. The	inverse	relation	is	a	one	to	many,	meaning	that	each	stage	can	have	many	tasks. 
- Each	task	can	have	many	tags	–	that’s	a	many	to	many	relation.	The	inverse	relation, of	course,	is	also	a	many	to	many,	since	each	tag	can	also	have	many	tasks. 

Let’s	add	the	corresponding	relation	fields	to	the	to-do	tasks	in	our `todo_ui/todo_model.py`	file: 
```
class	TodoTask(models.Model):
    _inherit	=	'todo.task'
    stage_id	=	fields.Many2one('todo.task.stage',	'Stage')
    tag_ids	=	fields.Many2many('todo.task.tag',	string='Tags')
```
The	preceding	code	shows	the	basic	syntax	for	these	fields,	setting	the	related	model	and the	field’s	title	string.	The	convention	for	relational	field	names	is	to	append	_id	or	_ids to	the	field	names,	for	to	one	and	to	many	relations,	respectively. 

As	an	exercise,	you	may	try	to	also	add	on	the	related	models,	the	corresponding	inverse relations:  The	inverse	of	the	Many2one	relation	is	a	One2many	field	on	stages:	each	stage	can have	many	tasks.	We	should	add	this	field	to	the	Stage	class. The	inverse	of	the	Many2many	relation	is	also	a	Many2many	field	on	tags:	each	tag	can also	be	used	on	many	tasks. 

Let’s	have	a	closer	look	at	relational	field	definitions. 
 

**Many	to	one	relations**

Many2one	accepts	two	positional	arguments:	the	related	model	(corresponding	to	the  comodel	keyword	argument)	and	the	title	string.	It	creates	a	field	in	the	database	table with	a	foreign	key	to	the	related	table. 

Some	additional	named	arguments	are	also	available	to	use	with	this	type	of	field: 

- ondelete	defines	what	happens	when	the	related	record	is	deleted.	Its	default	is	set  null,	meaning	it	is	set	to	an	empty	value	if	the	related	record	is	deleted.	Other possible	values	are	restrict,	raising	an	error	preventing	the	deletion,	and	cascade also	deleting	this	record. 
- context	and	domain	are	meaningful	for	the	web	client	views.	They	can	be	set	on	the model	to	be	used	by	default	on	any	view	where	the	field	is	used.	They	will	be	better explained	in the Chapter	6, 	*Views	-	Designing	the	User	Interface*. 
- `auto_join=True`	allows	the	ORM	to	use	SQL	joins	when	doing	searches	using	this relation.	By	default	this	is	False	to	be	able	to	enforce	security	rules.	If	joins	are	used, the	security	rules	will	be	bypassed,	and	the	user	could	have	access	to	related	records the	security	rules	wouldn’t	allow,	but	the	SQL	queries	will	be	more	efficient	and	run faster. 


**Many	to	many	relations**

The	Many2many	minimal	form	accepts	one	argument	for	the	related	model,	and	it	is recommended	to	also	provide	the	string	argument	with	the	field	title. 

At	the	database	level,	this	does	not	add	any	column	to	the	existing	tables.	Instead,	it automatically	creates	a	new	relation	table	with	only	two	ID	fields	with	the	foreign	keys	to the	related	tables.	The	relation	table	name	and	the	field	names	are	automatically	generated. The	relation	table	name	is	the	two	table	names	joined	with	an	underscore	with	`_rel` appended	to	it.
 
These	defaults	can	be	manually	overridden.	One	way	to	do	it	is	to	use	the	longer	form	for the	field	definition: 
```
# TodoTask class: Task	<-> Tag	relation (long	form): 
tag_ids	= fields.Many2many( 'todo.task.tag',	# related model
                            'todo_task_tag_rel', # relation table name
                            'task_id', # field for "this" record
                            'tag_id', #field	for "other" record
                             string='Tasks')
``` 
Note	that	the	additional	arguments	are	optional.	We	could	just	set	the	name	for	the	relation table	and	let	the	field	names	use	the	automatic	defaults. 

If	you	prefer,	you	may	use	the	long	form	using	keyword	arguments	instead: 
```
# TodoTask class: Task	<-> Tag	relation (long	form): 
tag_ids	= fields.Many2many(comodel_name='todo.task.tag', #related model
                           relation='todo_task_tag_rel', #relation able	name
                           column1='task_id', # field for "this" record
                           column2='tag_id', # field for "other" record
                           string='Tasks') 
```
Like	many	to	one	fields,	many	to	many	fields	also	support	the	domain	and	context keyword	attributes. 

On	some	rare	occasions	we	may	have	to	use	these	long	forms	to	override	the	automatic defaults,	in	particular,	when	the	related	models	have	long	names	or	when	we	need	a second	many	to	many	relation	between	the	same	models. 

*Tip*  
*PostgreSQL	table	names	have	a	limit	of	63	characters,	and	this	can	be	a	problem	if	the automatically	generated	relation	table	name	exceeds	that	limit.	That	is	a	case	where	we should	manually	set	the	relational	table	name	using	the	relation	attribute. *

The	inverse	of	the	Many2many	relation	is	also	a	Many2many	field.	If	we	also	add	a  Many2many	field	to	the	tags,	Odoo	infers	that	this	many	to	many	relation	is	the	inverse	of the	one	in	the	task	model. 

The	inverse	relation	between	tasks	and	tags	can	be	implemented	like	this: 
``` 
# class	Tag(models.Model): #
    _name	= 'todo.task.tag' 
    #Tag	class	relation to Tasks: 
    task_ids	=	fields.Many2many( 'todo.task', # related model 
                                           string='Tasks') 
```


**One	to	many	inverse	relations**
  
The	inverse	of	a	Many2one	can	be	added	to	the	other	end	of	the	relation.	This	has	no impact	on	the	actual	database	structure,	but	allows	us	easily	browse	from	the	“one”	side the	“many”	side	records.	A	typical	use	case	is	the	relation	between	a	document	header	and its	lines. 

On	our	example,	with	a	One2many	inverse	relation	on	stages,	we	could	easily	list	all	the tasks	in	that	stage.	To	add	this	inverse	relation	to	stages,	add	the	code	shown	here: 
```
# class	Stage(models.Model): #
    _name	= 'todo.task.stage' 
    #Stage	class	relation	with	Tasks:
    tasks	=	fields.One2many('todo.task',#related	model
                                        'stage_id',#field for	"this"	on	related	model 
                                        'Tasks	in	this	stage') 
```
The	One2many	accepts	three	positional	arguments:	the	related	model,	the	field	name	in	that model	referring	this	record,	and	the	title	string.	The	two	first	positional	arguments correspond	to	the	comodel_name	and	inverse_name	keyword	arguments. 

The	additional	keyword	parameters	available	are	the	same	as	for	many	to	one:	context,  domain,	ondelete	(here	acting	on	the	“many”	side	of	the	relation),	and	auto_join. 
 

**Hierarchical	relations**

Parent-child	relations	can	be	represented	using	a	Many2one	relation	to	the	same	model,	to let	each	record	reference	its	parent.	And	the	inverse	One2many	makes	it	easy	for	a	parent	to keep	track	of	its	children. 

Odoo	also	provides	improved	support	for	these	hierarchic	data	structures:	faster	browsing through	tree	siblings,	and	simpler	search	with	the	additional	child_of	operator	in	domain expressions. 

To	enable	these	features	we	need	to	set	the	_parent_store	flag	attribute	and	add	the helper	fields:	parent_left	and	parent_right.	Mind	that	this	additional	operation	comes at	storage	and	execution	time	penalties,	so	it’s	best	used	when	you	expect	to	read	more frequently	than	write,	such	as	a	the	case	of	a	category	tree. 

Revisiting	the	tags	model	defined	in	the	`todo_ui/todo_model.py`	file,	we	should	now	edit it	to	look	like	this: 
```
class	Tags(models.Model):
    _name	=	'todo.task.tag'
    _parent_store	=	True 	#
    _parent_name	=	'parent_id'
    name	=	fields.Char('Name')
    parent_id	=	fields.Many2one('todo.task.tag','Parent	Tag', ondelete='restrict')
    parent_left	=	fields.Integer('Parent	Left',	index=True)
    parent_right	=	fields.Integer('Parent	Right',	index=True) 
```
Here,	we	have	a	basic	model,	with	a	parent_id	field	to	reference	the	parent	record,	and the	additional	`_parent_store`	attribute	to	add	hierarchic	search	support.	When	doing	this, the	`parent_left`	and	`parent_right`	fields	also	have	to	be	added. 

The	field	referring	to	the	parent	is	expected	to	be	named	`parent_id`.	But	any	other	field name	can	be	used	by	declaring	it	with	the	`_parent_name`	attribute. 

Also,	it	is	often	convenient	to	add	a	field	with	the	direct	children	of	the	record: 
```
child_ids	=	fields.One2many( 'todo.task.tag', 'parent_id',	'Child	Tags') 
 

**Referencing	fields	using	dynamic	relations**

So	far,	the	relation	fields	we’ve	seen	can	only	reference	one	model.	The	Reference	field type	does	not	have	this	limitation	and	supports	dynamic	relations:	the	same	field	is	able	to refer	to	more	than	one	model. 

We	can	use	it	to	add	a	To-do	Task	field,	Refers	to ,	that	can	either	refer	to	a	User	or	a Partner: 
```
# class	TodoTask(models.Model):
    refers_to	=	fields.Reference([('res.user',	'User'),('res.partner',	'Partner')], 'Refers	to') 
```
You	can	see	that	the	field	definition	is	similar	to	a	Selection	field,	but	here	the	selection list	holds	the	models	that	can	be	used.	On	the	user	interface,	the	user	will	first	pick	a model	from	the	list,	and	then	pick	a	record	from	that	model. 

This	can	be	taken	to	another	level	of	flexibility:	a	Referencable	Models 	configuration table	exists	to	configure	the	models	that	can	be	used	in	Reference	fields.	It	is	available	in the	Settings	|	Technical	|	Database	Structure 	menu.	When	creating	such	a	field	we	can set	it	to	use	any	model	registered	there,	with	the	help	of	the	referencable_models() function	in	the	openerp.addons.res.res_request	module.	In	Odoo	version	8,	it	is	still using	the	old-style	API,	so	we	need	to	wrap	it	to	use	with	the	new	API: 
```
from	openerp.addons.base.res	import	res_request def	referencable_models(self):
    return	res_request.referencable_model (self,	self.env.cr,	self.env.uid,	context=self.env.context) 
```
Using	the	preceding	code,	the	revisited	version	of	the	Refers	to	field	would	look	like this: 
```
# class	TodoTask(models.Model):
     refers_to	=	fields.Reference( referencable_models,	'Refers	to') 
```


**Computed	fields**

Fields	can	have	values	calculated	by	a	function,	instead	of	simply	reading	a	database stored	value.	A	computed	field	is	declared	just	like	a	regular	field,	but	has	an	additional argument	compute	with	the	name	of	the	function	used	to	calculate	it. 

In	most	cases	computed	fields	involve	writing	some	business	logic,	so	we	will	develop this	topic	more	in Chapter	7, 	*ORM	Application	Logic	-	Supporting	Business	Processes*.	We can	still	explain	them	here,	but	keeping	the	business	logic	side	as	simple	as	possible. 
Let’s	work	on	an	example:	stages	have	a	fold	field.	We	will	add	to	tasks	a	computed	field with	the	Folded? 	flag	for	the	corresponding	stage. 

We	should	edit	the	TodoTask	model	in	the	todo_ui/todo_model.py	file	to	add	the following: 
```
#	class	TodoTask(models.Model):
    stage_fold	=	fields.Boolean('Stage	Folded?', compute='_compute_stage_fold')
    @api.one	@api.depends('stage_id.fold') def _compute_stage_fold(self):
        self.stage_fold	= self.stage_id.fold 
```
The	preceding	code	adds	a	new	stage_fold	field	and	the	`_compute_stage_fold`	method used	to	compute	it.	The	function	name	was	passed	as	a	string,	but	it’s	also	allowed	to	pass it	as	a	callable	reference	(the	function	identifier	with	no	quotes). 
Since	we	are	using	the	`@api.one`	decorator,	self	will	represent	a	single	record.	If	we	used `@api.multi`	instead,	it	would	represent	a	recordset	and	our	code	would	need	to	handle	the iteration	over	each	record. 

The	`@api.depends`	is	necessary	if	the	computation	uses	other	fields:	it	tells	the	server when	to	recompute	stored	or	cached	values.	It	accepts	one	or	more	field	names	as arguments	and	dot-notation	can	be	used	to	follow	field	relations. 

The	computation	function	is	expected	to	assign	a	value	to	the	field	or	fields	to	compute.	If it	doesn’t,	it	will	error.	Since	self	is	a	record	object,	our	computation	is	simply	to	get	the Folded? 	field	using	`self.stage_id.fold`.	The	result	is	achieved	by	assigning	that	value (writing	it)	to	the	computed	field,	`self.stage_fold`.
 
We	won’t	be	working	yet	on	the	views	for	this	module,	but	you	can	make	a	quick	edit	on the	task	form	to	confirm	if	the	computed	field	is	working	as	expected:	using	the Developer	Menu 	pick	the	Edit	View 	option	and	add	the	field	directly	in	the	form	XML. Don’t	worry:	it	will	be	replaced	by	the	clean	module	view	on	the	next	upgrade. 
 


**Search	and	write	on	computed	fields**
  
The	computed	field	we	just	created	can	be	read,	but	it	can’t	be	searched	or	written.	This can	be	enabled	by	providing	specialized	functions	for	that.	Along	with	the	compute function,	we	can	also	set	a	search	function,	implementing	the	search	logic,	and	the  inverse	function,	implementing	the	write	logic. 
In	order	to	do	this,	our	computed	field	declaration	becomes	like	this: 
```
# class	TodoTask(models.Model):
	stage_fold	=	fields.Boolean
        string='Stage	Folded?', 								
        compute='_compute_stage_fold', # store=False) # the	default 			 
        search='_search_stage_fold', 								
        inverse='_write_stage_fold') 
```
The	supporting	functions	are: 
```
def _search_stage_fold(self, operator,	value):
    return	[('stage_id.fold',	operator,	value)] 

def	_write_stage_fold(self): 								
    self.stage_id.fold	=	self.stage_fold 
```
The	search	function	is	called	whenever	a	`(field,	operator,	value)`	condition	on	this field	is	found	in	a	search	domain	expression.	It	receives	the	operator	and	value	for	the search	and	is	expected	to	translate	the	original	search	element	into	an	alternative	domain search	expression. 

The	inverse	function	performs	the	reverse	logic	of	the	calculation,	to	find	the	value	to write	on	the	source	fields.	In	our	example,	it’s	just	writing	on	`stage_id.fold`. 
 

**Storing	computed	fields**

Computed	field’s	values	can	also	be	stored	on	the	database,	by	setting	store	to	True	on their	definition.	They	will	be	recomputed	when	any	of	their	dependencies	change.	Since the	values	are	now	stored,	they	can	be	searched	just	like	regular	fields,	so	a	search function	is	not	needed. 
 

**Related	fields**

The	computed	field	we	implemented	in	the	previous	section	is	a	special	case	that	can	be automatically	handled	by	Odoo.	The	same	effect	can	be	achieved	using	Related	fields. They	make	available, directly	on	a	model,	fields	that	belong	to	a	related	model,	accessible using	a	dot-notation	chain.	This	makes	them	usable	in	situations	where	dot-notation	can’t be	used,	such	as	UI	forms. 

To	create	a	related	field,	we	declare	a	field	of	the	needed	type,	just	like	with	regular computed	fields,	and	instead	of	compute,	use	the	related	attribute	indicating	the	dot- notation	field	chain	to	reach	the	desired	field. 

To-do	tasks	are	organized	in	customizable	stages	and	these	is	turn	map	into	basic	states. We	will	make	them	available	on	tasks,	and	will	use	this	for	some	client-side	logic	in	the next	chapter. 
Similarly	to	stage_fold,	we	will	add	a	computed	field	on	the	task	model,	but	now	using the	simpler	Related	field: 
```
# class	TodoTask(models.Model):
    stage_state	=	fields.Selection( 								
         related='stage_id.state', 								
         string='Stage	State') 
```
Behind	the	scenes,	Related	fields	are	just	computed	fields	that	conveniently	implement  search	and	inverse.	This	means	that	we	can	search	and	write	on	them	out	of	the	box, without	having	to	write	any additional	code. 
 

**Model	constraints** 

To	enforce	data	integrity,	models	also	support	two	types	of	constraints:	SQL	and	Python. 

SQL	constraints	are	added	to	the	table	definition	in	the	database	and	implemented	by PostgreSQL.	They	are	defined	using	the	class	attribute	_sql_constraints.	It	is	a	list	of tuples	with	the	constraint	identifier	name,	the	SQL	for	the	constraint,	and	the	error message	to	use. 

A	common	use	case	is	to	add	unique	constraints	to	models.	Suppose	we	didn’t	want	to allow	the	same	user	to	have	two	active	tasks	with	the	same	title: 
```
# class	TodoTask(models.Model):
    	_sql_constraints	=	[ 								
            ('todo_task_name_uniq',
             'UNIQUE	(name,	user_id,	active)', 
             'Task	title	must	be	unique!')] 
```
Since	we	are	using	the	user_id	field	added	by	the	todo_user	module,	this	dependency should	be	added	to	the	depends	key	of	the	`__openerp__.py`	manifest	file. 

Python	constraints	can	use	a	piece	of	arbitrary	code	to	check	conditions.	The	checking function	needs	to	be	decorated	with	@api.constrains	indicating	the	list	of	fields	involved in	the	check.	The	validation	is	triggered	when	any	of	them	is	modified,	and	will	raise	an exception	if	the	condition	fails: 
```
from	openerp.exceptions	import	ValidationError #
class	TodoTask(models.Model):
	@api.one @api.constrains('name') 
        def	_check_name_size(self): 								
            if	len(self.name)	<	5:
                   raise	ValidationError('Must	have	5	chars!') 
```
The	preceding	example	prevents	saving	task	titles	with	less	than	5	characters. 
 

** Summary**

We	went	through	a	thorough	explanation	of	models	and	fields,	using	them	to	extend	the To-do	app	with	tags	and	stages	on	tasks.	You	learned	how	to	define	relations	between models,	including	hierarchical	parent/child	relations.	Finally,	we	saw	simple	examples	of computed	fields	and	constraints	using	Python	code. 

In	the	next	chapter,	we	will	work	on	the	user	interface	for	these	back-end	model	features, making	them	available	in	the	views	used	to	interact	with	the	application. 