---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Juguemos Poker (Segunda Parte)"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-01-05T20:40:39-03:00
lastmod: 2020-01-05T20:40:39-03:00
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

Llegó el momento de terminar nuestro ejercicio [planteado la semana pasada](/blog/2019/12/29/juguemos-poker.html).


Recordemos los pasos que planteamos para resolver este mini proyecto:

1. Modelar las manos de Poker
2. Definir cómo representaremos las manos de poker.
3. Definir las reglas que permiten determinar un ganador.
4. Crear un motor que revuelva y reparta las cartas.
5. Crear una interfaz de usuario en modo consola que permita jugar poker.

Los pasos 1, 2 y 3 los resolvimos en el post anterior, y la solución se encuentra en este repositorio: https://github.com/lnds/desafios-programando.org/tree/master/2019-12-29/poker

Ahora vamos a ejecutar los dos pasos que nos faltan.

## Programando el _front end_

Para esto, he creado otra biblioteca que nombre cartas, y que se encuentra en este repo: https://github.com/lnds/desafios-programando.org/tree/master/2020-01-05/poker-rust/cartas.

Van a notar que hay un grado de duplicidad con respecto al proyecto presentado la semana pasada. Se podría simplificar y eso puede ser un ejercicio interesante para ustedes. Pero decidí escribir toda la capa del _"front end"_ totalmente aislada de la capa de back, aunque haya algo de redundancia. 

Si esto fuera un proyecto entre dos personas, esta sería una buena manera de dividirlo. Uno desarrollador trabaja en los pasos 1, 2 y 3, mientras que el otro trabaja en los pasos 4 y 5.
Hay un momento en que se integrarán, a través de una única interfaz pública, algo que veremos más adelante.

Nuestras estructuras internas en el crate `cartas` son las siguiente:

```rust
#[derive(Clone, Debug, IntoEnumIterator, PartialEq, Eq, PartialOrd, Ord, Copy)]
pub enum Pinta {
    Picas,
    Corazones,
    Diamantes,
    Treboles,
}


#[derive(Clone, Debug, PartialEq, Eq, PartialOrd, Ord, Copy)]
pub enum Color {
    Rojo,
    Negro,
}

#[derive(Clone, Debug, IntoEnumIterator, PartialEq, Eq, PartialOrd, Ord, Copy)]
pub enum Orden {
    As,
    Dos,
    Tres,
    Cuatro,
    Cinco,
    Seis,
    Siete,
    Ocho,
    Nueve,
    Diez,
    Jack,
    Queen,
    King,
}

#[derive(Clone, Debug, PartialEq, Eq, PartialOrd, Ord, Copy)]
pub struct Naipe {
    pub orden: Orden,
    pub pinta: Pinta,
    pub color: Color,
}
```

Esto contiene las estructuras básicas. El motor que revuelve y reparte cartas está en la estructura Baraja:

```rust
pub struct Baraja(Vec<Naipe>);

impl Baraja {
    pub fn new() -> Self {
        let mut naipes = iproduct!(Pinta::into_enum_iter(), Orden::into_enum_iter())
            .map(|(pinta, orden)| Naipe::new(orden.clone(), pinta.clone()))
            .into_iter()
            .collect::<Vec<Naipe>>();
        naipes.reverse();
        Baraja(naipes)
    }

    pub fn tomar(&mut self) -> Option<Naipe> {
        self.0.pop()
    }

    pub fn barajar(&mut self) {
        let mut rng = rand::thread_rng();
        self.0.shuffle(&mut rng);
    }

    pub fn repartir(&mut self, n: usize) -> Mano {
        Mano::new((0..n).flat_map(|_| self.tomar()).collect())
    }

    pub fn contar(&self) -> usize {
        self.0.len()
    }
}
```

