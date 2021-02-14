---
description: O cómo se gestiona la memoria en Rust
---

# Ownership, borrowing y lifetimes

Esta es una lista de palabras inglesas muy utilizadas en el ámbito de la gestión de la memoria en Rust con la traducción que he utilizado para este documento.

* _**Allocation:**_ Asignar.
* _**Bind:**_ Enlazar.
* _**Borrowing**_: Préstamo.
* _**Borrows**_: Pedir prestado.
* _**Dereference**_: "Desreferencia". No he encontrado una palabra mejor en español y me he inventado ésta.
* _**Drop**_: Soltar.
* _**Dropped**_: Soltado.
* _**Free**_: Liberar.
* _**Garbage collection**_: Recolección de basura.
* _**Heap**_: Montón.
* _**Lifetime**_: Tiempo de vida.
* _**Manual memory allocation**_: Asignación manual de memoria.
* _**Owner**_: Propietario.
* _**Ownership**_: Propiedad.
* _**Pointer**_: Puntero.
* _**Reference**_: Referencia.
* _**Runtime**_: Entorno de ejecución.
* _**Scope**_: Ámbito.
* _**Size**_: Tamaño.
* _**Stack**_: Pila.
* _**Unbind:**_ Desligar.

## _Ownership_, _Borrowing_ & _Lifetime_: Propiedad, préstamo y tiempo de vida

Para poder entender cómo se gestiona la memoria en Rust antes es necesario conocer, de una manera muy superficial, cómo se usa la memoria de un ordenador.

### Variables y valores

Siempre pensé en una variable como en una caja donde se guarda un valor, y esa caja era un trocito de memoria.

A la variable se le podían ir asignando valores, unos reemplazando a los otros y cuando no necesitaba más esa variable, esa caja, la destruía y listos.

Frases que están en mi cabeza como "a la variable num se le asigna el valor 1" o "num vale 1", sumado a la sintaxis que utilizan muchos lenguajes de programación para declarar variables y darles un valor, me hacían pensar que primero estaba la variable y luego el valor que se guardaba en ella.

Pero lo que sucede es ligeramente diferente. Primero está el valor y luego está la variable. Primero el valor se guarda en la memoria y luego se crea una variable que se enlaza \(_bind_\) con ese valor \(realmente enlaza con la dirección de la memoria en la que guarda el valor\). 

Cuando pasamos una variable como parámetro a una función no pasamos el valor de una caja a otra, sino que enlazamos la variable que recibe el parámetro a ese valor. Las funciones no retornan variables, retornan los valores para que estos sean enlazados a la variable que recibe el resultado de la función.

El cambio es sutil, pero el concepto de enlace es muy útil para entender ciertos aspectos de la gestión de la memoria que veremos más adelante.

### Pila y montón

La pila y el montón son partes de la memoria donde podemos almacenar datos.

