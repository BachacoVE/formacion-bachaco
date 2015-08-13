Capítulo 8. QweB -  Creando vistas Kanban y Reportes
=====

**QWeb** es un motor (engine) de plantillas por odoo. Está basado en XML y es utilizado para generar fragmentos y páginas html. QWeb fue introducido por primera vez en la versión 7.0 para habilitar vistas kanban más ricas, y con las versión 8.0, también se usa para la generación de reportes y páginas web CMS (CMS: Sistemas Manejadores de Contenido).

Aquí aprenderás acerca de la sintaxis QWeb y como usarla para crear tus propias vistas kanban reportes personalizados.

Para entender los tableros kanban, **kanban** es una palabra de origen japonés que es usada para representar un método de gestión de colas (queue) de trabajo. Fue inspirado del Sistema de Producción y Fabricación Ligera (lean) de Toyota, y se ha vuelto popular en la la industria del software con su adopción en las metodologías Ágiles.

El **tablero kanban** es una herramienta para visualizar la cola de trabajo. Los artículos (items) de trabajo están representados por tarjetas que son organizadas en columnas represantando las **etapas** (stages) del proceso de trabajo. Nuevos artículos de trabajo inician en la columna más a la izquierda y viaja a través del tablero hasta que alcanzan la columna más a la derecha, representando el trabajo completado.

**Iniciándose con el tablero kanban**

La simplicidad y el impacto visual del tablero kanban los hace excelente para soportar procesos de negocio simples. Un ejemplo básico de un tablero kanban puede tener tres columnas, como se muestra en la siguiente imagen: “ToDo”, “Doing” y “Done” (Por hacer, haciendo y hecho), pero, por supuesto puede ser extendido a cualquier paso de un proceso específico que necesitemos:


![280_1](/images/Odoo Development Essentials - Daniel Reis-280_1.jpg)

Las vistas kanban una característica distintiva de Odoo, haciendo fácil implementar estos tableros. Aprendamos cómo usarlos.


**Kanban views**

En las vistas de formulario, usamos mayormente elementos XML específicos, tales como `<field>` y `<group>`, y algunos elementos HTML, tales como `<h1>` o `<div>`. Con las vistas kanban, es un poco lo opuesto; ellas son plantillas basadas en HTML y soportan solo dos elementos específicos de Odoo, `<field>` y `<button>`. 

El HTML puede ser generado dinámicamente usando el motor de plantilla Qweb. Éste procesa los atributos de etiqueta especiales en los elementos HTML para producir el HTML final para ser presentado por el cliente web. Esto proporciona mucho control sobre cómo renderizar el contenido, pero también permite hacer diseños de vistas más complejas.

Las vistas kanban son tan flexibles que pueden haber muchas formas diferentes de diseñarlas, y puede ser difícil proveer una receta para seguir. Una buena regla general es encontrar un vista kanban existente similar a lo que queremos alcanzar, y crear nuestro nuevo trabajo de vista kanban basada en ella.

Observando las vistas kanban usadas en los módulos estándar, es posible identificar dos estilos de vistas kanban principales: viñeta y tarjeta

Ejemplos de las vistas kanban de estilo **viñeta** pueden ser encontrados en **Clientes, Productos**, y también, **Aplicaciones y Módulos**. Ellos usualmente no tienen borde y son decorados con imágenes en el lado de la izquierda, tal como se muestra en la siguiente imagen:

![281_1](/images/Odoo Development Essentials - Daniel Reis-281_1.jpg)

El estilo kanban **tarjeta** es usualmente usada para mostrar tarjetas organizadas en columnas para las etapas de procesos. Ejemplo de esto son las **Oportunidades CRM y las Tareas de Proyectos**. El contenido principal es mostrado en el área superior de la tarjeta y la información adicional puede ser mostrada en las áreas inferior derecha e inferior izquierda, tal como se muestra en la siguiente imagen:

![281_2](/images/Odoo Development Essentials - Daniel Reis-281_2.jpg)

Veremos el esqueleto y elementos típicos usados en ambos estilos de vistas tal que puedas sentirte cómodo adaptándolos a tus casos de usos particular.

**Diseña vistas kanban**

La primera cosa es crear un nuevo módulo agregando nuestras vistas kanban a la lista de tareas por hacer. En un trabajo del mundo real, una situación de uso de un módulo para esto podría ser, probablemente, excesiva y ellas podrían ser perfectamente agregadas directamente en el módulo todo_ui. Pero para una explicación más clara, usaremos un nuevo módulo y evitaremos demasiados, y posiblemente confusos, cambios en archivos ya creados. Lo nombraremos todo_kanban y crearemos los archivos iniciales tal como sigue:


