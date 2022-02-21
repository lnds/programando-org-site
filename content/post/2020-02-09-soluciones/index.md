---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Soluciones"
slug: "soluciones"
subtitle: ""
summary: ""
authors: [admin]
tags: []
categories: []
date: 2020-02-09T08:02:05-03:00
lastmod: 2020-02-09T08:02:05-03:00
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

En este post se encuentran las soluciones a los dos últimos desafíos. Espero que los próximos desafíos vayan atrayendo más participantes, quizás haya que colocar incentivos mayores :wink:

## Primer desafío: Calculando Capicúas

Publicado hace dos semanas, se trata de calcular el número entre 100 y 1000 que requiere el máximo número de iteraciones para calcular su capicúa (tal como la definimos en ese [mismo post](/blog/2020/01/26/calculando-capicuas.html)).

Participaron dos personas, Denis Fuenzalida con una [solución en Clojure](https://gist.github.com/dfuenzalida/efcc554552be626bab08d22a39195de0) y Rodrigo Tobar con [una solución en C](https://gist.github.com/rtobar/ae6f7cee231bb0bb9bf3c95559f95b88).

La solución de Denis es bastante breve, unas 20 líneas de código y encuentra el número correcto, que corresponde al 880, cuya capicúa es 8813200023188.

La solución en C de Rodrigo es mucho más extensa porque hay que lidiar con el overflow que se produce al usar tipos de datos nativos (de 64 bits, por ejemplo). Son casi 150 líneas de código.

Lo que pasa es que en este ejercicio es muy fácil que si usamos tipos nativos de datos para los enteros tendremos overflow en las operaciones aritméticas, esto nos puede llevar a falsos positivos.

Esto es muy interesante, porque en algunos lenguajes este tipo de overflows no se notan o producen excepciones fatales. Así que muy bien Rodrigo por darse cuenta de esto. Por desgracia no he tenido el tiempo para verificar el comportamiento de la solución de Rodrigo, así que si él lee este post, por favor agradeceré que indique los resultados.

Bueno, veamos mi solución. Para resolver esto me basé en el código que usamos la vez anterior para calcular las capicúas del 11 al 99. La primera diferencia es que modifiqué la función ```reverse```:

```rust
type Int = u64;

fn reverse(num: Int) -> Option<Int> {
    fn calc_capicua(num: Int, rev: Int) -> Option<Int> {
        if num == 0 {
            Some(rev)
        } else {
            calc_capicua( num.checked_div(10)?,  
            ( rev.checked_mul(10)? ).checked_add( num.checked_rem(10)? ) ?)
        }
    }
    calc_capicua(num, 0)
}
```

He definido el tipo Int como un alias de u64, lo que me permite cambiar el tipo de datos en futuros experimentos.

Noten que la función ahora retorna un ```Option<Int>```, esto es así porque usaremos _"checked operators"_, que es la forma que tenemos en Rust para lidiar con el control de los overflow. Por ejemplo, en vez de hacer ```a + b```, hacemos ```a.checked_add(b)```. Estas operaciones retornan un Option, de modo que si se produce un overflow el resultado es siempre ```None```.

Lo malo de esto es que el código es más dificil de leer.

Otra cosa que quiero que noten es el uso del operador ? de Rust. Es una abreviación. Cuando tenemos ```expresion?```, si expresión es None entonces la función retorna None de inmediato. 

Por ejemplo:

```rust
  let a = reverse(10)?;
```

es equivalente a

```rust
  let a = match reverse(10) {
    None => return None,
    Some(b) => b
  };
```

Con esto, he modificado mi función para calcular la capicú del siguiente modo:

```rust
const LIMIT: Int = 1_000_000;

fn encontrar_capicua(num: Int, n: Int) -> Option<(Int, Int, Int)> {
    let rev = reverse(num)?;
    let sum = num + rev;
    if n > LIMIT {
        None
    } else if sum == reverse(sum)? {
        Some((num, sum, n))
    } else {
        encontrar_capicua(sum, n + 1).map(|(_, sum, n)| (num, sum, n))
    }
}
```

Ahora recibe un argumento ```n``` que es la cantidad de iteraciones.
Si n excede el límite retornamos ```None```.
El resto es muy similar, salvo que ahora tenemos un argumento más, y en el resultado agregamos las iteraciones.

Con todo esto la solución final queda así:

```rust
type Int = u64;

fn main() {
    if let Some((num, c, n)) = (100..=1000)
        .flat_map(|i| encontrar_capicua(i, 1))
        .max_by(|(_, _, a), (_, _, b)| a.cmp(b))
    {
        println!("num = {}, su capicua es {}, iteraciones: {}", num, c, n);
    }
}

const LIMIT: Int = 1_000_000;

fn encontrar_capicua(num: Int, n: Int) -> Option<(Int, Int, Int)> {
    let rev = reverse(num)?;
    let sum = num + rev;
    if n > LIMIT {
        None
    } else if sum == reverse(sum)? {
        Some((num, sum, n))
    } else {
        encontrar_capicua(sum, n + 1).map(|(_, sum, n)| (num, sum, n    ))
    }
}

fn reverse(num: Int) -> Option<Int> {
    fn calc_capicua(num: Int, rev: Int) -> Option<Int> {
        if num == 0 {
            Some(rev)
        } else {
            calc_capicua(num.checked_div(10)?, (rev.checked_mul(10)?).checked_add(num.checked_rem(10)?)?)
        }
    }
    calc_capicua(num, 0)
}
```

La solución en Rust toma unas 35 líneas de código y maneja correctamente el overflow, y es bastante eficiente en tiempo de ejecución.


## Buscando la Gran Capicúa

El segundo problema [fue enunciado la semana pasada](/blog/2020/02/02/feliz-dia-de-la-gran-capicua.html) con ocasión del 2 de febrero de 2020.

La idea es buscar fechas entre el año 1 y el 9999 que sean similares al 2 de febrero de 2020, es decir, sean capicúa expresados como mes/día/año, año/mes/día o día/mes/año.

Sólo participó en esta ocasión Denis Fuenzalida, con una solución en Clojure como siempre. Su solución está acá: https://gist.github.com/dfuenzalida/9a159fc46104687e84be25a9a8377e9a

Efectivamente no considera bisiestos, no tampoco todos los formatos posibles que rellenen con cero.

Mi solución es de fuerza bruta y escrita muy a la rápida, sólo considera rellenar con cero los tres números, o simplemente expresar los tres números sin relleno.

Este es el código:

```rust
fn main() {
    let days1 = vec![0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
    let days2 = vec![0, 31, 29, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
    for year in 1..10_000 {
        for (month, days) in (if is_leap(year) { &days1 } else { &days2 }).iter().enumerate().take(12+1).skip(1) {
            for day in 1..=*days {
                if check_zero_filled(year, month, day as usize) {
                    println!(
                        "year={}, month={}, day={}, f1: {}, f2: {}, f3: {}",
                        year,
                        month,
                        day,
                        format_date_1(year, month, day),
                        format_date_2(year, month, day),
                        format_date_1(year, day as usize, month)
                    );
                }
                else if check_unfilled(year, month, day) {
                    println!(
                        "year={}, month={}, day={} f1: {}, f2: {}, f3: {}",
                        year,
                        month,
                        day,
                        format_date_3(day, month, year),
                        format_date_3(month, day, year),
                        format_date_3(year, month, day)
                    );
                }
            }
        }
    }
}

fn is_leap(year: usize) -> bool {
    year % 4 == 0 && year % 100 != 0 || year % 400 == 0
}

fn check_zero_filled(year: usize, month: usize, day: usize) -> bool {
    let s = format_date_1(year, month, day);
    if s == s.chars().rev().collect::<String>() {
        let t = format_date_2(year, month, day);
        if t == t.chars().rev().collect::<String>() {
            let r = format_date_2(year, day, month);
            if r == r.chars().rev().collect::<String>() {
                return true;
            }
        }
    }
    false
}

fn check_unfilled(year: usize, month: usize, day: usize) -> bool {
    let s = format_date_3(year, month, day);
    if s == s.chars().rev().collect::<String>() {
        let t = format_date_3(day, month, year);
        if t == t.chars().rev().collect::<String>() {
            let r = format_date_3(month, day, year);
            if r == r.chars().rev().collect::<String>() {
                return true;
            }
        }
    }
    false
}

fn format_date_1(year: usize, month: usize, day: usize) -> String {
    if year < 100 {
        format!("{:02}{:02}{:02}", year, month, day)
    } else if year < 1000 {
        format!("{:03}{:02}{:02}", year, month, day)
    } else {
        format!("{:04}{:02}{:02}", year, month, day)
    }
}

fn format_date_2(year: usize, month: usize, day: usize) -> String {
    if year < 100 {
        format!("{:02}{:02}{:02}", day, month, year)
    } else if year < 1000 {
        format!("{:02}{:02}{:03}", day, month, year)
    } else {
        format!("{:02}{:02}{:04}", day, month, year)
    }
}

fn format_date_3(year: usize, month: usize, day: usize) -> String {
    format!("{}{}{}", year, month, day)
}

```

Según esto hay 62 fechas en 10.000 años que cumplen esta característica.

  year=1, month=1, day=1 f1: 111, f2: 111, f3: 111
  year=1, month=1, day=11 f1: 1111, f2: 1111, f3: 1111
  year=1, month=11, day=1 f1: 1111, f2: 1111, f3: 1111
  year=1, month=11, day=11 f1: 11111, f2: 11111, f3: 11111
  year=2, month=2, day=2 f1: 222, f2: 222, f3: 222
  year=2, month=2, day=22 f1: 2222, f2: 2222, f3: 2222
  year=3, month=3, day=3 f1: 333, f2: 333, f3: 333
  year=4, month=4, day=4 f1: 444, f2: 444, f3: 444
  year=5, month=5, day=5 f1: 555, f2: 555, f3: 555
  year=6, month=6, day=6 f1: 666, f2: 666, f3: 666
  year=7, month=7, day=7 f1: 777, f2: 777, f3: 777
  year=8, month=8, day=8 f1: 888, f2: 888, f3: 888
  year=9, month=9, day=9 f1: 999, f2: 999, f3: 999
  year=11, month=1, day=1 f1: 1111, f2: 1111, f3: 1111
  year=11, month=1, day=11 f1: 11111, f2: 11111, f3: 11111
  year=11, month=11, day=1 f1: 11111, f2: 11111, f3: 11111
  year=11, month=11, day=11, f1: 111111, f2: 111111, f3: 111111
  year=22, month=2, day=2 f1: 2222, f2: 2222, f3: 2222
  year=22, month=2, day=22 f1: 22222, f2: 22222, f3: 22222
  year=33, month=3, day=3 f1: 3333, f2: 3333, f3: 3333
  year=44, month=4, day=4 f1: 4444, f2: 4444, f3: 4444
  year=55, month=5, day=5 f1: 5555, f2: 5555, f3: 5555
  year=66, month=6, day=6 f1: 6666, f2: 6666, f3: 6666
  year=77, month=7, day=7 f1: 7777, f2: 7777, f3: 7777
  year=88, month=8, day=8 f1: 8888, f2: 8888, f3: 8888
  year=99, month=9, day=9 f1: 9999, f2: 9999, f3: 9999
  year=111, month=1, day=1 f1: 11111, f2: 11111, f3: 11111
  year=111, month=1, day=11 f1: 111111, f2: 111111, f3: 111111
  year=111, month=11, day=1 f1: 111111, f2: 111111, f3: 111111
  year=111, month=11, day=11, f1: 1111111, f2: 1111111, f3: 1111111
  year=222, month=2, day=2 f1: 22222, f2: 22222, f3: 22222
  year=222, month=2, day=22 f1: 222222, f2: 222222, f3: 222222
  year=333, month=3, day=3 f1: 33333, f2: 33333, f3: 33333
  year=444, month=4, day=4 f1: 44444, f2: 44444, f3: 44444
  year=555, month=5, day=5 f1: 55555, f2: 55555, f3: 55555
  year=666, month=6, day=6 f1: 66666, f2: 66666, f3: 66666
  year=777, month=7, day=7 f1: 77777, f2: 77777, f3: 77777
  year=888, month=8, day=8 f1: 88888, f2: 88888, f3: 88888
  year=999, month=9, day=9 f1: 99999, f2: 99999, f3: 99999
  year=1010, month=1, day=1, f1: 10100101, f2: 01011010, f3: 10100101
  year=1111, month=1, day=1 f1: 111111, f2: 111111, f3: 111111
  year=1111, month=1, day=11 f1: 1111111, f2: 1111111, f3: 1111111
  year=1111, month=11, day=1 f1: 1111111, f2: 1111111, f3: 1111111
  year=1111, month=11, day=11, f1: 11111111, f2: 11111111, f3: 11111111
  year=2020, month=2, day=2, f1: 20200202, f2: 02022020, f3: 20200202
  year=2121, month=12, day=12, f1: 21211212, f2: 12122121, f3: 21211212
  year=2222, month=2, day=2 f1: 222222, f2: 222222, f3: 222222
  year=2222, month=2, day=22 f1: 2222222, f2: 2222222, f3: 2222222
  year=3030, month=3, day=3, f1: 30300303, f2: 03033030, f3: 30300303
  year=3333, month=3, day=3 f1: 333333, f2: 333333, f3: 333333
  year=4040, month=4, day=4, f1: 40400404, f2: 04044040, f3: 40400404
  year=4444, month=4, day=4 f1: 444444, f2: 444444, f3: 444444
  year=5050, month=5, day=5, f1: 50500505, f2: 05055050, f3: 50500505
  year=5555, month=5, day=5 f1: 555555, f2: 555555, f3: 555555
  year=6060, month=6, day=6, f1: 60600606, f2: 06066060, f3: 60600606
  year=6666, month=6, day=6 f1: 666666, f2: 666666, f3: 666666
  year=7070, month=7, day=7, f1: 70700707, f2: 07077070, f3: 70700707
  year=7777, month=7, day=7 f1: 777777, f2: 777777, f3: 777777
  year=8080, month=8, day=8, f1: 80800808, f2: 08088080, f3: 80800808
  year=8888, month=8, day=8 f1: 888888, f2: 888888, f3: 888888
  year=9090, month=9, day=9, f1: 90900909, f2: 09099090, f3: 90900909
  year=9999, month=9, day=9 f1: 999999, f2: 999999, f3: 999999

Con esto cerramos estos dos desafíos. Estén atentos para nuevos artículos y desafíos.

Las soluciones se encuentran en mi repositorio en GitHub: https://github.com/lnds/desafios-programando.org/tree/master/2020-02-09

Esta semana no habrá desafío.