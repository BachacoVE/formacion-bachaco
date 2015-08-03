Capítulo 3. Herencia - Extendiendo la Funcionalidad de las Aplicaciones Existentes
===

Una de las características más poderosas de Odoo es la capacidad para agregar características son modificar directamente los objetos subyacentes. Esto se logra a través de mecanismos de herencia, que funcionan como capas para la modificación por encima de los objetos existentes. 

Estas modificaciones puede suceder en todos los niveles: modelos, vistas, y lógica de negocio. En vez de modificar directamente un módulo existente, creamos un módulo nuevo para agregar las modificaciones previstas.

Aquí, aprenderá como escribir sus propios módulos de extensión, confiriéndole facultades para aprovechar las aplicaciones base o comunitarias. Como ejemplo, aprenderá como agregar las características de mensajería y redes sociales de Odoo a sus propios módulos.
 
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

Para aplicar un modelo usamos una clase Python con un atributo `__inherit`. Este identifica el modelo que será ampliado. La clase nueva hereda todas las características del modelo padre, y solo necesitamos declarar las modificaciones que queremos introducir.

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

Las siguientes dos líneas son declaraciones de campos comunes. El `user_id` representa un usuario desde el modelo Users, `res.users`. Es un campo de “Many2one” equivalente a una clave foránea en el argot de base de datos. El `date_deadline` es un simple campo de fecha. En el Capítulo 5, explicaremos con mas detalle los tipos de  campos disponibles en Odoo.

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

Es mejor evitar cambiar la función distintiva del método (esto es, mantener los mismos argumentos) para asegurarnos que las llamadas a el sigan funcionando adecuadamente. En caso que necesite agregar parámetros adicionales, hágalos opcionales (con un valor predeterminado).

La acción original de Borrar Todas las tareas Finalizadas no es apropiada para nuestro módulos de tareas compartidas, ya que borra todas las tareas sin importar a quien le pertenecen. Necesitamos modificarla para que borre solo las tareas del usuario actual.

Para esto, sobrescribiremos el método original con una nueva versión que primero encuentre las tareas completadas del usuario actual, y luego las desactive:
```
@api.multi
def do_clear_done(self):
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

Lo que acabamos de hacer fue sobrescribir completamente el método padre, reemplazándolo con una implementación nueva. 

Pero esto no es lo que usualmente querremos hacer. En vez de esto, ampliaremos la lógica actual y agregaremos operaciones adicionales. De lo contrario podemos dañar operaciones existentes. La lógica existente es insertada dentro de un método sobrescrito usando el comando `super()` de Python para llamar a la versión padre del método.

Veamos un ejemplo de esto: podemos escribir una versión mejor de `do_toggle_done()` que solo ejecute la acción sobre las Tareas asignadas a nuestro usuario:
```
@api.one
def do_toggle_done(self):
    if self.user_id != self.env.user:
        raise Exception('Only	the responsible can do this!')
    else:
        return super(TodoTask, self).do_toggle_done()
```
Estas son las técnicas básicas para sobrescribir y ampliar la lógica de negocio definida en las clases del modelo. Veremos ahora como extender las vistas de la interfaz con las usuarias y usuarios.


**Ampliar las vistas**

Vistas de formulario, listas y búsqueda son definidas usando las estructuras de arco de XML. Para ampliar las vistas necesitamos una manera de modificar este XML. Esto significa localizar los elementos XML y luego introducir modificaciones en esos puntos. Las vistas heredadas permiten esto.

Una vista heredada se ve así:
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
El campo `inherit_id` identifica la vista que sera ampliada, a través de la referencia de su identificador externo usando el atributo especial “ref”. Los identificadores externos serán discutidos con mayor detalle en el Capítulo 4.

La forma natural de localizar los elementos XML es usando expresiones XPath. Por ejemplo, tomando la vista que fue definida en el capítulo anterior, la expresión XPtah  para localizar el elemento `<field name="is_done">`es `//field[@name]='is_done'`. Esta expresión encuentra un elemento con un atributo “name” igual a “is_done”. Puede encontrar mayor información sobre XPath en: [https://docs.python.org/2/library/xml.etree.elementtree.html#xpath-support](https://docs.python.org/2/library/xml.etree.elementtree.html#xpath-support).

Tener atributos “name” en los elementos es importante porque los hace mucho más fácil de seleccionar como puntos de extensión. Una vez que el punto de extensión es localizado, puede ser modificado o puede tener elementos XML agregados cerca de él.

