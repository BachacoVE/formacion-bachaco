Chapter 8. Qweb To Do, Doing, Done,s learn how to use them. 
=====

![281_1](/images/Odoo Development Essentials - Daniel Reis-281_1.jpg)

![281_2](/images/Odoo Development Essentials - Daniel Reis-281_2.jpg)

**Kanban	views**

In	form	views,	we	use	mostly	specific	XML	elements, such	as	<field>	and	<group>,	and few	HTML	elements,	such as	<h1>	or	<div>.	With	kanban	views,	its	possible to	identify	two main	kanban	view	styles:	vignette	and card	kanbans. 

Examples	of	vignette 	style	kanban	views	can	be found	for	Customers ,	Products ,	and	also Apps	& Modules .	They	usually	have	no	border	and	are	decorated with	an	image	on	the left-hand	side,	as	shown in the	following	image: 

The	card 	style	kanban	is	usually	used	to display cards	organized	in	columns	for	the process stages. Examples	are	CRM	Opportunities 	and	Project	Tasks .	The main	content is	displayed	in	the	card	top	area	and additional	information	can	be	displayed	in	the bottom-right	and	bottom-left	areas,	as	shown	in the	following	image: 

We	will	see	the	skeleton	and	typical	elements	used	in  both	styles	of	views	so	that	you	can feel comfortable adapting	them	to	your	particular	use	cases. 


**Design	kanban	views**

First	thing	is	to	create	a	new	module adding our	kanban	views	to	to-do	tasks.	In	a	real- world	work,	situation	using	a	module	for	this would	probably	be	excessive	and	they	could perfectly well be	added	directly	in	the	todo_ui	module.	But	for a clearer	explanation,	we will	use	a	new	module and avoid	too	many,	and	possibly	confusing, changes	in already created	files.	We	will	name	it	todo_kanban and create	the	usual	initial	files	as	follows: 
```
$	cd	~/odoo-dev/custom-addons $	mkdir	todo_kanban $	touch	todo_kanban/__init__.py  
```
Now,	edit	the	descriptor	file	todo_kanban/__openerp__.py as follows:
``` 
{'name':	'To-Do	Kanban', 	'description':	'Kanban	board	for	to-do	tasks.', 	'author':	'Daniel	Reis', 	'depends':	['todo_ui'], 	'data':	['todo_view.xml']	} 
```
Next,	create	the	XML	file	where	our	shiny	new kanban	views	will	go	and	set	kanban	as the default view	on	the	to-do	task	Extending	Existing Applications*, for	more	details. 

Before	starting	with	the	kanban	views,	we	need	to add	a	couple	of	fields	to	the	to-do	tasks model. 


**Priority	and	kanban	state**

The	two	fields	that	are	frequently	used	in	kanban views	are	priority	and	kanban	state. Priority 	lets users	organize	their	work	items,	signaling	what should	be	addressed	first. Kanban	state 	signals whether a	task	is	ready	to	move	to	the	next	stage	or is blocked	for some	reason.	Both	are	supported	by selection fields	and	have	specific	widgets	to	use	on forms	and kanban	views. 

To	add	these	fields	to	our	model,	we	will	add a todo_kanban/todo_task.py	file,	as	shown in	the following: 
```
from	openerp	import	models,	fields class	TodoTask(models.Model): 				_inherit	=	'todo.task' 				priority	=	fields.Selection( 								[('0',	'Low'),	('1',	'Normal'),	('2',	'High')], 									'Priority',	default='1') 				kanban_state	=	fields.Selection( 								[('normal',	'In	Progress'), 									('blocked',	'Blocked'), 									('done',	'Ready	for	next	stage')], 									'Kanban	State',	default='normal') 
Let	--> 										<field	name="a_field"	/> 										<templates> 												<t	t-name="kanban-box"> 														<!--	HTML – >
```

