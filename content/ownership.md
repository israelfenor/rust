+++
title = "Ownership: o cómo se gestiona la memoria en Rust (parte I)"
summary = "Este apunte surge de mi necesidad de querer entender bien las bases de cómo Rust gestiona la memoria para seguir avanzando en el aprendizaje de este lenguaje. Siento que si esta parte no la comprendo lo acabaré pagando."
slug = "ownership-gestion-de-memoria-rust"
date = "2021-03-06"

+++

> Aviso: Este apunte aún está inacabado, lo iré actualizando según voy investigando y aprendiendo, por tanto puede contener errores de todo tipo.

Esta es una lista de palabras inglesas muy utilizadas en el ámbito de la gestión de la memoria en Rust con la traducción que he utilizado para este documento.

* ***Allocation***: Asignar.
* ***Bind***: Enlazar.
* ***Borrowing***: Préstamo.
* ***Borrows***: Pedir prestado.
* ***Copy***: Copiar.
* ***Dereference***: *Desreferencia*. No he encontrado una palabra mejor en español y me he inventado ésta.
* ***Drop***: Soltar.
* ***Dropped***: Soltado.
* ***Free***: Liberar.
* ***Garbage collection***: Recolección de basura.
* ***Heap***: Montón.
* ***Lifetime***: Tiempo de vida.
* ***Manual memory allocation***: Asignación manual de memoria.
* ***Owner***: Propietario.
* ***Ownership***: Propiedad.
* ***Pointer***: Puntero.
* ***Reference***: Referencia.
* ***Runtime***: Entorno de ejecución.
* ***Scope***: Ámbito.
* ***Size***: Tamaño.
* ***Stack***: Pila.
* ***Unbind***: Desligar.

Para poder entender ***cómo*** se gestiona la memoria en Rust antes es necesario conocer, aunque sea de una manera superficial, cómo se usa la memoria de un ordenador.

## Variables y datos

Siempre imaginé una variable como una caja donde se guarda un dato. Y que esa caja era una porción de memoria.

A esa caja se le podían ir guardando datos, unos reemplazando a los otros y cuando no necesitaba más esos datos, "destruía" esa caja, esa variable, y listos.

Frases que estaban en mi cabeza como "a la variable num se le asigna el valor 1" o "num vale 1", sumado a la sintaxis que utilizan muchos lenguajes de programación para declarar variables y asignarles un dato, me hacían pensar que primero existe la variable y luego el dato que se guarda en ella.

Pero lo que sucede es ligeramente diferente. Primero está el dato y luego está la variable. Primero el dato se guarda en una porción de la memoria y luego se crea una variable que se enlaza \(_bind_\) con esa porción de memoria. Ese enlace consiste en una dirección de memoria, que al igual que en un callejero, nos permite saber dónde de toda la memoria disponible está guardado el dato.

Cuando declaramos una variable y le asignamos un dato, estamos enlazando la variable al dato. De la misma manera que cuando pasamos una variable como parámetro a una función no pasamos el dato de una caja a otra, sino que enlazamos la variable que recibe el parámetro a ese dato. Lo mismo sucede con el retorno de funciones, enlazamos el dato de retorno a la variable que espera ese dato de retorno.

El cambio es sutil, pero el concepto de enlace es muy útil para entender ciertos aspectos de la gestión de la memoria que a continuación.

## Pila y montón

La pila \(_stack_\) y el montón \(_heap_\) son dos tipos de memoria donde podemos almacenar datos.

