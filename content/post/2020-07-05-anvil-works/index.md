---

title: "Explorando Anvil Works"
subtitle: ""
summary: ""
authors: [admin]
tags: []
categories: []
date: 2020-07-05T19:55:51-04:00
lastmod: 2020-07-08T19:55:51-04:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

En este periodo de pandemia hemos visto a muchas empresas en la situación de tener que acelerar sus ciclos 
de  desarrollo de software, y eso ha coincidido con
una tendencia de adopción de herramientas de desarrollo de software usando plataformas
[LowCode](https://en.wikipedia.org/wiki/Low-code_development_platform).

Esta corriente tiene sus pro y contra, pero hay que reconocer que los desarrolladores siempre hemos
sufrido la presión por entregar software cada vez más rápido. 

En estas últimas semanas he estado investigando algunas de estas herramientas, y fue a través de un dato
que me entregó Agustín Villena que llegué a Anvil.Works, una herramienta de desarrollo LowCode que permite
desarrollar en Python.

Anvil me recuerda en muchos aspectos a las herramientas RAD de los años noventa, como Delphi o Visual Basic, pero
esta vez el IDE se encuentra en la nube.

Pueden acceder a esta en su sitio: https://anvil.works, y pueden crear una cuenta gratuita, que es bastante amplia para poder
aprender a usarla. Les sugiero seguir los tutoriales que ellos mismos proporcionan en esta dirección: https://anvil.works/learn.

La documentación es bastante buena, y puede ser una excelente alternativa para prototipar o construir un producto
mínimo viable. El plan pagado incluye el soporte para operar tu aplicación, con su propio dominio, y los precios
me parecen son razonables, sobretodo si vas a empezar tu propio negocio, o quieres crear una aplicación que soporte tu negocio.

Con el fin de mostrar las capacidades de esta aplicación, voy a construir un prototipo muy simple de un sistema
de atención por video conferencia para tele medicina.

No voy a enseñar a usar Anvil.Works, para eso están los tutoriales, así que este artículo asume cierto grado de familiaridad
con la herramienta. Si les interesa y hay suficientes comentarios puedo armar un video mostrando esta herramienta en mayor detalle.

## Doctor Chapatín y Asociados

El Doctor Chapatin se ha asociado con otros distinguidos médicos y necesita poder atender a sus pacientes
de forma remota. Así que nos han pedido crear una aplicación que permita implementar la atención de las consultas
usando video conferencia.

Vamos a construir un prototipo que nos permita validar con nuestros clientes qué es lo que quieren. Veamos cuanto nos demoramos.


### Crear la aplicación (1 minuto)

Presionamos el botón "Create App" que aparece en nuestra página inicial en Anvil.Works y el servicio nos pide que elijamos el tema, para esto
seleccionamos Material Design:

![](/blog/2020/07/05/explorando-anvil-works/images/crear-app.png)

Una vez creada la app entramos a nuestro espacio de trabajo:

![](/blog/2020/07/05/explorando-anvil-works/images/entorno-de-trabajo.png)

Haremos click en donde dice "Material Design 3", en la parte superior, entre el logo de Anvil y el botón Run, este es el nombre de la aplicación, así que lo cambiaremos:

![](/blog/2020/07/05/explorando-anvil-works/images/renombrar-app.png)

Fíjense que tenemos un formulario llamado Form1 que se puede observar en la columna del lado izquierdo. Esto se encuentra bajo un área
denominada "CLIENT CODE". Este componente además tiene un ícono con la forma de un rayo, lo que indica que es la pantalla que aparece
al activar la aplicación. Aprovecharemos de cambiarle el nombre, presionando el menú desplegable y le pondremos HomePage.


### Creando el modelo de datos (3 minutos)

Antes de seguir con la Interfaz de usuario crearemos nuestro modelo de datos. Para eso presionamos el signo "+" que sale
en la sección SERVICES. Acá seleccionamos el servicio "Data Tables", tal como se nos explica en la documentación: https://anvil.works/docs/data-tables

![](/blog/2020/07/05/explorando-anvil-works/images/agregar-servicio-data-tables.png)

Lo primero es modelar nuestra tabla de médicos, y la llenaremos con los datos de los socios de la clínica, la que quedará así:


![](/blog/2020/07/05/explorando-anvil-works/images/tabla-medicos.png)

Esta tabla la usaremos para identificar los socios de nuestra clínica en el homepage.

Ojo, es importante dar el permiso para que esta tabla sea accesible desde los servicios de datos, para eso se habilita en la parte
superior de esta página donde dice "Permissions".

### Construyendo la capa de servicio de datos (2 minutos)

Para poder acceder a los datos debemos habilitar un servicio en el server, tal como se nos explica en la documentación: https://anvil.works/docs/server.

En Anvil.Works hay una clara separación de responsabilidades, y es en los _ServerModules_ donde se se recomienda implementar
 la lógica de acceso a los datos.

Para esto clickeamos el botón "+" en SERVER CODE y agregamos un _server module_:

![](/blog/2020/07/05/explorando-anvil-works/images/agrega-server-module.png)

Y en este `ServerModule1`, agregamos el siguiente código:

```
@anvil.server.callable
def buscar_medicos():
  return app_tables.medicos.search(
    tables.order_by("nombre", ascending=True)
  )
```

Esto será usado para visualizar los datos en nuestra interfaz de usuario. Como pueden ver el código es bastante fácil de entender.

La API para acceder a los datos es bastante sencilla y se encuentra documentada acá: https://anvil.works/docs/api/anvil.tables

Veamos cómo podemos visualizar esto en nuestra home page.


### Completando nuestra homepage (5 minutos)

En el AppBrowser elegimos el form HomePage, agregamos un label que tomamos de la columna de la derecha denominada Toolbox, esto
es bien sencillo y sólo implica aplicar "drag and drop" para llevar las componentes. El label lo dejamos como título de la aplicación,
quedando así:


![](/blog/2020/07/05/explorando-anvil-works/images/titulo-homepage.png)


Ahora vamos a crear un panel para visualizar los datos de un médico, para esto agregamos un form en CLIENT CODE:

![](/blog/2020/07/05/explorando-anvil-works/images/agregar-panel.png)

Al presionar esta opción se nos pide seleccionar el tipo de formulario, elegiremos un panel en blanco:

![](/blog/2020/07/05/explorando-anvil-works/images/agregar-panel-medicos.png)

El que renombraremos como Panel Médicos, a este le agregaremos una componente tipo "Card" que elegiremos del Toolbox.

Agregaremos 2 componentes tipo `Label` y una tipo `Image`.

Configuraremos el primer label para que despliegue el nombre del médico, para eso ajustaremos las propiedades `name` que dejaremos como `label_nombre`, agregaremos un `databinding` de tipo text asociado a `item['nombre']` (es decir, al campo nombre de la tabla de datos).
Ajustaremos otras propiedades como `role` y pondremos el texto en `bold`, tal como se aprecia en esta imagen:

![](/blog/2020/07/05/explorando-anvil-works/images/label-nombre.png)

Haremos lo mismo con la imagen y el segundo label.


Luego volveremos a seleccionar el Homepage y le agregaremos una componente de tipo `Repating Panel`:

![](/blog/2020/07/05/explorando-anvil-works/images/repeating-panel.png)


Este permite colocar otras componentes en su interior, para eso se define la propiedad `item_template` del `repeating panel` para que
usa el PanelMedicos que definimos recién:

![](/blog/2020/07/05/explorando-anvil-works/images/panel_de_medicos.png)

Ahora escribiremos código, para eso presionaremos el botón CODE de nuestra HomePage y agregaremos el siguiente método en la clase
`HomePage`:

```
def refrescar_panel(self):
    self.panel_de_medicos.items = anvil.server.call('buscar_medicos')
```


y modificaremos el constructor ``__init__`` para que invoque este método:

```
def __init__(self, **properties):
    # Set Form properties and Data Bindings.
    self.init_components(**properties)

    self.refrescar_panel()
```


Y con esto podemos apretar el botón RUN y si todo está bien veremos lo siguiente:


![](/blog/2020/07/05/explorando-anvil-works/images/first-run.png)


### Implementando la video conferencia (10 minutos)

Hasta ahora todo lo que hemos hecho está soportado por Anvil.Works por default, pero para simular la video
conferencia usaremos el servicio Jitsi Meet.

Anvil.Works nos permite agregar bibliotecas de javascript nativas, para esto agregamos una `Native Library` del siguiente modo:

![](/blog/2020/07/05/explorando-anvil-works/images/native-library.png)


El código que agregamos es el siguiente:

```
<script src="https://meet.jit.si/external_api.js"></script>
```

Y lo que hará Anvil es agregarlo en la sección  `<head>` de nuestra aplicación.

Luego de esto, necesitamos crear un panel especial para desplegar la interfaz de Jitsi. Para esto
crearemos un panel tipo html:

![](/blog/2020/07/05/explorando-anvil-works/images/custom-html-form.png)

Nombraremos a este panel `JitsiPanel` y editaremos su contenido presionando el botón `Edit` en `Custom HTML`:

![](/blog/2020/07/05/explorando-anvil-works/images/editar-custom-html.png)

Al hacer esto aparece un editor en la parte inferior del área de diseño, en este editor copiaremos este código (borrando el contenido previo):

```
<div anvil-slot="default"></div>
<div id="jitsi"></div>

<script>
  function initJitsi(domain, options) {
    options.parentNode = this.find("#jitsi")[0]
    window.japi = new JitsiMeetExternalAPI(domain, options);
  }
</script>

<style>
  #jitsi iframe {
    width: 100%;
    border: none;
  }
</style>
```

Así se ve en el IDE (si presionas donde dice "Expand HTML Editor", en la parte inferior del editor):

![](/blog/2020/07/05/explorando-anvil-works/images/jitsi-panel-editado.png)

Después de esto agregaremos un botón en este panel y configuraremos su DATA BINDING con el siguiente código:

```
"INICIAR VIDEO CONFERENCIA CON {}".format(self.item['nombre'])
```

![](/blog/2020/07/05/explorando-anvil-works/images/boton-video-conferencia.png)


Haremos doble click sobre el botón y se abrirá el editor de código, donde escribiremos el siguiente código:


```
def boton_video_conferencia_click(self, **event_args):
    """This method is called when the button is clicked"""
    
    room = "Consulta-Doctor-{}".format(self.item['nombre'])
    self.call_js("initJitsi", "meet.jit.si", 
                 {"roomName": room, 
                  "height": 500})
    self.boton_video_conferencia.visible = False
```

Para finalizar agregaremos el `JitsiPanel` en el `PanelMedicos`, para eso abrimos el formulario `PanelMedicos` y arrastraremos
el JitsiPanel a la parte inferior debajo de la imagen y la descripción, el formulario deberá quedar así:

![](/blog/2020/07/05/explorando-anvil-works/images/agregar-jitsi-panel.png)

Con esto nuestro prototipo ya está listo, al presionar run podemos elegir un médico e iniciar una video conferencia con él:


![](/blog/2020/07/05/explorando-anvil-works/images/video-conferencia.png)


### Llamando al doctor (5 minutos)

Ahora lo que falta es notificar al médico que se conecte a la sala de conferencia, para esto le enviaremos un correo
con un enlace para que acceda a la sala que previamente hemos creado en la plataforma Jitsi.meet.

Primnero agregaremos soporte para despachar email agregando el servicio respectivo:

![](/blog/2020/07/05/explorando-anvil-works/images/agregar-email.png)

Luego agregamos el siguiente código en nuestro `ServerModule1`:

```
@anvil.server.callable
def notificar_medico(medico, room):
  email_medico = medico['email']
  anvil.email.send(to=email_medico,
                   subject="nueva video llamada medica",
                   html="""
  <h1>Nueva Sesi&oacute;</h1>
  <p>
  Lo est&acute; esperando un paciente en una video llamda.<br>
  Haga click en el enlace de abajo para ingresar:<br>
  <b><a href="https://meet.jit.si/{}">INGRESAR A VIDEO LLAMADA</a></b></p>
  </p>
  """.format(medico['nombre'], room))
```

https://meet.jit.si/Consulta-Doctor-Dr.%20House

Además hay que acordarse de incluir el respectivo import al inicio de `ServerModule1`:

```
import anvil.email
```


Con esto estamos casi listos, debemos agregar una llamada más en el click del botón que activa la video conferencia,
con lo que el método (que se encuentra en `JitsiPanel`) queda así:

```
def boton_video_conferencia_click(self, **event_args):
    """This method is called when the button is clicked"""
    
    room = "Consulta-Doctor-{}".format(self.item['nombre'])
    self.call_js("initJitsi", "meet.jit.si", 
                 {"roomName": room, 
                  "height": 500})
    self.boton_video_conferencia.visible = False

    anvil.server.call('notificar_medico', self.item, room)
```


Y si todo sale bien, a nuestra casilla de correo llegará un email con el link.

## Cierre

Con todo hemos invertido unos 26 minutos en desarrollar un prototipo que incluye video conferencia!

Bastante impresionante la verdad. Claro que yo tenía claro que debía ejecutar en cada paso, en realidad me tomó unas 3 horas
investigar y escribir este post, aún así es bastante poco tiempo.

Lo otro es que tuve que escribir muy pocas líneas de código.

Si tienen cuenta en Anvil.Works pueden copiar mi aplicación usando este link: https://anvil.works/build#clone:M6ILKTAYE3BCIMSG=WWRAJOFMNG6NQYNDFMBAANI3

Recuerden, si este post tiene sobre 20 comentarios publicaré un video más amplio sobre Anvil.Works, este demo
y quizás otras cosas.
