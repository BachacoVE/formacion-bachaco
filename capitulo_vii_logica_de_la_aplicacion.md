Capítulo 7. Lógica de Aplicación ORM – Soportando Procesos de Negocio
=====

En este cap+itulo, aprenderás a escribir código para soportar la lógica de negocio en tus modelos y además aprenderás como puede ser activada en eventos y en acciones de usuario. Usando la API de programación de Odoo, podemos escribir lógica compleja y los asistentes nos permiten proveer una interacción rica con el usuario con estos programas.
 
To-do	wizard  With	the	wizards,	we	can	ask	users	to	input information	to	be	used	in	some	processes. Suppose	our	to-do	app users	regularly	need	to	set	deadlines	and	the	responsible	persons for a	large	number	of	tasks.	We	could	use	an	assistant	to help	them	with	this.	It	should allow	them	to	pick	the	tasks	to be updated	and	then	choose	the	deadline	date	and/or	the responsible	user	to	set	on	them. 

We	will	start	by	creating	a	new	module	for	this	feature: todo_wizard.	Our	module	will have	a	Python	file	and	an XML file,	so	the	todo_wizard/__openerp__.py	description	will be	as	shown in	the	following	code: 
```
{	'name':	'To-do	Tasks	Management	Assistant', 
      'description':	'Mass edit	your	To-Do	backlog.',
      'author': 'Daniel	Reis', 'depends':	['todo_user'],
      'data': ['todo_wizard_view.xml'],	
} 
```
The	todo_wizard/__init__.py	file	to	load	our	code	is	just	one	line, as	follows: 
```
from	.	import	todo_wizard_model 
```
Next,	we	need	to	describe	the	data	model	supporting	our	wizard. 
 

**Wizard	model**

A	wizard	displays	a	form	view	to	the	user,	usually	in a dialog	window,	with	some	fields	to be	filled	in.	These will then	be	used	by	the	wizard	logic.
 
This	is	implemented	using	the	model/view	architecture	used	for regular views,	with	a difference:	the	supporting	model	is	based	on models.TransientModel	instead	of `models.Model`.
 
This	type	of	model	is	also	stored	in	the	database,	but	the data	is	expected	to	be	useful	only until	the	wizard	is completed	or	canceled.	Server	vacuum	processes	regularly	clean up	old wizard	data	from	the	corresponding	database	tables. 

The	todo_wizard/todo_wizard_model.py	file	will	define	the	three	fields we	need:	the lists	of	tasks	to	update,	the	user	responsible	for them,	and	the	deadline	to	set	on	them,	as shown	here: 
```
#	-*-	coding:	utf-8	-*- from	openerp	import	models,	fields,	api from	openerp	import	exceptions		#	will	be	used	in	the	code 
import	logging _logger	=	logging.getLogger(__name__) 
class	TodoWizard(models.TransientModel): 		_name	=	'todo.wizard' task_ids	=	fields.Many2many('todo.task',	string='Tasks') new_deadline	=	fields.Date('Deadline	to	Set') new_user_id	=	fields.Many2one( 'res.users',string='Responsible	to	Set') 
```
It’s	worth	noting	that	if	we	used	a	*one	to	many*	relation, we would	have	to	add	the	inverse *many	to	one*	field	on	to-do	tasks. We	should	avoid	*many	to	one*	relations	between	transient and regular	models,	and	so	we	used	a	*many	to	many*	relation that	fulfills	the	same	purpose without	the	need	to	modify the	to-do task	model. 

We	are	also	adding	support	to	message	logging.	The	logger is	initialized	with	the	two	lines just	before	the	TodoWizard,	using the	Python	logging	standard	library.	To	write	messages to the log	we	can	use: 
```
_logger.debug('A	DEBUG	message') _logger.info('An	INFO	message') _logger.warning('A	WARNING	message') _logger.error('An	ERROR	message') 
```
We	will	see	some	usage	examples	in	this	chapter. 
 

**Wizard	form**

The	wizard	form	view	looks	exactly	the	same	as	regular	forms, except	for	two	specific elements: 

- A	<footer>	section	can	be	used	to	place	the	action buttons. 
- A	special	cancel	button	type	is	available	to	interrupt the	wizard	without	performing any	action. 

This	is	the	content	of	our	`todo_wizard/todo_wizard_view.xml`	file:
``` 
<openerp> 		<data> 				<record	id="To-do	Task	Wizard"	model="ir.ui.view"> 						<field	name="name">To-do	Task	Wizard</field> 						<field	name="model">todo.wizard</field> 						<field	name="arch"	type="xml"> 
								<form> 										<div	class="oe_right"> 												<button	type="object"	name="do_count_tasks"	string="Count"	/> 												<button	type="object"	name="do_populate_tasks"	string="Get	All"	 /> 										</div> 										<field	name="task_ids"	/> 										<group> 												<group>	<field	name="new_user_id"	/>	</group> 												<group>	<field	name="new_deadline"	/>	</group> 										</group> 										<footer> 												<button	type="object"	name="do_mass_update"	string="Mass	 Update"	class="oe_highlight"	attrs="{'invisible':	 [('new_deadline','=',False),	('new_user_id',	'=',False)]}"	/> 												<button	special="cancel"	string="Cancel"/> 										</footer> 								</form> 						</field> 				</record> 
				<!--	More	button	Action	→ <act_window	id="todo_app.action_todo_wizard"	name="To-Do	Tasks	Wizard" src_model="todo.task"	res_model="todo.wizard"	view_mode="form"	target="new" multi="True"	/> 		</data> 	</openerp> 
```
The	window	action	we	see	in	the	XML	adds	an	option to the	More 	button	of	the	to-do	task form	by	using	the src_model attribute.	target="new"	makes	it	open	as	a	dialog window. 

