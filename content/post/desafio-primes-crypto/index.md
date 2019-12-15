---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Desafio Primes Crypto"
subtitle: ""
summary: ""
authors: []
tags: [desafios, números primos, criptografía, desafio20191215]
categories: []
date: 2019-12-15T13:55:06-03:00
lastmod: 2019-12-15T13:55:06-03:00
featured: true
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

Los números primos son usados en criptografía de varias maneras, pero principalmente para generar claves.

Hay una forma de codificar bien ingeniosa, que no se usa mucho pues sólo permite operar con mensajes muy cortos y requiere mucho esfuerzo de computación. Se las voy a describir a continuación.

Lo primero es definir nuestro alfabeto, para efectos de este desafío definiremos el alfabeto como las letras minúsculas del alfabeto inglés, esto es un conjunto de 26 caracteres: 

    "abcdefghijklmnopqrstuvwxyz"

A cada letra se le asigna un valor numérico. 
A la letra a le asignaremos el valor 1, a la b el valor 2 y así sucesivamente.

    a 1
    b 2
    c 3
    d 4
    e 5
    f 6
    g 7
    ...


También definiremos que nuestros mensajes tendrán un largo máximo de 10 caracteres.

Usaremos como clave los primeros 10 números primos:

    2, 3, 5,  7,  11,  13,  17,   19,  23,  29

Para encriptar elevamos el número primo en la posición de cada letra al valor de la letra y luego multiplicamos todos los valores.

Ejemplos:

    "hola" = 2^8 * 3^15 * 5^12 * 7^1 = 6277646812500000000
  
    "abc" = 2^1 * 3^2 * 5^3 = 2250

Acá el operador ^ es el operador potencia, así 2^8 corresponde a 2 elevado a 8.

Usando esta técnica he cifrado 5 palabras que deben descubrir. 

Los valores cifrados de estas cinco palabras son:

    40233680140239064308212773133312764943748000000000000000000

    45087592346382488832366326449069283133412065695070241853706487031250000

    8679608265978819446726615827418826755071534791227586000000000000000

    27914797455418138364383974433340682351379394531250000

    1059246326897806934956526951511712774422030066044771506277207862450866815727414664067933302815246198122035444843750

Valores que también pueden encontrar en este archivo: https://github.com/lnds/desafios-programando.org/blob/master/2019-12-15/ciphers.txt

Deben dejar la respuesta y el código o descripción de la técnica que usaron para descubrir las palabras en los comentarios.

Si participan más de 10 habrá un regalo en el ganador.