Qweb	template	one	or more.	The	main	template	to	be	used must	be	named	kanban-box.	Other	templates	are allowed	for	HTML	fragments	to	include	in	the	main template. 

The	templates	use	standard	HTML,	but	can	include	the <field>	tag	to	insert	model	fields. Some	Qweb special	directives	for	dynamic	content	generation can also	be	used,	like	the  t-name	used	in	the	previous example. 

All	model	fields	used	have	to	be	declared	with	a <field>	tag.	If	they	are	used	only	in expressions,	we have	to	declare	them	before	the	<templates>	section. One	of	these	fields is	allowed	to	have	an	aggregated value,	displayed	at	the	top	of	the	kanban	columns. This	is done	by	adding	an	attribute	with	the aggregation to	use,	for	example: 
```
<field	name="effort_estimated"	sum="Total	Effort"	/> 
```
Here	the	sum	for	the	estimated	effort	field	is presented	at	the	top	of	the	kanban	columns with	the label	text	Total	Effort .	Supported	aggregations	are sum,	avg,	min,	max,	and	count. 

The	<kanban>	top	element	also	supports	a	few interesting attributes: 

- default_group_by:	This	sets	the	field	to	use	for	the default	column	groups. 
- default_order:	This	sets	a	default	order	to	use for	the	kanban	items. 
- quick_create="false":	This	disables	the	quick	create option	on	the	kanban	view. 
- class:	This	adds	a	CSS	class	to	the	root	element of the	rendered	kanban	view.

Now	lets	have a	closer	look	at	it. 


**Actions	in	kanban	views**

In	QWeb	templates,	the	<a>	tag	for	links	can	have a	type	attribute.	It	sets	the	type	of action	the	link will	perform	so	that	links	can	act	just	like	the buttons	in	regular	forms.	So in	addition	to	the <button>	elements,	the	<a>	tags	can	also	be	used	to	run Odoo	actions. 

As	in	form	views,	the	action	type	can	be	action or object,	and	it	should	be	accompanied by	a	name attribute,	identifying	the	specific	action	to execute.	Additionally,	the	following action	types	are also	available: 

- open:	This	opens	the	corresponding	form	view. 
- edit:	This	opens	the	corresponding	form	view	directly in	edit	mode. 
- delete:	This	deletes	the	record	and	removes	the	item from	the	kanban	view. 


**The	card	kanban	view**

The	card 	kanban	can	be	a	little	more	complex. It	has	a	main	content	area	and	two	footer sub -containers,	aligned	to	each	of	the	card	sides.	A button	opening	an	action	menu	may also	be	featured at	the	card
```
			<div	class="oe_kanban_bottom_right"></div> 												<div	class="oe_kanban_footer_left"></div> 								</div> 				</div> </t> 
A	card 	kanban	is	more	appropriate	for	the	to-do	tasks,	so	instead	of	the	view	described	in the	previous	section,	we	would	be	better	using	the	following: 
<t	t-name="kanban-box"> 				<div	class="oe_kanban_card"> 								<div	class="oe_kanban_content"> 												<!--	Option	menu	will	go	here!	--> 												<h4><a	type="open"> 																<field	name="name"	/> 												</a></h4> 												<field	name="tags"	/> 												<ul> 																<li><field	name="user_id"	/></li> 																<li><field	name="date_deadline"	/></li> 												</ul> 												<div	class="oe_kanban_bottom_right"> 																<field	name="kanban_state" 																							widget="kanban_state_selection"/> 												</div> 												<div	class="oe_kanban_footer_left"> 																<field	name="priority"	widget="priority"/> 												</div> 								</div> 				</div> </t> 
```
So	far	we	have	seen	static	kanban	views,	using a combination	of	HTML	and	special	tags (field,  button, a). But	we	can	have	much	more	interesting	results	using dynamically generated	HTML	content.	Lets	display	(the	DOM). 

This	is	not	meant	to	be	technically	exact.	It is just	a	mind	map	that	can	be	useful	to understand how things	work	in	kanban	views. 