You	might	also	have	noticed	attrs	in	the	Mass	Update 	button used to	make	it	invisible until	either	a	new	deadline	or responsible	user	is	selected. 

This	is	how	our	wizard	will	look: 
 
![251_1](/images/Odoo Development Essentials - Daniel Reis-251_1.jpg)
 

**Wizard	business	logic**

Next	we	need	to	implement	the	actions	performed	while	clicking on the	Mass	Update  button.	The	method	called	by	the	button is do_mass_update	and	it	should	be	defined	in	the `todo_wizard/todo_wizard_model.py`	file,	as	shown	in	the	following	code:
``` 
				@api.multi 				def	do_mass_update(self): self.ensure_one() if	not	(self.new_deadline	or	self.new_user_id): raise	 exceptions.ValidationError('No	data	to	update!') #	else: _logger.debug('Mass	update	on	Todo	Tasks	%s',self.task_ids.ids) if self.new_deadline:self.task_ids.write({'date_deadline': self.new_deadline}) 								if self.new_user_id:self.task_ids.write({'user_id':	 self.new_user_id.id}) return	True 
```
Our	code	can	handle	only	one	wizard	instance	at	a	time. We	could	have	used	@api.one, but	it	is	not	advised	to	do so in	wizards.	In	some	cases,	we	want	the	wizard	to return a window	action	telling	the	client	what	to	do	next.	That is	not	possible	with	@api.one,	since it	would	return	a	list of	actions	instead	of	a	single	one. 

Because	of	this,	we	prefer	to	use	@api.multi	but	then	we use ensure_one()	to	check	that self	represents	a	single	record. It should	be	noted	that	self	is	a	record	representing the data	on	the	wizard	form. 

The	method	begins	by	validating	if	a	new	deadline	date or	responsible	user	was	given,	and raises	an	error	if	not.	Next, we	demonstrate	writing	a	message	to	the	server	log.  If the validation	passes,	we	write	the	new	values	given	to	the selected tasks.	We	are	using the	write	method	on	a	record	set, such	as	the	task_ids	to	many	field	to	perform	a	mass update. 

This	is	more	efficient	than	repeating	a	write	on	each	record in a loop. Now	we	will	work	on	the	logic	behind	the	two	buttons at the	top:	Count 	and	Get	All . 
 

**Raising	exceptions**

When	something	is	not	right,	we	will	want	to	interrupt	the program	with	an	error	message. This	is	done	by	raising	an exception.	Odoo	provides	a	few	additional	exception	classes	to the ones	available	in	Python.	These	are	examples	for	the	most	useful ones:
``` 
from	openerp	import	exceptions raise	exceptions.Warning('Warning	message') raise	exceptions.ValidationError('Not	valid	message') 
The	Warning	message	also	interrupts	execution	but	can	sound	less	severe	that	a 
ValidationError.	While	it’s	not	the	best	user	interface,	we	take	advantage	of	that	on	the Count 	button	to	display	a	message	to	the	user: 
				@api.multi 				def	do_count_tasks(self): 								Task	=	self.env['todo.task'] 								count	=	Task.search_count([]) 								raise	exceptions.Warning('There	are	%d	active	tasks.'	%	count) 
 ```

**Auto-reloading	code	changes**

When	you’re	working	on	Python	code,	the	server	needs	t be restarted	every	time	the	code is	changed	to	reload	it.	To make	life	easier	for	developers	an	--auto-reload	option	is available.	It	monitors	the	source	code	and	automatically reloads it if	changes	are	detected. Here	is	an	example	of	it’s usage: 
```
$	./odoo.py	-d	v8dev	--auto-reload
```
But	this	is	a	Linux-only	feature.	If	you	are	using Debian/Ubuntu	box	to	run	the	server,	as recommended	in	 Odoo Development Essentials - Daniel Reiss.html#45 Chapter	1 ,	*Getting	Started with Odoo	Development*,	it	should	work	for you.	The	pyinotify	Python package	is	required,	and	it	should	be	installed	either through apt-get	or	pip,	as	shown	here: 
```
$	sudo	apt-get	install	python-pyinotify		#	using	OS packages $	pip	install	pyinotify		#	using	pip,	possibly in	a	virtualenv  
```
 
**Actions	on	the	wizard	dialog**  

Now	suppose	we	want	a	button	to	automatically	pick	all the	to-do	tasks	to	spare	the	user from	picking	them	one	by	one. That’s	the	point	of	having	the	Get	All 	button	in	the form. The	code	behind	this	button	will	get	a	record	set with	all	active	tasks	and	assign	it	to	the tasks	in	the many	to	many	field. 

But	there	is	a	catch	here.	In	dialog	windows,	when	a	button is	pressed,	the	wizard	window is	automatically	closed.	We didn’t	face	this	problem	on	the	Count	button	because	it uses	an exception	to	display	it’s	message;	so	the	action fails and	the	window	is	not	closed. 

Fortunately	we	can	work	around	this	behavior	by	returning	an action	to	the	client	that reopens	the	same	wizard.	The model	methods	are	allowed	to	return	an	action	for	the web client	to	perform,	in	the	form	of	a	dictionary	describing the	window	action	to	execute. This	dictionary	uses	the	same attributes	used	to	define	window	actions	in	module	XML. 

