Capítulo 5.	Modelos – Estructura de los Datos de la Aplicación
===

En los capítulos anteriores, vimon un resumen de extremo a extremo sobre la creación de módulos nuevos para Odoo. En el Capítulo 2, se construyo una aplicación totalmente nueva, y en el Capítulo 3, exploramos la herencia y como usarla para crear un módulo de extensión para nuestra aplicación. En el Capítulo 4, discutimos como agregar datos iniciales y de demostración a nuestros módulos.

En estos resúmenes, tocamos todas las capas que componen el desarrollo de aplicaciones “backend” para Odoo. Ahora, en los siguientes capítulos, es hora de explicar con más detalle todas estas capas que conforman una aplicación: modelos, vistas, y lógica de negocio.

En este capítulo, aprenderá como diseñar las estructuras de datos que soportan una aplicación, y como representar las relaciones entre ellas.


**Organizar las características de las aplicaciones en módulos**

Como hicimos anteriormente, usaremos un ejemplo para ayudar a explicar los conceptos. Una de las mejores cosas de Odoo es tener la capacidad de tomar una aplicación o módulo existente y agregar, sobre este, las características que necesite. Así que continuaremos mejorando nuestros módulos to-do, y pronto formaran una aplicación completa!

Es una buena práctica dividir las aplicaciones Odoo en varios módulos pequeños, cada uno responsable de una característica específica. Esto reduce la complejidad general y hace el mantenimiento y la actualización más fácil.

El problema de tener que instalar todos esos módulos individuales puede ser resuelto proporcionando un módulo de la aplicación que empaquete todas esas características, a través de sus dependencias. Para ilustrar este enfoque implementaremos las características adicionales usando módulos to-do nuevos.


**Introducción al modulo todo_ui**

En el capítulo anterior, primero creamos una aplicación para tareas por hacer personales, y luego la ampliamos para que las tareas por hacer pudieran ser compartidas con otras personas.

Ahora queremos llevar a nuestra aplicación a otro nivel agregandole una pizarra kanban y otras mejoras en la interfaz. La pizarra kanban nos permitirá organizar las tareas en columnas, de acuerdo a sus estados, como En Espera, Lista, Iniciada o Culminada.

Comenzaremos agregando la estructura de datos para permitir esa visión. Necesitamos agregar los estados y sería bueno si añadimos soporte para las etiquetas, permitiendo que las tareas estén organizadas por categoría.

La primera cosa que tenemos que comprender es como nuestra data estará estructurada para que podamos diseñar los Modelos que la soportan. Ya tenemos la entidad central: las tareas por hacer. Cada tarea estará en un estado, y las tareas pueden tener una o más etiquetas. Esto significa que necesitaremos agregar dos modelos adicionales, y tendrán estas relaciones:

- Cada tarea tiene un estado, y puede haber muchas tareas en un estado.
- Cada tarea puede tener muchas etiquetas, y cada etiqueta puede estar en muchas tareas.

Esto significa que las tareas tiene una relación muchos a uno con los estados, y una relación muchos a muchos con las etiquetas. Por otra parte, las relaciones inversas son: los estados tiene una relación uno a muchos con las tareas y las etiquetas tienen una relación muchos a muchos con las tareas.

Comenzaremos creando el módulo nuevo todo_ui y agregaremos los estados y los modelos de etiquetas.

Hemos estado usado el directorio `~/odoo-dev/custom-addons/` para alojar nuestros módulos. Para crear el módulo nuevo junto a los existentes, podemos usar estos comandos en la terminal:
```
$ cd ~/odoo-dev/custom-addons 
$ mkdir todo_ui 
$ cd todo_ui 
$ touch todo_model.py 
$ echo "from . Import todo_model" > __init__.py
```  
Luego, debemos editar el archivo manifiesto `__openerp__.py` con este contenido:
```
{ 
   'name': 'User interface improvements to the To-Do app',
   'description': 'User friendly features.',
   'author': 'Daniel Reis',
   'depends': ['todo_app']	
}
``` 
Note que dependemos de todo_app y no de todo_user. En general, es buena idea mantener los módulos tan independientes como sea posible. Cuando un módulo aguas arriba es modificado, puede impactar todos los demás módulos que directa o indirectamente dependen de el. Es mejor si podemos mantener al mínimo el número de dependencias, e igualmente evitar concentrar un gran número de dependencias, como: `todo_ui → todo_user → todo_app` en este caso.

Ahora podemos instalar el módulo en nuestra base de datos de trabajo y comenzar con los modelos.


**Crear modelos**

Para que las tareas por hacer tengan una pizarra kanban, necesitamos estados. Los estados son columnas de la pizarra, y cada tarea se ajustará a una de esas columnas.