Next	we	will	explore	the	several	QWeb	directives available,	using	examples	that	enhance our	to-do task kanban	card. 


**Conditional	rendering	with	t-if**

The	t-if	directive,	used	in	the	previous	example, accepts	a	JavaScript	expression	to	be evaluated. The	tag	and	its	content	will	be	rendered	if	the condition	evaluates	to	true. 

For	example,	in	the	card	kanban,	to	display	the	Task effort	estimate,	only	if	it	has	a	value, after	the date_deadline	field,	add	the	following: 
```
<t	t-if="record.effort_estimate.raw_value	>	0"><li>Estimate	<field	 name="effort_estimate"/></li></t> 
```
The	JavaScript	evaluation	context	has	a	record object	representing	the	record	being rendered,	with the	fields	requested	from	the	server.	The	field values can	be	accessed	using either	the	raw_value	or	the	value attributes: 

- raw_value:	This	is	the	value	returned	by	the read() server	method,	so	itt	be	able	to	go	into	that detail.	For	reference	purposes,	the	following identifiers are available	in	QWeb	expression	evaluation: 
- widget:	This	is	a	reference	to	the	current KanbanRecord widget	object,	responsible for	the	rendering	of	the current	record	into	a	kanban	card.	It	exposes some	useful helper	functions	we	can	use. 
- record:	This	is	a	shortcut	for	widget.records	and provides	access	to	the	fields available,	using	dot notation. 
- read_only_mode:	This	indicates	if	the	current	view	is in read	mode	(and	not	in	edit mode).	It	is	a shortcut for widget.view.options.read_only_mode.
- instance:	This	is	a	reference	to	the	full	web client instance.

It	is	also	noteworthy	that	some	characters	are	not allowed	inside	expressions.	The	lower than	sign	(*<*) is	such	a	case.	You	may	use	a	negated	*>=* instead.	Anyway,	alternative symbols	are	available	for inequality	operations	as	follows: 

- lt:	This	is	for	less	than. 
- lte:	This	is	for	less	than	or	equal	to. 
- gt:	This	is	for	greater	than. 
- gte:	This	is	for	greater	than	or	equal	to.


**Rendering	values	with	t-esc	and	t-raw**

We	have	used	the	<field>	element	to	render	the	field content.	But	field	values	can	also	be presented directly	without	a	<field>	tag.	The	t-esc	directive evaluates	an	expression	and renders	its	HTML	escaped value,	as	shown	in	the	following: 
```
<t	t-esc="record.message_follower_ids.raw_value"	/> 
```
In	some	cases,	and	if	the	source	data	is	ensured to be	safe,	t-raw	can	be	used	to	render	the field raw	value,	without	any	escaping,	as	shown	in	the following	code: 
```
<t	t-raw="record.message_follower_ids.raw_value"	/> 
```

**Loop	rendering	with	t-foreach**

A	block	of	HTML	can	be	repeated	by	iterating through	a	loop.	We	can	use	it	to	add	the avatars	of	the	task	followers	to	the	tasks	start by rendering	just	the	Partner	IDs	of	the	task,	as follows: 
```
<t	t-foreach="record.message_follower_ids.raw_value"	t-as="rec"> 				<t	t-esc="rec"	/>; </t> 
```
The	t-foreach	directive	accepts	a	JavaScript	expression evaluating	to	a	collection	to iterate.	In	most cases,	this	will	be	just	the	name	of	a	*to	many* relation	field.	It	is	used	with	a  t-as	directive	to set	the	name	to	be	used	to	refer	to	each	item	in the	iteration. 