La pila tiene una estructura, justamente como su nombre indica, de pila. En la pila se guardan los datos uno encima del otro y se quitan de uno en uno empezando por el último que se puso. A este funcionamiento se le llama [LIFO](https://es.wikipedia.org/wiki/Last_in%2C_first_out), _Last In, First Out_. Podemos imaginarlo como una pila de libros, complicado quitar el libro que hay más abajo sin quitar antes los que hay encima.

El montón no tiene una estructura fija tan "estricta" como la pila, es más un espacio. En la que los datos se van guardando allí donde hay espacio libre. Siguiendo con la analogía anterior, podríamos decir que es una estantería donde podemos poner algunos libros continuos y otros no. Unos en un estante y otros en otro.

Qué va en la pila y qué va en el montón depende, generalizando, del tipo del dato que queremos almacenar y más precisamente del tamaño \(_size_\) de la porción de memoria necesaria para almacenar ese dato.

Todo dato que requiera de una cantidad de memoria que es conocida en tiempo de compilación y no cambiará a lo largo de la ejecución del programa, se almacena en la pila, y cuando esa cantidad es desconocida en tiempo de compilación o cambiará a lo largo de la ejecución del programa, se almacena en el montón.

* Se guardan en la pila, por ejemplo: enteros, flotantes, booleanos, caracteres, punteros...
* Se guardan en el montón, por ejemplo: cadenas de texto, listas, vectores...

Veamos un ejemplo de dato almacenado en la pila:

```rust
let i: i32 = 10;
// El dato 10 es almacenado en la pila ya que conocemos la cantidad de bytes
// necesarios para almacenar ese dato (el mismo que para almacenar cualquier
// dato soportado por el tipo i32, ya sea un 200 o un 1239)
```

```text
      +----+---+
Pila  | 10 | i |
      +----+---+
```

Ahora veamos un ejemplo de dato almacenado en el montón:

```rust
let texto: String = String::from("Hola, mundo");
// El dato "Hola, mundo" es almacenado en el montón ya que la variable cadena al
// ser de tipo String es susceptible de cambiar su tamaño si se cambia su contenido.
// Es diferente el tamaño necesario para almacenar "Hola, mundo" que
// para almacenar "Adiós mundo cruel"
```

La variable `texto` se guarda en memoria de la siguiente manera: en el montón se guarda el dato \(en este caso la cadena de texto\) y en la pila se almacena un puntero \(_pointer_\) a ese espacio en el montón junto con la capacidad de ese espacio y el tamaño del dato.

```text
              puntero
            /    capacidad
           /    /     tamaño
          /    /     /   variable
         /    /     /   /
       +---+----+----+-------+
Pila   | * | 13 | 12 | texto |
       +-|-+----+----+-------+
         |
       [-|----------- capacidad -----------------------]
         |
       +-v-+---+---+---+---+---+---+---+---+---+---+---+
Montón | H | o | l | a | , |   | m | u | n | d | o |   |
       +---+---+---+---+---+---+---+---+---+---+---+---+

       [-------------- tamaño ---------------------]
```

La manera en cómo se almacenan y se borran los datos en el montón determina cómo gestiona la memoria un lenguaje de programación y por tanto cómo se programa en ese lenguaje.

## Gestión de la memoria

Todos los lenguajes de programación transfieren al programador, en mayor o menor medida, la gestión de la memoria del montón, que principalmente se refiere a la responsabilidad de almacenar datos en memoria ocupando memoria libre \(_allocation\)_ y borrar esos datos cuando ya nos son necesarios, liberando la memoria ocupada \(_free_\).

Generalizando, esa gestión puede ser de dos maneras:

* mediante un **recolector de basura** \(_garbage collector_\), donde el programador no tiene que pensar ni preocuparse dónde ni cómo los datos son almacenados ni borrados, ya que es el propio entorno de ejecución \(_runtime_\) del lenguaje el que se preocupa por ti. Lenguajes como Javascript o Java entre muchos funcionan de esta manera.
* mediante la **asignación manual de memoria** \(_Manual memory allocation_\), en la que la gestión completa de la memoria recae sobre el programador. Lenguajes como C y C++ funcionan de esta manera. Es el programador quien tiene que especificar cómo y dónde almacenar los datos y cuando borrarlos.

El recolector de basura facilita la vida al desarrollador a costa de una pérdida de rendimiento y de control. Mediante la asignación manual de memoria tienes el rendimiento y control, a cambio de una mayor complejidad de código.

Pero existe una tercera manera de gestionar la memoria. La forma en que lo hace Rust. Mediante la propiedad \(_ownership_\) y los préstamos \(_borrowing_\).

## Propiedad

En Rust, todo dato tiene un único propietario \(_owner_\). Ser propietario de un dato implica ser el único que puede acceder al dato y determina el tiempo \(_lifetime_\) en el que el dato permanece en la memoria y puede ser accedido y manipulado.

### La propiedad empieza con una asignación

Asignar un dato a una variable \(por tanto enlazar la variable a ese dato\) hace que esa variable sea la propietaria de ese dato.

```rust
fn main () {
    let num: i32 = 10;
    println!("El valor de num es: {}", num);
}
// El dato 10 está enlazado a la variable num.
// La variable num es la propietaria del dato 10.
```

### La propiedad termina con el ámbito

Cuando termina el ámbito \(_scope\)_ de una variable, se rompe el enlace \(_unbind_\) entre la variable y el dato del que es propietaria y comporta el borrado automático del dato de la memoria \(y la liberación de esa porción de memoria\). En Rust se dice que el dato es soltado \(_dropped_\).

```rust
fn main () {
    {
        let num: i32 = 10;
        println!("Este es el ámbito de num y su valor es: {}", num);
    }
    println!("Esto está fuera del ámbito de num y su valor es: {}", num);
}
// La declaración de la variable num ocurre dentro de un bloque delimitado entre {}.
// El ámbito de la variable num es ese bloque de código.

// Una vez se sale del ámbito, el dato enlazado con la variable num (un 10) es 
// borrado de la memoria (es soltado).

// No se puede usar la variable num fuera de su ámbito por tanto no se puede 
// acceder al dato 10 más allá de ese ámbito. La sentecia: 
// println!("Esto está fuera del ámbito de num y su valor es: {}", num); 
// da un error de compilación.
```

### La propiedad se mueve con el cambio de asignación

Asignar una variable a otra transfiere la propiedad del valor de una a la otra y comporta la eliminación de la variable propietaria original.

```rust
fn main () {
    let hola: String = String::from("Hola, mundo");
    let saludo = hola;

    println!("El valor de saludo es: {}", saludo);
}
// El resultado de compilar y ejecutar este código es:
// "El valor de saludo es: Hola, mundo"
```

La propiedad del dato "Hola, mundo" ha pasado de la variable `hola` a la variable `saludo`.

Tras el cambio de propiedad, la variable `hola` deja de existir y ya no se puede usar para acceder al dato "Hola, mundo" y es `saludo` la que sí que puede acceder, ya que ahora es su propietaria. Esto podemos verlo en el siguiente código:

```rust
fn main () {
    let hola: String = String::from("Hola, mundo");
    let saludo = hola;

    println!("El valor de hola es: {} y el valor de saludo es: {}", hola, saludo);
}
```

Si compilamos el código anterior obtenemos lo siguiente:

```rust
let hola: String = String::from("Hola, mundo");
    ---- move occurs because `hola` has type `String`, which does not implement the `Copy` trait
 let saludo = hola;
              ---- value moved here

 println!("El valor de hola es: {} y el valor de saludo es: {}", hola, saludo);
                                                                 ^^^^ value borrowed here after move

// Sin entrar en detalle, el compilador nos está diciendo en las líneas 2 y 4, 
// dónde la propiedad del dato se ha movido de la variable hola a la variable saludo. 
// En la línea 7 nos dice dónde se ha intentado usar la variable hola
// que ya ha dejado de existir y por tanto no puede acceder al dato.
```

A diferencia de otros lenguajes de programación en Rust `let saludo = hola` no hace que `saludo` y `hola` estén enlazadas con el mismo dato. O que `saludo` tenga una copia del dato al que está enlazada `hola`. Debido al mecanismo de la propiedad, el enlace con el dato pasa, automáticamente, de estar en `hola` para estar en `saludo`, su propiedad se ha transferido o mejor, como se dice en Rust, se ha movido.

Este cambio de asignación y por tanto el cambio de propietario también sucede cuando pasamos una variable como parámetro de una función.

```rust
fn main () {
    let hola: String = String::from("Hola, mundo");

    saludo(hola);

    println!("El valor de hola es: {}", hola);
}

fn saludo (texto: String) {
    println!("{}", texto);
}
```

Compilar el código anterior nos muestra el siguiente mensaje \(he eliminado expresamente parte del mensaje que nos da el compilador\):

```rust
let hola: String = String::from("Hola, mundo");
    ---- move
saludo(hola);
       ---- value moved here
  
println!("El valor de hola es: {}", hola);
                                    ^^^^
```

Por último, este cambio de asignación también ocurre cuando se retorna un dato de una función, pero en este caso puesto que al retornar un dato, el ámbito de la función se acaba y no podemos usar la variable que tiene la propiedad inicial, no nos encontraremos con los errores comentados.

> * Cada dato tiene una variable enlazada que es propietaria de ese dato
> * Solo puede haber un único propietario de un dato al mismo tiempo
> * Cuando se acaba el ámbito del propietario el dato es eliminado de la memoria

Que **la propiedad empieza con una asignación** y que **la propiedad termina con el ámbito**, son aplicables tanto para datos que se almacenan en la pila como en el montón. Pero que **la propiedad se mueve con el cambio de asignación**, funciona de diferente manera dependiendo de si los datos se almacenan en la pila o en el montón. Para entender esa diferencia hablaré de mover (*move*) y copiar (*copy*).

## Mover y copiar

Cuando asignamos una variable enlazada a un dato almacenado en la pila a otra variable, la segunda variable se enlaza a una copia del dato original. Quedando la primera variable enlazada y propietaria del dato original y la segunda variable enlazada y propietaria del dato copiado.

```rust
fn main () {
    let i: i32 = 10;
    let j: i32 = i;
    println!("El valor de i es: {}", i);
    println!("El valor de j es: {}", j);
}
```



## Enlaces de referencia

Un listado de todo aquello de lo que me he servido para aprender y poder escribir estos apuntes. Sincero agradecimiento a cada uno de sus autores.

* [https://tourofrust.com/chapter\_5\_es.html](https://tourofrust.com/chapter_5_es.html)
* [https://www.softax.pl/blog/rust-lang-in-a-nutshell-1-introduction/](https://www.softax.pl/blog/rust-lang-in-a-nutshell-1-introduction/)
* [https://fasterthanli.me/articles/i-am-a-java-csharp-c-or-cplusplus-dev-time-to-do-some-rust](https://fasterthanli.me/articles/i-am-a-java-csharp-c-or-cplusplus-dev-time-to-do-some-rust)
* [https://github.com/Dhghomon/easy\_rust/blob/master/README.md](https://github.com/Dhghomon/easy_rust/blob/master/README.md)
* [https://www.youtube.com/playlist?list=PLVvjrrRCBy2JSHf9tGxGKJ-bYAN\_uDCUL](https://www.youtube.com/playlist?list=PLVvjrrRCBy2JSHf9tGxGKJ-bYAN_uDCUL)
* [https://learning-rust.github.io/docs/c1.ownership.html](https://learning-rust.github.io/docs/c1.ownership.html)
* [https://blog.thoughtram.io/rust/2015/05/11/rusts-ownership-model-for-javascript-developers.html](https://blog.thoughtram.io/rust/2015/05/11/rusts-ownership-model-for-javascript-developers.html)
* [https://blog.thoughtram.io/ownership-in-rust/](https://blog.thoughtram.io/ownership-in-rust/)
* [https://blog.thoughtram.io/references-in-rust/](https://blog.thoughtram.io/references-in-rust/)
* [https://blog.thoughtram.io/lifetimes-in-rust/](https://blog.thoughtram.io/lifetimes-in-rust/)
* [https://www.tutorialspoint.com/rust/rust\_ownership.htm](https://www.tutorialspoint.com/rust/rust_ownership.htm)
* [https://depth-first.com/articles/2020/01/27/rust-ownership-by-example/](https://depth-first.com/articles/2020/01/27/rust-ownership-by-example/)
* [https://blog.logrocket.com/introducing-the-rust-borrow-checker/](https://blog.logrocket.com/introducing-the-rust-borrow-checker/)
* [https://dev.to/imaculate3/that-s-so-rusty-ownership-493c](https://dev.to/imaculate3/that-s-so-rusty-ownership-493c)
* [https://medium.com/@bugaevc/understanding-rust-ownership-borrowing-lifetimes-ff9ee9f79a9c](https://medium.com/@bugaevc/understanding-rust-ownership-borrowing-lifetimes-ff9ee9f79a9c)
* [https://willcrichton.net/notes/rust-memory-safety/](https://willcrichton.net/notes/rust-memory-safety/)
* [https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html](https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html)
* [https://blog.skylight.io/rust-means-never-having-to-close-a-socket/](https://blog.skylight.io/rust-means-never-having-to-close-a-socket/)
* [https://medium.com/@thomascountz/ownership-in-rust-part-1-112036b1126b](https://medium.com/@thomascountz/ownership-in-rust-part-1-112036b1126b)
* [https://medium.com/@thomascountz/ownership-in-rust-part-2-c3e1da89956e](https://medium.com/@thomascountz/ownership-in-rust-part-2-c3e1da89956e)