Agreguemos el siguiente código al archivo `todo_ui/todo_model.py`:
```
#-*- coding: utf-8 -*- 
from openerp import models, fields, api 

class	Tag(models.Model):
    _name = 'todo.task.tag'
    name = fields.Char('Name', 40, translate=True) 

class	Stage(models.Model):
    _name = 'todo.task.stage'
    _order = 'sequence,name'
    _rec_name = 'name' 	# the	default
    _table = 'todo_task_stage' #the	default
    name = fields.Char('Name', 40, translate=True)
    sequence = fields.Integer('Sequence') 
```
Aquí, creamos los dos Modelos nuevos a los cuales haremos referencia en las tareas por hacer.

Enfocándonos en los estados de las tareas, tenemos una clase Python, Stage, basada en la clase `models.Model`, que define un modelo nuevo, `todo.task.stage`. También definimos dos campos, “name” y “sequence”. Podemos ver algunos atributos del modelo, (con la barra baja, `_`, como prefijo) esto es nuevo para nosotros. Demos le una mirada más profunda.

**Atributos del Modelo**

Las clases del modelo pueden tener atributos adicionales usados para controlar alguno de sus comportamientos:

- `_name`: Este es el identificador interno para el modelo que estamos creando. 
- `_order`: Este fija el orden que será usado cuando se navega por los registros del modelo. Es una cadena de texto que es usada como una clausula SQL “order by”, así que puede ser cualquier cosa permitida.
- `_rec_name`: Este indica el campo a usar como descripción del registro cuando se hace referencia a él desde campos relacionados, como una relación muchos a uno. De forma predeterminada usa el campo de nombre, el cual esta frecuentemente presente en los modelos. Pero este atributo nos permite usar cualquier otro campo para este propósito.
- `_table`: Este es el nombre de la tabla de la base de datos que soporta el modelo. Usualmente, se deja para que sea calculado automáticamente, y es el nombre del modelo con el carácter de piso bajo (`_`) que reemplaza a los puntos. Pero puede ser configurado para indicar un nombre de tabla específico.

Para completar, también podemos tener atributos `inherit` y `_inherits`, como explicamos en el Capítulo 3.
 

**Modelos y Clases Python**

Los modelos de Odoo son representados por las clases Python. En el código precedente, tenemos una clase Python llamada Stage, basada en la clase `models.Model`, usada para definir el modelo nuevo `todo.task.stage`.

Los modelos de Odoo son mantenidos en un registro central, también denominado como piscina en las versiones anteriores. Es un diccionario que mantiene las referencias de todas las clases de modelos disponibles en la instancia, a las cuales se les puede hacer referencia por el nombre del modelo. Específicamente, el código en un método del modelo puede usar `self.env['x]` o `self.env.get('x')` para obtener la referencia a la clase que representa el modelo x.

Puede observar que los nombres del modelo son importantes ya que son la llave para acceder al registro. La convención para los nombres de modelo es usar una lista de palabras en minúscula unidas con puntos, como `todo.task.stage`. Otros ejemplos pueden verse en los módulos raíz de Odoo `project.project`, `project.task` o `project.task.type`. 

Debemos usar la forma singular: `todo.task` en vez de `todo.tasks`. Por cuestiones históricas se pueden encontrar módulos raíz, que no sigan dicha convención, como `res.users`, pero no es la norma.

Los nombres de modelo deben ser únicos. Debido a esto, la primera palabra deberá corresponder a la aplicación principal con la cual esta relacionada el módulo. En nuestro ejemplo, es “todo”. De los módulos raíz tenemos, por ejemplo, project, crm, o sale.

Por otra parte, las clases Python, son locales para el archivo Python en la cual son declaradas. El identificador usado en ellas es solo significante para el código en ese archivo.

Debido a esto, no se requiere que los identificadores de clase tengan como prefijo a la aplicación principal a la cual están relacionados. Por ejemplo, no hay problema en llamar simplemente Stage a nuestra clase para el modelo `todo.task.stage`. No hay riesgo de colisión con otras posibles clases con el mismo nombre en otros módulos.

Se pueden usar dos convenciones diferentes para los identificadores de clase: “snake_case” o “CamelCase”. Históricamente, el código Odoo ha usado el “snake_case”, y es aún muy frecuente encontrar clases que usan esa convención. Pero la tendencia actual en usar “CamelCase”, debido a que es el estándar definido para Python por la convenciones de codificación PEP8. Puede haber notado que estamos usando esta última forma. 


**Modelos Transitorios y Abstractos**