Como un ejemplo práctico, para agregar el campo `date_deadline` antes del campo “is_done”, debemos escribir en el arco:
```
<xpath expr="//field[@name]='is_done'" position="before">
    <field name="date_deadline" /> 
</xpath> 
```
Afortunadamente Odoo proporciona una notación simplificada para eso, así que la mayoría de las veces podemos omitir la sintaxis XPath. En vez del elemento “xpath” anterior podemos usar el tipo de elementos que queramos localizar y su atributo distintivo. 

Lo anterior también puede ser escrito como: 
```
<field name="is_done" position="before">
<field name="date_deadline" /> </field>`
```
Agregar campos nuevos, cerca de campos existentes es hecho frecuentemente, por lo tanto la etiqueta `<field>` es usada frecuentemente como el localizador. Pero cualquier otra etiqueta puede ser usada: `<sheet>`, `<group>`, `<div>`, entre otras. El atributo “name” es generalmente la mejor opción para hacer coincidir elementos, pero a veces, podemos necesitar usar cadenas (el texto mostrado en un “label”) o el elemento de clase CSS.

El atributo de posición usado con el elemento localizador es opcional, y puede tener los siguientes valores:


- after: Este es agregado al elemento padre, después del nodo de coincidencia.
- before: Este es agregado al elemento padre, antes del nodo de coincidencia.
- inside (el valor predeterminado): Este es anexado al contenido del nodo de coincidencia.
- replace: Este reemplaza el nodo de coincidencia. Si es usado con un contenido vacío, borra un elemento.
- attributes: Este modifica los atributos XML del elemento de coincidencia (más detalles luego de esta lista).
La posición del atributo nos permite modificar los atributos del elemento de coincidencia. Esto es hecho usando los elementos `<attribute name=”attr-name”>` con los valores del atributo nuevo.

En el formulario de Tareas, tenemos el campo Activo, pero tenerlo visible no es muy útil. Quizás podamos esconderlo de la usuaria y el usuario. Esto puede ser realizado configurando su atributo invisible: 
```
<field name="active" position="attributes">
    <attribute name="invisible">1<attribute/> 
</field> 
```
Configurar el atributo “invisible” para esconder un elemento es una buena alternativa para usar el localizador de reemplazo para eliminar nodos. Debería evitarse la eliminación, ya que puede dañar las extensiones de modelos que pueden depender del nodo eliminado.

Finalmente, podemos poner todo junto, agregar los campos nuevos, y obtener la siguiente vista heredada completa para ampliar el formulario de tareas por hacer:
```
<record id="view_form_todo_task_inherited" model="ir.ui.view">
    <field name="name">Todo Task form – User extension</field>
    <field name="model">todo.task</field>
    <field name="inherit_id" ref="todo_app.view_form_todo_task"/>
    <field name="arch" type="xml">
        <field name="name" position="after">
            <field name="user_id" />
        </field>
        <field name="is_done" position="before">
            <field name="date_deadline" /> 
        </field>
        <field name="name" position="attributes">
            <attribute name="string">I have to…<attribute/>
        </field>
    </field> 
</record>
``` 
Esto debe ser agregado al archivo `todo_view.xml` en nuestro módulo, dentro de las etiquetas `<openerp>` y `<data>`, como fue mostrado en el capítulo anterior.

*Nota*
* Las vistas heredadas también pueden ser heredadas, pero debido a que esto crea dependencias más complicadas, debe ser evitado *.

No podemos olvidar agregar el atributo datos al archivo descriptor `__openerp__.py`:
``` 
'data':	['todo_view.xml'], 
``` 


**Ampliando mas vistas de árbol y búsqueda**

Las extensiones de las vistas de árbol y búsqueda son también definidas usando la estructura de arco XML, y pueden ser ampliadas de la misma manera que las vistas de formulario. Seguidamente mostramos un ejemplo de la ampliación  de vistas de lista y búsqueda.

Para la vista de lista, queremos agregar el campo usuario:
```
<record id="view_tree_todo_task_inherited" model="ir.ui.view">
    <field name="name">Todo Task tree – User extension</field>
    <field name="model">todo.task</field>
    <field name="inherit_id" ref="todo_app.view_tree_todo_task"/>
    <field name="arch"	type="xml">
        <field name="name" position="after">
            <field name="user_id" />
        </field>
    </field> 
