---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Nuevos Desafios"
subtitle: ""
summary: ""
authors: []
tags: [desafios, desafio20191208]
categories: []
date: 2019-12-08T11:39:03-03:00
lastmod: 2019-12-08T11:39:03-03:00
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

# Nuevos Desafíos

Unos años atrás empecé este blog con el fin de escribir de manera dedicada sobre programación, de un modo más detallado y técnico de cómo lo hacía en mi blog principal, ["La Naturaleza del Software"](https://www.lnds.net/).

Hoy empieza una nueva era de este sitio, que retomaremos con una sección que fue bien popular, los desafíos de programación.

Cada semana trataré de publicar un problema con el desarrollo de su solución en algún lenguaje de programación, el que será compartido en un Gist o algún repositorio en gitHub para que lo analicen. Junto a esto les dejaré planteado un desafío para que ustedes resuelvan antes de la semana que sigue. Así que sin más dilaciones partamos.

# Crackeando Claves

Este desafío nace de una mini conversación en Twitter con Denis Fuenzalida [@dfuenzal](https://twitter.com/dfuenzal), quien publicó el siguiente tweet este fin de semana:

{{<tweet 1203063885856534528>}}

Lo que plantea Denis es lo fácil que es "crackear" claves sencillas aunque usemos un algoritmo relativamente fuerte como SHA-512.

Aparte de este importante hecho que debe ser considerado cuando nos preocupamos de la ciberseguridad, me nación la curiosidad por la implementación particular del código de Denis. Así que le pregunté por cuantas líneas de código tenía su solución. Denis tuvo la amabilidad de publicar el Gist con su código:

{{< gist dfuenzalida 52f63122f19ccf0ed82ef10a1362deeb  >}}

La verdad es que yo pensé que el código tendría pocas líneas de código, dado lo conciso que es Clojure. Pero no consideré que debía implementar una función para generar claves en SHA-512. Aún así es notable cómo en menos de 20 lineas de código tengamos un algoritmo de fuerza bruta para determinar claves.

Decidí escribir una solución alternativa en Rust que dejé en el siguiente Gist:

{{< gist lnds 44b4d7b2cb439272b3829138ba1beb47>}}

El código hace lo mismo que la solución de Denis, pero al ser Rust un lenguaje compilado se ejecuta en menos tiempo que la solución en Clojure de Denis.

Lo que hace este desafío es buscar la coincidencia para una clave de 4 letras (todas minúsculas) del alfabeto inglés de 26 letras. En este caso buscamos una colisión en SHA512 para la palabra "help".

# Desafio 2018-12-08

Ahora viene el desafío para esta semana.

He elegido 5 números de 4 dígitos cada uno. Para cada uno generé su hash 512 el que he expresado en hexadecimal. Estos cinco números se encuentran en el siguiente archivo:

https://github.com/lnds/desafios-programando.org/blob/master/2019-12-08/hashes.txt

El desafío consiste en descubrir cuáles son estos números. Además deben compartir el código que usaron para encontrarlos. 

Deben colocar un link a su solución en los comentarios. 

Espero sus respuestas!