En el código precedente, y en la vasta mayoría de los modelos Odoo, las clases están basadas en el clase `models.Model`. Este tipo de modelos tienen bases de datos persistentes: las tablas de las bases de datos son creadas para ellos y sus registros son almacenados hasta que son borrados explícitamente.

Pero Odoo proporciono otros dos tipos de modelo: modelos Transitorios y Abstractos.

Los modelos transitorios están basados en la clase `models.TransientModel` y son usados para interacción con el usuario y la usuaria tipo asistente. Sus datos es aún almacenada en la base de datos, pero se espera que sea temporal. Un trabajo de aspiradora limpia periódicamente los datos viejos de esas tablas.

Los modelos abstractos están basados en la clase `models.AbstractModel` y no tienen  almacén vinculado a ellos. Actúan como una característica de re uso configurada para ser mezclada con otros modelos. Esto es hecho usando las capacidades de herencia de Odoo.
 

![185_1](/images/Odoo Development Essentials - Daniel Reis-185_1.jpg)


**Inspeccionar modelos existentes**

La información sobre los modelos y los campos creados con clases Python esta disponible a través de la interfaz. En el menú principal de Configuraciones, seleccione la opción de menú Técnico | Estructura de Base de Datos | Modelos. Allí, encontrará la lista de todos los modelos disponibles en la base de datos. Al hacer clic en un modelo de la lista se abrirá un formulario con sus detalles.

Esta es una buena herramienta para inspeccionar la estructura de un Modelo, ya que se tiene en un solo lugar el resultado de todas las adiciones que pueden venir de diferentes módulos. En este caso, como puede observar en el campos En Módulos, en la parte superior derecha, las definiciones de `todo.task` vienen de los módulos `todo_app` y `todo_user`.
En el área inferior, tenemos disponibles algunas etiquetas informativas: una referencia rápida de los Campos del modelo, los Derechos de Acceso concedidos, y también lista las Vistas disponibles para este modelo.

Podemos encontrar el Identificador Externo del modelo, activando el Menú de Desarrollador y accediendo a la opción Vista de Metadatos. Estos son generados automáticamente pero bastante predecibles: para el modelo `todo.task`, el Identificador Externo es `model_todo_task`.

*Tip*
* Los formularios del Modelo pueden ser editados! Es posible crear y modificar modelos, campos y vistas desde aquí. Puede usar esto para construir prototipos antes de colocarlos definitivamente dentro de los propios modelos. * 
 

**Crear campos**

Después de crear un modelo nuevo, el siguiente paso es agregar los campos. Vamos a explorar diferentes tipos de campos disponibles en Odoo.


**Tipos básicos de campos**

Ahora tenemos un modelo Stage y vamos a ampliarlo para agregar algunos campos adicionales. Debemos editar el archivo `todo_ui/todo_model.py`, removiendo algunos atributos innecesarios incluidos antes con propósitos descriptivos:
```
class	Stage(models.Model):
    _name  = 'todo.task.stage'
    _order = 'sequence,name'	

    # String fields:
    name  = fields.Char('Name',40)
    desc  =	fields.Text('Description')
    state =	fields.Selection([('draft','New'),('open','Started' ('done','Closed')],'State')
    docs  = fields.Html('Documentation')

    #	Numeric fields:
    sequence      = fields.Integer('Sequence')
    perc_complete = fields.Float('%	Complete',(3,2))
    
    # Date fields:
    date_effective = fields.Date('Effective Date')
    date_changed   = fields.Datetime('Last Changed')

    # Other	fields:
    fold  = fields.Boolean('Folded?')
    image =	fields.Binary('Image')
``` 
Aquí tenemos un ejemplo de tipos de campos no relacionales disponibles en Odoo, con los argumentos básicos esperados por cada función. Para la mayoría, el primer argumento es el título del campo, que corresponde al atributo palabra clave de cadena. Es un argumento opcional, pero se recomienda colocarlo. De lo contrario, sera generado automáticamente un título por el nombre del campo.

Existe una convención para los campos de fecha que usa “date” como prefijo para el nombre. Por ejemplo, deberíamos usar “date_effective” en vez de “effective_date”. Esto también puede aplicarse a otros campos, como “amount_”, “price_” o “qty_”.
Algunos otros argumentos están disponibles para la mayoría de los tipos de campo:

