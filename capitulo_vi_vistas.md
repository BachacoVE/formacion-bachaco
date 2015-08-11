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

Como hemos visto en capítulos anteriores, las vistas de formulario cumplir con una diseño simple o un diseño de documento de negocio, similar a un documento en papel.

Ahora veremos como diseñar vistas de negocio y usar los elementos y widgets disponibles. Esto es hecho usualmente heredando la vista base. Pero para hacer el código más simple, crearemos una vista completamente nueva para las tareas por hacer que sobre escribirá la definida anteriormente.

De hecho, el mismo modelo puede tener diferentes vistas del mismo tipo. Cuando se abre un tipo de vista para un modelo a través de una acción, se selecciona aquella con la prioridad más baja. O como alternativa, la acción puede especificar exactamente el identificador de la vista que se usará. La acción que definimos al principio de este capítulo solo hace eso; el `view_id` le dice a la acción que use específicamente el formulario con el ID `view_form_todo_task_ui`. Esta es la vista que crearemos a continuación.


**Vistas de negocio**
 
En una aplicación de negocios podemos diferenciar los datos auxiliares de los datos principales del negocio. Por ejemplo, en nuestra aplicación los datos principales son las tareas por hacer, y las etiquetas y los estados son tablas auxiliares.

Estos modelos de negocio pueden usar diseños de vista de negocio mejorados para mejorar la experiencia del usuario y la usuaria. Si vuelve a ejecutar la vista del formulario de tarea agregada en el Capítulo 2, notará que ya sigue la estructura de vista de negocio.

La vista de formulario correspondiente debe ser agregada después de las acciones y los elementos del menú, que agregamos anteriormente, y su estructura genérica es esta:
```
<record id="view_form_todo_task_ui"	model="ir.ui.view">
    <field name="name">view_form_todo_task_ui</field>
    <field name="model">todo.task</field>
    <field name="arch" type="xml">
        <form>
            <header><!-- Buttons and status widget --> </header>
            <sheet><!--	Form	content --> </sheet>
            <!-- History and communication: →
            <div class="oe_chatter">
                <field name="message_follower_ids" widget="mail_followers" />
                <field name="message_ids" widget="mail_thread" />
		</div>
        </form>
    </field>
</record> 
```
Las vistas de negocio se componen de tres área visuales:

- Un encabezado, “header”
- Un “sheet” para el contenido
- Una sección al final de historia y comunicación, “history and communication”.

La sección historia y comunicación, con los widgets de red social en la parte inferior, es agregada por la herencia de nuestro modelo de `mail.thread` (del módulo mail), y agrega los elementos del ejemplo XML mencionado anteriormente al final de la vista de formulario. También vimos esto en el Capítulo 3.


**La barra de estado del encabezado**

La barra de estado en la parte superior usualmente presenta el flujo de negocio y los botones de acción.

Los botones de acción son botones regulares de formulario, y lo más común es que el siguiente paso sea resaltarlos, usando `class=”oe_highlight”`. En `todo_ui/todo_view.xml` podemos ampliar el encabezado vacío para agregar le una barra de estado:
```
<header>
    <field name="stage_state" invisible="True" />
    <button	name="do_toggle_done" type="object" attrs="{'invisible' [('stage_state','in',['done','cancel'])]}" string="Toggle Done" class="oe_highlight" />
    <!-- Add stage statusbar:	… --> 
</header> 
```
Los botones de acción disponible puede diferir dependiendo en que parte del proceso se encuentre el documento actual. Por ejemplo, un botón Marcar como Hecho no tiene sentido si ya estamos en el estado “Hecho”.

Esto se realiza usando el atributo “states”, que lista los estados donde el botón debería estas visible, como esto: `states="draft,open"`.

Para mayor flexibilidad podemos usar el atributo “attrs”, el cual forma condiciones donde el botón debería ser invisible: `attrs="{'invisible' [('stage_state','in', ['done','cancel'])]`.
 
Estas características de visibilidad también están disponibles para otros elementos de la vista, y no solo para los botones. Veremos esto en detalle más adelante en este capítulo.


**El flujo de negocio**

El flujo de negocio es un widget de barra de estado que se encuentra en un campo el cual representa el punto en el flujo donde se encuentra el registro. Usualmente es un campo de selección “State”, o un campo “Stage” muchos a uno. En ambos casos puede encontrarse en muchos módulos de Odoo.

