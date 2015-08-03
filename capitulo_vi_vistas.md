Chapter	6.	Views	–	Designing	the	User Interface
====

This	chapter	will	help	you	build	the	graphical	interface	for	your	applications.	There	are several	different	types	of	views	and	widgets	available.	The	concepts	of	context	and domain	also	play	an	important	role	for	an	improved	user	experience,	and	you	will	learn more	about	them. 

The	todo_ui	module	has	the	model	layer	ready,	and	now	it	needs	the	view	layer	with	the user	interface.	We	will	add	new	elements	to	the	UI	as	well	as	modify	existing	views	that were	added	in	previous	chapters. 

The	best	way	to	modify	existing	views	is	to	use	inheritance,	as	explained	in	 Chapter	3 , *Inheritance	–	Extending	Existing	Applications*.	However,	for	the	sake	of	clarity,	we	will overwrite	the	existing	views,	replacing	them	with	completely	new	views.	This	will	make the	topics	easier	to	explain	and	follow. 

A	new	XML	data	file	for	our	UI	needs	to	be	added	to	the	module,	so	we	can	start	by editing	the	`__openerp__.py`	manifest	file.	We	will	need	to	use	some	fields	from	the 
todo_user	module,	so	it	must	be	set	as	a	dependency:
``` 
{ 'name':	'User	interface	improvements	to	the	To-Do	app',
  'description':	'User	friendly	features.',
  'author':	'Daniel	Reis',
  'depends':	['todo_user'],
  'data':	['todo_view.xml']
} 
```
Let’s	get	started	with	the	menu	items	and	window	actions. 
 

**Window	actions** 

Window	actions	give	instructions	to	the	client-side	user	interface.	When	a	user	clicks	on	a menu	item	or	a	button	to	open	a	form,	it’s	the	underlying	action	that	instructs	the	user interface	what	to	do. 

We	will	start	by	creating	the	window	action	to	be	used	on	the	menu	items,	to	open	the	to- do	tasks	and	stages	views.	Create	the	todo_view.xml	data	file	with	the	following	code: 
```
<?xml	version="1.0"?>
    <openerp>
        <data>
            <act_window	id="action_todo_stage"	name="To-Do	Task	Stages" res_model="todo.task.stage"	view_mode="tree,form"	/> 
	    <act_window	id="todo_app.action_todo_task"	name="	To-Do	Tasks"	 res_model="todo.task"	view_mode="tree,form,calendar,gantt,graph"	 target="current "context="{'default_user_id':	uid}" domain="[]" limit="80"	/> 
            <act_window	id="action_todo_task_stage"	name="To-Do	Task	Stages" res_model="todo.task.stage"	src_model="todo.task" multi="False"/>	
        </data> 
     </openerp> 
```
Window	actions	are	stored	in	the	`ir.actions.act_window`	model,	and	can	be	defined	in XML	files	using	the	`<act_window>`	shortcut	that	we	just	used. 
The	first	action	opens	the	task	stages	model,	and	uses	only	the	basic	attributes	for	a window	action. 

The	second	action	uses	an	ID	in	the	todo_app	namespace	to	overwrite	the	original	to-do task	action	of	the	todo_app	module.	It	uses	the	most	relevant	window	actions	attributes: 

- name:	This	is	the	title	displayed	on	the	views	opened	through	this	action. 
- res_model:	This	is	the	identifier	of	the	target	model. 
- view_mode:	These	are	the	view	types	to	make	available.	The	order	is	relevant	and	the first	in	the	list	is	the	view	type	opened	by	default. 
- target:	If	this	is	set	to	new,	it	will	open	the	view	in	a	dialog	window.	By	default,	it	is 
current,	and	opens	the	view	in	the	main	content	area. 
- context:	This	sets	context	information	on	the	target	views,	which	can	be	used	to	set default	values	on	fields	or	activate	filters,	among	other	things.	We	will	cover	its details	later	in	this	chapter. 
- domain:	This	is	a	domain	expression	setting	a	filter	for	the	records	that	will	be available	in	the	opened	views. 
- limit:	This	is	the	number	of	records	for	each	list	view	page,	80	by	default. 

The	window	action	already	includes	the	other	view	types	that	we	will	be	exploring	in	this chapter:	calendar,	Gantt,	and	graph.	Once	these	changes	are	installed,	the	corresponding buttons	will	be	seen	at	the	top-right	corner,	next	to	the	list	and	form	buttons.	Notice	that  these	won’t	work	until	we	create	the	corresponding	views. 