- Char, acepta un segundo argumento opcional, “size”, que corresponde al tamaño máximo del texto. Es recomendable usarlo solo si se tiene una buena razón.
- Text, se diferencia de Char en que puede albergar texto de varias líneas, pero espera los mismos argumentos.
- Selecction, es una lista de selección desplegable. El primer argumento es la lista de opciones seleccionables y el segundo es la cadena de título. La lista de selección es una tupla `('value', 'Title')` para el valor almacenado en la base de datos y la cadena de descripción correspondiente. Cuando se amplía a través de la herencia, el argumento “selection_add” puede ser usado para agregar opciones a la lista de selección existente.
- Html, es almacenado como un campo de texto, pero tiene un manejo específico para presentar el contenido HTML en la interfaz.
- Integer, solo espera un argumento de cadena de texto para el campo de título.
- Float, tiene un argumento opcional, una tupla `(x,y)` con los campos de precisión: x como el número total de dígitos; y representa los dígitos decimales.
- Date y Datetime, estos datos son almacenados en tiempo UTC. Se realizan conversiones automáticas, basadas en las preferencias del usuario o la usuaria, disponibles a través del contexto de la sesión de usuario. Esto es discutido con mayor detalle en el Capítulo 6.
- Boolean, solo espera sea fijado el campo de título, incluso si es opcional. Binary también espera este único argumento.

Además de estos, también existen los campos relacionales, los cuales serán introducidos en este mismo capítulo. Pero por ahora, hay mucho que aprender sobre los tipos de campos y sus atributos.


**Atributos de campo comunes**

Los campos también tienen un conjunto de atributos los cuales podemos usar, y los explicaremos aquí con más detalle:

- string, es el título del campo, usado como su etiqueta en la UI. La mayoría de las veces no es usado como palabra clave, ya que puede ser fijado como un argumento de posición.
- default, fija un valor predefinido para el campo. Puede ser un valor estático o uno fijado anticipadamente, pudiendo ser una referencia a una función o una expresión lambda.
- size, aplica solo para los campos Char, y pueden fijas el tamaño máximo permitido.
- translate, aplica para los campos de texto, Char, Text y Html, y hacen que los campos puedan ser traducidos: puede tener varios valores para diferentes idiomas.
- help, proporciona el texto de ayuda desplegable mostrado a los usuarios y usuarias.
- `readonly = True`, hace que el campo no pueda ser editado en la interfaz.
- `required = True`, hace que el campo sea obligatorio.
- `index = True`, creara un índice en la base de datos para el campo.
- `copy = False`, hace que el campo sea ignorado cuando se usa la función Copiar. Los campos no relacionados de forma predeterminada pueden ser copiados.
- groups, permite limitar la visibilidad y el acceso a los campos solo a determinados grupos. Es una lista de cadenas de texto separadas por comas, que contiene los ID XML del grupo de seguridad.
- states, espera un diccionario para los atributos de la UI dependiendo de los valores se estado del campo. Por ejemplo: `states={'done':[('readonly', True)]}`. Los atributos que pueden ser usados son, “readonly”, “required” y “invisible”.

Para completar, a veces se usando dos atributos más cuando se actualiza entre versiones principales de Odoo:

- `deprecated = True`, registra un mensaje de alerta en cualquier momento que el campo sea usado.
- `oldname = 'field'`, es usado cuando un campo es re nombrado en una versión nueva, permitiendo que la data en el campo viejo sea copiada automáticamente dentro del campo nuevo.


**Nombres de campo reservados**

Unos cuantos nombres de campo estan reservados para ser usados por el ORM:

- id, es un número generado automáticamente que identifica de forma única a cada registro, y es usado como clave primaria en la base de datos. Es agregado automáticamente a cada modelo.

Los siguientes campos son creados automáticamente en los modelos nuevos, a menos que el atributo `_log_access=False` sea fijado:

- `create_uid`, para el usuario que crea el registro.
- `created_date`, para la fecha y la hora en que el registro es creado.
- `write_uid`, para el último usuario que modifica el registro.
- `write_date`, para la última fecha y hora en que el registro fue modificado.

Esta información esta disponible desde el cliente web, usando el menú Modo Desarrollador y seleccionando la opción Vista de Metadatos.

Hay algunos efectos integrados que esperan nombres de campo específicos. Debemos evitar usarlos para otros propósitos que aquellos para los que fueron creados. Algunos de ellos incluso están reservados y no pueden ser usados para ningún otro propósito:

- name, es usado de forma predeterminada como el nombre del registro que será mostrado. Usualmente es un Char, pero se permiten otros tipos de campos. Puede ser sobre escrito configurando el atributo `_rec_name` del modelo.
- active (tipo Boolean), permite desactivar registros. Registro con `active==False` serán excluidos automáticamente de las consultas. Para acceder a ellos debe ser agregada la condición `('active','=', False)` al dominio de búsqueda o agregar `'active_test':False` al contexto actual.
- sequence (tipo Integer),, si esta presente en una vista de lista, permite definir manualmente el orden de los registros. Para funcionar correctamente debe estar también presente en el `_order` del modelo.
- state (tipo Selection), representa los estados básicos del ciclo de vida del registro, y puede ser usado por el atributo “field” del estado para modificar de forma dinámica la vista: algunos campos de formulario pueden ser solo lectura, requeridos o invisibles en estados específicos del registro. 
- `parent_id`, `parent_left`, y `parent_right`; tienen significado especial para las relaciones jerárquicas padre/hijo. En un momento las discutiremos con mayor detalle.

