---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Juguemos Poker"
subtitle: ""
summary: ""
authors: [admin]
tags: []
categories: []
date: 2019-12-29T13:35:32-03:00
lastmod: 2019-12-29T13:35:32-03:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

header:
  image: "featured.jpg"

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

Este año me he dedicado a aprender el [lenguaje de programación Rust](https://www.rust-lang.org/) en profundidad. Tal como han apreciado en los desafíos previos, mis soluciones han sido presentadas en este lenguaje.

Incluso hasta me he vuelto una especie de evangelista de Rust, medio en broma, medio en serio. Salvo [en una charla](https://www.meetup.com/Santiago-Scala-Meetup/photos/30258503/), donde hice una breve introducción, y los desafíos que he abordado en La Naturaleza del Software, he abordado muy superficialmente mi fascinación por este lenguaje de programación. Así que voy a remediar eso con algunos posts al respecto.

![](/blog/2019/12/29/juguemos-poker/images/image1.jpg)

Me parece que una buena forma de entender un lenguaje es implementando algún juego simple, que podamos sofisticar con cada iteración. Así que voy a escribir un par de post en que implementaré una interfaz de linea de comandos para jugar Poker. 

Espero con esto exponer algunos de los aspectos del sistema de tipos de Rust y la manera en que este lenguaje aborda el paradigma funcional. Así que vamos con esto.

## Organización de este proyecto

Para implementar este juego de poker seguiremos los siguientes pasos:

1. Modelar las manos de Poker
2. Definir cómo representaremos las manos de poker.
3. Definir las reglas que permiten determinar un ganador.
4. Crear un motor que revuelva y reparta las cartas.
5. Crear una interfaz de usuario en modo consola que permita jugar poker.

Este post abordará los primeros tres pasos. La próxima semana abordaremos los otros dos.

## Un sistema de tipos para el Poker

Lo primero que haremos, antes de implementar el juego propiamente tal, es construir un sistema de tipos que nos permita modelar las reglas que permiten determinar quien gana una partida de Poker.

Para simplificar la interfaz de usuario modelaremos nuestras manos de carta como un string, que contendrá cada carta separadas por espacios en blanco. Las cartas serán a su vez un string con el valor (A,2..10, J, Q, K) más la pinta (P, C, D, T) (♠, ♥, ♦, ♣). 

Así las manos se ingresarán de la siguiente manera a nuestro sistema:

    "AT 2D 10T 3T KC"
    "10D 9T 10T QT JD"

Lo primero que haremos es modelar nuestras cartas en forma interna, para eso usaremos este struct:

```rust
struct Carta(u8, char);
```

Esto es lo que se llama una [tuple struct](https://doc.rust-lang.org/stable/rust-by-example/custom_types/structs.html). La definimos como una tupla que contiene el valor expresado como un u8 (es decir un byte, un entero sin signo de 8 bits) y la pinta como un char.

Rust soporta en parte el paradigma orientado al objeto, así que podemos crear una `Carta` a partir de un string de la siguiente manera:

```rust
impl Carta {

    pub fn new(carta: &str) -> Option<Carta> {
        let valor = match &carta[0..carta.len() - 1] {
            "A" => 14,
            "J" => 11,
            "Q" => 12,
            "K" => 13,
            s => s.parse().ok().filter(|&x| x >= 2 && x <= 10)?,
        };
        let pinta = carta.chars().last().filter(|c| PINTAS.contains(c))?;
        Some(Carta(valor, pinta))
    }
}
```

Mediante la palabra reservado `impl` lo que hacemos es implementar métodos asociados a una struct. 

En este caso tenemos un patrón común que es usar el método static new para Carta, esto permite crear una estructura `Carta` a partir de strings. Esto es similar a un constructor.

Pero noten que estamos retornand `Option<Carta>`, la razón es que la transformación puede fallar, si ese el el caso queremos manejar el error usando la Option monad.

La estructura monádica `Option<T>` retorna dos valores: `None` o `Some(valor)`, donde valor es de tipo `T`..

Es lo que hacemoso en este caso, si tenemos problemas al parsear la función `Carta::new()` retornará None.

Lo primero que hacemos es determinar el valor haciendo:

```rust
let valor = match &carta[0..carta.len() - 1] {
    "A" => 14,
    "J" => 11,
    "Q" => 12,
    "K" => 13,
    s => s.parse().ok().filter(|&x| x >= 2 && x <= 10)?,
};
```

Noten como tomamos el string menos el último caracter en la expresión `&carta[0..carta.len()-1]`. Noten que al as (`"A"`) le damos el valor 14, esto permite resolver una regla que dice que el AS tiene más valor que otras cartas. 

La clave está en la expresión:

```rust
    s => s.parse().ok().filter(|&x| x >= 2 && x <= 10)?,
```

`s` contiene el string. El método parse() retorna un tipo `Result` (ver https://doc.rust-lang.org/std/result/). Como lo que queremos es un tipo Option, lo que hacemos es usar el método `Ok()` que convierte un `Result` en un `Option`. Luego aplico el método `filter()` para verificar que el valor parseado esté entre 2 y 10. Estas son operaciones monádicas, así que toda esa expresión retorna un `Option<u8>`. 

Pero noten que hay un signo de interrogación al final de la expresión. En Rust esto significa que si toda la expresión retorna un `None` entonces la función que contiene a la expresión retornará `None`. Es una suerte de atajo. 

Esto es equivalente a escribir:

```rust
match s.parse().ok().filter(|&x| x >= 2 && x <= 10)) {
  Some(val) => val,
  None => return None;
}
```

Usar el ? al final de expresiones que retornan `Option` o `Result` es muy común en Rust, es [syntactic sugar](https://en.wikipedia.org/wiki/Syntactic_sugar).

Con este antecedente podemos entender el ? al final de la siguiente expresión:

```rust
        let pinta = carta.chars().last().filter(|c| PINTAS.contains(c))?;
```

Acá usamos algunas características de los iteradores de Rust, que es propio del paradigma funcional.

Todo string en Rust tiene un método `chars()`, que retorna un iterador que recorre todos los caracteres de un string. El método `last()` de un iterador retorna el último elemento. Verificamos que ese último caracter tenga un valor válido para una pinta, chequeando que esté en el siguiente arreglo:

```rust
const PINTAS:[char;4] = ['P','C','T','D'];
```

Acá vemos como se declaran [constantes en Rust](https://doc.rust-lang.org/stable/rust-by-example/custom_types/constants.html). En este caso un arreglo de 4 caracteres.

Escribamos algunos tests para ver que implementamos bien la estructura `Carta`

```rust
#[test]
fn cartas() {
    assert!(Carta::new("11C").is_none());
    assert!(Carta::new("AP").is_some());
    let carta = Carta::new("KT");
    assert_eq!(carta.unwrap().0, 13);

    let cartas: Vec<Carta> = "AT 2D 10T 3T KC"
        .split_whitespace()
        .flat_map(|s| Carta::new(s))
        .collect();
    assert_eq!(cartas.len(), 5);

    let cartas: Vec<Carta> = "10D 9Z 10T QT JD"
        .split_ascii_whitespace()
        .flat_map(|s| Carta::new(s))
        .collect();
    assert_eq!(cartas.len(), 4);
}
```

### Clasificando las manos

Una mano de poker, de acuerdo a [este artículo en Wikipedia](https://es.wikipedia.org/wiki/Manos_de_p%C3%B3quer), puede clasificarse en:


- Escalera de Color
- Poker
- Full House
- Color
- Escalera
- Trio
- Doble Par
- Par
- Carta más alta

(El artículo menciona a la escala real, pero la omitiremos en este ejercicio).

Dado esto vamos a crear un tipo de datos enumerado para todos nuestras posibles manos:


```rust
#[derive(Debug, PartialEq, Eq, PartialOrd, Ord, Clone)]
enum Mano {
    Invalida,
    CartaAlta(u8, u8, u8, u8, u8),
    Par(u8, u8, u8, u8),
    DoblePar(u8, u8, u8),
    Trio(u8, u8, u8),
    Escala(u8),
    Color(u8),
    FullHouse(u8, u8),
    Poker(u8, u8),
    EscalaDeColor(u8),
}
```

Fíjense que cada enum tiene valores asociados, esto se debe a que para desempatar debemos considerar algunos valores adicionales. Por ejemplo, si tenemos los siguientes pares:

    p1: "2C 3T 4P 2D KT"
    p2: "3P 8D 2T JD 2P"

Ganará p1, puesto que tiene una carta más alta que es el rey de trébol (KT). De este modo, representaremos estas manos del siguiente modo:

    p1: Par(2, 13, 4, 3)
    p2: Par(2, 11, 8, 3)

Nuestra solución debe ser general y soportar varias barajas, es por esto que el caso de Poker almacenamos la carta más alta:

    p3: "4T JT JD JP JC" => Poker(11, 4)
    p4: "JT 8D JP JC JD" => Poker(11, 8)

En este caso, p4 es mayor que p3.

Dado que usamos un tipo enumerado podemos establecer una relación de orden, la que está dada por el orden en que se escriben los tipos, de este modo `Par` es menor que `Trio` y `Escalera` es menor que `FullHouse`.

Junto con esta definición debemos indicar que nuestro tipo enumerado `Mano` implemente ciertas operaciones de comparación, esto se logra mediante la anotación `#[derive()]`. En este caso le indicamos al compilador que queremos que implemente los traits PartialOrd, Ord, PartialEq y Eq.

Puedes aprender más sobre los traits acá: https://doc.rust-lang.org/stable/rust-by-example/trait.html

Lo importante es que con esto no tenemos que escribir código para comparar manos de poker, basta con clasificar una mano en alguno de estos tipos enumerados y listo. Y eso es lo que haremos a continuación.


```rust

impl Mano {
    fn new(cartas: &[Carta]) -> Self {
        if cartas.len() != 5 {
            return Mano::Invalida;
        }

        let grupos = cartas
            .iter()
            .sorted_by_key(|c| c.0)
            .group_by(|&c| c.0)
            .into_iter()
            .map(|(_, g)| g.map(|c| c.0).collect())
            .sorted_by_key(|v: &Vec<u8>| v.len())
            .collect::<Vec<_>>();
        let clasi = &grupos.iter().map(|v| v.len()).collect::<Vec<usize>>();
        match &clasi[..] {
            [_, 4] => Mano::Poker(grupos[1][0], grupos[0][0]),
            [2, 3] => Mano::FullHouse(grupos[1][0], grupos[0][0]),
            [_, _, 3] => Mano::Trio(grupos[2][0], grupos[1][0], grupos[0][0]),
            [_, 2, 2] => Mano::DoblePar(grupos[2][0], grupos[1][0], grupos[0][0]),
            [_, _, _, 2] => {
                Mano::Par(grupos[3][0], grupos[0][0], grupos[0][0], grupos[0][0])
            }
            _ => {
                // no encontramos grupos, así que vemos si tenemos escalas o color
                let cartas: Vec<Carta> = cartas
                    .iter()
                    .sorted_by(|&x, &y| y.cmp(&x))
                    .cloned()
                    .collect();
                let escala = cartas
                    .windows(2)
                    .all(|w| w[0].0 == w[1].0 + 1 || w[0].0 == 14 && w[1].0 == 5);
                let color = cartas.iter().all(|c| c.1 == cartas[0].1);

                if !escala && !color {
                    Mano::CartaAlta(cartas[0].0, cartas[1].0, cartas[2].0, cartas[3].0, cartas[4].0)
                } else {
                    // corrige caso en que tenemos un as
                    let max: u8 = cartas
                        .iter()
                        .map(|c| if c.0 == 14 { 1 } else { c.0 })
                        .max()
                        .unwrap();

                    if escala && color {
                        Mano::EscalaDeColor(max)
                    } else if escala {
                        Mano::Escala(max)
                    } else {
                        Mano::Color(max)
                    }
                }
            }
        }
    }
}
```

Lo que hacemos es que el constructor de `Mano` recibe un arreglo de cartas con la representación de la mano y realizamos la clasificación en el constructor.

Este código es bien complejo, así que vamos a desmenuzarlo paso a paso.

Lo primero es validar que una mano debe tener 5 cartas:

```rust
 if cartas.len() != 5 {
    return Mano::Invalida;
}
```

Lo segundo es agrupar:

```rust
let grupos = cartas
            .iter()
            .sorted_by_key(|c| c.0)
            .group_by(|&c| c.0)
            .into_iter()
            .map(|(_, g)| g.map(|c| c.0).collect())
            .sorted_by_key(|v: &Vec<u8>| v.len())
            .collect::<Vec<_>>();
````

Acá usamos un paquete (crate en la jerga de Rust) llamado `itertools`, que permite agrupar.

Recordemos que las cartas se definen así:

```rust
#[derive(Debug, PartialEq, PartialOrd, Eq, Ord, Clone)]
struct Carta(u8, char);
````

Noten que agregamos anotaciones para poder ordenar las cartas.

Como `Carta` es un tuple struct, para acceder a sus elementos usamos .0 para acceder al valor y .1 para acceder al color (o pinta) de la carta.

Entonces lo que hacemos es que obtenemos un iterador sobre el arreglo de cartas al hacer `cartas.iter()`.
Este iterador lo ordenamos en forma ascendente por el valor al hacer `.sorted_by_key(|c| c.0)`.
Luego agrupamos por valor de la carta haciendo `.group_by(|&c| c.0)`.
El operador `.into_iter()` es necesario por la forma en que opera itertools, no lo explicaré por ahora.

La función group_by crea tuplas consistente en el valor usado para agrupar y una lista o grupo con los valores. De este modo obtenemos los pares, trios, cuartetos, etc. Lo que queremos es que estos grupos queden almacenados como vectores que contengan sólo el valor de la carta (sin la pinta), eso lo  logramos haciendo: `.map(|(_, g)| g.map(|c| c.0).collect())`. 

Por último ordenamos estos vectores por el largo de cada uno, mediante la llamada a `.sorted_by_key(|v:&Vec<u8>| v.len())`. La llamada final `.collect::<Vec<_>>()` agrupa todo a un vector de vectores.

Por ejemplo, si nuestra entrada es esta:

    [Carta(2, 'D'), Carta(10,'C'), Carta(10, 'D'), Carta(2, 'T'), Carta(10, 'T')]

En grupos tendremos como resultado:

    [[2, 2], [10, 10, 10]]

Esto es un Full House, con un par de 2 y un par de 10.

El truco para clasificar es convertir el arreglo `grupos` a algo así:

    [2, 3]

Eso se logra con esta expresión:

```rust
  let clasi = grupos.iter().map(|v| v.len()).collect::<Vec<usize>>();
```

Simplemente mapeamos cada vector a su largo.

Dado esto se entiende el [match](https://doc.rust-lang.org/rust-by-example/flow_control/match.html):

```rust
  match &clasi[..] {
      [_, 4] => Mano::Poker(...),
      [2, 3] => Mano::FullHouse(...),
      [_, _, 3] => Mano::Trio(...),
      [_, 2, 2] => Mano::DoblePar(...),
      [_, _, _, 2] =>  Mano::Par(...),
      _ => {...}
  }  
```

El problema es cuando no tenemos grupos, que es lo que abordamos en la expresión del match que empeiza con `_ => `:

Primero ordenamos las cartas:

```rust
let cartas: Vec<Carta> = cartas
                    .iter()
                    .sorted_by(|&x, &y| y.cmp(&x))
                    .cloned()
                    .collect();
```

Este ordenamiente es de mayor a menor. Ignoren el método `cloned()` por ahora, tiene que ver con conceptos propios de Rust que investigaremos en otra oportunidad.

¿Cómo determinanos si hay una escala?

Esta es la expresión:

```rust
let escala = cartas
    .windows(2)
    .all(|w| w[0].0 == w[1].0 + 1 || w[0].0 == 14 && w[1].0 == 5);
```

Lo que hacemos es usar `.windows(2)` que genera un iterador que corresponde a ventas de 2 elementos en el arreglo, estas ventanas se van desplazando a lo largo del arreglo. Entonces verificamos si el elemento 0 de la ventanta es menor en 1 del siguiente elemento en la ventana. Hay que tener cuidado con el caso particular del AS (que vale 14), en esa situación hay un caso de borde con la siguiente configuración "A2345", que quedaría ordenado así: "A5432", y dado que A es 14, tendriamos este arreglo: [14,5,4,3,2], y no se cumple la primera condición. Eso explica la expresión: `|| w[0].0 == 14 && w[1].0 == 5`.


Determinar si hay color es mucho más fácil:

```rust
let color = cartas.iter().all(|c| c.1 == cartas[0].1);
```

Simplemente queremos que todas los colores (o pintas) sean iguales al color de la primera carta. Eso es lo que hace el método `all()` que requiere que se cumpla la condición para todos los elementos de un iterador.

Y así se entiend la parte final:

```rust
if !escala && !color {
    Mano::CartaAlta(
        cartas[0].0,
        cartas[1].0,
        cartas[2].0,
        cartas[3].0,
        cartas[4].0,
    )
} else {
    ...
}
```

Si no hay escala ni color devolvemos un mano de tipo CartaAlta, con todos los valores de las cartas (ordenados de mayor a menor).

De lo contrarior hay que clasificar el tipo de escala o color.

Fíjense en el caso e de las escalas debemos devolver un valor entre 1 y 13, para que la comparación funcione, es por esto que determinamos la variable max:

```rust
  let max: u8 = cartas
        .iter()
        .map(|c| if c.0 == 14 { 1 } else { c.0 })
        .max()
        .unwrap();

    if escala && color {
        Mano::EscalaDeColor(max)
    } else if escala {
        Mano::Escala(max)
    } else {
        Mano::Color(max)
    }
````

Y con esto tenemos todo lo necesario para determinar un ganador de una partida de poker.


### Determinando las manos ganadoras


Esto lo encapsularemos en la función `manos_ganadoras()`:

```rust
pub fn manos_ganadoras<'a>(manos: &[&'a str]) -> Option<Vec<&'a str>> {
    let manos = manos
        .iter()
        .map(|mano| {
            let mano_poker = Mano::new(
                mano.split_whitespace()
                    .flat_map(|carta| Carta::new(carta))
                    .collect::<Vec<Carta>>()
                    .as_slice()
            );

            (mano_poker, *mano)
        })
        .sorted_by(|x, y| y.cmp(&x))
        .collect::<Vec<(Mano, &'a str)>>();

    if manos.iter().any(|(mano, _)| *mano == Mano::Invalida) {
        None
    } else {
        let winner = &manos[0].0;
        Some(
            manos
                .iter()
                .take_while(|mano| mano.0 == *winner) // empates
                .map(|m| m.1)
                .collect(),
        )
    }
}
```

Esta función recibe un arreglo con manos expresados como strings, y retorna None si hay algún error, o un vector con todas las manos ganadoras (cosideramos el empate).

Fíjense que la función está declarada como pública mediante la keyword `pub`, la razón es porque esta será nuestra interfaz con el resto del juego. Todo el código que hemos escrito hasta ahora es interno, el resto del juego sólo opera con strings.

Lo primero que hacemos es convertir el arreglo de strings en manos de poker, la clave está en esta expresión:

```rust
let mano_poker = Mano::new(
    mano.split_whitespace()
        .flat_map(|carta| Carta::new(carta))
        .collect::<Vec<Carta>>()
        .as_slice()
);
```

Lo que hacemos acá es separar el string y pasar cada carta a `Carta::new()`.

De este modo si tenemos este string:

    "10D 9T 10T QT JD"

La expresión ejecuta estos pasos:

    "10D 9T 10T QT JD"
    -> ["10D", "9T", "10T", "QT", "JD"]
    -> [Carta(10,'D'), Carta(9, 'T'), Carta(10, 'T'), Carta(12, 'T), Carta(11, 'D')]


Como ordenamos el arreglo de manos

```rust
let manos = manos
    .iter()
    .map(|mano| {
        ...
        (mano_poker, *mano)
    })
    .sorted_by(|x, y| y.cmp(&x))
    .collect::<Vec<(Mano, &'a str)>>();
```

Esto deja la mano ganadora en el primer lugar.

Debemos verificar que no existan manos inválidas: 


```rust
if manos.iter().any(|(mano, _)| *mano == Mano::Invalida) {
        None
    } 
```


Por último, verificamos si hay empates, eso explica esta expresión:

```rust
let winner = &manos[0].0;
...
manos
  .iter()
  .take_while(|mano| mano.0 == *winner) // empates
  .map(|m| m.1)
  .collect()
```

Con esto ya tenemos lo necesario para armar nuestro juego, que veremos en el post de la siguiente semana.

## Ejercicios

¿Puedes escribir mejor código para esto?
¿Puedes escribirlo en otro lenguaje de programación?
∫ ¿Cuáles son las mayores diferencias?

## Notas

Este post está basado en mi solución al siguiente ejercicio encontrado en el sitio [Exercism.io](https://exercism.io/): https://exercism.io/tracks/rust/exercises/poker

Mi solución la pueden ver en este link: https://exercism.io/tracks/rust/exercises/poker/solutions/3e42a0f977a8483886cfd358ec575752

El código fuente para este desafío está acá: https://github.com/lnds/desafios-programando.org/tree/master/2019-12-29/poker


La foto es Kenny Rogers tomado del video [The Gambler](https://www.youtube.com/watch?v=7hx4gdlfamo)