In	the	previous	example,	we	loop	through	the	task followers,	stored	in	the  message_follower_ids field. Since	there	is	limited	space	on	the	kanban card,	we	could have	used	the	slice()	JavaScript function	to	limit	the	number	of	followers	to display,	as shown	in	the	following: 
```
t-foreach="record.message_follower_ids.raw_value.slice(0,	3)" 
```
The	rec	variable	holds	each	iterations	avatar stored in	the	database.	Kanban	views	provide	a helper function	to	conveniently	generate	that:	kanban_image(). It	accepts	as	arguments the	model	name,	the	field name	holding	the	image	we	want,	and	the	ID	for the	record	to retrieve. 

With	this,	we	can	rewrite	the	followers	loop	as follows: 
```
<div> 		<t	t-foreach="record.message_follower_ids.raw_value.slice(0,	3)" 					t-as="rec"> 				<img	t-att-src="kanban_image( 																						'res.partner',	'image_small',	rec)" 									class="oe_kanban_image	oe_kanban_avatar_smallbox"/> 		</t> </div> 
```
We	used	it	for	the	src	attribute,	but	any	attribute can	be	dynamically	generated	with	a	`t-  att-` prefix. 

String	substitution	in	attributes	with	`t-attf-` prefixes.
  
Another	way	to	dynamically	generate	tag	attributes is using	string	substitution.	This	is helpful	to	have parts	of	larger	strings	generated	dynamically,	such as	a	URL	address	or CSS	class	names. 

The	directive	contains	expression	blocks	that	will	be evaluated	and	replaced	by	the	result. These	are	delimited either	by	{{	and	}}	or	by	#{	and	}.	The content	of	the	blocks	can	be any	valid	JavaScript expression	and	can	use	any	of	the	variables	available for	QWeb expressions,	such	as	record	and	widget. 

Now	lets rework	it	to	use	a	sub-template.	We should start	by	adding	another	template	to	our	XML file, inside	the	<templates>	element,	after	the	`<t	t-name="kanban-box">`	node,	as	shown in	the	following: 
```
<t	t-name="follower_avatars"> <div> 		<t	t-foreach="record.message_follower_ids.raw_value.slice(0,	3)" 					t-as="rec"> 				<img	t-att-src="kanban_image( 									'res.partner',	'image_small',	rec)" 									class="oe_kanban_image	oe_kanban_avatar_smallbox"/> 		</t> </div> </t> 
```
Calling	it	from	the	kanban-box	main	template	is	quite straightforwardfor	eacht	exist	in	the	caller3s	value when	performing	the	sub-template	call	as	follows: 
```
<t	t-call="follower_avatars"> 				<t	t-set="arg_max"	t-value="3"	/> </t> 
```
The	entire	content	inside	the	t-call	element	is also	available	to	the	sub-template	through the	magic variable	0.	Instead	of	the	argument	variables,	we can	define	an	HTML	code fragment	that	could	be inserted	in	the	sub-template	using	`<t	t-raw="0"	/>`. 


**Other	QWeb	directives**

We	have	gone	through	through	the	most	important	Qweb directives,	but	there	are	a	few more	we	should be aware	of.	Weve	seen	the	basics	about	kanban views and	QWeb	templates.	There	are	still	a	few techniques	we	can	use	to	bring	a	richer	user experience	to	our	kanban	cards. 


**Adding	a	kanban	card	option	menu**

Kanban	cards	can	have	an	option	menu,	placed at the	top	right.	Usual	actions	are	to	edit	or delete the	record,	but	any	action	callable	from	a	button is possible.	There	is	also	available a	widget	to set the card
```
 </a></li> 								</t> 								<t	t-if="widget.view.is_action_enabled('delete')"> 								<li><a	type="delete">Delete</a></li> 								</t> 								<!--	Color	picker	option:	--> 								<li><ul	class="oe_kanban_colorpicker" 																data-field="color"/></li> 				</ul> </div> 
```
It	is	basically	an	HTML	list	of	<a>	elements.	The	Edit  and	Delete 	options	use	QWeb	to make	them	visible	only when	their	actions	are	enabled	on	the	view.	The 
widget.view.is_action_enabled	function	allows	us	to inspect if	the	edit	and	delete actions	are	available	and	to decide what	to	make	available	to	the	current	user. 