Usamos la biblioteca [rand](https://crates.io/crates/rand) para poder revolver nuestros naipes, lo que se usa en la función `barajar()`. La función `tomar()` extrae una a una las cartas. La función `repartir()` retorna una `Mano` que corresponde a esta estructura:

```rust
#[derive(Debug, PartialEq)]
pub struct Mano(Vec<Naipe>);

impl Mano {
    fn new(naipes: Vec<Naipe>) -> Self {
        Mano(naipes.iter().sorted().cloned().collect())
    }

    pub fn valor(&self) -> String {
        self.0
            .iter()
            .map(|n| n.valor())
            .collect::<Vec<String>>()
            .join(" ")
    }

    pub fn cambiar(&mut self, pos: usize, naipe: Naipe) {
        self.0[pos] = naipe;
    }
}
```

La función `cambiar()` se usa en el juego para poder cambiar un naipe de la mano.

La otra función importante es `valor()`, que transforma una mano en un string que será usado por la función `manos_ganadoras()` que desarrollamos la semana pasada en la biblioteca `poker` (que pueden ver acá: https://github.com/lnds/desafios-programando.org/blob/master/2019-12-29/poker/src/lib.rs).


## Integración

La integración de nuestro front end con nuestro back end se realiza a través de la siguiente función:

```rust
pub fn mano_ganadora<'a>(mano1: &'a Mano, mano2: &'a Mano) -> Option<&'a Mano> {
    let m1 = &mano1.valor();
    let m2 = &mano2.valor();
    let resultado = poker::manos_ganadoras(&[m1, m2])?;
    if resultado.len() == 2 {
        return None;
    }
    if resultado[0] == mano1.valor() {
        return Some(&mano1);
    }
    return Some(&mano2);
}
```

La función original `poker::manos_ganadoras()` de nuestro backend permite el empate, en este caso permitimos sólo una mano ganadora. Fíjense en el uso de la función `valor()`, que transforma cada mano al formato de string necesario para ser usado por la función `poker::manos_ganadoras()`.

## La interfaz de usuario

La interfaz de usuario es la linea de comandos, y para esto usamos la consola. Para hacer más atractiva la interacción usamos el crate `colored` (https://crates.io/crates/colored). Es por esto que nuestra biblioteca Cartas implementa el trait Display para casi todas las estructuras:

```rust

impl fmt::Display for Naipe {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        let naipe = format!("{}{:>2.2}", self.pinta, self.orden);
        let naipe_decorado = if self.color == Color::Rojo {
            naipe.red()
        } else {
            naipe.black()
        };
        write!(f, "{}", naipe_decorado.bold().on_white())
    }
}

impl fmt::Display for Pinta {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        let icon = match self {
            &Pinta::Picas => "♠",
            &Pinta::Corazones => "♥",
            &Pinta::Diamantes => "♦",
            &Pinta::Treboles => "♣",
        };
        write!(f, "{:<1}", icon)
    }
}
```

De este modo las cartas de corazones y diamantes aparecen de color rojo en la consola, además de usar los íconos respectivos.

Todo el resto de la interacción con el usuario se encuentra escrita en otro proyecto rust, en este caso no se trata de una biblioteca, como `poker` y `cartas`. Se trata de un proyecto que implementa una aplicación y se llama `poker_consola`: https://github.com/lnds/desafios-programando.org/tree/master/2020-01-05/poker-rust/poker-consola.

El código de la aplicación se encuentra en este archivo: https://github.com/lnds/desafios-programando.org/blob/master/2020-01-05/poker-rust/poker-consola/src/main.rs.

No lo analizaremos en detalle, pero acá se encuentran todas las funciones que interactúan con el usuario, y se apoya en las otras dos bibliotecas: `poker` y `cartas`.

## Rust WorkSpaces

Si revisan el archivo `Cargo.toml` del proyecto `poker-consola` encontrarán lo siguiente:

```rust
[dependencies]

cartas = { path = "../cartas" }
colored = "1.9.1"
```

Por otro lado, el archivo `Cargo.toml` del proyecto `cartas` contiene lo siguiente:

```rust
[dependencies]
poker = { path = "../../../2019-12-29/poker" }

colored = "1.9.1"
enum-iterator = "0.5.0"
rand = "0.7.2"
itertools = "0.8.2"
```

Esta es la forma en que creamos una dependencia que se encuentra dentro de nuestro disco duro en un proyecto [Cargo](https://doc.rust-lang.org/cargo/).

Para que todo esto trabaje en conjunto, he creado un [Cargo Workspace](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html), el que se encuentra en la raiz de este repo:
https://github.com/lnds/desafios-programando.org/tree/master/2020-01-05/poker-rust

Esto se hace con un archivo `Cargo.toml` con la siguiente sintaxis:

```rust
[workspace]

members = [
	"cartas",
	"poker-consola",
]

```

Fíjense que no se hace referencia a "poker". Lo correcto seria que poker esté en el mismo nivel que `cartas` y `poker-consola`. Como no quiero copiar el codigo, ni tampoco mover la carpeta, decidí dejarlo así. No hay manera de crear un Cargo Workspace referenciando una carpeta externa a la raiz de donde se encuentra el archivo Cargo.toml, al menos no en la actual versión de Rust (y no tendría mucho sentido).

Esto es parte de la gestión de código que permite la herramienta Cargo y que es uno de los aspectos más interesantes de Rust, la organización del código nos permite tener una gestión bastante ordenada de los módulos, lo que es útil cuando trabajamos en proyectos grandes.
    
Cuando ejecutas el programa se ve más o menos así:

![](/blog/2020/01/05/juguemos-poker-segunda-parte/images/sesion.png)


Y con esto damos por terminado este mini proyecto, espero que les despierte la curiosidad por aprender Rust. Seguiré escribiendo sobre este lenguaje, pero los invito a leer su documentación oficial, estos dos enlaces son buenas introducción para todo lo que hemos visto:

El libro, "The Rust Programming Language": https://doc.rust-lang.org/book/

El libro "The Cargo Book": https://doc.rust-lang.org/cargo/

Nos vemos la próxima semana con un nuevo desafío, espero sus comentarios.