```
$ cd ~/odoo-dev/custom-addons
$ mkdir todo_kanban 
$ touch todo_kanban/__init__.py
```

Ahora, edita el archivo descriptor todo_kanban/__opernerp__.py tal como sigue:

```
{'name': 'To-Do Kanban',
'description': 'Kanban board for to-do tasks.',
'author': 'Daniel Reis', 
'depends': ['todo_ui'],
'data': ['todo_view.xml'] }
```


Next, create the XML file where our shiny new kanban views will go and set kanban as the default view on the to-do task 

Lo que sigue es crear el archivo XML donde nuestras nuevas y brillantes vistas kanban irán y establecer kanban como la vista por defecto en la acción (action) de ventana de las tareas por hacer (to-do tasks), tal como se muestre a continuación:

```
<?xml version=”1.0”?>
<openerp>
    <data>
        <!-- Agrega el modo de vista kanban al menu Action: -->
    <act_window id=”todo_app.action_todo_task” name=”To-Do Tasks”  res_model=”todo.task” view_mode=”kanban,tree,form,calendar,gantt,graph” context=”{'search_default_filter_my_tasks':True}” />
        <!-- Agregar vista kanban -->
          <record id=”To-do Task Kanban” model=”ir.ui.view”>
            <field name=”name”>To-do Task Kanban</field>
            <field name=”model”>todo.task</field>
            <field name=”arch” type=”xml”>
               <!-- vacío por ahora, pero el Kanban irá aquí! -->
            </field>
         </record></data>
</openerp>
```

Ahora tenemos ubicado el esqueleto básico para nuestro módulo. Las plantillas usada en las vistas kanban y los reportes son extendidos usando las ténicas regulares usadas para otras vistas, por ejemplos usandos expresiones XPATH. Para más detalles, ve al [Capítulo 3](capitulo_iii_herencia.md), Herencia – Extendiendo Aplicaciones Existentes.

Antes de iniciar con las vistas kanban, necesitamos agregar un para de campos en el modelo tareas por hacer. (to-do tasks model)

**Prioridad y estado (state) kanban**

Los dos campos que son frecuentemente usados en las vistas kanban son: priority y kanban state. **Priority** permite a los usuarios organizar sus elementos de trabajo, señalando lo que debería estar ubicado primero. **Kanban state** señala cuando una tarea está lista para pasar a la siguiente etapa o si es bloqueada por alguna razón. Ambos son soportados por campos selection y tienen widgets específicos para ser usados en las vistas de formulario y kanban.

Para agrega estos campos a nuestro modelo, agregaremos al archivo todo_kanban/todo_task.py, tal como se muestra a continuación:

```
from openerp import models, fields
    class TodoTask(models.Model):
        _inherit = 'todo.task'
        priority = fields.Selection([('0','Low'),('1','Normal'),('2','High')],'Priority',default='1')
        kanban_state = fields.Selection([('normal', 'In Progress'),('blocked', 'Blocked'),('done', 'Ready for next stage')], 'Kanban State', default='normal')
```


No olvidemos el archivo todo_kanban/__init__.py que cargará el código precedente:

`from . import todo model`

Elementos de la vista kanban

La arquitectura de la vista kanban tiene un elemento superior <kanban> y la siguiente estructura básica:

```
<kanban> 
    <!-- Fields to use in expressions... --> 
    <field name="a_field" /> 
    <templates> 
        <t   t-name="kanban-box">
               <!-- HTML Qweb template	... --> 
        </t> 
    </templates> 
</kanban> 
```

El elemento <templates> contiene las plantillas para los fragmentos HTML a usar —uno o más. La plantilla principal a ser usada debe ser nombrada kanban-box. Otras plantillas son permitidas para fragmentos HTML para se incluido en la plantilla principal.

Las plantillas usan html estandar, pero pueden incluir etiquetas `<field>` para insertar campos del modelo. También pueden ser usadas algunas directivas especiales de Qweb para la generación dinámica de contenido, tal como el t-name usado en el ejemplo previo.

Todos los campos del modelo usados deben ser declarados con una etiqueta `<field>`. Si ellos son usados solo en expresiones, tenemos que declararlos antes de la sección `<templates>`. Uno de esos campos se le permite tener un valor agregado, mostrado en en el área superior de las columnas kanban. Esto se logra mediante la adición de un atributo con la agregación a usar, por ejemplo:

`<field name="effort_estimated" sum="Total Effort" />`

Aquí, la suma para el campo de estimación de esfuerzo es presentada en el área superios de las columnas kanban con la etiqueta Total Effort. Las agregaciones soportadas son sum, avg, min, max y count.