El “Stage” es un campo muchos a uno que se usa en un modelo donde los pasos del proceso están definidos. Debido a esto pueden ser fácilmente configurados por el usuario u la usuaria final para adecuarlo a sus procesos específicos de negocio, y son perfectos para el uso de pizarras kanban.

El “State” es una lista de selección que muestra los pasos estables y principales de un proceso, como Nuevo, En Progreso, o Hecho. No pueden ser configurados por el usuario o usuaria final, pero son fáciles de usar en la lógica de negocio. Los “States” también tienen soporte especial para las vistas: el atributo “state” permite que un elemento este habilitado para ser seleccionado por el usuario o usuaria dependiendo en el estado en que se encuentre el registro.

*Tip* 
* Es posible obtener un beneficio de ambos mundos, a través del uso de “stages” que son mapeados dentro de los “states”. Esto fue lo que hicimos en el capítulo anterior, haciendo disponible a “State” en los documentos de tareas por hacer a través de un campo calculado.*

Para agregar un flujo de “stage” en nuestro encabezado de formulario:
```
<!--	Add	stage	statusbar:	...	--> 
<field name="stage_id" widget="statusbar" clickable="True" options="{'fold_field': 'fold'}"	/> 
```
El atributo “clickable” permite hacer clic en el widget, para cambiar la etapa o el estado del documento. Es posible que no queramos esto si el progreso del proceso debe realizarse a través de botones de acción.

En el atributo “options” podemos usar algunas configuraciones específicas: 

- `fold_fields`, cuando de usa “stages”, es el nombre del campo que usa el “stage” del modelo usa para indicar en cuales etapas debe ser mostrado “fold”. 
- `statusbar_visible`, cuando se usa “states”, lista los estados que deben estar siempre visibles, para mantener ocultos los estados de excepción que se usan para casos menos comunes. Por ejemplo: `statusbar_visible=”draft,open.done”`.

La hoja canvas es el área del formulario que contiene los elementos principales del formulario. Esta diseñada para parecer un documento de papel, y sus registros de datos, a veces, puede ser referidos como documentos.

La estructura general del documento tiene estos componentes:

- Información de título y subtítulo
- Un área de botón inteligente, es la parte superior derecha de los campos del encabezado del documento.
- Un cuaderno con páginas en etiquetas, con líneas de documento y otros detalles.


**Título y subtítulo**

Cuando se usa el diseño de hoja, los campos que están fuera del bloque `<group>` no se mostrarán las etiquetas automáticamente. Es responsabilidad de la persona que desarrolla controlar si se muestran las etiquetas y cuando.

También se puede usar las etiquetas HTML para hacer que el título resplandezca. Para mejores resultados, el título del documento debe estar dentro de un “div” con la clase `oe_title`:

```
<div class="oe_title">
    <label for="name" class="oe_edit_only"/>
    <h1><field name="name"/></h1>
    <h3>
        <span class="oe_read_only">By</span>
        <label for="user_id" class="oe_edit_only"/>
        <field name="user_id" class="oe_inline"	/>
    </h3>
</div> 
```
Aquí podemos ver el uso de elementos comúnes de HTML como  div, span, h1 y h3.


**Etiquetas y campos**

Las etiquetas de los campos no son mostradas fuera de las secciones `<group>`, pero podemos mostrarlas usando el elemento  `<label>`:

- El atributo “for” identifica el campo desde el cual tomaremos el texto de la etiqueta.
- El atributo “string” sobre escribe el texto original de la etiqueta del campo.
- Con el atributo “class” también podemos usar las clases CSS para controlar la presentación. Algunas clases útiles son:

  - `oe_edit_only` para mostrar lo solo cuando el formulario este modo de edición.
  - `oe_read_only`  para mostrar lo solo cuando el formulario este en modo de lectura.