**Adding	colors	to	kanban	cards**

The	color	picker	option	allows	the	user	to choose the	color	of	a	kanban	card.	The	color	is stored	in	a	model	field	as	a	numeric index. 

We	should	start	by	adding	this	field	to	the to-do	task	model,	by	adding	to `todo_kanban/todo_model.py`	the	following	line: 
```
				color	=	fields.Integer('Color	Index') 
```
Here	we	used	the	usual	name	for	the	field,	color, and this	is	what	is	expected	in	the	data-  field	attribute on	the	color	picker. 

Next,	for	the	colors	selected	with	the	picker	to have	any	effect	on	the	card,	we	must	add some dynamic	CSS	based	on	the	color	field	value. On the	kanban	view,	just	before	the  <templates>	tag, we	must	also	declare	the	color	field,	as	shown in the	following: 
```
<field	name="color"	/> 
```
And,	we	need	to	replace	the	kanban	card	top	element, <div	class="oe_kanban_card">, with	the	following: 
```
<div	t-attf-class="oe_kanban_card 																			#{kanban_color(record.color.raw_value)}"> 
```
The	kanban_color	helper	function	does	the	translation of the	color	index	into	the corresponding	CSS	class name. 

And	that).	A	helper	function	for this	is	available in	kanban	views. 

For	example,	to	limit	our	to-do	task	titles	to the	first	32	characters,	we	should	replace	the 
<field	name="name"	/>	element	with	the	following: 
```
<t	t-esc="kanban_text_ellipsis(record.name.value,	32)"	/> 
```

**Custom	CSS	and	JavaScript	assets**

As	we	have	seen,	kanban	views	are	mostly	HTML and	make	heavy	use	of	CSS	classes. We	have	been introducing	some	frequently	used	CSS	classes	provided by	the	standard product.	But	for	best	results,	modules can	also	add	their	own	CSS. 

We	are	not	going	into	details	here	on	how	to	write CSS,	but	itt	work,	since	we	havenWebkit	HTML	to PDF.s	probably	not	what	you	will	get	now	on	your system.	Lett	display	the	You	need	Wkhtmltopdf	to	print a	pdf	version	of	the	reports	time	library  

- user:	This is	the	record	for	the	user	running	the report 
- res_company:	This	is	the	record	for	the	current	user Designing	the	User	Interface*,	with	an additional	widget to set	the	widget	to	use	to	render	the	field. 

A	common	example	is	a	monetary	field,	as	shown in	the	following: 
```
<span	t-field="o.amount" 						t-field-options='{ 								"widget":	"monetary", 								"display_currency":	"o.pricelist_id.currency_id"}'/> 
```
A	more	sophisticated	case	is	the	contact	widget,	used to	format	addresses,	as	shown	in the	following: 
```
<div	t-field="res_company.partner_id"	t-field-options='{ 							"widget":	"contact", 							"fields":	["address",	"name",	"phone",	"fax"], 							"no_marker":	true}'	/> 
```
By	default,	some	pictograms,	such	as	a	phone,	are displayed	in	the	address.	The  no_marker="true"	option disables	them. 


**Enabling	language	translation	in	reports**

A	helper	function,	translate_doc(),	is	available	to dynamically	translate	the	report content	to	a	specific language. 

It	needs	the	name	of	the	field	where	the language	to	use	can	be	found.	This	will	frequently be the	Partner	the	document	is	to	be	sent	to,	usually stored	at	partner_id.lang.	In	our case,	we	dons	also a	less	efficient	method. 

If	you	cans	growing	in importance	in	the	Odoo	toolset. Finally,	you	had	an	overview	on	how	to	create reports, also	using	the	QWeb	engine. 

In	the	next	chapter,	we	will	explore	how	to	leverage the	RPC	API	to	interact	with	Odoo from	external applications. 