</record> 
```
Para la vista de búsqueda, agregaremos una búsqueda por usuario, y filtros predefinidos para las tareas propias del usuario y tareas no asignadas a alguien.
```
<record id="view_filter_todo_task_inherited" model="ir.ui.view">
    <field name="name">Todo Task tree – User extension</field>
    <field name="model">todo.task</field>
    <field name="inherit_id"	ref="todo_app.view_filter_todo_task"/>
    <field name="arch"	type="xml">
        <field name="name" position="after">
            <field name="user_id" />
            <filter name="filter_my_tasks" string="My	Tasks" 	           
                    domain="[('user_id','in',[uid,False])]"	/>
            <filter name="filter_not_assigned" string="Not Assigned" 
                    domain="[('user_id','=',False)]" />
        </field>
    </field> 
</record> 
```
No se preocupe demasiado por la sintaxis específica de las vistas. Describiremos esto con más detalle en el Capítulo 6.


** Más sobre el uso de la herencia para ampliar los modelos **

Hemos visto lo básico en lo que se refiere a la ampliación de modelos “in place”, lo cual es la forma más frecuente de uso de la herencia. Pero la herencia usando el atributo `_inherit` tiene mayores capacidades, como la mezcla de clases.

También tenemos disponible el método de herencia delegada, usando el atributo `_inherits`. Esto permite a un modelo contener otros modelos de forma transparente a la vista, mientras por detrás de escena cada modelo gestiona sus propios datos. 

Exploremos esas posibilidades en más detalle.
 

** Copiar características usando herencia por prototipo **

El método que usamos anteriormente para ampliar el modelo solo usa el atributo `_inherit`. Definimos una clase que hereda el modelo `todo.task`, y le agregamos algunas características. La clase `_name` no fue fijada explícitamente; implícitamente fue también `todo.task`.

Pero usando el atributo `_name` nos permitió crear una mezcla de clases, configurando le el modelo que queremos ampliar. Aquí mostramos un ejemplo:
```
from openerp import models 
class TodoTask(models.Model): 
    _name = 'todo.task'
    _inherit = 'mail.thread' 
```
Esto amplia el modelo `todo.task` copiando las características del modelo `mail.thread`. El modelo `mail.thread` implementa la mensajería de Odoo y la función de seguidores, y es re usable, por lo tanto es fácil agregar esas características a cualquier modelo.

Copiar significa que los métodos y los campos heredados estarán disponibles en el modelo heredero. Por campos significa que estos serán creadas y almacenados en las tablas de la base de datos del modelo objetivo. Los registros de datos de el modelo original (heredado) y el nuevo modelo (heredero) son conservados sin relación entre ellos. Solo son compartidas las definiciones.

Estas mezclas son usadas frecuentemente como modelos abstractos, como el `mail.thread` usado en el ejemplo. Los modelos abstractos son como los modelos regulares excepto que no es creada ninguna representación de ellos en la base de datos. Actúan como plantillas, describen campos y la lógica para ser reusado en modelos regulares. 

Los campos que definen solo serán creados en aquellos modelos regulares que hereden de ellos. En un momento discutiremos en detalle como usar eso para agregar `mail.thread` y sus características de redes sociales a nuestro módulo. En la práctica cuando se usan las mezclas rara vez heredamos de modelos regulares, porque esto puede causar duplicación de las mismas estructuras de datos.

Odoo proporciona un mecanismo de herencia delegada, el cual impide la duplicación de estructuras de datos, por lo que es usualmente usada cuando se hereda de modelos regulares. Veamos esto con mayor detalle.
 

** Integrar Modelos usando herencia delegada **

La herencia delegada es el método de extensión de modelos usado con menos frecuencia, pero puede proporcionar soluciones muy convenientes. Es usada a través del atributo `_inherits` (note la 's' adicional) con un mapeo de diccionario de modelos heredados con campos relacionados a él.

Un buen ejemplo de esto es el modelo estándar Users, `res.users`, que tiene un modelo Partner integrado:
```
from openerp import models, fields 

class	User(models.Model):
    _name      = 'res.users'
    _inherits  = {'res.partner': 'partner_id'}
    partner_id = fields.Many2one('res.partner') 