We	will	use	a	helper	function	for	the	window	action dictionary	to	reopen	the	wizard window,	so	that	it	can	be easily	reused	in	several	buttons,	as	shown	here: 
```
				@api.multi 				def	do_reopen_form(self): self.ensure_one() return	{	'type':	'ir.actions.act_window', 'res_model':	self._name,		 #	this	model	'res_id':	self.id, #	the	current	wizard	record	'view_type':	 'form', 'view_mode': 'form', 'target':	'new'} 
```
It	is	worth	noting	that	the	window	action	could	be anything else,	like	jumping	to	a	specific form	and	record,	or opening another	wizard	form	to	ask	for	additional	user	input. 

Now	the	Get	All 	button	can	do	its	job	and	keep	the	user working	on	the	same	wizard: 
```
				@api.multi 				def	do_populate_tasks(self): self.ensure_one() Task	= self.env['todo.task'] 								all_tasks =	Task.search([]) self.task_ids	=	all_tasks #	reopen	wizard	form	on	same	wizard	record return	self.do_reopen_form() 
```
Here	we	can	see	how	to	get	a	reference	to	a	different model,	which	is	todo.task	in	this case,	to	perform	actions on it.	The	wizard	form	values	are	stored	in	the transient model and	can	be	read	and	written	as	in	regular	models. We can	also	see	that	the	method	sets	the  task_ids	value	with	the list	of	all	active	tasks. 

Note	that	since	self	is	not	guaranteed	to	be	a	single record, we validate	that	using self.ensure_one().	We	shouldn’t	use	the `@api.one`	decorator	because	it	would	wrap	the returned	value	in a list.	Since	the	web	client	expects	to	receive	a	dictionary and	not	a	list, it	wouldn’t	work	as	intended. 
  

**Working	with	the	server**

Our	server	code	will	usually	run	inside	a	method	of a model,	as	is	the	case	for  do_mass_update()	in	the	preceding code.  In	this	context,	self	represents	the	recordset	being	acted	upon. 

Instances	of	model	classes are	actually	recordsets.	For	actions executed from	views,	this	will	be	only	the	record currently	selected	on it. If	it’s	a	form	view,	it	is	usually	a	single record,	but in tree	views, there	can	be	several	records. 

The	self.env	object	allows	us	to	access	our	running environment;	this	includes	the information	on	the	current session, such	as	the	current	user	and	session	context,	and	also access all	the	other	models	available	in	the	server. 
To	better	explore	programming	on	the	server	side,	we	can use	the	server	interactive console,	where	we	have	an	environment similar	to	what	we	can	find	inside	a	model method. 

This	is	a	new	feature	for	version	9.	It	has	been	back ported	as	a	module	for	version	8,	and it	can	be downloaded	from	the	link	 https://www.odoo.com/apps/modules/8.0/shell/ https://www.odoo.com/apps/modules/8.0/shell/ .	It	just needs	to	be	placed somewhere	in	your	add-ons	path,	and	no	further	installation is necessary,	or	you	can	use	the	following	commands	to	get	the code	from	GitHub	and	make the	module	available	in	our	custom add-ons	directory: 
```
$	cd	~/odoo-dev $	git	clone	https://github.com/OCA/server-tools.git	-b	8.0 $	ln	-s	server-tools/shell	custom-addons/shell $	cd	~/odoo-dev/odoo  
To	use	this,	run	odoo.py	with	the	shell	command	and	the	database	to	use	as	shown	here: 
$	./odoo.py	shell	-d	v8dev  
```
You	will	see	the	server	start	up	sequence	in	the	terminal ending	in	a	>>>	Python	prompt. Here,	self	represents	the record	for	the	administrator	user	as	shown	here: 
```
>>>	self res.users(1,) >>>	self.name u'Administrator' >>>	self._name 'res.users' >>>	self.env <openerp.api.Environment	object	at	0xb3f4f52c>  
```
In	the	session	above,	we	do	some	inspection	on	our environment.	self	represents	a  res.users	recordset	containing	only the	record	with	ID	1	and name	Administrator.	We can	also confirm	the	recordset’s	model	name	inspecting	self._name,	and	confirm that 
self.env	is	a	reference	for	the	environment. 

As	usual,	you	can	exit	the	prompt	using	*Ctrl*	+	*D*. This	will	also	close	the	server	process and	bring	you	back	to	the system	shell	prompt. 
 
The	Model	class	referenced	by	self	is	in	fact	a	recordset,	an terable	collection	of	records. Iterating	through	a	recordset returns	individual	records. 

The	special	case	of	a	recordset	with	only	one	record	is called	a	singleton .	Singletons behave	like	records,	and	for	all practical	purposes	are	the	same	thing	as	a	record.	This particularity	means	that	a	record	can	be	used	wherever	a recordset	is	expected. 

Unlike	multi-element	recordsets,	singletons	can	access	their	fields using	the	dot	notation, as	shown	here: 
```
>>>	print	self.name Administrator >>>	for	rec	in	self: print	rec.name Administrator  
```
In	this	example,	we	loop	through	the	records	in	the	self recordset	and	print	out	the content	of	their	name	field.	It contains only	one	record,	so	only	one	name	is	printed	out.	As you can see,	self	is	a	singleton	and	behaves	as	a	record, but	at the same	time	is	iterable like	a	recordset. 
 

**Using	relation	fields**
As	we	saw	earlier,	models	can	have	relational	fields:	many to	one ,	one	to	many, 	and many	to	many .	These	field	types have	recordsets	as	values. 

In	the	case	of	many	to	one,	the	value	can	be	a	singleton or an	empty	recordset.	In	both cases,	we	can	directly	access	their field	values.	As	an	example,	the	following	instructions are correct and	safe: 
```
>>>	self.company_id res.company(1,) >>>	self.company_id.name u'YourCompany' >>> self.company_id.currency_id res.currency(1,) >>> self.company_id.currency_id.name u'EUR'  
```
Conveniently,	an	empty	recordset	also	behaves	like	a	singleton, and	accessing	its	fields does	not	return	an	error	but	just returns False.	Because	of	this,	we	can	traverse	records using	dot notation	without	worrying	about	errors	from	empty	values,	as shown	here: 
```
>>>	self.company_id.country_id res.country() >>>	self.company_id.country_id.name False  
```