Hasta ahora hemos discutido los los valores escalares de los campos. Pero una buena parte de una estructura de datos de la aplicación es sobre la descripción de relaciones entre entidades. Veamos algo sobre esto ahora.


**Relaciones entre modelos**

Viendo nuestro diseño del módulo, tenemos estas relaciones:

- Cada tarea tiene un estado – esta es una relación muchos a uno, también conocida como una clave foránea. La relación inversa es de uno a muchos, que ssignifica que cada estado puede tener muchas tareas. 

- Cada tarea puede tener muchas etiquetas – esta es una relación muchos a muchos. La relación inversa , obviamente, es también una relación muchos a muchos, debido a que cada etiqueta puede también tener muchas tareas.

Agreguemos los campos de relación correspondientes al archivo `todo_ui/todo_model.py`: 
```
class	TodoTask(models.Model):
    _inherit = 'todo.task'
    stage_id = fields.Many2one('todo.task.stage', 'Stage')
    tag_ids = fields.Many2many('todo.task.tag', string='Tags')
```
El código anterior muestra la sintaxis básica para estos campos. Configurando el modelo relacionado y el campo de título. La convención para los nombres de campo relacionales es agregar a los nombres de campos `_id` o `_ids`, para las relaciones de uno y muchos, respectivamente.

Como ejercicio puede intentar agregar en los modelos relacionados, las relaciones inversas correspondientes: La relación inversa de Many2one es un campo One2many en los estados: cada estado puede tener muchas tareas. Deberíamos agregar este campo a la clase Stage. La relación inversa de Many2many es también un campo Many2many en las etiquetas: cada etiqueta puede ser usada en muchas tareas.

Veamos con mayor detalle las definiciones de los campos relacionales.


**Relaciones Muchos a uno**

Many2many, acepta dos argumentos de posición: el modelo relacionado (que corresponde al argumento de palabra clave del comodelo) y la cadena de título. Este crea un campo en la table de la base de datos con una clave foránea a la tabla relacionada.

Algunos nombres adicionales de argumentos también están disponibles para ser usados con estos tipos de campo:

- ondelete, define lo que pasa cuando el registro relacionado es eliminado. De forma predeterminada esta fijado como null, lo que significa que al ser eliminado el registro relacionado se fija a un valor vacío. Otros valores posibles son “restrict”, que arroja un error que previene la eliminación, y “cascade” que también elimina este registro.
- context y domain, son significativos para las vistas del cliente. Pueden ser configurados en el modelo para ser usados de forma predeterminada en cualquier vista donde sea usado el campo. Estos serán explicados con más detalle en el Capítulo 6.
- `auto_join = True`, permite que el ORM use uniones SQL haciendo búsquedas usando esta relación. De forma predeterminada esto esta fijado como False para reforzas las reglas de seguridad. Si son usadas uniones, las reglas de seguridad serán pasadas por alto, y el usuario podrá tener acceso a los registros relacionados que las reglas de seguridad no le permitirían, pero las consultas SQL serán más eficientes y se ejecutarán con mayor rapidez.


**Relaciones muchos a muchos**
La forma mas simple de la relación Many2many acepta un argumento para el modelo relacionado, y es recomendable también proporcionar el argumento de cadena con el título del campo.

En el nivel de base de datos, esto no agrega ninguna columna a las tablas existentes. Por el contrario, automáticamente crea una tabla nueva de relación de solo dos campos con las claves foráneas de las tablas relacionadas. El nombre de la tabla de relación es el nombre de ambas tablas unidos por un símbolo de guión bajo (`_`) con `_rel` anexado.

Estas configuración predeterminadas pueden del sobre escritas manualmente. Una forma de hacerlo es usar la forma larga para la definición del campo:
```
# TodoTask class: Task <-> Tag relation (long form): 
tag_ids = fields.Many2many( 'todo.task.tag', # related model
                            'todo_task_tag_rel', # relation table name
                            'task_id', # field for "this" record
                            'tag_id', #field for "other" record
                             string='Tasks')
``` 
Note que los argumentos adicionales son opcionales. Podemos simplemente fijar el nombre para la tabla de relación y dejar que los nombres de los campos usen la configuración predeterminada.