Un ejemplo interesante es reemplazar el texto con un ícono:
```
<label for="name" string=" " class="fafa-wrench"/> 
```
Odoo empaqueta los íconos “Font Awesome”, que se usan aquí. Los íconos disponibles puede encontrar se en  [http://fontawesome.org](http://fontawesome.org).
  

**Botones inteligentes**

El área superior izquierda puede tener una caja invisibles para colocar botones inteligentes. Estos funcionan como los botones regulares pero pueden incluir información estadística. Como ejemplo agregaremos un botón para mostrar el número total de tareas realizadas por el dueño de la tarea por hacer actual.

Primero necesitamos agregar el campo calculado correspondiente a `todo_ui/todo_model.py`. Agregue lo siguiente a la clase TodoTask:
```
@api.one def compute_user_todo_count(self): 
    self.user_todo_count = self.search_count([('user_id', '=', self.user_id.id)])
    user_todo_count      = fields.Integer('User	To-Do	Count', compute='compute_user_todo_count') 
```
Ahora agregaremos la caja del botón con un botón dentro de ella. Agregue lo siguiente justo después del bloque div `oe_title`:  
```
<div name="buttons" class="oe_right oe_button_box">
    <button class="oe_stat_button" type="action" icon="fa-tasks" name="%(todo_app.action_todo_task)d" string="" context="{'search_default_user_id': user_id, 'default_user_id': user_id}" help="Other to-dos for this user" >
        <field string="To-dos" name="user_todo_count" widget="statinfo"/>
    </button>
</div> 
```
El contenedor para los botones es un div con las clases `oe_button_box` y `oe_right`, para que este alineado con la parte derecha del formulario.

En el ejemplo el botón muestra el número total de las tareas por hacer que posee el documento responsable. Al hacer clic en el, este las inspeccionara, y si se esta creando tareas nuevas el documento responsable original será usado como predeterminado. 

Los atributos usados para el botón son:

- `class="oe_stat_button"`, es para usar un estilo rectángulo en vez de un botón.
- icon, es el ícono que será usaso, escogido desde el conjunto de íconos de Font Awesome.
- type, será usualmente una acción para la acción de ventana, y name será el ID de la acción que será ejecutada. Puede usarse la formula `%(id-acción-externa)d`, para transformar el ID externo en un número de ID real. Se espera que esta acción abra una vista con los registros relacionados.
- string, puede ser usado para agregar texto al botón. No se usa aquí porque el campo que lo contiene ya proporciona un texto.
- context, fija las condiciones estándar en la vista destino, cuando se haga clic a través del botón, para los filtros de datos y los valores predeterminados para los registros creados.
- help, es la herramienta de ayuda que será mostrada. 

Por si solo el botón es un contenedor y puede tener sus campos dentro para mostrar estadísticas. Estos son campos regulares que usan el widget “statinfo”.

El campo debe ser un campo calculado, definido en el módulo subyacente. También podemos usar texto estático en vez de o junto a los campos de “statinfo”, como : `<div>User's To-dos</div>`

 
**Organizar el contenido en un formulario**

El contenido principal del formulario debe ser organizado usando etiquetas `<group>`. Un grupo es una cuadrícula con dos columnas. Un campo y su etiqueta ocupan dos columnas, por lo tanto al agregar campos dentro de un grupo, estos serán apilados verticalmente. 

Si anidamos dos elementos `<group>` dentro de un grupo superior, tendremos dos columnas de campos con etiquetas, una al lado de la otra.
```
<group name="group_top">
    <group name="group_left">
        <field name="date_deadline"	/>
        <separator string="Reference"/>
        <field name="refers_to"/>
    </group>
    <group name="group_right">
        <field name="tag_ids" widget="many2many_tags"/>
    </group>
</group> 
```
Los grupos pueden tener un atributo “string”, usado para el título de la sección. Dentro de una sección de grupo, los títulos también pueden agregarse usando un elemento “separator”.

*Tip*  
*Intente usar la opción Alternar la Disposición del Esquema del Formulario del menú de Desarrollo: este dibuja líneas alrededor de cada sección del formulario, permitiendo un mejor entendimiento de como esta organizada la vista actual.*


**Cuaderno con pestañas**

Otra forma de organizar el contenido es el cuaderno, el cual contiene múltiples secciones a través de pestañas llamadas páginas. Esto puede usarse para mantener algunos datos fuera de la vista hasta que sean necesarios u organizar un largo número de campos por tema.

No necesitaremos esto en nuestro formulario de tareas por hacer, pero el siguiente es un ejemplo que podríamos agregar en el formularios de etapas de la tarea:
```
<notebook>
    <page string="Whiteboard" name="whiteboard">
        <field name="docs"/>
    </page>
    <page name="second_page">
        <!-- Second page content →
    </page>
</notebook> 
```
Se considera una buena practica tener nombres en las páginas, esto hace que la ampliación de estas por parte de otros módulo sea más fiable


**Elementos de la vista**

Hemos visto como organizar el contenido dentro de un formulario, usando elementos como encabezado, grupo y cuaderno. Ahora, podemos ahondar en los elementos de campo y botón y que podemos hacer con ellos.
 

**Botones**
  
Los botones soportar los siguientes atributos:

- icon. A diferencia de los botones inteligentes, los íconos disponibles para los botones regulares son aquellos que se encuentran en `addons/web/static/src/img/icons`. 
- string, es el texto de descripción del botón.
- type, puede ser “workflow”, “object” o “action”, para activar una señal de flujo de trabajo, llamar a un método Python o ejecutar una acción de ventana.
- name, es el desencadenante de un flujo de trabajo, un método del modelo, o la ejecución de una acción de ventana, dependiendo del “type” del botón.
- args, se usa para pasar parámetros adicionales al método, si el “type” es “object”.
- context, fija los valores en el contexto de la sesión, el cual puede tenet efecto luego de la ejecución de la acción de ventana, o al llamar a un método de Python. En el último caso, a veces puede ser usado como un alternativa a “args”.
- confirm, agrega un mensaje con el mensaje de texto preguntando por una confirmación.
- `special="cancel"`, se usa en los asistentes, para cancelar o cerrar el formulario. No debe ser usado con “type”.


**Campos**

Los campos tiene los siguientes atributos disponibles. La mayoría es tomado de los que fue definido en el modelo, pero pueden ser sobre escritos en la vista. Los atributos generales son:

- name: identifica el nombre técnico del campo.
- string: proporciona la descripción de texto de la etiqueta para sobre escribir aquella provista por el modelo.
- help: texto de ayuda a ser usado y que reemplaza el proporcionado por el modelo. 
- placeholder: proporciona un texto de sugerencia que será mostrado dentro del campo.
- widget: sobre escribe el widget predeterminado usado por el tipo de campo. Exploraremos los widgets disponibles mas adelante en este mismo capítulo.
- options: contiene opciones adicionales para ser usadas por el widget. 
- class: proporciona las clases CSS usadas por el HTML del campo.
- `invisible="1"`: invisibiliza el campo.
- `nolabel="1"`: no muestra la etiqueta del campo, solo es significativo para los campos que se encuentran dentro de un elemento `<group>`.
- `readonly="1"`: no permite que el campo sea editado.
- `required="1"`: hace que el campo sea obligatorio.


Atributos específicos para los tipos de campos:

- sum, avg: para los campos numéricos, y en las vistas de lista/árbol, estos agregan un resumen al final con el total o el promedio de los valores.
- `password="True"`: para los campos de texto, muestran el campo como una campo de contraseña.
- filename: para campos binarios, es el campo para el nombre del archivo.
- `mode="tree"`: para campos One2many, es el tipo de vista usado para mostrar los registros. De forma predeterminada es de árbol, pero también puede ser de formulario, kanban o gráfico.

Para los atributos Boolean en general, podemos usar True o 1 para habilitarlo y False o 0 (cero) para deshabilitarlo. Por ejemplo, `readonly=”1”` y `realonly=”True”` son equivalentes.


**Campos relacionales**

En los campos relacionales, podemos tener controles adicionales referentes a los que el usuario o la usuaria puede hacer. De forma predeterminada el usuario y la usuaria pueden crear nuevos registros desde estos campos (también conocido como creación rápida) y abrir el formulario relacionado al registro. Esto puede ser deshabilitado usando el atributo del campo “options”:
```
options={'no_open': True, 'no_create': True} 
```
El contexto y el dominio también son particulares en los campos relacionales. El contexto puede definir valores predeterminados para los registros relacionados, y el dominio puede limitar los registros que pueden ser seleccionados, por ejemplo, basado en otro campo del registro actual. Tanto el contexto como el dominio pueden ser definidos en el modelo, pero solo son usados en la vista.


**Widgets de campo**

Cada tipo de campo es mostrado en el formulario con el widget predeterminado apropiado. Pero otros widget adicionales están disponible y pueden ser usados:

Widgets para los campos de texto:

- email: convierte al texto del correo electrónico en un elemento “mail-to” ejecutable.
- url: convierte al texto en un URL al que se puede hacer clic.
- html: espera un contenido en HTML y lo representa; en modo de edición usa un editor WYSIWYG para dar formato al contenido sin saber HTML.

Widgets para campos numéricos: 

- handle: específicamente diseñado para campos de secuencia, este muestra una guía para dibujar líneas en una vista de lista y re ordenarlos manualmente.
- float_time: da formato a un valor decimal como tiempo en horas y minutos.
- monetary: muestra un campo decimal como un monto en monedas. La moneda a usar puede ser tomada desde un campo como `options="{'currency_field': 'currency_id'}"`. 
- progressbar: presenta un decimal como una barra de progreso en porcentaje, usualmente se usa en un campo calculado que computa una tasa de culminación.

Algunos widget para los campos relacionales y de selección:

- `many2many_tags`: muestran un campo muchos a muchos como una lista de etiquetas.
- selection: usa el widget del campo Selección para un campo mucho a uno.
- radio: permite seleccionar un valor para una opción del campo de selección usando botones de selección (radio buttons).
- `kanban_state_selection`: muestra una luz de semáforo para la lista de selección de esta kanban.
- priority: representa una selección como una lista de estrellas a las que se puede hacer clic.


**Eventos on-change**

A veces necesitamos que el valor de un campo sea calculado automáticamente cuando cambia otro campo. El mecanismo para esto se llama `on-change`.

Desde la versión o, los eventos `on-change` están definidos en la capa del modelo, sin necesidad de ningún marcado especial en las vistas. Es se hace creando los métodos para realizar el calculo y enlazándolos al campo(s) que desencadenara la acción, usando el decorador `@api.onchenge('field1','field2')`.

En las versiones anteriores, ente enlace era hecho en la capa de vista, usando el atributo “onchange” para fijar el método de la clase que sería llamado cuando el campo cambiara. Esto todavía es soportado, pero es obsoleto. Tenga en cuenta que los métodos  `on-change` con el estilo viejo no pueden ser ampliados usando la API nueva. Si necesita hacer esto, deberá usar la API vieja.


**Vistas dinámicas**

Los elementos visibles como un formulario también pueden ser cambiados dinámicamente, dependiendo, por ejemplo de los permisos de usuario o la etapa del proceso en la cual esta el documento.

Estos dos atributos nos permiten controlar la visibilidad de los elemento en la interfaz:

- groups: hacen al elemento visible solo para los miembros de los grupos de seguridad específicos. Se espera una lista separada por coma de los ID XML del grupo.
- states: hace al elemento visible solo cuando el documento esta en el estado especificado. Espera una lista separada por coma de los códigos de “State”, y el modelo del documento debe tener un campo “state”.

Para mayor flexibilidad, podemos fijar la visibilidad de un elemento usando expresiones evaluadas del lado del cliente. Esto puede hacerse usando el atributoo “attrs” con un diccionario que mapea el atributo “invisible” al resultado de una expresión de dominio.

Por ejemplo, para hacer que el campo `refers_to` sea visible en todos los estados menos “draft”: 
```
<field name="refers_to" attrs="{'invisible': [('state','=','draft')]}"	/> 
```
El atributo “invisible” esta disponible para cualquier elemento, no solo para los campos. Podemos usarlo en las páginas de un cuaderno o en grupos, por ejemplo.

El “attrs” también puede fijar valores para otros dos atributos: readonly y required, pero esto solo tiene sentido para los campos de datos, convirtiéndolos en campos que no pueden ser editados u obligatorios. Con esto podemos agregar alguna lógica de negocio haciendo a un campo obligatorio, dependiendo del valor de otro campo, o desde un cierto estado mas adelante.
 

**Vistas de lista**

Comparadas con las vistas de formulario, las vistas de listas son mucho más simples. Una vista de lista puede contener campos y botones, y muchos de los atributos de los formularios también están disponibles.

Aquí se muestra un ejemplo de una vista de lista para nuestra Tareas por Hacer:
```
<record id="todo_app.view_tree_todo_task"	model="ir.ui.view">
    <field name="name">To-do Task Tree</field>
    <field name="model">todo.task</field>
    <field name="arch" type="xml">
        <tree editable="bottom" colors="gray:is_done==True" fonts="italic: state!='open'" delete="false">
            <field name="name"/>
            <field name="user_id"/>
        </tree>
    </field>
</record> 
```
Los atributos para el elemento “tree” de nivel superior son:

- editable: permite que los registros sean editados directamente en la vista de lista. Los valores posibles son “top” y “bottom”, los lugares en donde serán agregados los registros nuevos.

- colors: fija dinámicamente el color del texto para los registros, basándose en su contenido. Es una lista separada por punto y coma de valores `color:condition`. “color” es un color válido CSS (vea [http://www.w3.org/TR/css3-color/#html4](http://www.w3.org/TR/css3-color/#html4) ), y “condition” es una expresión Python que evalúa el contexto del registro actual.

- fonts: modifica dinámicamente el tipo de letra para los registro basándose en su contexto. Es similar al atributo “colors”, pero este fija el estilo de la letra a “bold”, “italic” o “underline”.
 
- create, delete, edit: si se fija a “false” (en minúscula), deshabilita la acción correspondiente en la vista de lista.
 

**Vistas de búsqueda**

Las opciones de búsqueda disponibles en las vistas son definidas a través de una vista de lista. Esta define los campos que serán buscados cuando se escriba en la caja de búsqueda. También provee filtros predefinidos que pueden ser activados con un clic, y opciones de agrupación de datos para los registros en las vistas de lista o kanban.

Aquí se muestra una vista de búsqueda para las tareas por hacer:
```
<record id="todo_app.view_filter_todo_task" model="ir.ui.view">
    <field name="name">To-do Task Filter</field>
    <field name="model">todo.task</field>
    <field name="arch" type="xml">
        <search>
            <field name="name" domain_filter="['|', ('name','ilike',self),('user_id','ilike',self)]"/>
            <field name="user_id"/>
        	<filter name="filter_not_done" string="Not Done" domain="[('is_done','=',False)]"/>
            <filter name="filter_done" string="Done" domain="[('is_done','!=',False)]"/>
            <separator/>
            <filter name="group_user" string="By User" context="{'group_by':'user_id'}"/>
        </search>
    </field>
</record>
``` 
Podemos ver dos campos que serán buscados: “name” y “user_id”. En “name” tenemos una regla de filtro que hace la “búsqueda si” tanto en la descripción como en el usuario responsable. Luego tenemos dos filtros predefinidos, filtrando las “tareas no culminadas” y “tareas culminadas”. Estos filtros pueden ser activados de forma independiente, y serán unidos por un operador “OR” si ambos son habilitados. Los bloques de “filters” separados por un elemento `<separator/>` serán unidos por un operador “AND”.

El tercer filtro solo fija un contexto o ”group-by”. Esto le dice a la vista que agrupe los registros por ese campo, `user_id` en este caso.

Los elementos “filed” pueden usar los siguientes atributos:
 
- name: identifica el campo.
- string: proporciona el texto de la etiqueta que será usado, en vez del predeterminado.
- operator: nos permite usar un operador diferente en vez del predeterminado - `=` para campos numéricos y “ilike” para otros tipos de campos.
- filter_domain: puede usarse para definir una expresión de dominio específica para usar en la búsqueda, proporcionando mayor flexibilidad que el atributo “operator”. El texto que será buscado se referencia en la expresión usando “self”.
- groups: permite hacer que la búsqueda en el campo solo este disponible para una lista de grupos de seguridad (identificado por los Ids XML)

Estos son los atributos disponibles para los elementos “filter”:
 
- name: en un identificador, usado para la herencia o para habilitar la a través de la clave `search_default_` en el contexto de acciones de ventana.
- string: proporciona el texto de la etiqueta que se mostrara para el filtro (obligatorio) 
- domain: proporciona la expresión de dominio del filtro para ser añadida al dominio activo.
- context: es un diccionario de contexto para agregarlo al contexto actual. Usualmente este fija una clave `group_by` con el nombre del filtro que agrupara los registros.
- groups: permite hacer que el filtro de búsqueda solo este disponible para una lista de grupos.


**Otros tipos de vista**

Los tipos de vista que se usan con mayor frecuencia son los formularios y las listas, discutidos hasta ahora. A parte de estas, existen otros tipos de vista, y daremos un vistazo a cada una de ellas. Las vistas kanban no serán discutidas aquí, ya que las veremos en el Capítulo 8.

Recuerde que los tipos de vista disponibles están definidos en el atributo `view_mode` de la acción de ventana correspondiente.


**Vistas de calendario**

Como su nombre lo indica, esta presenta los registros en un calendario. Una vista de calendario para las tareas por hacer puede ser de la siguiente manera:
```
<record id="view_calendar_todo_task" model="ir.ui.view">
    <field name="name">view_calendar_todo_task</field>
    <field name="model">todo.task</field>
    <field name="arch" type="xml">
        <calendar date_start="date_deadline" color="user_id" display="[name], Stage[stage_id]">
            <!-- Fields used for the text of display attribute →
            <field name="name" />
            <field name="stage_id"	/>
        </calendar>
    </field>
</record> 
```
Los atributos de “calendar” son los siguientes:

- `date_start`: El campo para la fecha de inicio (obligatorio).
- `date_end`: El campo para la fecha de culminación (opcional).
- `date_delay`: El campo para la duración en días. Este puede ser usado en vez de `date_end`.
- color: El campo para colorear las entradas del calendario. Se le asignará un color a cada valor en el calendario, y todas sus entradas tendrán el mismo color.
- display: Este es el texto que se mostrará en las entradas del calendario. Los campos pueden ser insertados usando `[<field>]`. Estos campos deben ser declarados dentro del elemento “calendar”.


**Vistas de Gantt**

Esta vista presenta los datos en un gráfico de Gantt, que es útil para la planificación. Las tareas por hacer solo tiene un campo de fecha para la fecha de límite, pero podemos usarla para tener una vista funcional de un gráfico Gantt básico:
```
<record id="view_gantt_todo_task" model="ir.ui.view">
    <field name="name">view_gantt_todo_task</field>
    <field name="model">todo.task</field>
    <field name="arch" type="xml">
        <gantt date_start="date_deadline" default_group_by="user_id" />
    </field>
</record> 
```
Los atributos que puede ser usados para las vistas Gantt son los siguientes.

- `date_start`: El campo para la fecha de incio (obligatorio).
- `date_stop`: El campo para la fecha de culminación. Puede ser reemplazado por `date_delay`.
- `date_delay`: El campo con la duración en días. Puede usarse en vez de `date_stop`.
- progress: Este campo proporciona el progreso en porcentaje (entre 0 y 100).
- `default_group_by`: Este campo se usa para agrupar las tareas Gantt.
 

**Vistas de gráfico**

Los tipos de vista de gráfico proporcionan un análisis de los datos, en forma de gráfico o una tabla pivote interactiva.

Agregaremos una tabla pivote a las tareas por hacer. Primero, necesitamos agregar un campo. En la clase TodoTask, del archivo `todo_ui/todo_model.py`, agregue este línea:
``` 
effort_estimate = fields.Integer('Effort Estimate') 
```
También debe ser agregado al formulario de tareas por hacer para que podamos fijar datos allí. Ahora, agreguemos la vista de gráfico con una tabla pivote:
```
<record id="view_graph_todo_task" model="ir.ui.view">
    <field name="name">view_graph_todo_task</field>
    <field name="model">todo.task</field>
    <field name="arch" type="xml">
        <graph type="pivot">
            <field name="stage" type="col" />
            <field name="user_id"	/>
            <field name="date_deadline"	interval="week" />
            <field name="effort_estimate" type="measure" />
        </graph>  
    </field>
</record> 
```
El elemento “graph” tiene el atributo “type” fijado a “pivot”. También puede ser “bar” (predeterminado), “pie” o “line”. En el caso que sea “bar”, gráfico de barras, adicionalmente se puede usar `stacked=”True”` para hacer un gráfico de barras apilado.

“graph” debería contener campos que pueden tener estos posibles atributos: 

- name: Identifica el campo que será usado en el gráfico, así como en otras vistas.
- type: Describe como será usado el campo, como un grupo de filas (predeterminado), “row”, como un grupo de columnas, “col”, o como una medida, “mesure”.
- interval: Solo es significativo para los campos de fecha, es un intervalo de tiempo para agrupar datos de fecha por “day”, “week”, “month”, “quarter” o “year”.
 

**Resumen**

Aprendió más sobre las vistas e Odoo que son usadas para la construcción de la interfaz. Comenzamos agregando opciones de menú y acciones de ventana usadas para abrir las vistas. Fueron explicados en detalle los conceptos de contexto y dominio.

También aprendió como diseñar vistas de lista y configurar opciones de búsqueda usando las vistas de búsqueda. Luego, se describieron de modo general los otros tipos de vista disponibles: calendario, Gantt y gráfico. Las vistas Kanban será estudiadas mas adelante, cuando aprenda como usar Qweb.

Ya hemos vistos los modelos y las vistas. En el próximo capítulo, aprenderá como implementar la lógica de negocio del lado del servidor.
