---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Felíz Día De La Gran Capicúa"
subtitle: "Nuevo desafío"
summary: ""
slug: "feliz-dia-de-la-gran-capicua"
authors: [admin]
tags: [capicúas, desafíos, fechas]
categories: []
date: 2020-02-02T11:32:04-03:00
lastmod: 2020-02-02T11:32:04-03:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "Día de la marmota"
  focal_point: ""
  preview_only: false
  position: 3


# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
El 2 de febrero de cada año se celebra en Estados Unidos el ["Día de la Marmota"](https://es.wikipedia.org/wiki/D%C3%ADa_de_la_Marmota). Pero hoy tiene una gracia particular. 

No importa la forma en que lo expresemos la actual fecha, 2 de febrero de 2020, obtenemos siempre un [palíndromo](https://es.wikipedia.org/wiki/Pal%C3%ADndromo): 02-02-2020 o 2020-02-02, pero como ya aclaramos antes, cuando se trata de números les decimos [capicúas](/blog/2020/01/19/desafio-capicuas.html).

En los dos últimos posts hemos estado calculando capicúas y dejamos [un desafío la semana pasada](/blog/2020/01/26/calculando-capicuas.html), cuya solución publicaré más tarde. 

Pues bien, quiero aprovechar esta magna fecha para plantearles un nuevo desafío, que llamaremos "El Día de la Gran Capicúa".

Este 2 de febrero de 2020 tiene otra curiosidad adicional, aparte de coincidir con el día de la marmota, es además el día 33 del año, y como este año es bisiesto, nos quedan 333 días para que termine el año.

Todas estas coincidencias me dan la idea para este nuevo desafío, el que tendrá un premio sorpresa si es que tiene una participación de más de 5 concursantes.

# Desafío del Día de la Gran Capicúa

Deben encontrar todos los días entre el 1-1-1 hasta el 31-12-9999 que cumplan con ser capicúa, en cualquiera las siguientes formas de anotar las fechas:

- AAAAMMDD (o formato ISO)
- DDMMAAAA (formato usado en Chile y varios paises europeos)
- MMDDAAAA (formato usado en USA y otros paises)

Sugiero primero determinar que es Capicúa en formato ISO y después validar los otros casos, considerando como válido tener un cero (0) a la izquierda.

Ejemplos de días de Gran Capicúa: 1-1-1, 5-5-55, 11-11-1111, 02-02-2020.

(*)  Noten que el caso especial de 02-02-2020 no es en estricto rigor una capicúa, pero sí si lo expresamos como ISO: 2020-02-02 sí.  Si lo expresamos así: 2-2-2020 tampoco es capicúa, así que definiremos que se puede "arreglar" la fecha agregando el 0 cuando permita armar una capicúa. Consideren también los casos de los años menores a 1000: 5-5-55, etc, en que no concideramos el 0 porque si no no podrían ser capicúas. (Técnicamente, no estamos usando el formato ISO porque este nos exige rellenar con 0 los días y meses inferiores a 10).

Bonus 1 (opcional): determinen si hay otros días de este tipo que dividan el año de la forma en que lo hace el 2 de febrero de 2020, es decir, es el día 33 de año y quedan 333 días.

Bonus 2 (opcional): determinen si hay una regularidad entre los días gran capicúa, es decir, ocurren cada XXX años, o sólo los bisiestos, o cualquier otra propiedad que puedan determinar.

No es necesario que escriban un programa, siempre que expongan detalladamente cómo determinaron sus resultados.

Deben dejar sus resultado en los comentarios.

Éxito con este desafío, y recuerden que habrá un premio sorpresa si participan más de cinco, así que inviten a sus amigos programadores, matemáticos o aficionados a estas cosas. 