Si prefiere, puede usar la forma larga usando los argumentos de palabra clave:
```
# TodoTask class: Task	<-> Tag relation (long form): 
tag_ids = fields.Many2many(comodel_name='todo.task.tag', #related model
                           relation='todo_task_tag_rel', #relation able name
                           column1='task_id', # field for "this" record
                           column2='tag_id', # field for "other" record
                           string='Tasks') 
```
Como los campos muchos a uno, los campos muchos a muchos también soportan los atributos de palabra clave de dominio y contexto.

En algunas raras ocasiones tendremos que usar estas formas largas para sobre escribir las configuraciones automáticas predeterminadas, en particular, cuando los modelos relacionados tengan nombres largos o cuando necesitemos una segunda relación muchos a muchos entre los mismos modelos.

* Tip *
* Los nombres de las  tablas PostgreSQL tienen 63 caracteres como límite, y esto puede ser un problema su la tabla de relación generada automáticamente excede ese limite. Este es uno de los casos cuando tendremos que configurar manualmente el nombre de la tabla de relación usando el atributo “relation”.*

Lo inverso a la relación Many2many es también un campo Many2many. Si también agregamos un campo Many2many a las etiquetas, Odoo infiere que esta relación de muchos a muchos es la inversa a la del modelo de tareas.

La relación inversa entre tareas y etiquetas puede ser implementada así:
``` 
# class Tag(models.Model): #
    _name = 'todo.task.tag' 

    #Tag class relation to Tasks: 
    task_ids = fields.Many2many( 'todo.task', # related model string='Tasks') 
```


**Relaciones inversas de uno a muchos**

La inversa de Many2many puede ser agregada al otro extremo de la relación. Esto no tiene un impacto real en la estructura de la base de datos, pero nos permite navegar fácilmente desde “un” lado a “muchos” lados de los registros. Un caso típico es la relación entre un encabezado de un documento y sis líneas.

En nuestro ejemplo, con una relación inversa One2many en estados, fácilmente podemos listas todas las tareas que se encuentran en un estado. Para agregar esta relación inversa a los estados, agregue el código mostrado a continuación:
```
# class Stage(models.Model): #
    _name = 'todo.task.stage' 

    #Stage class relation with Tasks:
    tasks = fields.One2many('todo.task',# related model 'stage_id',#field for	
                             "this" on related model 'Tasks in this stage') 
```
One2many acepta tres argumentos de posición: el modelo relacionado, el nombre del campo en aquel modelo que referencia este registro, y la cadena de título. Los dos primeros corresponden a los argumentos `comodel_name` y `inverse_name`.

Los parámetros adicionales disponibles son los mismos que para el muchos a uno: contexto, dominio, ondelete (aquí actuá en el lado “muchos” de la relación), y `auto_join`.


**Relaciones jerárquicas**

Las relaciones padre-hijo pueden ser representadas usando una relación Many2one al mismo modelo, para dejar que cada registro haga referencia a su padre. Y la inversa One2many hace más fácil para un padre mantener el registro de sus hijos.

Odoo también provee soporte mejorado para estas estructuras de datos jerárquicas: navegación más rápida a través de árboles hermanos, y búsquedas más simples con el operador `child_of` en las expresiones de dominio.

Para habilitar esas características debemos configurar el atributo `_parent_store` y agregar los campos de ayuda: `parent_left` y `parent_right`. Tenga en cuenta que estas operaciones adicionales traen como consecuencia penalizaciones en materia de almacenamiento y ejecución, así que es mejor usarlo cuando se espere ejecutar más lecturas que escritura, como es el caso de un árbol de categorías.

Revisando el modelo de etiquetas definido en el archivo `todo_ui/todo_model.py`, ahora editaremos para que luzca así:
```
class	Tags(models.Model):
    _name         = 'todo.task.tag'
    _parent_store = True #
    _parent_name  = 'parent_id'
    name = fields.Char('Name')
    parent_id     = fields.Many2one('todo.task.tag','Parent Tag', ondelete='restrict')
    parent_left   = fields.Integer('Parent Left', index=True)
    parent_right  = fields.Integer('Parent	Right', index=True) 
```
Aquí tenemos un modelo básico, con un campos `parent_id` que referencia al registro padre, y el atributo adicional `_parent_store` para agregar soporte a búsquedas jerárquicas.  

Se espera que el campo que hace referencia al padre sea nombrado `parent_id`. Pero puede usarse cualquier otro nombre declarándolo con el atributo `_parent_name`. 

También, es conveniente agregar un campo con el hijo directo del registro:
```
child_ids = fields.One2many('todo.task.tag', 'parent_id', 'Child Tags') 
```

**Hacer referencia a campos usando relaciones dinámicas**

