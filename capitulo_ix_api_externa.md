Chapter	9. External API	– Integration with Other Systems
===

Until	now,	we	have	been	working	with	server-side	code.	However,	the	Odoo	server	also provides	an	external	API,	which	is	used	by	its	web	client	and	is	also	available	for	other client	applications. 

In	this	chapter,	we	will	learn	how	to	use	the	Odoo	external	API	from	our	own	client programs.	For	simplicity,	we	will	focus	on	Python-based	clients. 
 

**Setting up a Python client**

The	Odoo	API	can	be	accessed	externally	using	two	different	protocols:	XML-RPC	and JSON-RPC.	Any	external	program	capable	of	implementing	a	client	for	one	of	these protocols	will	be	able	to	interact	with	an	Odoo	server.	To	avoid	introducing	additional programming	languages,	we	will	keep	using	Python	to	explore	the	external	API. 

Until	now,	we	have	been	running	Python	code	only	on	the	server.	This	time,	we	will	use Python	on	the	client	side,	so	it’s	possible	you	might	need	to	do	some	additional	setup	on your	workstation. 

To	follow	the	examples	in	this	chapter,	you	will	need	to	be	able	to	run	Python	files	on	your work	computer.	The	Odoo	server	requires	Python	2,	but	our	RPC	client	can	be	in	any language,	so	Python	3	will	be	just	fine.	However,	since	some	readers	may	be	running	the server	on	the	same	machine	they	are	working	on	(hello	Ubuntu	users!),	it	will	be	simpler for	everyone	to	follow	if	we	stick	to	Python	2. 