The	third	window	action	demonstrates	how	to	add	an	option	under	the	More 	button,	at	the top	of	the	view.	These	are	the	action	attributes	used	to	do	so. 

- multi:	This	flag,	if	set	to	True,	makes	it	available	in	the	list	view.	Otherwise,	it	will be	available	in	the	form	view. 
 

**Menu	items**

Menu	items	are	stored	in	the	ir.ui.menu	model,	and	can	be	searched	for	in	the	Settings  menu	by	navigating	to	Technical	|	User	Interface	|	Menu	Items .	If	we	search	for Messaging ,	we	will	see	that	it	has	Organizer 	as	one	of	its	submenus.	With	the	help	of	the developer	tools	we	can	find	the	XML	ID	for	that	menu	item:	it	is	mail.mail_my_stuff. 

We	will	replace	the	existing	To-do	Task 	menu	item	with	a	submenu	that	can	be	found	by navigating	to	Messaging	|	Organizer .	In	the	`todo_view.xml} ,	after	the	window	actions, add	this	code:
```
<menuitem	id="menu_todo_task_main"	name="To-Do"	 parent="mail.mail_my_stuff"	/>	<menuitem	id="todo_app.menu_todo_task"	 name="To-Do	Tasks"	parent="menu_todo_task_main"	sequence="10"	 action="todo_app.action_todo_task"	/>
<menuitem	id="menu_todo_task_stage"	 name="To-Do	Stages"	parent="menu_todo_task_main"	sequence="20"	 action="action_todo_stage"	/> 
```
The	menu	option	data	for	the	`ir.ui.menu`	model	can	also	be	loaded	using	the	`<menuitem>` shortcut	element,	as	used	in	the	preceding	code. 

The	first	menu	item,	To-Do, 	is	a	child	of	the	mail.mail_my_stuff	Organizer 	menu option.	It	has	no	action	assigned,	since	it	will	be	used	as	a	parent	for	the	next	two	options. 

The	second	menu	option	rewrites	the	option	defined	in	the	todo_app	module	so	that	it	is relocated	under	the	To-Do 	main	menu	item. 
The	third	menu	item	adds	a	new	option	to	access	the	to-do	stages.	We	will	need	it	in	order to	add	some	data	to	be	able	to	use	stages	in	to-do	tasks. 
 

**Context	and	domain**

We	have	stumbled	on	context	and	domain	several	times.	We	have	also	seen	that	window actions	are	able	to	set	values	on	them,	and	that	relational	fields	can	also	use	them	in	their attributes.	Both	concepts	are	useful	to	provide	richer	user	interfaces.	Let’s	see	how. 
 

**Session	context**

The	context	is	a	dictionary	carrying	session	data	to	be	used	by	client-side	views	and	by server	processes.	It	can	transport	information	from	one	view	to	another,	or	to	the	server- side	logic.	It	is	frequently	used	in	window	actions	and	relational	fields	to	send	information to	the	views	opened	through	them. 
Odoo	sets	some	basic	information	about	the	current	session	on	the	context.	The	initial session	information	can	look	like	this: 
```
{'lang':	'en_US',	'tz':	'Europe/Brussels',	'uid':	1} 
```
We	have	information	on	the	current	user	ID	and	the	language	and	time	zone	preferences for	the	user	session. 

When	using	an	action	on	the	client,	such	as	clicking	on	a	button,	information	about	the currently	selected	records	is	added	to	the	context: 

the	active_id	key	is	the	ID	of	the	selected	record	on	a	form, the	active_model	key	is	the	model	of	the	current	record, the	active_ids	key	is	the	list	of	IDs	selected	in	the	tree/list	view. 

The	context	can	also	be	used	to	provide	default	values	on	fields	or	to	enable	filters	on	the target	view.	To	set	on	the	user_id	field	a	default	value	corresponding	to	the	session’s current	user	we	would	use: 
```
{'default_user_id':	uid} 
```
And	if	the	target	view	has	a	filter	named	filter_my_tasks,	we	can	enable	it	using: 
```
{'search_default_filter_my_tasks':	True} 
``` 


**Domain	expressions**

Domains	are	used	to	filter	data	records.	Odoo	parses	them	to	produce	the	SQL	WHERE expressions	that	are	used	to	query	the	database. 
When	used	on	a	window	action	to	open	a	view,	domain	sets	a	filter	on	the	records	that	will be	available	in	that	view.	For	example,	to	limit	to	only	the	current	user’s	Tasks: 
```
domain=[('user_id',	'=',	uid)] 
```
The	uid	value	used	here	is	provided	by	the	session	context. 
When	used	on	a	relation	field,	it	will	limit	the	selection	options	available	for	that	field. The	domain	filter	can	also	use	values	from	other	fields	on	the	view.	With	this	we	can	have different	selection	options	available	depending	on	what	was	selected	on	another	field.	For example,	a	contact	person	field	can	be	made	to	show	only	the	persons	for	the	company that	was	selected	on	a	previous	field. 

A	domain	is	a	list	of	conditions,	where	each	condition	is	a	`('field',	'operator', value)`	tuple. 

The	left-hand	field	is	where	the	filter	will	be	applied	to,	and	can	use	dot-notation	on relation	fields. 

The	operators	that	can	be	used	are: 

The	usual	comparison	operators:	`<`,	`>`,	`<=`,	`>=`,	`=`,	and	`!=`	are	available. 

`=`like	to	match	against	the	value	pattern	where	the	underscore	symbol	matches	any single	character,	and	`%`	matches	any	sequence	of	characters. 
like	for	case-sensitive	match	against	the	`%value%`	SQL	pattern,	and	ilike	for	a case	insensitive	match.	The	not	like	and	not	ilike	operators	do	the	inverse operation. 

child_of	finds	the	direct	and	indirect	children,	if	parent/child	relations	are configured	in	the	target	model. 

in	and	not	in	check	for	inclusion	in	a	list.	In	this	case,	the	right-hand	value	should be	a	Python	list.	These	are	the	only	operators	that	can	be	used	with	list	values.	A curious	special	case	is	when	the	left-hand	is	a	to-many	field:	here	the	in	operator performs	a	contains	operation. 

The	right-hand	value	 can	be	a	constant	or	a	Python	expression	to	be	evaluated.	What	can be	used	in	these	expressions	depends	on	the	evaluation	context	available	(not	to	be confused	with	the	session	context,	discussed	in	the	previous	section).	There	are	two possible	evaluation	contexts	for	domains:	client	side	or	server	side. 

For	field	domains	and	window	actions,	the	evaluation	is	made	client-side.	The	evaluation context	here	includes	the	fields	available	in	the	current	view,	and	dot-notation	is	not available.	The	session	context	values,	such	as	uid	and	active_id,	can	also	be	used.	The 
datetime	and	time	Python	modules	are	available	to	use	in	date	and	time	operations,	and also	a	context_today()	function	returning	the	client	current	date. 
 
Domains	used	in	security	record	rules	and	in	server	Python	code	are	evaluated	on	the server	side.	The	evaluation	context	has	the	fields	of	the	current	record	available,	and	dot- notation	is	allowed.	Also	available	is	the	current	session’s	user	record.	Using	user.id here	is	the	equivalent	to	using	uid	in	the	client	side	evaluation	context. 
The	domain	conditions	can	be	combined	using	the	logical	operators:	'&'	for	“AND”	(the default),	'|'	for	“OR”,	and	'!'	for	“negation.”
 
The	negation	is	used	before	the	condition	to	negate.	For	example,	to	find	all	tasks	not belonging	to	the	current	user:	`['!',	('user_id',	'=',	uid)]`
 
The	“AND”	and	“OR”	operate	on	the	next	two	conditions.	For	example:	to	filter	tasks	for the	current	user	or	without	a	responsible	user: 
```
['|',	('user_id',	'=',	uid),	('user_id',	'=',	False)] 
```
A	more	complex	example,	used	in	server-side	record	rules: 
```
['|',	('message_follower_ids',	'in',	[user.partner_id.id]), 						'|',	('user_id',	'=',	user.id), 											('user_id',	'=',	False)]
``` 
This	domain	filters	all	records	where	the	followers	(a	many	to	many	relation	field)	contain the	current	user	plus	the	result	of	the	next	condition.	The	next	condition	is	again	the	union of	two	other	conditions:	the	records	where	the	user_id	is	the	current	session	user	or	it	is not	set. 
 

**Form	views**

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