**Querying	models**

With	self	we	can	only	access	the	method’s	recordset.	But	the self.env	environment reference	allows	us	to	access	any	other model.
 
For	example,	self.env['res.partner']	returns	a	reference	to	the Partners	model	(which is	actually	an	empty	recordset).	We	can	then use	search()	or	browse()	on	it	to	generate recordsets. 

The	search()	method	takes	a	domain	expression	and	returns a recordset	with	the	records matching	those	conditions.	An	empty	domain [] will	return	all	records.	If	the	model	has the	active special field,	by	default	only	the	records	with	active=True	will	be considered. A	few	optional	keyword	arguments	are	available,	as shown	here:
 
- order:	This	is	a	string	to	be	used	as	the	ORDER	BY clause	in	the	database	query. This	is	usually	a	comma-separated	list	of	field	names. 
- limit:	This	sets	a	maximum	number	of	records	to retrieve. 
- offset:	This	ignores	the	first	n	results;	it	can	be	used with	limit	to	query	blocks	of records	at	a	time. 

Sometimes	we	just	need	to	know	the	number	of	records meeting certain	conditions.	For that	we	can	use	search_count(),	which returns	the	record	count	instead	of	a	recordset. 

The	browse()	method	takes	a	list	of	IDs	or	a	single ID and	returns	a	recordset	with	those records.	This	can	be convenient	for	the	cases	where	we	already	know	the	IDs	of	the records	we	want. 

Some	usage	examples	of	this	are	shown	here: 
```
>>>	self.env['res.partner'].search([('name',	'like',	'Ag')]) res.partner(7,	51) >>>	self.env['res.partner'].browse([7,	51]) res.partner(7,	51)  
 ```

**Writing	on	records**

Recordsets	implement	the	active	record	pattern.	This	means	that we	can	assign	values	on them,	and	these	changes	will	be made	persistent	in	the	database.	This	is	an	intuitive	and convenient	way	to	manipulate	data,	as	shown	here: 
```
>>>	admin	=	self.env['res.users'].browse(1) >>>	admin.name	=	'Superuser' >>>	print	admin.name Superuser  
```
Recordsets	have	three	methods	to	act	on	their	data:	create(), write(), and	unlink(). 

The	create()	method	takes	a	dictionary	to	map	fields	to values	and	returns	the	created record.	Default	values	are automatically	applied	as	expected,	which	is	shown	here: 
```
>>>	Partner	=	self.env['res.partner'] >>>	new	= Partner.create({'name':	'ACME',	'is_company':	True}) >>>	print	new res.partner(72,)  
```
The	unlink()	method	deletes	the	records	in	the	recordset, as	shown	here: 
```
>>>	rec	=	Partner.search([('name',	'=',	'ACME')]) >>>	rec.unlink() True  
```
The	write()	method	takes	a	dictionary	to	map	fields	to values.	These	are	updated	on	all elements	of	the recordset and nothing	is	returned,	as	shown	here: 
```
>>>	Partner.write({'comment':	'Hello!'})  
```
Using	the	active	record	pattern	has	some	limitations;	it updates	only	one	field	at	a	time. On	the	other	hand,	the write() method	can	update	several	fields	of	several records	at the same	time	by	using	a	single	database	instruction.	These differences	should	be	kept	in	mind for	the	cases	where	performance can	be	an	issue. 

It	is	also	worth	mentioning	copy()	to	duplicate	an	existing record;	it	takes	that	as	an optional	argument	and	a	dictionary with	the	values	to	write	on	the	new	record.	For example,  to create	a	new	user	copying	from	the	Demo	User: 
```
>>>	demo	=	self.env.ref('base.user_demo') >>>	new	=	demo.copy({'name': 'Daniel',	'login':	'dr',	'email':''}) >>>	self.env.cr.commit()  
```
Remember	that	fields	with	the	copy=False	attribute	won’t	be copied. 

 
**Transactions	and	low-level	SQL**

Database	writing	operations	are	executed	in	the	context	of a database	transaction.	Usually we	don’t	have	to	worry	about	this	as the server	takes	care	of	that	while	running	model methods. 

But	in	some	cases,	we	may	need	a	finer	control	over	the transaction.	This	can	be	done through	the	database	cursor self.env.cr,	as	shown	here: 

- self.env.cr.commit():	This	commits	the	transaction’s	buffered	write operations. 
- self.env.savepoint():	This	sets	a	transaction	savepoint	to	rollback to. 
- self.env.rollback():	This	cancels	the	transaction’s	write	operations since	the	last savepoint	or	all	if	no	savepoint	was	created. 


* Tip *  
* In	a	shell	session,	your	data	manipulation	won’t	be	made effective	in	the	database until	you	use	self.env.cr.commit(). *

With	the	cursor	execute()	method,	we	can	run	SQL	directly in the	database.	It	takes	a string	with	the	SQL	statement	to run and a second	optional	argument	with	a	tuple	or	list	of values to use as	parameters	for	the	SQL.	These	values	will	be	used	where	%s placeholders are	found. 