El elemento superior <kanban> también soporta algunos atributos interesantes:

- default_group_by: Establece el campo a usar para la agrupación por defecto de columnas
- default_order: Establece un orden por defecto para usarse en los elementos kanban
- quick_create=”false”: Deshabilita la opción de creación rápida en la vista kanban
- class: Añade una clase CSS al elemento raíz en la vista kanban renderizada.

Ahora démosle una mirada más de cerca a las plantillas Qweb usadas en las vistas kanban.

La vista kanban viñeta

Para las plantilas QWeb de las viñetas kanban, el esqueleto se ve así:

```
<t t-name="kanban-box"> 
    <div class="oe_kanban_vignette"> 
        <!-- Left side image:--> 
        <img class="oe_kanban_image" name="..." /> 
            <div class="oe_kanban_details"> 
                <!-- Title and data --> 
                <h4>Title</h4>
                <br>Other data <br/> 
                <ul>
                     <li>More data</li> 
                </ul> 
           </div> 
    </div> 
</t> 
```

Puedes ver las dos clases CSS principales provistas para los kanban de estilo viñeta:  oe_kanban_vignette para el contenedor superior y oe_kanban_details para el contenido de datos.

La vista completa de viñenta kanban para las tareas por hacer es como sigue:

```
<kanban> 
    <templates> 
        <t t-name="kanban-box"> 
           <div class="oe_kanban_vignette"> 
              <img t-att-src="kanban_image('res.partner', 'image_medium', record.id.value)" class="oe_kanban_image"/> 
                <div class="oe_kanban_details"> 
                    <!--	Title	and	Data	content	--> 
                    <h4><a type="open"> 
                        <field name="name"/> </a></h4> 
                        <field name="tags" /> 
                           <ul> 
                              <li><field name="user_id" /></li> 
                               <li><field name="date_deadline"/></li> 
                            </ul> 
                        <field name="kanban_state" widget="kanban_state_selection"/> 
                        <field name="priority" widget="priority"/> 
                </div> 
            </div> 
        </t> 
    </templates> 
</kanban> 
```

Podemos ver los elementos discutidos hasta ahora, y también algunos nuevos. En la etiqueta <img>, tenemos el atributo QWeb especial t-att-src. Esto puede calcular el contenido src de la imagen desde un campo almacenado en la base de datos. Explicaremos esto en otras directivas QWeb en un momento. También podemos ver el uso del atributo especial type en la etiqueta `<a>`. Echémosle un vistazo más de cerca.

**Acciones en las vistas kanban**

En las plantillas Qweb, la etiqueta <a> para enlaces puede tener un atributo type.  Este establece el tipo de acción que el enlace ejecutará para que los enlaces puedan actuar como los botones en los formularios regulares. En adición a los elementos `<button>`, las etiquetas `<a>` también pueden ser usadas para ejecutar acciones Odoo.

Así como en las vistas de formulario, el tipo de acción puede ser acción u objeto, y debería ser acompañado por atributo nombre, que identifique la acción específica a ejecutar. Adicionalmente, los siguentes tipos de acción también están disponibles:

- open: Abre la vista formulario correspondiente
- edit: Abre la vista formulario correspondiente directamente en el modo de edición
- delete: Elimina el regitro y remueve el elemento de la vista kanban.

**La vista kanban de tarjeta**
El kanban de **tarjeta** puede ser un poco más complejo. Este tiene un área de contenido principal y dos sub-contenedores al pie, alineados a cada lado de la tarjeta. También podría contener un botń de apertura de una acción de menú en la esquina superior derecha de la tarjeta.

El esqueleto para esta plantilla se vería así:

<t	t-name="kanban-box">
				<div	class="oe_kanban_card">
								<div	class="oe_dropdown_kanban	oe_dropdown_toggle">
													<!--	Top-right	drop	down	menu	-->
											</div>
								<div	class="oe_kanban_content">
												<!--	Content	fields	go	here...	-->
												<div	class="oe_kanban_bottom_right"></div>
												<div	class="oe_kanban_footer_left"></div>
								</div>
				</div>
</t>