La pila tiene una estructura, justamente como su nombre indica, de pila. En la pila se guardan los datos "uno encima del otro" y se quitan de uno en uno empezando por el último que se puso. A esto se le llama [LIFO](https://es.wikipedia.org/wiki/Last_in%2C_first_out), _Last In, First Out_. Pensemos en una pila de libros, complicado quitar el libro que hay más abajo sin quitar antes los que hay encima.

El montón no tiene una estructura fija tan "estricta" como la pila, es más un espacio. En la que los datos se van guardando allí donde hay espacio libre. Siguiendo con la analogía anterior, podríamos decir que es una estantería donde podemos poner algunos libros continuos y otros no. Unos en un estante y otros en otro.

Qué va en la pila y qué va en el montón depende, generalizando, del tipo del dato que queremos almacenar. Más precisamente del tamaño \(_size_\) de la memoria necesario para almacenar ese dato.

Todo dato que requiera de una cantidad de memoria conocida en tiempo de compilación se almacena en la pila, y cuando esa cantidad sea desconocida en tiempo de compilación, se almacena en el montón.

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

Ahora veamos un ejemplo de datos almacenado en el montón:

```rust
let texto: String = String::from("Hola, mundo");
// El dato "Hola, mundo" es almacenado en el montón ya que la variable cadena al
// ser de tipo String cambiará su tamaño si se cambia su contenido.
// Es diferente el tamaño necesario para almacenar "Hola, mundo" que
// para almacenar "Hasta luego"
```

La variable `texto` se guarda en memoria de la siguiente manera: en el montón se guarda el dato \(en este caso la cadena de texto\) y en la pila se almacena un puntero \(_pointer_\) a ese espacio en el montón junto con la capacidad de ese espacio y el tamaño del contenido.

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
      [--|------------capacidad------------------------]
         |
       +-v-+---+---+---+---+---+---+---+---+---+---+---+
Montón | H | o | l | a | , |   | m | u | n | d | o |   |
       +---+---+---+---+---+---+---+---+---+---+---+---+

       [---------------tamaño----------------------]
```

Dependiendo de la manera en cómo se almacena y se borran datos en el montón  determina, principalmente, cómo se gestiona la memoria en un lenguaje de programación.

### Gestión de la memoria

Todos los lenguajes de programación transfieren al programador, en mayor o menor medida, la responsabilidad de gestionar la memoria. Principalmente almacenar datos ocupando memoria libre \(_allocation\)_ y borrar esos datos cuando ya nos son necesarios, liberando la memoria ocupada \(_free_\).

Esa gestión puede ser de dos maneras:

* mediante un **recolector de basura** \(_garbage collector_\), donde el programador no tiene que pensar ni preocuparse dónde ni cómo los datos son almacenados ni borrados. De eso se encarga el propio entorno de ejecución \(_runtime_\) del lenguaje. Lenguajes como PHP, Python, Javascript o Java entre muchos funcionan de esta manera.
* mediante la **asignación manual de memoria** \(_Manual memory allocation_\), en la que la gestión completa de la memoria recae sobre el programador. Lenguajes como C y C++ funcionan de esta manera. Es el programador quien tiene que especificar cómo y dónde almacenar los datos y cuando borrarlos.

El recolector de basura facilita la vida al desarrollador a costa de una pérdida de rendimiento y de control. Mediante la asignación manual de memoria tienes el rendimiento y control, a cambio de una mayor complejidad de código.

Pero existe una tercera manera de gestionar la memoria. La forma en que lo hace Rust, mediante la propiedad \(_ownership_\) y los préstamos \(_borrowing_\).

### Propiedad

En Rust, todo valor tiene un único propietario \(_owner_\). Ser propietario de un valor implica ser el único que puede acceder al valor y determina el tiempo \(_lifetime_\) en el que el valor permanece en la memoria y puede ser accedido y manipulado.

#### La propiedad empieza con una asignación

Asignar un valor a una variable \(por tanto enlazar la variable a ese valor\) hace que esa variable sea la propietaria de ese valor.

```rust
fn main () {
    let num: i32 = 10;
    println!("El valor de num es: {}", num);
}
// El valor 10 está enlazado a la variable num.
// La variable num es la propietaria del valor 10.
```

#### La propiedad acaba con el ámbito

Cuando termina el ámbito \(_scope\)_ de una variable, se rompe el enlace \(_unbind_\) entre la variable y el valor del que es propietaria y comporta el borrado del valor de la memoria \(también se dice que se libera la memoria\). En Rust se dice que el valor es soltado \(_dropped_\).

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

// Una vez se sale del ámbito, el valor enlazado con la variable num (10) es 
// borrado de la memoria (es soltado).

// No se puede usar la variable num fuera de su ámbito ni acceder al valor 10 más
// allá de ese ámbito. La sentecia: 
// println!("Esto está fuera del ámbito de num y su valor es: {}", num); 
// da un error de compilación.
```

#### La propiedad cambia con la reasignación

Asignar una variable a otra transfiere la propiedad del valor de una a la otra.

```rust
fn main () {
    let hola: String = String::from("Hola, mundo");
    let saludo = hola;

    println!("El valor de saludo es: {}", saludo);
}
// El resultado de compilar y ejecutar este código es:
// "El valor de saludo es: Hola, mundo"
```

La propiedad del dato \(Hola, mundo\) ha pasado de la variable `hola` a la variable `saludo`.

La variable `hola` ya no puede acceder al dato y es `saludo` la que sí que puede acceder, ya que ahora es su propietaria. Esto podemos verlo en el siguiente código:

```rust
fn main () {
    let hola: String = String::from("Hola, mundo");
    let saludo = hola;

    println!("El valor de hola es: {} y el valor de saludo es: {}", hola, saludo);
}
```

Si compilamos el código anterior obtenemos lo siguiente:

```rust
  |
2 |     let hola: String = String::from("Hola, mundo");
  |         ---- move occurs because `hola` has type `String`, which does not implement the `Copy` trait
3 |     let saludo = hola;
  |                  ---- value moved here
4 | 
5 |     println!("El valor de hola es: {} y el valor de saludo es: {}", hola, saludo);
  |                                                                     ^^^^ value borrowed here after move

```

Del mensaje del compilador por ahora obviemos _occurs because \`hola\` has type \`String\`,_ _which does not implement the \`Copy\` trait_ y quedémonos con:

```rust
let hola: String = String::from("Hola, mundo");
    ---- move
let saludo = hola;
             ---- value moved here
println!("El valor de hola es: {} y el valor de saludo es: {}", hola, saludo);
                                                                ^^^^
```

El compilador nos está diciendo dónde \(líneas 2 y 4\) la propiedad del dato se ha movido de la variable `hola` a la variable `saludo`y dónde \(línea 5\) se ha intentado usar la variable `hola` \(que ya no puede acceder al dato\).

Esta reasignación y por tanto el cambio de propietario también sucede cuando pasamos una variable como parámetro de una función.

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

Compilar el código anterior nos muestra el siguiente mensaje \(sigamos obviando algunos detalles\):

```rust
2 |     let hola: String = String::from("Hola, mundo");
  |         ---- move
3 |     
4 |     saludo(hola);
  |            ---- value moved here
5 |     
6 |     println!("El valor de hola es: {}", hola);
  |                                         ^^^^
```

Por último, esta reasignación también ocurre cuando se retorna un valor de una función, pero en este caso puesto que al retornar un valor, el ámbito de la función se acaba y no podemos usar la variable que tiene la propiedad inicial, no nos encontraremos con estos errores.

{% hint style="info" %}
* Cada valor tiene una variable enlazada que llamamos propietario
* Solo puede haber un único propietario de un valor al mismo tiempo
* Cuando se acaba el ámbito del propietario el valor es soltado
{% endhint %}



### Enlaces de referencia

Un listado de todo aquello de lo que me he servido para aprender y poder escribir este documento. Sincero agradecimiento a cada uno de sus autores.

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

