---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "El Camino del Backend Developer: Lenguajes de Programación"
subtitle: ""
summary: ""
authors: [admin]
tags: [aprendizaje, roadmaps, competencias, 'lenguajes de programación']
categories: ['backend developer']
date: 2021-08-07T11:48:50-04:00
lastmod: 2021-08-07T11:48:50-04:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false
  placement: 3


# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

Hace unos años atrás presenté una charla en [Starsconf](https://www.starsconf.com) titulada ["Esos Raros Lenguajes Nuevos"](https://www.youtube.com/watch?v=Hp9HwLPYkjI)[^1], creo que es la presentación más divertida que he hecho. 

El tema principal es la idea de lo positivo que aprender otros lenguajes de programación distintos al que usas en el día a día en tu trabajo.

En particular, recomiendo aprender lo que algunos llaman los lenguajes de Perlis[^2]. Este concepto nace de una frase de Alan Perlis que dice:

    A language that doesn’t affect the way you think about programming is not worth knowing. — Alan Perlis

De todas maneras todos los programadores somos programadores políglotas.

## El Programador Políglota

Cuando desarrollamos software lo normal es usar más de un lenguaje.

Por ejemplo, un programador full-stack en una aplicación web típca podría trabajar con los siguientes lenguajes:

- HTML
- CSS
- Javascript
- Java (o Python, o PHP)
- SQL
- YAML
- JSON
- Etc.

Cuando explico esto no falta quien dice que HTML y CSS no son lenguajes de programación, pero yo no he dicho que estos sean lenguajes de programación[^3].

Si recuerdan bien, en nuestro anterior artículo introducimos el concepto de [Lenguajes Formales](/blog/2021/06/28/el-camino-del-backend-developer-lenguajes-formales/).

Todos estos lenguajes que usamos al programar son **lenguajes formales**, pero de distintos tipos.

Por ejemplo, casi todos los lenguajes que terminan en ML son _"Markups Lenguajes"_, lenguajes de marcado. Estos son  lenguajes formales que permiten incorporar etiquetas o marcas, las que son interpretadas por los programas que procesan los archivos escritos en estos lenguajes. Al interpretar estos archivos las marcas dirigen la traducción, típicamente para formatear el contenido. 

Por otro lado, algunos lenguajes de marcado nos permiten estructurar datos. Es por eso que los usamos para compartir información, por ejemplo cuando los usamos para "serializar" datos a través de un protocolo como HTTP, los ejemplos clásicos son XML o JSON.

Y están los lenguajes de programación propiamente tal, como Java o SQL.

Vamos a decir que un lenguaje de programación es un lenguaje formal que nos permite expresar una computación o algoritmo.

O en otras palabras, son los lenguajes con los que escribimos programas, y los programas son un conjunto de instrucciones que permiten que un computador ejecute algún tipo de computación.

Sin embargo no todos los lenguajes de programación tienen el mismo poder computacional.

##  Turing Completo

En la Teoría de la Computación, el término "Túring Completo" se refiere a todo sistema que tiene el poder computacional equivalente a una "Máquina Universal de Turing". Y ¿qué es eso?

En 1931 Alan Turing decidió reformular ciertos resultados de la lógica matemática, expresando las formulaciones de teoremas en términos de algoritmos que podían ser interpretados por una máquina teórica.

Un algoritmo es “un conjunto preescrito de instrucciones o reglas bien definidas, ordenadas y finitas que permite realizar una actividad mediante pasos sucesivos que no generen dudas a quien deba realizar dicha actividad.” O sea, un programa :smile:.

Lo que pretendía Turing era estudiar los límites de las máquinas que realizan cálculos automáticos. Era una época en que se estaban sentando las bases teóricas de los futuros computadores. Ya era posible contar con los elementos para construir estas máquinas y Turing quería ver cual era el poder de estas máquinas. Hasta qué punto estas máquinas serían capaces de ayudarnos a resolver problemas complejos.

{{<figure caption="Alan Turing" src="Alan_Turing.jpg">}}

Turing describió una máquina teórica que manipula los símbolos escritos en una cinta siguiendo las reglas escritas en una tabla. La máquina de Turing consta de un cabezal que puede leer y/o escribir en una cinta infinita de papel. La máquina puede realizar sólo las siguientes operaciones: 

- Avanzar el cabezal hacia la derecha
- Avanzar el cabezal hacia la izquierda
- Leer el contenido de una celda de la cinta
- Borrar el contenido de una celda 
- Escribir contenido en la cinta.


{{<figure caption="Imagen artística de la Máquina de Turing" src="Maquina-de-Turing.png">}}

Junto a la máquina hay una tabla, que representa las acciones que se deben hacer al interpretar los símbolos de la máquina. 

Por ejemplo:

| Símbolo | Estado | Escribir | Mover      | Siguiente Estado |
| --------|--------|----------|------------|------------------|
|    0    |  E0    |    1     | Derecha    |      E1          |
|    0    |  E1    |    1     | Izquierda  |      E0          |
|    0    |  E2    |    1     | Izquierda  |      E1          |
|    1    |  E0    |    1     | Izquierda  |      E2          |
|    1    |  E1    |    1     | Derecha    |      E1          |
|    1    |  E2    |    1     | Derecha    |      ALTO        |


La Máquina de Turing se inicializa en el estado `E0`. La columna `Símbolo` nos indica cuál es el símbolo que se encuentra en la celda apuntada por el cabezal. La columna `Escribir` nos dice qué símbolo se debe escribir en la cinta cuando nos encontramos en el estado dado por la columna `Estado` y el símbolo actual en la cinta. La columna `Mover` nos dice a donde deberemos mover el cabezal tras realizar la escritura. Y el estado se modifica al valor dado por la columna `Siguiente Estado`.

La **Máquina Universal de Turing** extiende este modelo al permitir que la configuración de la Máquina de Turing (la tabla), se almacena en la misma cinta. Esto permite a la Máquina Universal simular a cualquier Máquina de Turing particular.

Ahora bien. Tecnicamente no existen Máquinas de Turing ni tampoco Máquinas de Turing Universales, porque requeriríamos de una cantidad infinita de almacenamiento. Pero una computadora es lo más cercano que tenemos a una Máquina Universal de Turing.

Es fácil ver que un programa se aproxima a una Máquina de Turing.

Si creamos un sistema que obtenga el mismo poder computacional que una Máquina de Turing (o se aproxime en términos prácticos a la misma), entonces nuestro sistema es Turing Completo.

Así que varios lenguajes de programación son Turing Completos.

¿Por qué?

Porque la mayor parte de los lenguajes de programación tienen la capacidad de avanzar y retroceder en la cinta (es decir, la memoria).

Aunque no todos los lenguajes de programación son Turing Completos, por ejemplo SQL-92[^4], en la práctica casi todos los lenguajes de programación de uso general lo son. Así que no es algo que nos deba preocupar demasiado.

### Lenguajes de Dominio Específico

Hay algunos lenguajes diseñados para aplicaciones específicas, estos son los lenguajes de dominio específico o DSL (por su sigla en inglés).

Gradle, Make, UnrealScript, YACC, Puppet, YAML, Dockerfile, etc. Son lenguajes creados para poder configurar sistemas, orquestar acciones o ejecutar acciones específicas en ciertos entornos.

Incluso podemos crear nuestros propios DSL usando parte de las técnicas que aprendimos en el capítulo anterior. En el reino de los DSL es más probable encontrar lenguajes que no son Turing Completos, porque sus diseñadores no están interesados en darle esas "capacidades universales".


## Clasificando los lenguajes

Cuando yo estudié se usaba una clasificación "generacional" de los lenguajes de programación. Que dividía los lenguajes desde la primera generación representada por el lenguaje de máquina, hasta la cuarta generación que era un conjunto de herramientas que permite combinar componentes pre construidias para genera programas. Pero esa clasificación ya está un tanto en desuso, Una de las razones es que casi todos los lenguajes actuales serían lenguajes de tercera generación. 

Más interesante es la clasificación por paradigmas.

## Paradigmas de Programación

Hay varios paradigmas de computación que nos permiten clasificar a los distintos lenguajes de programación.

Hay dos tipos principales de modelos de programación, la **programación declarativa** y la **programación imperativa**.

En la programación declarativa nos interesa expresar qué queremos lograr como resultado. En la programación imperativa nos concentramos en todos los pasos que debemos ejecutar para lograr un resultado.

### Paradigmas Imperativos

- **Por procedimiento**: es el paradigma más usado, porque está fuertemente basado en los modelos de Turing o Von Neumman de la computación. Acá tenemos a los distintos lenguajes de máquina, los respectivos lenguajes de ensamblado (assembly) y lenguajes tradicionales  como COBOL, Basic, Pascal o C. Acá especificamos cada uno de los pasos para ejecutar el algoritmo.

- **Orientado el Objeto**: acá se encapsula la computación en elementos llamados objetos. Estos objetos se comunican entre si mediante mensajes o métodos, los que se implementan de un modo imperativo. Representantes de este paradigma son: SmallTalk, C# o Java.

### Paradigmas Declarativos

- **Programación Funcional**: basados en el modelo de Church de la computación, es un modelo más matemático donde todo está basado en la declaración de una serie de predicados. Acá tenemos a lenguajes como Haskell o Scheme.

- **Programación Lógica**: en este modelo se establecen relaciones lógicas entre predicados y átomos. El representante más importante es Prolog, pero existen otros como Datalog.

### Multi Paradigmas 

La mayor parte de los lenguajes que veremos son multi paradigmas. Por ejemplo, Python tiene elementos funcionales y orientados al objeto.


## ¿Qué lenguajes aprender y usar?

Nuestro [mapa](https://roadmap.sh/backend) nos propone la siguiente lista:

- [PHP](https://www.php.net)
- [Java](https://go.java/)
- [C#](https://docs.microsoft.com/en-us/dotnet/csharp/)
- [Ruby](https://www.ruby-lang.org/)
- [Python](https://www.python.org)
- [Javascript](https://www.javascript.com)
- [Go](https://golang.org)
- [Rust](https://www.rust-lang.org)

Java, Python,  Javascript y PHP son los lenguajes más usados hoy en día, con un fuerte crecimientode Python en los últimos años.
Muchos de los ejemplos en esta serie los he escrito en Python, lo que muestra la flexibilidad de este lenguaje, que se usa tanto en backend, scripting de sistemas, e incluso en ciencia de datos e inteligencia artificial.

Java es el lenguaje de facto en el mundo empresarial. Algunos lo llaman el nuevo COBOL, y seguirá vigente por años. Lo mismo pasa con C# que se sostiene por su popularidad en los entornos del ecosistema de Microsoft. Si tu objetivo es trabajar en entornos empresariales, cualquiera de estos dos lenguajes te sirven. 

Ruby y PHP son lenguajes muy usados. PHP de hecho es uno de los lenguajes más populares, y gran parte de internet se sostiene sobre este lenguaje. Ruby fue muy popular hace unos años, pero su uso ha decaido en favor de Python o incluso Elixir, creo que un factor fue su desempeño, a pesar de que ha mejorado mucho, pero no ha logrado recuperarse. El éxito de este lenguaje se debió al framework Ruby on Rail que permite desarrollar sitios webs muy rápidos. Quizás el servicio más grande que opera usando Ruby es GitHub. PHP consolidó su éxito con su uso en Facebook, que ha llegado a crear un entorno de alto performance para este lenguaje.

Javascript se ha convertido en un lenguaje importante para el desarrollo backend, y probablemente lo domine por mucho tiempo, gracias al desarrollo de [Node.js](https://nodejs.org/en/). Hablaremos un poco sobre en futuros artículos.

Por último Go y Rust representan las tendencias actuales y futuras. 

Si bien Rust es un lenguaje más orientado a la programación de sistemas, se ha vuelto muy popular por motivos técnicos y culturales. Su forma innovadora de manejar la memoria, evita un montón de errores de programación y que generan vulnerabilidades muy comunes. Pero además, su comunidad es muy activa y comprometida, así que está en crecimiento.


Go es un lenguaje muy pragmático, rápido y muy bien diseñado (entre sus creadores están nada menos que Ken Thompson y Rob Pike). Aunque fue creado por Google y usado por empresas como Uber y Dropbox, aún no está entre los más populares.


Pero fuera de estos lenguajes creo que vale la pena echarle una mirada a Elixir, Clojure, Scala, Kotlin y Swift. 

En esta serie nos concentraremos en Python, Javascript, Rust y Go[^5]. Revisaremos las características principales de estos cuatro lenguajes en los próximos capítulos. Y para mostrarles algo distinto agregaremos un capítulo dedicado a Elixir.

¿Cuáles son tus lenguajes de programación favoritos?

Si te gustó puedes apoyar esta serie a donando tu aporta a través de [Ko-Fi](https://ko-fi.com/lnds/), en este enlace: https://ko-fi.com/lnds/donate

{{<koffe>}}


[^1]: Por desgracia el audio está desfasado.
[^2]: http://blog.fogus.me/2011/08/14/perlis-languages/
[^3]: Tampoco lo dije en la charla, pero varios twitearon esta crítica, lo que demuestra que hay personas que no ponen atención y sólo quieren pelear.
[^4]: Pero hay dialectos de SQL que son Turing Completo, como PSQL, TSQL. Incluso se dice que SQL-99 sí es Turing Completo.
[^5]: Aunque hemos realizado algunos ejemplos en C, pero este no es un lenguaje muy adecuado para el desarrollo backend.