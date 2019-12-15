---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Solucion Desafio Crackeando Claves"
subtitle: ""
summary: ""
authors: []
tags: [desafios, desafio20191208]
categories: []
date: 2019-12-15T10:10:52-03:00
lastmod: 2019-12-15T10:10:52-03:00
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

El [desafío de la semana pasada](/blog/2019/12/08/nuevos-desafios.html) era una invitación para volver a plantear retos de programación en este sitio. Es natural que no tuviera tantas respuestas, puesto que estuvimos mucho tiempo sin publicar nada en este blog.

Pero aún así hubo tres respuestas que revisaremos más adelante. Ahora les entregaré la respuesta a ese desafío.

El desafío consistía en deducir cuales eran los números de 4 dígitos que corresponde a estos cinco hashes SHA-512

    d0d7cc0cbf21d971606685c15f896bbfd6b1f99e1c2a3fa8381c622f57fab8877ef56d8f88f634829adea9088db05326eaf6fb9253554dd873b11b59b341f09b
    d1dfa57bff540ec0baac90a1897f1b5e9a9a6bec30f070e33eabc4fc1fbed54dd76c4eb793dbd7accacd604903b6376325ea4e887b38e48bec4b76cf2c301549
    be3283db5ebd65a0db49edeaa8801e64bd2c0303d38bd94d4f38ec5596ba280c8368e64adc3a1f5600fb7642ecf6e911fdcd6fb1ec7492bbb2855ab9bd1962ce
    a4e709497bd88e9987529eca36bd3e5249d7ecd178c90c5999871b36f1016a7f040b4646665e49d8a344a77be6935af17282e7e57ae64c9bb487ad8a46666561
    15f13e170a5bb2759f79deaf4edb6f3c8ceae5fe09bb70c168f6b70f08e97ad736771c9f3fc34a721b3d4bbb5c18f48d965178f0274372767934ee89375fdb20

Estos hashes están expresados en hexadecimal. 

## La solución

La solución que se nos ocurre es un ataque de fuerza bruta.

Como sabemos que las claves almacenadas son pines de 4 dígitos el ataque de fuerza bruta es super simple, iteramos por los 10.000 posibles valores hasta encontrar los 5. Pero podemos hacerlo un poquito más rápido si usamos un hashset, de modo que cada vez que encontramos una clave imprimimos la respuesta y sacamos el elemento del hashset. De este modo no necesariamente recorreremos los 10.000 valores.

Considerando todo esto la solución completa en Rust es la siguiente:

```rust
use sha2::{Digest, Sha512};
use std::collections::HashSet;
use std::env;
use std::fs;

fn main() {
    // t0 nos sirve para medir tiempo
    let t0 = std::time::Instant::now();

    // toma de los argumentos el nombre del archivo de hashes
    let args: Vec<String> = env::args().collect();
    let filename = &args[1];

    // crea un hashset con los hahes que lee del archivo
    let mut hashes = fs::read_to_string(filename)
        .unwrap()
        .lines()
        .map(|s| s.to_string())
        .collect::<HashSet<String>>();

    // loop de búsqueda
    for i in 0..10000 {
        let candidate = format!("{:04.4}", i);
        let hash = format!("{:x}", Sha512::new().chain(&candidate).result());
        if hashes.contains(&hash) {
            println!("{}", candidate);
            hashes.remove(&hash);
        }
        if hashes.is_empty() { break }
    }

    // muestra el tiempo de ejecución
    let dur = t0.elapsed();
    let secs = dur.as_secs();
    let msec = dur.subsec_millis();
    println!("tiempo ocupado: {} segundos {} milisegundos", secs, msec);
}
```

El código completo con el proyecto Cargo se encuentra en mi repo Github en esta carpeta: https://github.com/lnds/desafios-programando.org/tree/master/2019-12-08/solucion.

Si ejecuto esto en mi computador se toma 5 milisegundos en encontrar las claves.
Hay que notar que si elimino la linea que dice: ```ìf hashes.is_empty() { break }```, el programa se demora 10 milisegundos. (¿Por qué exactamente el doble de tiempo?).

Con el programa anterior podemos determinar que los cinco valores que estamos buscando son:

        0101
        0812
        1214
        2336
        5689


## Reflexiones

Hay muchas personas que implementan el cifrado de claves usando algoritmos de hash como SHA512. Lo que es peor usan un algoritmo más anticuado, como MD5 o SHA-1. A priori es una mala estrategia.

Imaginen que estos 5 números hubieran sido los pines usados para una tarjeta de débito o crédito bancaria, y que los tuvieramos en nuestro poder. Ya tenemos un algoritmo rápido para desencriptarlos.

Es más, hay servicios que nos permiten desencriptar hashes, como la solución que prouso Christian Candia:

{{<gist ccandiaw 84eda2d17df2e8a45770cb9d1a7fd477>}}

Quien ocupa el servicio md5 decrypt (que en realidad opera con varios algoritmos de hash) para obtener la respuesta. No es el tipo de solución que estaba buscando, pero es efectiva, y nos muestra lo vulnerable que son estos algoritmos.

Las otras soluciones implementan el loop de fuerza bruta completo con las 10.000 iteraciones.

En el caso de Denis Fuenzalida usa ClojureScript:

{{<gist dfuenzalida 3a6614985bdcd01e72598bc0d126956b>}}

Y Rodrigo Tobal usa python:

{{<gist rtobar e1bde0a9dbd4f3ba6c4f30a0c1753a14>}}

Con todo esto debería qudarle claro lo vulnerables que son las claves encriptadas usando algoritmos de Hash, incluso con uno moderno como SHA-512.

### ¿Qué se puede hacer para evitar este tipo de ataques?

Una manera es usando lo que se llama agregar un SALT, que es un valor adicional al valor que vamos a encriptar, de este modo el universo de búsqueda se hace más dificil y tratar de crackear la solución toma mucho más tiempo (depende de el valor que uses para el SALT y que tanto la protejas).
Puedes leer más detalles en este artículo de Auth0: https://auth0.com/blog/adding-salt-to-hashing-a-better-way-to-store-passwords/. 

El uso de SALT es una buena medida de mitigación, pero la otra alternativa es usar otro tipo de algoritmos, como BCrypt que implementan lo que se llama Key Derivations Function, que tienen propiedades criptográficas más fuertes.

Eso es todo con este desafío, atentos al desafío de esta semana que viene luego.
