Chapter	10. Deployment Checklist – Going Live
===

In	this	chapter,	you	will	learn	how	to	prepare	your	Odoo	server	for	use	in	the	production environment. 

There	are	many	possible	strategies	and	tools	that	can	be	used	to	deploy	and	manage	an Odoo	production	server.	We	will	guide	you	through	one	way	of	doing	it. 

This	is	the	server	setup	checklist	that	we	will	follow: 

- Install	Odoo	from	the	source 
- Set	up	the	Odoo	configuration	file 
- Set	up	multiprocessing	workers 
- Set	up	the	Odoo	system	service 
- Set	up	a	reverse	proxy	with	SSL	support 

Let’s	get	started. 
 

**Installing	Odoo**

Odoo	has	Debian/Ubuntu	packages	available	for	installation.	With	these,	you	get	a working	server	process	that	automatically	starts	on	system	boot.	This	installation	process is	straightforward,	and	you	can	find	all	you	need	at [http://nightly.odoo.com](http://nightly.odoo.com).  

While	this	is	an	easy	and	convenient	way	to	install	Odoo,	here	we	prefer	running	from version-controlled	source	code	since	this	provides	better	control	over	what	is	deployed. 
 

**Installing	from	the	source	code**

Sooner	or	later,	your	server	will	need	upgrades	and	patches.	A	version	controlled repository	can	be	of	great	help	when	the	time	comes. 

We	use	git	to	get	our	code	from	a	repository,	just	like	we	did	to	install	the	development environment.	For	example: 
```
$	git	clone	https://github.com/odoo/odoo.git	-b	8.0	--depth=1  
```
This	command	gets	from	GitHub	the	branch	8.0	source	code	into	an	odoo/	subdirectory. At	the	time	of	writing,	8.0	is	the	default	branch,	so	the	-b	8.0	option	is	optional.	The	-- 
depth=1	option	was	used	to	get	a	shallow	copy	of	the	repository,	without	all	version history.	This	reduces	the	disk	space	used	and	makes	the	clone	operation	much	faster. 

It	might	be	worthwhile	to	have	a	slightly	more	sophisticated	setup,	with	a	staging environment	alongside	the	production	environment. 

With	this,	we	could	fetch	the	latest	source	code	version	and	test	it	in	the	staging environment,	without	disturbing	the	production	environment.	When	we’re	happy	with	the new	version,	we	would	deploy	it	from	staging	to	production. 

Let’s	consider	the	repository	at	~/odoo-dev/odoo	to	be	our	staging	environment.	It	was cloned	from	GitHub,	so	that	a	git	pull	inside	it	will	fetch	and	merge	the	latest	changes. But	it	is	also	a	repository	itself,	and	we	can	clone	it	for	our	production	environment,	as shown	in	the	following	example: 
```
$	mkdir	~/odoo-prd	&&	cd	~/odoo-prd 
$	git	clone	~/odoo-dev/odoo	~/odoo-prd/odoo/  
```
This	will	create	the	production	repository	at	~/odoo-prd/odoo	cloned	from	the	staging 
~/odoo-dev/odoo.	It	will	be	set	to	track	that	repository,	so	that	a	git	pull	inside	production will	fetch	and	merge	the	last	versions	from	staging.	Git	is	smart	enough	to	know	that	this is	a	local	clone	and	uses	hard	links	to	the	parent	repository	to	save	disk	space,	so	the	`--depth`	option	is	unnecessary. 

Whenever	a	problem	found	in	production	needs	troubleshooting,	we	can	checkout	in	the staging	environment	the	version	of	the	production	code,	and	then	debug	to	diagnose	and solve	the	issue,	without	touching	the	production	code.	Later,	the	solution	patch	can	be committed	to	the	staging	Git	history,	and	then	deployed	to	the	production	repository	using a	git	pull	command	on	it. 

*Note*  
*Git	will	surely	be	an	invaluable	tool	to	manage	the	versions	of	your	Odoo	deployments. We	just	scratched	the	surface	of	what	can	be	done	to	manage	code	versions.	If	you’re	not already	familiar	with	Git,	it’s	worth	learning	more	about	it.	A	good	starting	point	is  [http://git-scm.com/doc](http://git-scm.com/doc).* 
 

**Setting	up	the	configuration	file**

Adding	the	--save	option	when	starting	an	Odoo	server	saves	the	configuration	used	to the	~/.openerp_serverrc	file.	We	can	use	the	file	as	a	starting	point	for	our	server configuration,	which	will	be	stored	on	/etc/odoo,	as	shown	in	the	following	code: 
```
$	sudo	mkdir	/etc/odoo $	sudo	chown	$(whoami)	/etc/odoo $	cp	~/.openerp_serverrc	/etc/odoo/openerp-server.conf  
```
This	will	have	the	configuration	parameters	to	be	used	by	our	server	instance. 
The	following	are	the	parameters	essential	for	the	server	to	work	correctly: 

- addons_path:	This	is	a	comma-separated	list	of	the	paths	where	modules	will	be looked	up,	using	the	directories	from	left	to	right.	This	means	that	the	leftmost directories	in	the	list	have	a	higher	priority. 
- xmlrpc_port:	This	is	the	port	number	at	which	the	server	will	listen.	By	default,	port 8069	is	used. 
- log_level:	This	is	the	log	verbosity.	The	default	is	the	info	level,	but	using	the 
debug_rpc	level,	while	more	verbose,	adds	important	information	to	monitor	server performance. 

The	following	settings	are	also	important	for	a	production	instance: 

- admin_passwd:	This	is	the	master	password	to	access	the	web	client	database management	functions. It’s	critical	to	set	this	with	a	strong	password	or	an	empty value	to	deactivate	the	function. 
- dbfilter:	This	is	a	Python-interpreted	regex	expression	to	filter	the	databases	to	be listed.	For	the	user	to	not	be	prompted	to	select	a	database,	it	should	be	set	with 
`^dbname$`,	for	example,	`dbfilter	=	^v8dev$`. 
- logrotate=True:	This	will	split	the	log	into	daily	files	and	keep	only	one	month	of log	history. 
- data_dir:	This	is	the	path	where	the	attachment	files	are	stored.	Remember	to	have backups	on	it. 
- without_demo=True:	This	should	be	set	in	production	environments	so	that	new databases	do	not	have	demo	data	on	them. 

When	using	a	reverse	proxy,	the	following	settings	should	be	considered: 

- proxy_mode=True:	This	is	important	to	set	when	using	a	reverse	proxy.
- xmlrpc-interface:	This	sets	the	addresses	that	will	be	listened	to.	By	default,	it listens	to	all	0.0.0.0,	but	when	using	a	reverse	proxy,	it	can	be	set	to	127.0.0.1	in order	to	respond	only	to	local	requests. 

A	production	instance	is	expected	to	handle	significant	workload.	By	default,	the	server runs	one	process	and	is	capable	of	handling	only	one	request	at	a	time.	However,	a multiprocess	mode	is	available	so	that	concurrent	requests	can	be	handled.	The	option 
workers=N	sets	the	number	of	worker	processes	to	use.	As	a	guideline,	you	can	try	setting it	to	1+2*P,	where	P	is	the	number	of	processors.	The	best	setting	to	use	needs	to	be	tuned for	each	case,	since	it	depends	on	the	server	load	and	what	other	services	are	running	on the	server	(such	as	PostgreSQL). 

We	can	check	the	effect	of	the	settings	made	by	running	the	server	with	the	-c	or	-- 
config	option	as	follows: 
```
$	./odoo.py	-c	/etc/odoo/openerp-server.conf  
```

**Setting	up	as	a	system	service**

Next,	we	will	want	to	set	up	Odoo	as	a	system	service	and	have	it	started	automatically when	the	system	boots. 

The	Odoo	source	code	includes	an	init	script,	used	for	the	Debian	packaged	distribution. We	can	use	it	as	our	service	init	script	with	minor	modifications	as	follows: 
```
$	sudo	cp	~/odoo-prd/odoo/debian/init	/etc/init.d/odoo 
$	sudo	chmod	+x	/etc/init.d/odoo  
```
At	this	point,	you	might	want	to	check	the	content	of	the	init	script.	The	key	parameters are	assigned	to	variables	at	the	top	of	the	file.	A	sample	is	as	follows: 
```
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin DAEMON=/usr/bin/openerp-server NAME=odoo DESC=odoo CONFIG=/etc/odoo/openerp-server.conf LOGFILE=/var/log/odoo/odoo-server.log PIDFILE=/var/run/${NAME}.pid USER=odoo 
```
The	USER	variable	is	the	system	user	under	which	the	server	will	run,	and	you	probably want	to	change	it.	The	other	variables	should	be	adequate	and	we	will	prepare	the	rest	of the	setup	having	their	default	values	in	mind.	DAEMON	is	the	path	to	the	server	executable, 
CONFIG	is	the	configuration	file	to	use,	and	LOGFILE	is	the	log	file	location. 

The	DAEMON	executable	can	be	a	symbolic	link	to	our	actual	Odoo	location,	as	shown	in	the following: 
```
$	sudo	ln	-s	~/odoo-prd/odoo/odoo.py	/usr/bin/openerp-server $	sudo	chown	$(whoami)	/usr/bin/openerp-server  
```
Next	we	must	create	the	LOGFILE	directory	as	follows: 
```
$	sudo	mkdir	/var/log/odoo $	sudo	chown	$(whoami)	/etc/odoo  
```
Now	we	should	be	able	to	start	and	stop	our	Odoo	service	as	follows: 
```
$	sudo	/etc/init.d/odoo	start Starting	odoo:	ok  
```
We	should	now	be	able	to	get	a	response	from	the	server	and	see	no	errors	in	the	log	file, as	shown	in	the	following: 
```
$	curl	http://localhost:8069 <html><head><script>window.location	=	'/web'	+	location.hash;</script> </head></html> $	less	/var/log/odoo/odoo-server.log		#	show	the	log	file  
```
Stopping	the	service	is	done	in	a	similar	way,	as	shown	in	the	following: 
```
$	sudo	/etc/init.d/odoo	stop Stopping	odoo:	ok  
```

*Tip*  
*Ubuntu	provides	the	easier	to	remember	service	command	to	manage	services.	If	you prefer,	you	can	instead	use	sudo	service	odoo	start	and	sudo	service	odoo	stop.*

We	now	only	need	to	make	this	service	start	automatically	on	system	boot: 
```
$	sudo	update-rc.d	odoo	defaults  
```
After	this,	when	we	reboot	our	server,	the	Odoo	service	should	be	started	automatically and	with	no	errors.	It’s	a	good	time	to	check	that	all	is	working	as	expected. 
 

**Using	a	reverse	proxy**

While	Odoo	itself	can	serve	web	pages,	it	is	strongly	recommended	to	have	a	reverse proxy	in	front	of	it.	A	reverse	proxy	acts	as	an	intermediary	managing	the	traffic	between the	clients	sending	requests	and	the	Odoo	servers	responding	to	them.	Using	a	reverse proxy	has	several	benefits. 

On	the	security	side,	it	can	do	the	following: 

Handle	(and	enforce)	HTTPS	protocols	to	encrypt	traffic Hide	the	internal	network	characteristics Act	an	“application	firewall”	limiting	the	URLs	accepted	for	processing 

And	on	the	performance	side,	it	can	provide	significant	improvements: 

Cache	static	content,	thus	reducing	the	load	on	the	Odoo	servers Compress	content	to	speed	up	loading	times Act	as	a	load	balancer	distributing	load	between	several	servers 

Apache	is	a	popular	option	to	use	as	reverse	proxy.	Nginx	is	a	recent	alternative	with	good technical	arguments.	Here	we	will	choose	to	use	nginx	as	a	reverse	proxy	and	show	how	it can	be	used	perform	the	functions	mentioned	above. 


**Setting	up	nginx	for	reverse	proxy**

First,	we	should	install	nginx.	We	want	it	to	listen	on	the	default	HTTP	ports,	so	we	should make	sure	they	are	not	already	taken	by	some	other	service.	Performing	this	command should	result	in	an	error,	as	shown	in	the	following: 
```
$	curl	http://localhost curl:	(7)	Failed	to	connect	to	localhost	port	80  
```
If	not,	you	should	disable	or	remove	that	service	to	allow	nginx	to	use	those	ports.	For example,	to	stop	an	existing	Apache	server	you	should: 
```
$	sudo	/etc/init.d/apache2	stop  
```
Now	we	can	install	nginx,	which	is	done	in	the	expected	way: 
```
$	sudo	apt-get	install	nginx  
```
To	confirm	that	it	is	working	correctly,	we	should	see	a	“Welcome	to	nginx”	page	when visiting	the	server	address	with	a	browser	or	using	curl	http://localhost	in	the	server. 
Nginx	configuration	files	follow	the	same	approach	as	Apache:	they	are	stored	in /etc/nginx/available-sites/	and	activated	by	adding	a	symbolic	link	in  /etc/nginx/enabled-sites/.	We	should	also	disable	the	default	configuration	provided by	the	nginx	installation,	as	shown	in	the	following: 
```
$	sudo	rm	/etc/nginx/sites-enabled/default 
$	sudo	touch	/etc/nginx/sites-available/odoo 
$	sudo	ln	-s	/etc/nginx/sites-available/odoo	/etc/nginx/sites-enabled/odoo  
```
Using	an	editor,	such	as	nano	or	vi,	we	should	edit	our	nginx	configuration	file	as	follows: 
```
$	sudo	nano	/etc/nginx/sites-available/odoo  
```
First	we	add	the	upstreams,	the	backend	servers	nginx	will	redirect	traffic	to,	the	Odoo server	in	our	case,	which	is	listening	on	port	8069,	shown	in	the	following: 
```
upstream	backend-odoo	{ 		server	127.0.0.1:8069; } server	{ 		location	/	{ 				proxy_pass	http://backend-odoo; 		} } 
```
To	test	if	the	edited	configuration	is	correct,	use	the	following: 
```
$	sudo	nginx	-t  
```
In	case	you	find	errors,	confirm	the	configuration	file	is	correctly	typed.	Also,	a	common problem	is	for	the	default	HTTP	to	be	taken	by	another	service,	such	as	Apache	or	the default	nginx	website.	Double-check	the	instructions	given	before	to	make	sure	that	this	is not	the	case,	then	restart	nginx.	After	this,	we	can	have	nginx	to	reload	the	new configuration	as	follows: 
``` 
$	sudo	/etc/init.d/nginx	reload  
```
We	can	now	confirm	that	nginx	is	redirecting	traffic	to	the	backend	Odoo	server,	as	shown in	the	following: 
```
$	curl	http://localhost <html><head><script>window.location	=	'/web'	+	location.hash;</script> </head></html>  
``` 


**Enforcing	HTTPS**

Next	we	should	install	a	certificate	to	be	able	to	use	SSL.	To	create	a	self-signed certificate,	follow	the	following	steps: 
```
$	sudo	mkdir	/etc/nginx/ssl	&&	cd	/etc/nginx/ssl 
$	sudo	openssl	req	-x509	-newkey	rsa:2048	-keyout	key.pem	-out	cert.pem	- days	365	-nodes 
$	sudo	chmod	a-wx	*												#	make	files	read	only 
$	sudo	chown	www-data:root	*			#	access	only	to	www-data	group  
```
This	creates	an	ssl/	directory	inside	the	/etc/nginx/	directory	and	creates	a	password less	self-signed	SSL	certificate.	When	running	the	openssl	command,	some	additional information	will	be	asked,	and	a	certificate	and	key	files	are	generated.	Finally,	the ownership	of	these	files	is	given	to	the	user	www-data	used	to	run	the	web	server. 

*Note*  
*Using	self-signed	certificated	can	pose	some	security	risks,	such	as	man-in-the-middle attacks,	and	may	even	not	be	allowed	by	some	browsers.	For	a	robust	solution,	you	should use	a	certificate	signed	by	a	recognized	certificate	authority.	This	is	particularly	important if	you	are	running	a	commercial	or	e-commerce	website.*

Now	that	we	have	an	SSL	certificate,	we	are	ready	to	configure	nginx	to	use	it. 
To	enforce	HTTPS,	we	will	redirect	all	HTTP	traffic	to	it.	Replace	the	server	directive	we defined	previously	with	the	following: 
```
server	{ 		listen	80; 		add_header	Strict-Transport-Security	max-age=2592000; 		rewrite	^/.*$	https://$host$request_uri?	permanent; } 
```
If	we	reload	the	nginx	configuration	now	and	access	the	server	with	a	web	browser,	we will	see	that	the	http://	address	will	be	converted	into	an	https://	address. 
But	it	won’t	return	any	content	before	we	configure	the	HTTPS	service	properly,	by adding	the	following	server	configuration: 
```
server	{ 		listen	443	default; 		#	ssl	settings 		ssl	on; 		ssl_certificate					/etc/nginx/ssl/cert.pem; 		ssl_certificate_key	/etc/nginx/ssl/key.pem; 		keepalive_timeout	60; 		#	proxy	header	and	settings 		proxy_set_header	Host	$host; 		proxy_set_header	X-Real-IP	$remote_addr; 		proxy_set_header	X-Forward-For	$proxy_add_x_forwarded_for; 		proxy_set_header	X-Forwarded-Proto	$scheme; 		proxy_redirect	off; 	 
 
 		location	/	{ 				proxy_pass	http://backend-odoo; 		} } 
```
This	will	listen	to	the	HTTPS	port	and	use	the	/etc/nginx/ssl/	certificate	files	to	encrypt the	traffic.	We	also	add	some	information	to	the	request	header	to	let	the	Odoo	backend service	know	it’s	being	proxied.	For	security	reasons,	it’s	important	for	Odoo	to	make	sure the	proxy_mode	parameter	is	set	to	True.	At	the	end,	the	location	directive	defines	that	all request	are	passed	to	the	backend-odoo	upstream. 

Reload	the	configuration,	and	we	should	have	our	Odoo	service	working	through	HTTPS, as	shown	in	the	following: 
```
$	sudo	nginx	-t nginx:	the	configuration	file	/etc/nginx/nginx.conf	syntax	is	ok nginx:	configuration	file	/etc/nginx/nginx.conf	test	is	successful 
$	sudo	service	nginx	reload	 	*	Reloading	nginx	configuration	nginx 			...done. 
$	curl	-k	https://localhost	 <html><head><script>window.location	=	'/web'	+	location.hash;</script> </head></html>  
```
The	last	output	confirms	that	the	Odoo	web	client	is	being	served	over	HTTPS. 
 

**Nginx	optimizations**

Now,	it	is	time	for	some	fine-tuning	of	the	nginx	settings.	They	are	recommended	to enable	response	buffering	and	data	compression	that	should	improve	the	speed	of	the website.	We	also	set	a	specific	location	for	the	logs. 

The	following	configurations	should	be	added	inside	the	server	listening	on	port	443,	for example,	just	after	the	proxy	definitions: 
```
#	odoo	log	files access_log	/var/log/nginx/odoo-access.log; error_log		/var/log/nginx/odoo-error.log; 
#	increase	proxy	buffer	size proxy_buffers	16	64k; proxy_buffer_size	128k; 
#	force	timeouts	if	the	backend	dies proxy_next_upstream	error	timeout	invalid_header	http_500	http_502	 http_503; 
#	enable	data	compression gzip	on; gzip_min_length	1100; gzip_buffers	4	32k; gzip_types	text/plain	application/x-javascript	text/xml	text/css; gzip_vary	on; 
```
We	can	also	activate	static	content	caching	for	faster	responses	to	the	types	of	requests mentioned	in	the	preceding	code	example	and	to	avoid	their	load	on	the	Odoo	server. After	the	location	/	section,	add	the	following	second	location	section: 
```
location	~*	/web/static/	{ 		#	cache	static	data 		proxy_cache_valid	200	60m; 		proxy_buffering	on; 		expires	864000; 		proxy_pass	http://backend-odoo; } 
```
With	this,	the	static	data	is	cached	for	60	minutes.	Further	requests	on	those	requests	in that	interval	will	be	responded	to	directly	by	nginx	from	the	cache. 
 

**Long	polling**

Long	polling	is	used	to	support	the	instant	messaging	app,	and	when	using multiprocessing	workers,	it	is	handled	on	a	separate	port,	which	is	8072	by	default. 

For	our	reverse	proxy,	this	means	that	the	longpolling	requests	should	be	passed	to	this port.	To	support	this,	we	need	to	add	a	new	upstream	to	our	nginx	configuration,	as	shown in	the	following	code: 
```
upstream	backend-odoo-im	{	server	127.0.0.1:8072;	} 
```
Next,	we	should	add	another	location	to	the	server	handling	the	HTTPS	requests,	as shown	in	the	following	code: 
```
		location	/longpolling	{	proxy_pass	http://backend-odoo-im;} 
```
With	these	settings,	nginx	should	pass	these	requests	to	the	proper	Odoo	server	port. 
 

**Server	and	module	updates**

Once	the	Odoo	server	is	ready	and	running,	there	will	come	a	time	when	you	need	to install	updates	on	Odoo.	This	involves	two	steps:	first,	to	get	the	new	versions	of	the source	code	(server	or	modules),	and	second,	to	install	them. 

If	you	have	followed	the	approach	described	in	the	*Installing	from	the	source	code*	section, we	can	fetch	and	test	the	new	versions	in	the	staging	repository.	It	is	strongly	advised	that you	make	a	copy	of	the	production	database	and	test	the	upgrade	on	it.	If	v8dev	were	our production	database,	this	could	be	done	with	the	following	commands: 
```
$	dropdb	v8test	;	createdb	v8test 
$	pg_dump	v8dev	|	psqlpsql	-d	v8test 
$	cd	~/odoo-dev/odoo/ 
$	./odoo.py	-d	v8test	--xmlrpc-port=8080	-c	/etc/odoo/openerp-server.conf	- u	all  
```
If	everything	goes	OK,	it	should	be	safe	to	perform	the	upgrade	on	the	production	service. Remember	to	make	a	note	of	the	current	version	Git	reference,	in	order	to	be	able	to	roll back	by	checking	out	this	version	again.	Keeping	a	backup	of	the	database	before performing	the	upgrade	is	also	highly	advised. 

After	this,	we	can	pull	the	new	versions	to	the	production	repository	using	Git	and complete	the	upgrade,	as	shown	here: 
```
$	cd	~/odoo-prd/odoo/ 
$	git	pull 
$	./odoo.py	-c	/etc/odoo/openerp-server.conf	--stop-after-init	-d	v8dev	-u	 all 
$	sudo	/etc/init.d/odoo	restart  
``` 

**Summary**

In	this	chapter,	you	learned	about	the	additional	steps	to	set	up	and	run	Odoo	in	a	Debian- based	production	server.	The	most	important	settings	in	the	configuration	file	were	visited, and	you	learned	how	to	take	advantage	of	the	multiprocessing	mode. 

For	improved	security	and	scalability,	you	also	learned	how	to	use	nginx	as	a	reverse proxy	in	front	of	our	Odoo	server	processes. 

We	hope	this	covers	the	essentials	of	what	is	needed	to	run	an	Odoo	server	and	provide	a stable	and	secure	service	to	your	users. 