```
   <div class="oe_kanban_bottom_right"></div>             <div class="oe_kanban_footer_left"></div>         </div>     </div> </t> 
A card  kanban is more appropriate for the to-do tasks, so instead of the view described in the previous section, we would be better using the following: 
<t t-name="kanban-box">     <div class="oe_kanban_card">         <div class="oe_kanban_content">             <!-- Option menu will go here! -->             <h4><a type="open">                 <field name="name" />             </a></h4>             <field name="tags" />             <ul>                 <li><field name="user_id" /></li>                 <li><field name="date_deadline" /></li>             </ul>             <div class="oe_kanban_bottom_right">                 <field name="kanban_state"                        widget="kanban_state_selection"/>             </div>             <div class="oe_kanban_footer_left">                 <field name="priority" widget="priority"/>             </div>         </div>     </div> </t> 
```
So far we have seen static kanban views, using a combination of HTML and special tags (field,  button, a). But we can have much more interesting results using dynamically generated HTML content. Lets display (the DOM). 

This is not meant to be technically exact. It is just a mind map that can be useful to understand how things work in kanban views. 

Next we will explore the several QWeb directives available, using examples that enhance our to-do task kanban card. 


**Conditional rendering with t-if**

The t-if directive, used in the previous example, accepts a JavaScript expression to be evaluated. The tag and its content will be rendered if the condition evaluates to true. 

For example, in the card kanban, to display the Task effort estimate, only if it has a value, after the date_deadline field, add the following: 
```
<t t-if="record.effort_estimate.raw_value > 0"><li>Estimate <field  name="effort_estimate"/></li></t> 
```
The JavaScript evaluation context has a record object representing the record being rendered, with the fields requested from the server. The field values can be accessed using either the raw_value or the value attributes: 

- raw_value: This is the value returned by the read() server method, so itt be able to go into that detail. For reference purposes, the following identifiers are available in QWeb expression evaluation: 
- widget: This is a reference to the current KanbanRecord widget object, responsible for the rendering of the current record into a kanban card. It exposes some useful helper functions we can use. 
- record: This is a shortcut for widget.records and provides access to the fields available, using dot notation. 
- read_only_mode: This indicates if the current view is in read mode (and not in edit mode). It is a shortcut for widget.view.options.read_only_mode.
- instance: This is a reference to the full web client instance.

It is also noteworthy that some characters are not allowed inside expressions. The lower than sign (*<*) is such a case. You may use a negated *>=* instead. Anyway, alternative symbols are available for inequality operations as follows: 

- lt: This is for less than. 
- lte: This is for less than or equal to. 
- gt: This is for greater than. 
- gte: This is for greater than or equal to.


**Rendering values with t-esc and t-raw**

We have used the <field> element to render the field content. But field values can also be presented directly without a <field> tag. The t-esc directive evaluates an expression and renders its HTML escaped value, as shown in the following: 
```
<t t-esc="record.message_follower_ids.raw_value" /> 
```
In some cases, and if the source data is ensured to be safe, t-raw can be used to render the field raw value, without any escaping, as shown in the following code: 
```
<t t-raw="record.message_follower_ids.raw_value" /> 
```

**Loop rendering with t-foreach**

A block of HTML can be repeated by iterating through a loop. We can use it to add the avatars of the task followers to the tasks start by rendering just the Partner IDs of the task, as follows: 
```
<t t-foreach="record.message_follower_ids.raw_value" t-as="rec">     <t t-esc="rec" />; </t> 
```
The t-foreach directive accepts a JavaScript expression evaluating to a collection to iterate. In most cases, this will be just the name of a *to many* relation field. It is used with a  t-as directive to set the name to be used to refer to each item in the iteration. 

In the previous example, we loop through the task followers, stored in the  message_follower_ids field. Since there is limited space on the kanban card, we could have used the slice() JavaScript function to limit the number of followers to display, as shown in the following: 
```
t-foreach="record.message_follower_ids.raw_value.slice(0, 3)" 
```
The rec variable holds each iterations avatar stored in the database. Kanban views provide a helper function to conveniently generate that: kanban_image(). It accepts as arguments the model name, the field name holding the image we want, and the ID for the record to retrieve. 

With this, we can rewrite the followers loop as follows: 
```
<div>   <t t-foreach="record.message_follower_ids.raw_value.slice(0, 3)"      t-as="rec">     <img t-att-src="kanban_image(                       'res.partner', 'image_small', rec)"          class="oe_kanban_image oe_kanban_avatar_smallbox"/>   </t> </div> 
```
We used it for the src attribute, but any attribute can be dynamically generated with a `t-  att-` prefix. 

String substitution in attributes with `t-attf-` prefixes.
  
Another way to dynamically generate tag attributes is using string substitution. This is helpful to have parts of larger strings generated dynamically, such as a URL address or CSS class names. 

The directive contains expression blocks that will be evaluated and replaced by the result. These are delimited either by {{ and }} or by #{ and }. The content of the blocks can be any valid JavaScript expression and can use any of the variables available for QWeb expressions, such as record and widget. 