If	you	are	using	Ubuntu	or	a	Macintosh,	probably	Python	is	already	installed.	Open	a terminal	console,	type	python,	and	you	should	be	greeted	with	something	like	the following:
``` 
Python	2.7.8	(default,	Oct	20	2014,	15:05:29) [GCC	4.9.1]	on	linux2 Type	"help",	"copyright",",	"credits"	or	"license"	for	more	information. 
>>>  
```
*Note*  
*Windows	users	can	find	an	installer	and	also	quickly	get	up	to	speed.	The	official installation	packages	can	be	found	at	 [https://www.python.org/downloads/](https://www.python.org/downloads/).*  
 
 
**Calling the Odoo API using XML-RPC**

The	simplest	method	to	access	the	server	is	using	XML-RPC.	We	can	use	the	xmlrpclib library	from	Python’s	standard	library	for	this.	Remember	that	we	are	programming	a client	in	order	to	connect	to	a	server,	so	we	need	an	Odoo	server	instance	running	to connect	to.	In	our	examples,	we	will	assume	that	an	Odoo	server	instance	is	running	on the	same	machine	(localhost),	but	you	can	use	any	IP	address	or	server	name,	if	the server	is	running	on	another	machine. 
 

**Opening an XML-RPC connection**

Let’s	get	a	fist	contact	with	the	external	API.	Start	a	Python	console	and	type	the following: 
```
>>> import xmlrpclib 
>>> srv,	db	=	'http://localhost:8069',	'v8dev' >>>	user,	pwd	=	'admin',	'admin' 
>>>	common	=	xmlrpclib.ServerProxy('%s/xmlrpc/2/common'	%	srv) 
>>>	common.version() {'server_version_info':	[8,	0,	0,	'final',	0],	'server_serie':	'8.0',	 'server_version':	'8.0',	'protocol_version':	1} 
```
Here,	we	import	the	xmlrpclib	library	and	then	set	up	some	variables	with	the information	for	the	server	location	and	connection	credentials.	Feel	free	to	adapt	these	to your	specific	setup. 

Next,	we	set	up	access	to	the	server’s	public	services	(not	requiring	a	login),	exposed	at the	/xmlrpc/2/common	endpoint.	One	of	the	methods	that	are	available	is	version(), which	inspects	the	server	version.	We	use	it	to	confirm	that	we	can	communicate	with	the server. 

Another	public	method	is	authenticate().	In	fact,	this	does	not	create	a	session,	as	you might	be	led	to	believe.	This	method	just	confirms	that	the	username	and	password	are accepted	and	returns	the	user	ID	that	should	be	used	in	requests	instead	of	the	username, as	shown	here: 
```
>>>	uid	=	common.authenticate(db,	user,	pwd,	{}) 
>>>	print	uid 1 
```
 
**Reading	data	from	the	server**

With	XML-RPC,	no	session	is	maintained	and	the	authentication	credentials	are	sent	with every	request.	This	adds	some	overhead	to	the	protocol,	but	makes	it	simpler	to	use. 
Next,	we	set	up	access	to	the	server	methods	that	need	a	login	to	be	accessed.	These	are exposed	at	the	/xmlrpc/2/object	endpoint,	as	shown	in	the	following: 
```
>>>	api	=	xmlrpclib.ServerProxy('%s/xmlrpc/2/object'	%	srv) 
>>>	api.execute_kw(db,	uid,	pwd,	'res.partner',	'search_count',		[[]]) 70 
```
Here,	we	are	doing	our	first	access	to	the	server	API,	performing	a	count	on	the	Partner records.	Methods	are	called	using	the	execute_kw()	method	that	takes	the	following arguments: 
The	name	of	the	database	to	connect	to The	connection	user	ID The	user	password The	target	model	identifier	name The	method	to	call A	list	of	positional	arguments An	optional	dictionary	with	keyword	arguments 

The	preceding	example	calls	the	search_count	method	of	the	res.partner	model	with one	positional	argument,	[],	and	no	keyword	arguments.	The	positional	argument	is	a search	domain;	since	we	are	providing	an	empty	list,	it	counts	all	the	Partners. 

Frequent	actions	are	search	and	read.	When	called	from	the	RPC,	the	search	method returns	a	list	of	IDs	matching	a	domain.	The	browse	method	is	not	available	from	the RPC,	and	read	should	be	used	in	its	place	to,	given	a	list	of	record	IDs,	retrieve	their	data, as	shown	in	the	following	code: 
```
>>>	api.execute_kw(db,	uid,	pwd,	'res.partner',	'search',	[[('country_id',	 '=',	'be'),	('parent_id',	'!=',	False)]]) [43,	42] 
>>>	api.execute_kw(db,	uid,	pwd,	'res.partner',	'read',		[[43]],	{'fields':	 ['id',	'name',	'parent_id']}) [{'parent_id':	[7,	'Agrolait'],	'id':	43,	'name':	'Michel	Fletcher'}] 
```
Note	that	for	the	read	method,	we	are	using	one	positional	argument	for	the	list	of	IDs, 
[43],	and	one	keyword	argument,	fields.	We	can	also	notice	that	relational	fields	are retrieved	as	a	pair,	with	the	related	record’s	ID	and	display	name.	That’s	something	to keep	in	mind	when	processing	the	data	in	your	code. 

The	search	and	read	combination	is	so	frequent	that	a	search_read	method	is	provided to	perform	both	operations	in	a	single	step.	The	same	result	as	the	previous	two	steps	can be	obtained	with	the	following: 
```
>>>	api.execute_kw(db,	uid,	pwd,	'res.partner',	'search_read',	 [[('country_id',	'=',	'be'),	('parent_id',	'!=',	False)]],	{'fields':	 ['id',	'name',	'parent_id']}) 
```
The	search_read	method	behaves	like	read,	but	expects	as	first	positional	argument	a domain	instead	of	a	list	of	IDs.	It’s	worth	mentioning	that	the	field	argument	on	read	and 
search_read	is	not	mandatory.	If	not	provided,	all	fields	will	be	retrieved. 
 

**Calling	other	methods**

The	remaining	model	methods	are	all	exposed	through	RPC,	except	for	those	starting	with “_”	that	are	considered	private.	This	means	that	we	can	use	create,	write,	and	unlink	to modify	data	on	the	server	as	follows: 
```
>>>	api.execute_kw(db,	uid,	pwd,	'res.partner',	'create',	[{'name':	 'Packt'}]) 75 >>>	api.execute_kw(db,	uid,	pwd,	'res.partner',	'write',	[[75],	{'name':	 'Packt	Pub'}]) True 
>>>	api.execute_kw(db,	uid,	pwd,	'res.partner',	'read',	[[75],	['id',	 'name']]) [{'id':	75,	'name':	'Packt	Pub'}] >>>	api.execute_kw(db,	uid,	pwd,	'res.partner',	'unlink',	[[75]]) True 
```
One	limitation	of	the	XML-RPC	protocol	is	that	it	does	not	support	None	values.	The implication	is	that	methods	that	don’t	return	anything	won’t	be	usable	through	XML-RPC, since	they	are	implicitly	returning	None.	This	is	why	methods	should	always	finish	with	at least	a	return	True	statement. 
 
![328_1](/images/Odoo Development Essentials - Daniel Reis-328_1.jpg)

Writing	a	Notes	desktop	application  Let’s	do	something	interesting	with	the	RPC	API.	What	if	users	could	manage	their	Odoo to-do	tasks	directly	from	their	computer’s	desktop?	Let’s	write	a	simple	Python application	to	do	just	that,	as	shown	in	the	following	screenshot: 

For	clarity,	we	will	split	it	into	two	files:	one	concerned	to	interact	with	the	server backend,	note_api.py,	and	another	with	the	graphical	user	interface,	note_gui.py. 
 

**Communication	layer	with	Odoo**

We	will	create	a	class	to	set	up	the	connection	and	store	its	information.	It	should	expose two	methods:	get()	to	retrieve	task	data	and	set()	to	create	or	update	tasks. 
Select	a	directory	to	host	the	application	files	and	create	the	note_api.py	file.	We	can start	by	adding	the	class	constructor,	as	follows: 
```
import	xmlrpclib class	NoteAPI(): 				def	__init__(self,	srv,	db,	user,	pwd): 								common	=	xmlrpclib.ServerProxy( 												'%s/xmlrpc/2/common'	%	srv) 								self.api	=	xmlrpclib.ServerProxy( 												'%s/xmlrpc/2/object'	%	srv) 								self.uid	=	common.authenticate(db,	user,	pwd,	{}) 								self.pwd	=	pwd 								self.db	=	db 								self.model	=	'todo.task' 
```
Here	we	store	in	the	created	object	all	the	information	needed	to	execute	calls	on	a	model: the	API	reference,	uid,	password,	database	name,	and	the	model	to	use. 
Next	we	will	define	a	helper	method	to	execute	the	calls.	It	takes	advantage	of	the	object stored	data	to	provide	a	smaller	function	signature,	as	shown	next: 
```
				def	execute(self,	method,	arg_list,	kwarg_dict=None): 								return	self.api.execute_kw( 												self.db,	self.uid,	self.pwd,	self.model, 												method,	arg_list,	kwarg_dict	or	{}) 
```
Now	we	can	use	it	to	implement	the	higher	level	get()	and	set()	methods. 
The	get()	method	will	accept	an	optional	list	of	IDs	to	retrieve.	If	none	are	listed,	all records	will	be	returned,	as	shown	here: 
```
				def	get(self,	ids=None): 								domain	=	[('id','	in',	ids)]	if	ids	else	[] 								fields	=	['id',	'name'] 								return	self.execute('search_read',	[domain,	fields]) 
```
The	set()	method	will	have	as	arguments	the	task	text	to	write,	and	an	optional	ID.	If	ID is	not	provided,	a	new	record	will	be	created.	It	returns	the	ID	of	the	record	written	or created,	as	shown	here: 
```
				def	set(self,	text,	id=None): 								if	id: 												self.execute('write',	[[id],	{'name':	text}]) 								else: 												vals	=	{'name':	text,	'user_id':	self.uid} 												id	=	self.execute('create',	[vals])
``` 								return	id 
Let’s	end	the	file	with	a	small	piece	of	test	code	that	will	be	executed	if	we	run	the	Python file: 
``` 
if	__name__	==	'__main__': 				srv,	db	=	'http://localhost:8069',	'v8dev' 				user,	pwd	=	'admin',	'admin' 				api	=	NoteAPI(srv,	db,	user,	pwd) 				from	pprint	import	pprint 				pprint(api.get()) 
```
If	we	run	the	Python	script,	we	should	see	the	content	of	our	to-do	tasks	printed	out.	Now that	we	have	a	simple	wrapper	around	our	Odoo	backend,	let’s	deal	with	the	desktop	user interface. 
 

**Creating	the	GUI**

Our	goal	here	was	to	learn	to	write	the	interface	between	an	external	application	and	the Odoo	server,	and	this	was	done	in	the	previous	section.	But	it	would	be	a	shame	not	going the	extra	step	and	actually	making	it	available	to	the	end	user. 
To	keep	the	setup	as	simple	as	possible,	we	will	use	Tkinter	to	implement	the	graphical user	interface.	Since	it	is	part	of	the	standard	library,	it	does	not	require	any	additional installation.	It	is	not	our	goal	to	explain	how	Tkinter	works,	so	we	will	be	short	on explanations	about	it. 

Each	Task	should	have	a	small	yellow	window	on	the	desktop.	These	windows	will	have	a single	Text	widget.	Pressing	*Ctrl*	+	*N*	will	open	a	new	Note,	and	pressing	*Ctrl*	+	*S*	will write	the	content	of	the	current	note	to	the	Odoo	server. 

Now,	alongside	the	note_api.py	file,	create	a	new	note_gui.py	file.	It	will	first	import the	Tkinter	modules	and	widgets	we	will	use,	and	then	the	NoteAPI	class,	as	shown	in	the following: 
from	Tkinter	import	Text,	Tk import	tkMessageBox from	note_api	import	NoteAPI 

Next	we	create	our	own	Text	widget	derived	from	the	Tkinter	one.	When	creating	an instance,	it	will	expect	an	API	reference,	to	use	for	the	save	action,	and	also	the	Task’s	text and	ID,	as	shown	in	the	following: 
```
class	NoteText(Text): 				def	__init__(self,	api,	text='',	id=None): 								self.master	=	Tk() 								self.id	=	id 								self.api	=	api 								Text.__init__(self,	self.master,	bg='#f9f3a9', 																						wrap='word',	undo=True) 								self.bind('<Control-n>',	self.create) 								self.bind('<Control-s>',	self.save) 								if	id: 												self.master.title('#%d'	%	id) 								self.delete('1.0',	'end') 								self.insert('1.0',	text) 								self.master.geometry('220x235') 								self.pack(fill='both',	expand=1) 
```
The	Tk()	constructor	creates	a	new	UI	window	and	the	Text	widget	places	itself	inside	it, so	that	creating	a	new	NoteText	instance	automatically	opens	a	desktop	window. 
Next,	we	will	implement	the	create	and	save	actions.	The	create	action	opens	a	new empty	window,	but	it	will	be	stored	in	the	server	only	when	a	save	action	is	performed,	as shown	in	the	following	code: 
```
				def	create(self,	event=None): 								NoteText(self.api,	'') 				def	save(self,	event=None): 
 
 								text	=	self.get('1.0',	'end') 								self.id	=	self.api.set(text,	self.id) 								tkMessageBox.showinfo('Info',	'Note	%d	Saved.'	%	self.id) 
```
The	save	action	can	be	performed	either	on	existing	or	on	new	tasks,	but	there	is	no	need to	worry	about	that	here	since	those	cases	are	already	handled	by	the	set()	method	of 
NoteAPI. 

Finally,	we	will	add	the	code	that	retrieves	and	creates	all	note	windows	when	the	program is	started,	as	shown	in	the	following	code: 
```
if	__name__	==	'__main__': 				srv,	db	=	'http://localhost:8069',	'v8dev' 				user,	pwd	=	'admin',	'admin' 				api	=	NoteAPI(srv,	db,	user,	pwd) 				for	note	in	api.get(): 								x	=	NoteText(api,	note['name'],	note['id']) 				x.master.mainloop() 
```
The	last	command	runs	mainloop()	on	the	last	Note	window	created,	to	start	waiting	for window	events. 

This	is	a	very	basic	application,	but	the	point	here	is	to	make	an	example	of	interesting ways	to	leverage	the	Odoo	RPC	API. 
 

**Introducing	the	ERPpeek	client**

ERPpeek	is	a	versatile	tool	that	can	be	used	both	as	an	interactive	Command-line Interface 	(CLI )	and	as	a	Python	library ,	with	a	more	convenient	API	than	the	one provided	by	xmlrpclib.	It	is	available	from	the	PyPi	index	and	can	be	installed	with	the following: 
```
$	pip	install	-U	erppeek  
```
On	a	Unix	system,	if	you	are	installing	it	system	wide,	you	might	need	to	prepend	sudo	to the	command. 
 

**The	ERPpeek	API**

The	erppeek	library	provides	a	programming	interface,	wrapping	around	xmlrpclib, which	is	similar	to	the	programming	interface	we	have	for	the	server-side	code. 
Our	point	here	is	to	provide	a	glimpse	of	what	ERPpeek	has	to	offer,	and	not	to	provide	a full	explanation	of	all	its	features. 

We	can	start	by	reproducing	our	first	steps	with	xmlrpclib	using	erppeek	as	follows: 
```
>>>	import	erppeek 
>>>	api	=	erppeek.Client('http://localhost:8069',	'v8dev',	'admin',	 'admin') 
>>>	api.common.version() >>>	api.count('res.partner',	[]) >>>	api.search('res.partner',	[('country_id',	'=',	'be'),	('parent_id',	 '!=',	False)]) >>>	api.read('res.partner',	[43],	['id',	'name',	'parent_id']) 
```
As	you	can	see,	the	API	calls	use	fewer	arguments	and	are	similar	to	the	server-side counterparts. 

But	ERPpeek	doesn’t	stop	here,	and	also	provides	a	representation	for	Models.	We	have	the following	two	alternative	ways	to	get	an	instance	for	a	model,	either	using	the	model	() method	or	accessing	an	attribute	in	camel	case: 
```
>>>	m	=	api.model('res.partner') 
>>>	m	=	api.ResPartner 
```
Now	we	can	perform	actions	on	that	model	as	follows: 
```
>>>	m.count([('name',	'like',	'Packt%')]) 1 
>>>	m.search([('name',	'like',	'Packt%')]) [76] 
```
It	also	provides	client-side	object	representation	for	records	as	follows: 
```
>>>	recs	=	m.browse([('name',	'like',	'Packt%')]) 
>>>	recs <RecordList	'res.partner,[76]'> 
>>>	recs.name ['Packt'] 
```
As	you	can	see,	ERPpeek	goes	a	long	way	from	plain	xmlrpclib,	and	makes	it	possible	to write	code	that	can	be	reused	server	side	with	little	or	no	modification. 
 

**The	ERPpeek	CLI**

Not	only	can	erppeek	be	used	as	a	Python	library,	it	is	also	a	CLI	that	can	be	used	to perform	administrative	actions	on	the	server.	Where	the	odoo	shell	command	provided	a local	interactive	session	on	the	host	server,	erppeek	provides	a	remote	interactive	session on	a	client	across	the	network. 

Opening	a	command	line,	we	can	have	a	peek	at	the	options	available,	as	shown	in	the following: 
```
$	erppeek	--help  
```
Let’s	see	a	sample	session	as	follows: 
```
$	erppeek	--server='http://localhost:8069'	-d	v8dev	-u	admin Usage	(some	commands): 				models(name)																				#	List	models	matching	pattern 				model(name)																					#	Return	a	Model	instance (...) Password	for	'admin': Logged	in	as	'admin' v8dev	
>>>	model('res.users').count() 3 v8dev	
>>>	rec	=	model('res.partner').browse(43) v8dev	
>>>	rec.name 'Michel	Fletcher'  
```
As	you	can	see,	a	connection	was	made	to	the	server,	and	the	execution	context	provided	a reference	to	the	model()	method	to	get	model	instances	and	perform	actions	on	them. 

The	erppeek.Client	instance	used	for	the	connection	is	also	available	through	the	client variable.	Notably,	it	provides	an	alternative	to	the	web	client	to	manage	the	following modules	installed: 

- client.modules():	This	can	search	and	list	modules	available	or	installed 
- client.install():	This	performs	module	installation 
- client.upgrade():	This	orders	modules	to	be	upgraded 
- client.uninstall():	This	uninstalls	modules 

So,	ERPpeek	can	also	provide	good	service	as	a	remote	administration	tool	for	Odoo servers. 
 

** Summary**

Our	goal	for	this	chapter	was	to	learn	how	the	external	API	works	and	what	it	is	capable of.	We	started	exploring	it	using	a	simple	Python	XML-RPC	client,	but	the	external	API can	be	used	from	any	programming	language.	In	fact,	the	official	docs	provide	code examples	for	Java,	PHP,	and	Ruby. 

There	are	a	number	of	libraries	to	handle	XML-RPC	or	JSON-RPC,	some	generic	and some	specific	for	use	with	Odoo.	We	tried	not	point	out	any	libraries	in	particular,	except for	erppeek,	since	it	is	not	only	a	proven	wrapper	for	the	Odoo/OpenERP	XML-RPC	but because	it	is	also	an	invaluable	tool	for	remote	server	management	and	inspection. 

Until	now,	we	used	our	Odoo	server	instances	for	development	and	tests.	But	to	have	a production	grade	server,	there	are	additional	security	and	optimization	configurations	that need	to	be	done.	In	the	next	chapter,	we	will	focus	on	them. 