If	you’re	using	a	SELECT	query,	records	should	then be	fetched.	The	fetchall() function	retrieves	all	the	rows	as a list	of	tuples	and	dictfetchall()	retrieves	them	as	a list of dictionaries,	as	shown	in	the	following	example: 
```
>>>	self.env.cr.execute("SELECT	id,	login	FROM	res_users	WHERE login=%s	OR	id=%s",	('demo',	1)) >>> self.env.cr.fetchall()	 						[(4,	u'demo'), (1,	u'admin')]  
```
It’s	also	possible	to	run	data	manipulation	language	instructions (DML)	such	as	UPDATE and	INSERT.	Since	the	server	keeps	data caches, they	may	become	inconsistent	with	the actual	data	in	the database.	Because	of	that,	while	using	raw	DML,	the	caches	should be cleared	afterwards	by	using	self.env.invalidate_all(). 

* Note *  
* Caution! *  
* Executing	SQL	directly	in	the	database	can	lead	to	inconsistent data.	You	should	use	it only	if	you	are	sure	of	what	you are	doing. *
 

**Working	with	time	and	dates**

For	historical	reasons,	date	and	datetime	values	are	handled as strings	instead	of	the corresponding	Python	types.	Also datetimes	are	stored	in	the	database	in	UTC	time.	The formats used	in	the	string	representation	are	defined	by: 
```
openerp.tools.misc.DEFAULT_SERVER_DATE_FORMAT 
openerp.tools.misc.DEFAULT_SERVER_DATETIME_FORMAT 
``
They	map	to	`%Y-%m-%d`	and	`%Y-%m-%d	%H:%M:%S`	respectively. 

To	help	handle	dates,	fields.Date	and	fields.Datetime	provide a few	functions.	For example: 
```
>>>	from	openerp	import	fields >>>	fields.Datetime.now() '2014-12-08 23:36:09' >>>	fields.Datetime.from_string('2014-12-08	23:36:09') datetime.datetime(2014,	12,	8,	23,	36,	9)  
```
Given	that	dates	and	times	are	handled	and	stored	by	the	server in	a	naive	UTC	format, which	is	not	time	zone	aware	and	is probably	different	from	the	time	zone	that	the	user	is working	on, a few	other	functions	that	help	to	deal	with	this	are	shown	here: 

- fields.Date.today():	This	returns	a	string	with	the	current date in	the	format expected	by	the	server	and	using	UTC	as a reference.	This	is	adequate	to	compute default	values. 
- fields.Datetime.now():	This	returns	a	string	with	the current datetime	in	the	format expected	by	the	server	using	UTC	as a reference.	This	is	adequate	to	compute	default values. 
- fields.Date.context_today(record,	timestamp=None):	This	returns	a	string with the	current	date	in	the	session’s	context.	The	timezone value	is	taken	from	the record’s	context,	and	the	optional parameter to	use	is	datetime	instead	of	the	current time. 
- fields.Datetime.context_timestamp(record,	timestamp):	That	converts	a naive datetime	(without	timezone)	into	a	timezone	aware	datetime. 

The	timezone	is extracted	from	the	record’s	context,	hence	the name	of	the	function. 

To	facilitate	conversion	between	formats,	both	fields.Date	and fields.Datetime	objects provide	these	functions: 

- from_string(value):	This	converts	a	string	into	a	date	or datetime	object. 
- to_string(value):	This	converts	a	date	or	datetime	object into a	string	in	the	format expected	by	the	server. 
 

**Working	with	relation	fields**

While	using	the	active	record	pattern,	relational	fields	can be assigned	recordsets. 

- For	a	many	to	one	field,	the	value	assigned	must	be	a single	record	(a	singleton	recordset). 
- For	to-many	fields,	their	value	can	also	be	assigned	with	a recordset,	replacing	the	list	of linked	records,	if	any,	with	a new one.	Here	a	recordset	with	any	size	is	allowed. 

While	using	the	create()	or	write()	methods,	where	values	are assigned	using dictionaries,	relational	fields	can’t	be	assigned to recordset	values.	The	corresponding	ID, or	list	of	Ids should be	used. 

For	example,	instead	of	`self.write({'user_id':	self.env.user})`,	we should	rather use	`self.write({'user_id':	self.env.user.id})`. 
 

**Manipulating	recordsets**

We	will	surely	want	to	add,	remove,	or	replace	the elements in	these	related	fields,	and	so this	leads	to	the question: how can	recordsets	be	manipulated? 

Recordsets	are	immutable	but	can	be	used	to	compose	new recordsets.	Some	set operations	are	supported,	which	are	shown	here: 

- rs1	|	rs2:	This	results	in	a	recordset	with	all	element from both	recordsets. 
- rs1	+	rs2:	This	also	concatenates	both	recordsets	into	one. 
- rs1	&	rs2:	This	results	in	a	recordset	with	only	the elements present	in	both recordsets. 
rs1	-	rs2:	This	results	in	a	recordset	with	the	rs1 elements not	present	in	rs2. 

The	slice	notation	can	also	be	used,	as	shown	here: 

- rs[0]	and	rs[-1]	retrieve	the	first	element	and	the	last elements. 
- rs[1:]	results	in	a	copy	of	the	recordset	without	the first	element.	This	yields	the same	records	as	rs	– rs[0] but preserves	their	order. 

In	general,	while	manipulating	recordsets,	you	should	assume that the	record	order	is	not preserved.	However,	addition	and slicing are	known	to	keep	record	order. 

We	can	use	these	recordset	operations	to	change	the	list	by removing	or	adding	elements. You	can	see	this	in	the following	example: 

- self.task_ids	|=	task1:	This	adds	task1	element	if	it’s	not in the	recordset. 
- self.task_ids	-=	task1:	This	removes	the	reference	to	task1 if	it’s	present	in	the recordset. 
- self.task_ids	=	self.task_ids[:-1]:	This	unlinks	the	last record. 

While	using	the	create()	and	write()	methods	with	values	in a dictionary,	a	special syntax	is	used	to	modify	to	many fields. 

This	was	explained	in Chapter	4 ,	*Data Serialization	and	Module Data*,	in	the	section	*Setting	values	for	relation fields*. Refer	to the	following	sample	operations	equivalent	to	the preceding	ones	using	write(): 

- self.write([(4,	task1.id,	False)]):	This	adds	task1	to	the	member. 
- self.write([(3,	task1.id,	False)]):	This	unlinks	task1. 
- self.write([(3,	self.task_ids[-1].id,	False)]):	This	unlinks	the	last element. 
 

**Other	recordset	operations**

Recordsets	support	additional	operations	on	them. 

We	can	check	if	a	record	is	included	or	is	not	in a recordset	by	doing	the	following:  record	in	recordset  record	not in recordset.  These	operations	are	also	available: 

- recordset.ids:	This	returns	the	list	with	the	IDs	of	the recordset	elements. 
- recordset.ensure_one():	This	checks	if	it	is	a	single record	(singleton);	if	it’s	not,	it raises	a	ValueError exception. 
- recordset.exists():	This	returns	a	copy	with	only	the	records that still	exist. 
- recordset.filtered(func):	This	returns	a	filtered	recordset. 
- recordset.mapped(func):	This	returns	a	list	of	mapped values. 
- recordset.sorted(func):	This	returns	an	ordered	recordset. 

Here	are	some	usage	examples	for	these	functions: 
```
>>>	rs0	=	self.env['res.partner'].search([]) >>>	len(rs0)		#	how	many	records? 68 >>>	rs1	=	rs0.filtered(lambda	r:	r.name.startswith('A')) >>>	print	rs1 res.partner(3,	7,	6,	18,	51,	58,	39) >>>	rs2	=	rs1.filtered('is_company') >>>	print	rs2 res.partner(7,	6,	18) >>>	rs2.mapped('name') [u'Agrolait',	u'ASUSTeK',	u'Axelor'] >>>	rs2.mapped(lambda	r:	(r.id,	r.name)) [(7,	u'Agrolait'),	(6,	u'ASUSTeK'),	(18,	u'Axelor')] >>	rs2.sorted(key=lambda	r:	r.id,	reverse=True) res.partner(18,	7,	6)  
 ```
The	execution	environment  The	environment	provides	contextual	information used	by	the	server.	Every	recordset carries	its	execution	environment in	self.env	with	these	attributes: 

- env.cr:	This	is	the	database	cursor	being	used. 
- env.uid:	This	is	the	ID	for	the	session	user. 
- env.user:	This	is	the	record	for	the	session	user. 
- env.context:	This	is	an	immutable	dictionary	with	a	session context. 

The	environment	is	immutable,	and	so	it	can’t	be	modified.	But we can	create	modified environments	and	then	run	actions	using	them. 

These	methods	can	be	used	for	that: 

- env.sudo(user):	If	this	is	provided	with	a	user	record,	it returns	an	environment with	that	user.	If	no	user	is	provided, the	administrator	superuser	will	be	used,	which allows	running specific	queries	bypassing	security	rules. 
- env.with_context(dictionary):	This	replaces	the	context	with	a new one. 
- env.with_context(key=value,...):	This	sets	values	for	keys	in	the current	context. 

The	env.ref()	function	takes	a	string	with	an	External	ID and returns	a	record	for	it,	as shown	here: 
```
>>>	self.env.ref('base.user_root') res.users(1,)  
``` 
 

**Model	methods	for	client	interaction**

We	have	seen	the	most	important	model	methods	used	to	generate recordsets	and	how	to write	on	them.	But	there	are	a	few	more model	methods	available	for	more	specific actions,	as	shown	here: 
read([fields]):	This	is	similar	to	browse,	but	instead	of a recordset,	it	returns	a	list of	rows	of	data	with	the fields given	as	it’s	argument.	Each	row	is	a	dictionary.	It provides	a serialized	representation	of	the	data	that	can	be sent	through RPC protocols	and	is	intended	to	be	used	by client	programs	and not	in	server	logic. 

- `search_read([domain],	[fields],	offset=0,	limit=None,	order=None)`: This performs	a	search	operation	followed	by	a	read	on	the resulting	record	list.	It	is intended	to	be	used	by	RPC clients and	saves	them	the	extra	round	trip	needed	when doing	a	search first and	then	a	read. 
- load([fields],	[data]):	This	is	used	to	import	data	acquired from	a	CSV	file.	The first	argument	is	the	list	of	fields to import,	and	it	maps	directly	to	a	CSV	top	row. The	second argument	is	a	list	of	records,	where	each	record	is	a list of	string	values	to parse	and	import,	and	it	maps directly	to	the	CSV	data	rows	and	columns.	It implements	the features	of	CSV	data	import	described	in	 Chapter	4 ,	*Data Serialization	and	Module	Data*,	like	the	External	Ids support. It	is	used	by	the	web client	Import	feature.	It replaces the deprecated	import_data	method. 
- export_data([fields],	raw_data=False):	This	is	used	by	the	web	client Export function.	It	returns	a	dictionary	with	a	data	key containing	the	data–a	list	of	rows. The	field	names	can	use	the .id	and	/id	suffixes	used	in	CSV	files,	and	the	data	is in a	format	compatible	with	an	importable	CSV	file.	The	optional raw_data	argument allows	for	data	values	to	be	exported	with their	Python	types,	instead	of	the	string representation	used in	CSV. 

The	following	methods	are	mostly	used	by	the	web	client to render	the	user	interface	and perform	basic	interaction: 

- name_get():	This	returns	a	list	of	(ID,	name)	tuples	with the	text	representing	each record.	It	is	used	by	default to compute	the	display_name	value,	providing	the	text representation	of	relation	fields.	It	can	be	extended	to implement	custom	display representations,	such	as	displaying	the record	code	and	name	instead	of	only	the name. 
- name_search(name='',	args=None,	operator='ilike',	limit=100):	This	also returns a	list	of	(ID,	name)	tuples,	where	the	display	name	matches the text	in	the name	argument.	It	is	used	by	the	UI	while	typing in	a	relation	field	to	produce	the	list suggested	records matching	the	typed	text.	It	is	used	to	implement	product	lookup both	by	name	and	by	reference	while	typing	in	a	field	to pick	a	product. 
- name_create(name):	This	creates	a	new	record	with	only	the title	name	to	use	for	it. It	is	used	by	the	UI	for	the quick-create	feature,	where	you	can	quickly	create	a related record	by	just	providing	its	name.	It	can	be	extended	to provide	specific defaults	while	creating	new	records	through	this feature. 
 - default_get([fields]):	This	returns	a	dictionary	with	the default values	for	a	new record	to	be	created.	The	default	values may	depend	on	variables	such	as	the	current user	or	the session	context. 
- fields_get():	This	is	used	to	describe	the	model’s	field definitions,	as	seen	in	the View	Fields 	option	of	the developer	menu. 
- fields_view_get():	This	is	used	by	the	web	client	to retrieve the	structure	of	the	UI view	to	render.	It	can	be	given the	ID	of	the	view	as	an	argument	or	the	type	of	view we	want	using	view_type='form'.	Look	at	an	example	of	this: 
```
rset.fields_view_get(view_type='tree'). 
``` 

**Overriding	the	default	methods**

We	have	learned	about	the	standard	methods	provided	by	the API.	But	what	we	can	do with	them	doesn’t	end	there!	We can also	extend	them	to	add	custom	behavior	to	our models. 

The	most	common	case	is	to	extend	the	create()	and write() methods.	This	can	be	used to	add	the	logic	triggered	whenever these	actions	are	executed.	By	placing	our	logic	in	the appropriate	section	of	the	custom	method,	we	can	have	the code	run	before	or	after	the main	operations	are	executed. 

Using	the	TodoTask	model	as	an	example,	we	can	make	a	custom create(),	which	would look	like	this: 
```
@api.model def	create(self,	vals): 				#	Code	before create 				#	Can	use	the	`vals`	dict new_record	=	super(TodoTask,	self).create(vals) #	Code	after	create 				#	Can	use	the	`new` record	created 				return	new_record 
```
A	custom	write()	would	follow	this	structure: 
```
@api.multi def	write(self,	vals): 				#	Code	before write 				#	Can	use	`self`,	with	the	old values 				super(TodoTask,	self).write(vals) #	Code	after	write 				#	Can	use `self`,	with	the	new	(updated)	values 				return True 
```
These	are	common	extension	examples,	but	of	course	any standard method	available	for	a model	can	be	inherited	in	a similar way	to	add	to	it	our	custom	logic. 

These	techniques	open	up	a	lot	of	possibilities,	but	remember that	other	tools	are	also available	that	are	better	suited	for common	specific	tasks	and	should	be	preferred: 

To	have	a	field	value	calculated	based	on	another,	we	should use computed	fields.	An example	of	this	is	to	calculate	a total when the	values	of	the	lines	are	changed. To	have	field	default values calculated	dynamically,	we	can	use	a	field	default	bound to a function	instead	of	a	scalar	value. To	have	values set on other	fields	when	a	field	is	changed,	we	can	use	on-change functions.	An	example	of	this	is	when	picking	a	customer to set	the	document’s currency	to	the	corresponding	partner’s,	which can	afterwards	be	manually	changed by	the	user.	Keep	in	mind	that on-change	only	works	on	form	view	interaction	and not	on	direct write calls. For	validations,	we	should	use	constraint	functions decorated	with @api.constrains(fld1,fld2,...).	These	are	like	computed fields	but	are	expected to	raise	errors	when	conditions	are	not met	instead	of	computing	values. 
 

**Model	method	decorators**

During	our	journey,	the	several	methods	we	encountered	used API	decorators	like  @api.one.	These	are	important	for	the	server to know	how	to	handle	the	method.	We	have already	given	some explanation	of	the	decorators	used;	now	let’s	recap	the	ones	available and	when	they	should	be	used: 

- @api.one:	This	feeds	one	record	at	a	time	to	the	function. The	decorator	does	the recordset	iteration	for	us	and	self	is guaranteed	to	be	a	singleton.	It’s	the	one	to	use if	our logic	only	needs	to	work	with	each	record.	It	also	aggregates	the return	values of	the	function	on	each	record	in	a	list, which	can	have	unintended	side	effects. 
- @api.multi:	This	handles	a	recordset.	We	should	use	it when	our	logic	can	depend on	the	whole	recordset	and	seeing isolated records	is	not	enough,	or	when	we	need	a return	value	that is	not	a	list	like	a	dictionary	with	a	window	action. In practice	it	is the	one	to	use	most	of	the	time	as @api.one has	some	overhead	and	list	wrapping effects	on	result values. 
- @api.model:	This	is	a	class-level	static	method,	and	it does	not	use	any	recordset data.	For	consistency,	self	is	still a	recordset,	but	its	content	is	irrelevant. 
- @api.returns(model):	This	indicates	that	the	method	return instances of	the	model in	the	argument,	such	as	res.partner	or	self	for the	current	model. 

The	decorators	that	have	more	specific	purposes	that	were	explained in detail	in Chapter 5 , *Models	–	Structuring	Application	Data*	are	shown here:
 
- @api.depends(fld1,...):	This	is	used	for	computed	field	functions to identify	on what	changes	the	(re)calculation	should	be triggered. 
- @api.constrains(fld1,...):	This	is	used	for	validation	functions	to identify	on what	changes	the	validation	check	should	be triggered. 
- @api.onchange(fld1,...):	This	is	used	for	on-change	functions	to identify	the fields	on	the	form	that	will	trigger	the	action. 

In	particular	the	on-change	methods	can	send	a	warning message to the	user	interface.	For example,	this	could	warn	the	user	that the product	quantity	just	entered	is	not	available	on stock, without preventing	the	user	from	continuing.	This	is	done	by	having	the method return	a	dictionary	describing	the	following	warning message:
``` 
								return	{ 												'warning':	{ 																'title':	'Warning!', 																'message':	'The	warning	text'} 								} 
 ```

**Debugging**  

We	all	know	that	a	good	part	of	a	developer’s	work	is	to debug	code.	To	do	this	we	often make	use	of	a	code	editor that can	set	breakpoints	and	run	our	program	step	by	step. Doing	so with	Odoo	is	possible,	but	it	has	it’s	challenges. 

If	you’re	using	Microsoft	Windows	as	your	development	workstation, setting	up	an environment	capable	of	running	Odoo	code	from source	is	a	nontrivial	task.	Also	the	fact that	Odoo	is	a server	that	waits	for	client	calls	and	only	then	acts	on	them makes	it	quite different	to	debug	compared	to	client-side	programs. 

While	this	can	certainly	be	done	with	Odoo,	arguably	it	might	not be the	most	pragmatic approach	to	the	issue.	We	will introduce some	basic	debugging	strategies,	which	can	be	as effective	as	many sophisticated	IDEs	with	some	practice. 

Python’s	integrated	debug	tool	pdb	can	do	a	decent	job	at debugging.	We	can	set	a breakpoint	by	inserting	the	following line	in	the	desired	place: 
```
import	pdb;	pdb.set_trace() 
```
Now	restart	the	server	so	that	the	modified	code	is loaded. As	soon	as	the	program execution	reaches	that	line,	a	(pdb)	Python prompt	will	be	shown	in	the	terminal	window where	the	server is	running,	waiting	for	our	input. 

This	prompt	works	as	a	Python	shell,	where	you	can	run any	expression	or	command	in the	current	execution	context. This	means	that	the	current	variables	can	be	inspected	and even modified.	These	are	the	most	important	shortcut	commands	available: 

- h:	This	is	used	to	display	a	help	summary	of	the	pdb commands. 
- p:	This	is	used	to	to	evaluate	and	print	an	expression. 
- pp:	This	is	for	pretty	print,	which	is	useful	for	larger dictionaries	or	lists. 
- l:	This	lists	the	code	around	the	instruction	to	be	executed next. 
- n	(next):	This	steps	over	to	the	next	instruction. 
- s	(step):	This	steps	into	the	current	instruction. 
- c	(continue):	This	continues	execution	normally. 
- u(up):	This	allows	to	move	up	the	execution	stack. 
- d(down):	This	allows	to	move	down	the	execution	stack. 

The	Odoo	server	also	supports	the	--debug	option.	If	it’s used,	when	the	server	finds	an exception,	it	enters	into	a *post	mortem*	mode	at	the	line	where	the	error	was	raised.	This is a	pdb	console	and	it	allows	us	to	inspect	the program state	at	the	moment	where	the	error was	found. 

It’s	worth	noting	that	there	are	alternatives	to	the	Python built-in	debugger.	There	is	pudb that	supports	the	same	commands as pdb	and	works	in	text-only	terminals,	but	uses	a	more friendly graphical	display,	making	useful	information	readily	available such	as	the variables	in	the	current	context	and	their values. 
 
![275_1](/images/Odoo Development Essentials - Daniel Reis-275_1.jpg)

It	can	be	installed	either	through	the	system	package manager	or	through	pip,	as	shown here: 
```
$	sudo	apt-get	install	python-pudb		#	using	OS	packages $ pip	install	pudb		#	using	pip,	possibly	in	a virtualenv  
```
It	works	just	like	pdb;	you	just	need	to	use	pudb	instead	of pdb in	the	breakpoint	code. 

Another	option	is	the	Iron	Python	debugger	ipdb,	which	can be installed	by	using	the following	code: 
```
$	pip	install	ipdb  
```
Sometimes	we	just	need	to	inspect	the	values	of	some variables	or	check	if	some	code blocks	are	being	executed.	A	Python print	statement	can	perfectly	do	the	job	without stopping	the execution	flow.	As	we	are	running	the	server	in	a terminal window,	the printed	text	will	be	shown	in	the	standard output.	But it won’t	be	stored	to	the	server	log	if it’s	being written to a file. 

Another	option	to	keep	in	mind	is	to	set	debug	level	log messages	at	sensitive	points	of our	code	if	we	feel	that we	might	need	them	to	investigate	issues	in	a	deployed instance. It would	only	be	needed	to	elevate	that	server logging	level to	DEBUG	and	then	inspect	the log	files. 
 

**Summary**  

In	the	previous	chapters,	you	saw	how	to	build	models	and design	views.	Here	you	went	a little	further	learning	how to implement	business	logic	and	use	recordsets	to	manipulate model data. 

You	also	saw	how	the	business	logic	can	interact	with	the	user interface	and	learned	to create	wizards	that	dialogue	with	the user	and	serve	as	a	platform	to	launch	advanced processes. 

In	the	next	chapter,	our	focus	will	go	back	to	the	user interface,	and	you	will	learn	how	to create	powerful	kanban	views and	design	your	own	business	reports. 