Now lets rework it to use a sub-template. We should start by adding another template to our XML file, inside the <templates> element, after the `<t t-name="kanban-box">` node, as shown in the following: 
```
<t t-name="follower_avatars"> <div>   <t t-foreach="record.message_follower_ids.raw_value.slice(0, 3)"      t-as="rec">     <img t-att-src="kanban_image(          'res.partner', 'image_small', rec)"          class="oe_kanban_image oe_kanban_avatar_smallbox"/>   </t> </div> </t> 
```
Calling it from the kanban-box main template is quite straightforwardfor eacht exist in the caller3s value when performing the sub-template call as follows: 
```
<t t-call="follower_avatars">     <t t-set="arg_max" t-value="3" /> </t> 
```
The entire content inside the t-call element is also available to the sub-template through the magic variable 0. Instead of the argument variables, we can define an HTML code fragment that could be inserted in the sub-template using `<t t-raw="0" />`. 


**Other QWeb directives**

We have gone through through the most important Qweb directives, but there are a few more we should be aware of. Weve seen the basics about kanban views and QWeb templates. There are still a few techniques we can use to bring a richer user experience to our kanban cards. 


**Adding a kanban card option menu**

Kanban cards can have an option menu, placed at the top right. Usual actions are to edit or delete the record, but any action callable from a button is possible. There is also available a widget to set the card
```
 </a></li>         </t>         <t t-if="widget.view.is_action_enabled('delete')">         <li><a type="delete">Delete</a></li>         </t>         <!-- Color picker option: -->         <li><ul class="oe_kanban_colorpicker"                 data-field="color"/></li>     </ul> </div> 
```
It is basically an HTML list of <a> elements. The Edit  and Delete  options use QWeb to make them visible only when their actions are enabled on the view. The 
widget.view.is_action_enabled function allows us to inspect if the edit and delete actions are available and to decide what to make available to the current user. 


**Adding colors to kanban cards**

The color picker option allows the user to choose the color of a kanban card. The color is stored in a model field as a numeric index. 

We should start by adding this field to the to-do task model, by adding to `todo_kanban/todo_model.py` the following line: 
```
    color = fields.Integer('Color Index') 
```
Here we used the usual name for the field, color, and this is what is expected in the data-  field attribute on the color picker. 

Next, for the colors selected with the picker to have any effect on the card, we must add some dynamic CSS based on the color field value. On the kanban view, just before the  <templates> tag, we must also declare the color field, as shown in the following: 
```
<field name="color" /> 
```
And, we need to replace the kanban card top element, <div class="oe_kanban_card">, with the following: 
```
<div t-attf-class="oe_kanban_card                    #{kanban_color(record.color.raw_value)}"> 
```
The kanban_color helper function does the translation of the color index into the corresponding CSS class name. 

And that). A helper function for this is available in kanban views. 

For example, to limit our to-do task titles to the first 32 characters, we should replace the 
<field name="name" /> element with the following: 
```
<t t-esc="kanban_text_ellipsis(record.name.value, 32)" /> 
```

**Custom CSS and JavaScript assets**

As we have seen, kanban views are mostly HTML and make heavy use of CSS classes. We have been introducing some frequently used CSS classes provided by the standard product. But for best results, modules can also add their own CSS. 

We are not going into details here on how to write CSS, but itt work, since we havenWebkit HTML to PDF.s probably not what you will get now on your system. Lett display the You need Wkhtmltopdf to print a pdf version of the reports time library  

- user: This is the record for the user running the report 
- res_company: This is the record for the current user Designing the User Interface*, with an additional widget to set the widget to use to render the field. 

A common example is a monetary field, as shown in the following: 
```
<span t-field="o.amount"       t-field-options='{         "widget": "monetary",         "display_currency": "o.pricelist_id.currency_id"}'/> 
```
A more sophisticated case is the contact widget, used to format addresses, as shown in the following: 
```
<div t-field="res_company.partner_id" t-field-options='{        "widget": "contact",        "fields": ["address", "name", "phone", "fax"],        "no_marker": true}' /> 
```
By default, some pictograms, such as a phone, are displayed in the address. The  no_marker="true" option disables them. 


**Enabling language translation in reports**

A helper function, translate_doc(), is available to dynamically translate the report content to a specific language. 

It needs the name of the field where the language to use can be found. This will frequently be the Partner the document is to be sent to, usually stored at partner_id.lang. In our case, we dons also a less efficient method. 

If you cans growing in importance in the Odoo toolset. Finally, you had an overview on how to create reports, also using the QWeb engine. 

In the next chapter, we will explore how to leverage the RPC API to interact with Odoo from external applications. 