```
Con la herencia delegada el modelos `res.users` integra el modelo heredado `res.partner`, por lo tanto cuando un usuario (User) nuevo es creado, un socio (Partner) también es creado y se mantiene una referencia a este a través del campo `partner_id` de User. Es similar al concepto de polimorfismo en la programación orientada a objetos.

Todos los campos del modelo heredado, Partner, están disponibles como si fueran campos de User, a través del mecanismo de delegación. Por ejemplo, el nombre del socio y los campos de dirección son expuestos como campos de User, pero de hecho son almacenados en el modelo Partner enlazado, y no ocurre ninguna duplicación de la estructura de datos.

La ventaja de esto, comparada a la herencia por prototipo, es que no hay necesidad de repetir la estructura de datos en muchas tablas, como las direcciones. Cualquier modelo que necesite incluir un dirección puede delegar esto a un modelo Partner vinculado. Y si son introducidas algunas modificaciones en los campos de dirección del socio o validaciones, estas estarán disponibles inmediatamente para todos los modelos que vinculen con él!

*Nota* 
*Note que con la herencia delegada, los campos con heredados, pero los métodos no.


**Usar la herencia para agregar características redes sociales** 

El módulo de red social (nombre técnico mail) proporciona la pizarra de mensajes que se encuentra en la parte inferior de muchos formularios, también llamado Charla Abierta (Open Chatter), los seguidores se exhiben junto a la lógica relativa a los mensajes y notificaciones. Esto es algo que frecuentemente querremos agregar a nuestros modelos, así que aprendamos como hacerlo.

Las características de mensajería de red social son proporcionadas por el modelo `mail.thread` del modelo mail. Para agregarlo a un módulo personalizado necesitamos:

- Que el módulo dependa de mail.
- Que la clase herede de `mail.thread`.
- Tener agregados a la vista de formulario los widgets Seguidores e Hilo.

Opcionalmente, configurar las reglas de registro para seguidores.

Sigamos esta lista de verificación:

- En relación a la *#1*, debido a que nuestro módulo ampliado depende de `todo_app`, el cual a su vez depende de mail, la dependencia de mail esta implícita, por lo tanto no se requiere ninguna acción.

- En relación a la *#2*, la herencia a `mail.thread` es hecha usando el atributo `_inherit`. Pero nuestra clase ampliada de tareas por hacer ya esta usando el atributo `_inherit`.

Afortunadamente, también puede aceptar una lista de modelos desde los cuales heredar, así que podemos usar esto para hacer que incluya la herencia a `mail.thread`:
```
_name    = 'todo.task'
_inherit = ['todo.task', 'mail.thread'] 
```
El modelo `mail.thread` es un modelo abstracto. Los modelos abstracto son como los modelos regulares excepto que no tienen una representación en la base de datos; no son creadas tablas para ellos. Los modelos abstractos no están destinado a ser usados directamente. Pero se espera que sean usados en la mezcla de clases, como acabamos de hacer. 

Podemos pensar en los modelos abstractos como plantillas con características listas para usar. Para crear una clase abstracta solo necesitamos usar modelos abstractos. AbstractModel en vez de `models.Model`.

- Para la número *#3*, queremos agregar el widget de red social en la parte inferior del formulario. Podemos re usar la vista heredada que recien creamos, `view_form_todo_task_inherited`, y agregar esto a sus datos arco:
```
<sheet position="after">
    <div class="oe_chatter">
        <field name="message_follower_ids" widget="mail_followers" />
        <field name="message_ids" widget="mail_thread" />
    </div>
</sheet> 
```
Los dos campos que hemos agregado aquí no han sido declarados por explícitamente, pero son provistos por el modelo `mail.thread`.

El paso final es fijar las reglas de los registros de seguidores, esto solo es necesario si nuestro modelo tiene implementadas reglas de registro que limitan el acceso a otros usuarios. En este caso, necesitamos asegurarnos que los seguidores para cada registro tengan al menos acceso de lectura.

Tenemos reglas de registro en nuestro modelo de tareas por hacer así que necesitamos abordar esto, y es lo que haremos en la siguiente sección.
 

**Modificar datos**

A diferencia de las vistas, los registros de datos no tienen una estructura de arco XML y no pueden ser ampliados usando expresiones XPath. Pero aún pueden ser modificados reemplazando valores en sus campos.

El elemento `<record id=”x” model=”y”>` esta realizando una operación de inserción o actualización en un modelo: si x no existe, es creada; de otra forma, es actualizada / escrita.

Debido a que los registros en otros módulos pueden ser accedidos usando un identificador `<model>.<identifier>`, es perfectamente legal para nuestro módulo sobrescribir algo que fue escrito antes por otro módulo.

*Nota* 
*Note que el punto esta reservado para separar el nombre del módulo del identificador del objeto, así que no debe ser usado en identificadores. Para esto use la barra baja (`_`). *

Como ejemplo, cambiemos la opción de menú creada por el módulo `todo_app` en “My To Do”. Para esto agregamos lo siguiente al archivo `todo_user/todo_view.xml`:
```
<!-- Modify menu item -->
<record id="todo_app.menu_todo_task" model="ir.ui.menu">
    <field name="name">My To-Do</field>
