---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Solucion Desafio Primes-Crypto"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2019-12-22T22:24:03-03:00
lastmod: 2019-12-22T22:24:03-03:00
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

Antes que termine el día les entrego la solución al [desafío del 15 de diciembre: Primes-Crypto](/blog/2019/12/15/desafio-primes-crypto.html).

Antes de continuar quería mencionar que esta forma de cifrar la descubrí leyendo el cuento de ciencia ficción de Frederick Pohl titulado "El Oro al Final del Arco Estelar" ([The Gold at the StarBow's End](https://en.wikipedia.org/wiki/The_Gold_at_the_Starbow%27s_End)), publicado en 1972.

## Cifrado con números primos

Lo primero es explicar el algoritmo para cifrar que fue el que usé para plantear el desafío, el código en Rust es el siguiente:

```rust
use num::{pow, BigInt};
const PRIMES: [u32; 10] = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29];

fn crypt(message: &str) -> BigInt {
    message
        .bytes()
        .enumerate()
        .map(|(i, b)| {
            let exp = usize::from(b - b'a' + 1);
            let base = BigInt::from(PRIMES[i]);
            pow(base, exp)
        })
        .product()
}
```

El problema de este algoritmo de cifrado es que requiere de números muy grandes, es por eso que usamos el crate num: https://crates.io/crates/num

En cualquier lenguaje que implementes esto requerirás usar un tipo de datos que opere con números grandes. Aunque Rust soporta números de 128bits aún así no es suficiente.

Piensen además que nuestro desafío usa sólo los primeros 10 números primos. Para aplicar esto con mensajes más largos deberíamos tener una tabla con tantos números primos como el largo del mensaje. Por ejemplo, para encriptar un mensaje de 1.000 caracteres requerimos los primeros 1.000 números primos. 

Es por eso que este mecanismo de cifrado no se usa en la práctica. En el relato de Frederick Pohl se usa como un mecanismo de compresión para mandar un mensaje enorme, expresado como una simple operación matemática, pero que plantea un desafío enorme a los protagonistas de la historia.

## Descifrado

Para descifrar los mensajes debemos calcular los factores primos del número que corresponde al mensaje cifrado. Por ejemplo, si el mensaje es "abc", el valor calculado es:

    2^1 * 3^2 * 5^3

Los factores primos de este número son:

    {2, 3, 3, 5, 5, 5}

Entonces contamos la frecuencia de cada factor primo y así obtenemos el índice de cada letra:

    {(2,1), (3, 2), (5, 3)}

Por lo tanto el mensaje corresponde a la primera letra, a la segunda letra y a la tercera letra.

Eso se puede resolver con el siguiente algoritmo (en Rust):

```rust
fn decrypt(cipher: BigInt) -> String {
    // obtiene la frecuencia de cada factor primo
    let letters: Vec<usize> = factors(&cipher)
        .iter()
        .group_by(|&x| x)
        .into_iter()
        .map(|(_, g)| g.count())
        .collect();
    // mapea las frecuencias a caracteres y devuelve el string resultante
    letters
        .iter()
        .map(|&u| char::from(b'a' + u as u8 - 1))
        .collect::<String>()
}
```

No incluyo la función factors, pero la pueden consultar en el repo GitHub con la solución completa a esta desafío acá: https://github.com/lnds/desafios-programando.org/tree/master/2019-12-15/primes-crypt

### Respuestas al desafío

Las palabras cifradas eran: 

    1. turing
    2. dijstra
    3. programa
    4. desafio
    5. algoritmo

Gracias por participar a Rodrigo Guzman (con una solución en php), a Denis Fuenzalizda (en clojure), Rodrigo Tobar (Python), Joel Iturra (Java) y Marcelo Theone (en javascript). Son las cinco soluciones que pueden ver en los comentarios del post original.


