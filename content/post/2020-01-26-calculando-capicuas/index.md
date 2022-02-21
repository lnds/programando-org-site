---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Calculando Capicuas"
subtitle: "solución al desafío"
summary: ""
authors: [admin]
tags: [desafíos, capicúas, "programación funcional", "desafio20200126"]
categories: [desafíos, "programación funcional", "patrones", "patrones funcionales"]
date: 2020-01-26T15:27:41-03:00
lastmod: 2020-01-26T15:27:41-03:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "Imagen oroginal De Coso, CC BY-SA 3.0, https://commons.wikimedia.org/w/index.php?curid=1554831"
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
Ha llegado el momento de revisar la solución [al desafío de la semana pasada](/blog/2020/01/19/desafio-capicuas.html).

Recordemos que el objetivo es encontrar las capicúas de los números del 12 al 99.

Al plantear el desafío dijimos:

> Se obtiene una capicúa de un número sumando el número con su reverso hasta obtener una capicúa, por ejemplo, para el número 57:
>
>    57 + 75 = 132 + 231 = 363

Este desafío recibió dos soluciones, la de Rodrigo C. Gauzman, quien implementó dos soluciones, [una en PHP](https://github.com/rodrigore/desafio-capicuas) y [otra visual usando SwiftUI](https://github.com/rodrigore/desafio-capicuas-swiftui).

La segunda es de Denis Fuenzalida en Clojure: https://gist.github.com/dfuenzalida/4b26711cd1345391714cb61bfbdbb960

Ambas soluciones son correctas, pero Rodrigo usa una técnica muy común, que es convertir el número a string y después invertir el string.

Esa técnica si bien es válida, es ineficiente en términos computacionales. Una mejor forma de calcular el reverso de un número es usando operaciones aritméticas.

La siguiente es una manera de hacerlo:

```
// esto es seudo código
func reverse(num: int) -> int 
    var reversed = 0;
    var n = num;
    while n > 0:
        reversed = 10 * reversed + n % 10;
        n = n / 10;
    return reversed;
```

Un compilador puede optimizar esto y generar código muy eficiente para calcular el número reverso. Además convertir números a string puede fallar, sobretodo si usamos lenguajes dinámicos.

## Una solución funcional

Pero avancemos. Voy a proponerles una solución que use patrones propios de la programación funcional para resolver esto. La razón de hacerlo, es que aparte de ser más divertido e interesante, me permite sembrar las semillas para el próximo desafío.

Usaré Rust como lenguaje para resolver este ejercicio, pero esto se puede hacer en casi cualquier lenguaje que soporte el uso de la monad Option y tenga una buena implementación de "tail recursion".

Lo primero es crear una función que calcule el número reverso del parámetro que recibe.


```rust
fn reverse(num: u64) -> u64 {
    
    fn calc_capicua(num: u64, rev: u64) -> u64 {
        match num {
            0 => rev,
            _ => calc_capicua(num / 10, 10 * rev + num % 10),
        }
    }

    calc_capicua(num, 0)
}
```

Acá usamos una función recursiva, esto implementa el mismo loop que describimos antes usando un while.

Luego se trata de encontrar las capicúas

```rust
fn encontrar_capicua(num: u64) -> Option<(u64, u64)> {
    let rev = reverse(num);
    let sum = num + rev;
    if sum == reverse(sum) {
        Some((num, sum))
    } else {
        encontrar_capicua(sum).map(|(_, sum)| (num, sum))
    }
}
```

Para encontrar la capicua de un número debemos tomar este y sumarle su número reverso. Eso se chequea en la primera parte del if:

```rust
let rev = reverse(num);
let sum = num + rev;
if sum == reverse(sum) {
    Some((num, sum))
}
```

Si la suma del número con su reverso son capicúa entonces devolvemos el par del número con su suma. Que es lo que buscamos.

Si no se da esa condición debemos iterar con la suma, y eso es lo que hacemos en el else:

```rust
 } else {
    encontrar_capicua(sum).map(|(_, sum)| (num, sum))
}
```

Noten que lo que hacemos es que la función ```encontrar_capicua()``` desciende recursivamente, siempre devolviendo un Option con la tupla del número y su capicúa. Como esa recusión nos devolverá un número distinto en la tupla, lo que hacemos es que tomamos el último valor y reemplazamos el primer valor de la tupla por el número original.

Esto puede ser confuso la primera vez que lo ven, pero es un patrón funcional bastante útil. Les sugiero revisarlo y probarlo hasta entender cómo funciona.

Con esto el último paso es nuestro loop de 12 a 98 para encontrar lo que se nos pide.

Dado que ```encontrar_capicua()``` retorna un option usaremos esta propiedad para mapear cada valor y no tener que usar loops:

```rust
fn main() {
    (12..99)
        .flat_map(|i| encontrar_capicua(i))
        .for_each(|(n, c)| {
            println!("capicua de {} es {}", n, c);
        });
}
```

La solución completa está en GitHub en esta dirección: https://github.com/lnds/desafios-programando.org/tree/master/2020-01-26/capicuas y es la siguiente:


```rust
fn main() {
    (12..99)
        .flat_map(|i| encontrar_capicua(i))
        .for_each(|(n, c)| {
            println!("capicua de {} es {}", n, c);
        });
}

fn encontrar_capicua(num: u64) -> Option<(u64, u64)> {
    let rev = reverse(num);
    let sum = num + rev;
    if sum == reverse(sum) {
        Some((num, sum))
    } else {
        encontrar_capicua(sum).map(|(_, sum)| (num, sum))
    }
}

fn reverse(num: u64) -> u64 {
    fn calc_capicua(num: u64, rev: u64) -> u64 {
        match num {
            0 => rev,
            _ => calc_capicua(num / 10, 10 * rev + num % 10),
        }
    }
    calc_capicua(num, 0)
}

```

## Un nuevo desafío

La razón por la que este desafío es desde el 12 al 98 es que no se sabe si todos los números tienen capicúa. En particular para el número 196 aún no se encuentra la capicúa. 

El desafío que sigue requiere contar cuantas iteraciones se necesitan para calcular cada capicúa.

Recordemos el caso del 57:

    57 + 75 = 132 + 231 = 363

Para este caso la cantidad e iteraciones es 2.

Para el caso del 43:

  43 + 34 = 77

Tenemos 1 iteración.

Para el 77:

  77 + 77 = 154 + 451 = 605 + 506 = 1111

Tenemos 3 iteraciones.

Entonces, el desafío es encontrar el número con la mayor cantidad de iteraciones para determinar su capicúa, para los números entre 100 y 1000. Para efectos de este ejercicio, se deben descartar todos aquellos números en que la cantidad de  iteraciones excedan el millón.

Si hay más de 5 participantes diferentes tendremos premio a la mejor solución. 
El criterio será la brevedad del programa y sea el más rápido en ejecutar.