</record> 
<!-- Action to open To-Do Task list -->
<record model="ir.actions.act_window" id="todo_app.action_todo_task">
    <field name="context">
        {'search_default_filter_my_tasks': True}
    </field>
</record> 
```
 
**Ampliando las reglas de registro**

La aplicación Tareas por Hacer incluye una regla de registro para asegurar que cada tarea sea solo visible para el usuario que la he creado. Pero ahora, con al adición de las características sociales, necesitamos que los seguidores de la tarea también tengan acceso. El modelo de red social no maneja esto por si solo.

Ahora las tareas también pueden tener usuarios asignados a ellas, por lo tanto tiene más sentido tener reglas de acceso que funcionen para el usuario responsable en vez del usuario que creo la tarea.

El plan será el mismo que para la opción de menú: sobrescribir `todo_app.todo_task_user_rule` para modificar el campo `domain_force` a un valor nuevo.

Desafortunadamente, esto no funcionará esta vez. Recuerde que el `<data no_update=”1”>` que usamos anteriormente en el archivo XML de las reglas de seguridad: previene las operaciones posteriores de escritura.

Debido a que las actualizaciones del registro no están permitidas, necesitamos una solución alterna. Este será borrar el registro y agregar un reemplazo para este en nuestro módulo.

Para mantener las cosas organizadas, crearemos un archivo `security/todo_access_rules.xml` y agregaremos lo siguiente:
```
<?xml	version="1.0" encoding="utf-8"?> 
    <openerp>
        <data noupdate="1"> 
	    <delete	model="ir.rule" 
                 search="[('id''=',ref('todo_app.todo_task_user_rule'))]" /> 
	    <record	id="todo_task_per_user_rule" model="ir.rule">
                <field name="name">ToDo Tasks only for owner</field>
                <field name="model_id" ref="model_todo_task"/>
                <field name="groups" eval="[(4,	ref('base.group_user'))]"/>
                <field name="domain_force">
                    ['|',('user_id','in',	[user.id,False]),
                    ('message_follower_ids','in',[user.partner_id.id])]
                </field>
            </record> 
        </data>
    </openerp> 
```
Esto encuentra y elimina la regla de registro `todo_task_user_rule` del módulo `todo_app`, y crea una nueva regla de registro `todo_task_per_user`. El filtro de dominio que usamos ahora hace la tarea visible para el usuario responsable `user_id`, para todo el mundo si el usuario responsable no ha sido definido (igual a False), y para todos los seguidores. La regla se ejecutará en un contexto donde el usuario este disponible y represente la sesión del usuario actual. Los seguidores son socios, no objetos User, así que en vez de `user_id`, necesitamos usar `user.partner_id.id`.

*Tip* 
Cuando se trabaja en campos de datos con `<dara noupdate=”1”>` puede ser engañoso porque cualquier edición posterior no será actualizada en Odoo. Para evitar esto, use temporalmente `<data noupdate=”0”>` durante el desarrollo, y cambielo solo cuando haya terminado con el módulo.*

Como de costumbre, no debemos olvidar agregar el archivo nuevo al archivo descriptor `__openerp__.py` en el atributo “data”:
```
'data': ['todo_view.xml', 'security/todo_access_rules.xml'], 
```
Note que en la actualización de módulos, el elemento `<delete>` arrojará un mensaje de advertencia, porque el registro que será eliminado no existe más. Esto no es un error y la actualización se realizará con éxito, así que no es necesario preocuparse por esto.
 
 
**Resumen** 

Ahora debe ser capaz de crear módulos nuevos para ampliar los módulos existentes. Vimos como ampliar el módulo To-Do creado en los capítulos anteriores.

Se agregaron nuevas características en las diferentes capas que forman la aplicación. Ampliamos el modelo Odoo para agregar campos nuevos, y ampliamos los métodos con su lógica de negocio. Luego, modificamos las vistas para hacer disponibles los campos nuevos. Finalmente, aprendió como ampliar un modelo heredando de otros modelos, y usamos esto para agregar características de red social a nuestra aplicación.

Con estos tres capítulos, tenemos una vista general de las actividades mas comunes dentro del desarrollo en Odoo, desde la instalación de Odoo y configuración a la creación de módulos y extensiones. 

Los siguientes capítulos se enfocarán en áreas específicas, la mayoría de las cuales hemos tocado en estos primeros capítulos. En el siguiente capítulo, abordaremos la serialización de datos y el uso de archivos XML y CSV con más detalle.