Hasta ahora, los campos de relación que hemos visto puede solamente hacer referencia a un modelo. El tipo de campo Reference no tiene esta limitación y admite relaciones dinámicas: el mismo campo es capaz de hacer referencia a más de un modelo.

Podemos usarlos para agregar un campo, “Refers to”, a Tareas por Hacer que pueda hacer referencia a un User o un Partner:
```
# class TodoTask(models.Model):
    refers_to = fields.Reference([('res.user', 'User'),('res.partner', 'Partner')], 'Refers to') 
```
Puede observar que la definición del campo es similar al campo Selection, pero aquí la lista de selección contiene los modelos que pueden ser usados. En la interfaz, el usuario o la usuaria seleccionará un modelo de la lista, y luego elegirá un registro de ese modelo.

Esto puede ser llevado a otro nivel de flexibilidad: existe una tabla de configuración de Modelos Referenciables para configurar los modelos que pueden ser usados en campos Reference. Esta disponible en el menú Configuraciones | Técnico | Estructura de Base de Datos. Cuando se crea un campo como este podemos ajustarlo para que use cualquier modelo registrado allí, con la ayuda de la función `referencable_models()` en el módulo `openerp.addons.res.res_request`. En la versión 8 de Odoo, todavía se usa la versión antigua de la API, así que necesitamos  empaquetar lo para usarlo con la API nueva:
```
from openerp.addons.base.res import res_request 

def	referencable_models(self):
    return res_request.referencable_model (self, self.env.cr, self.env.uid, context=self.env.context) 
```
Usando el código anterior, la versión revisada del campo “Refers to” sera así:
```
# class TodoTask(models.Model):
    refers_to = fields.Reference( referencable_models, 'Refers to') 
```


**Campos calculados**

Los campos pueden tener valores calculados por una función, en vez de simplemente leer un valor almacenado en una base de datos. Un campo calculado es declarado como un campo regular, pero tiene el argumento “compute” adicional con el nombre de la función que se usará para calcularlo.

En la mayoría de los casos los campos calculados involucran alguna lógica de negocio, por lo tanto este tema se desarrollara con más profundidad en el Capítulo 7. Igual podemos explicarlo aquí, pero maneteniendo la lógica de negocio lo más simple posible.

Trabajamos en un ejemplo: los estados tienen un campo “fold”. Agregaremos a las tareas un campo calculado con la marca “Folded?” para el estado correspondiente.

Debemos editar el modelo TodoTask en el archivo `todo_ui/todo_model.py` para agregar lo siguiente:
```
# class TodoTask(models.Model):
    stage_fold = fields.Boolean('Stage Folded?', compute='_compute_stage_fold')
    @api.one @api.depends('stage_id.fold') 

def _compute_stage_fold(self):
    self.stage_fold = self.stage_id.fold 
```
El código anterior agrega un campo nuevo `stage_fold` y el método `_compute_stage_fold` que sera usado para calcular el campo. El nombre de la función es pasado como una cadena, pero también es posible pasarla como una referencia obligatoria (el identificador de la función son comillas).

Debido a que estamos usando el decorador `@api.one`, self tendrá un solo registro. Si en vez de esto usamos `@api.multi`, representara un conjunto de registros y nuestro código necesitará gestionar la iteración sobre cada registro.

El `@api.depends` es necesario si el calculo usa otros campos: le dice al servidor cuando re calcular valores almacenados o en cache. Este acepta uno o mas nombres de campo como argumento y la notación de puntos puede ser usada para seguir las relaciones de campo.

Se espera que la función de calculo asigne un valor al campo o campos a calcular. Si no lo hace, arrojara un error. Debido a que self es un objeto de registro, nuestro calculo es simplemente para obtener el campo “Folded?” usando `self.stage_id.fold`. El resultado es conseguido asignando ese valor (escribiéndolo) en el campo calculado, `self.stage_fold`.

No trabajaremos aún en las vistas para este módulo, pero puede hacer una edición rápida al formulario de tareas para confirmar si el campo calculado esta funcionando como es esperado: usando el Menú Desarrollador escoja la opción Edición de Vista y agregue el campo directamente en el XML del formulario. No se preocupe: será reemplazado por una vista limpia del módulo en la próxima actualización.


**Buscar y escribir en campos calculados**

El campo calculado que acabamos de crear puede ser leído, pero no se puede realizar una búsqueda ni escribir en el. Esto puede ser habilitado proporcionando funciones especiales para esto. A lo largo de la función de calculo también podemos colocar una función de búsqueda, que implemente la lógica de búsqueda, y la función inversa, que implemente la lógica de escritura.

Para hacer esto, nuestra declaración de campo calculado se convertirá en esto:
```
# class TodoTask(models.Model):
	stage_fold = fields.Boolean
        string   = 'Stage Folded?', 								
        compute  ='_compute_stage_fold', # store=False) # the default 			 
        search   ='_search_stage_fold', 								
        inverse  ='_write_stage_fold') 
```
Las funciones soportadas son:
```
def _search_stage_fold(self, operator, value):
    return [('stage_id.fold', operator, value)] 

def	_write_stage_fold(self):
    self.stage_id.fold = self.stage_fold 
```
La función de búsqueda es llamada en cuando en encontrada en este campo una condición `(field, operator, value)` dentro de una expresión de dominio de búsqueda.

La función inversa realiza la lógica reversa del cálculo, para hallar el valor que sera escrito en el campo de origen. En nuestro ejemplo, es solo escribir en `stage_id.fold`.


**Guardar campos calculados*

Los valores de los campos calculados también pueden ser almacenados en la base de datos, configurando “store” a “True” en su definición. Estos serán calculados cuando cualquiera de sus dependencias cambie. Debido a que los valores ahora estarán almacenados, pueden ser buscados como un campo regular, entonces no es necesaria una función de búsqueda.


**Campos relacionados**

Los campos calculados que implementamos en la sección anterior son un caso especial que puede ser gestionado automáticamente por Odoo. El mismo efecto puede ser logrado usando campos Relacionados. Estos hace disponible, de forma directa en un módulo, los campos que pertenecen a un modelo relacionado, que son accesibles usando la notación de puntos. Esto posibilita su uso en los casos en que la notación de puntos no pueda usarse, como los formularos de UI.

Para crear un campo relacionado, declaramos un campo del tipo necesario, como en los campos calculados regulares, y en vez de calcularlo, usamos el atributo “related” indicando la cadena de notación por puntos para alcanzar el campo deseado.

Las tareas por hacer están organizadas en estados personalizables y a su vez esto forma un mapa en los estados básicos. Los pondremos disponibles en las tareas, y usaremos esto para la lógica del lado del cliente en la próximo capítulo.

Agregaremos un campo calculado en el modelo tarea, similar a como hicimos a “stage_fold”, pero ahora usando un campo “Related”: 
```
# class TodoTask(models.Model):
    stage_state = fields.Selection(related='stage_id.state', string='Stage State') 
```
Detrás del escenario, los campos “Related” son solo campos calculados que convenientemente implementan las funciones de búsqueda e inversa.  Esto significa que podemos realizar búsquedas y escribir en ellos sin tener que agregar código adicional.


**Restricciones del Modelo**

Para reforzar la integridad de los datos, los modelos también soportan dos tipos de restricciones: SQL y Python.

Las restricciones SQL son agregadas a la definición de la tabla en la base de datos e implementadas por PostgreSQL. Son definidas usando el atributo de clase `_sql_constraints`. Este es una lista de tuplas con el nombre del identificador de la restricción, el SQL para la restricción, y el mensaje de error que se usara.

Un caso común es agregar restricciones únicas a los modelos. Suponga que no queremos permitir que el mismos usuario tenga dos tareas activas con el mismo título:
```
# class TodoTask(models.Model):
    _sql_constraints = [
        ('todo_task_name_uniq',
         'UNIQUE (name, user_id, active)',
         'Task title must be unique!')] 
```
Debido a que estamos usando el campos `user_id` agregado por el modulo `todo_user`, esta dependencia debe ser agregada a la clave “depends” del archivo manifiesto `__openerp__.py`.

Las restricciones Python pueden usar un pedazo arbitrario de código para verificar las condiciones. La función de verificación necesita ser decorada con `@api.constrains` indicando la lista de campos involucrados en la verificación. La validación es activada cuando cualquiera de ellos es modificado, y arrojara una excepción si la condición falla:
```
from openerp.exceptions import ValidationError #

class	TodoTask(models.Model):
     @api.one @api.constrains('name') 
     def _check_name_size(self): 								
        if len(self.name) < 5:
             raise ValidationError('Must have 5 chars!') 
```
El ejemplo anterior previene que el título de las tareas sean almacenado con menos de 5 caracteres.


**Resumen**

Vimos una explicación minuciosa de los modelos y los campos, usándolos para ampliar la aplicación de Tareas por Hacer con etiquetas y estados de las tareas. Aprendió como definir relaciones entre modelos, incluyendo relaciones jerárquicas padre/hijo. Finalmente, vimos ejemplos sencillos de campos calculados y restricciones usando código Python.

En el próximo capítulo, trabajaremos en la interfaz para las características “back-end” de ese modelo, haciéndolas disponibles para las vistas que se usan para interactuar con la aplicación.
