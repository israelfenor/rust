---
description: O cómo se gestiona la memoria en Rust
---

# Ownership, borrowing y lifetimes

Esta es una lista de palabras inglesas muy utilizadas en el ámbito de la gestión de la memoria en Rust con la traducción que he utilizado para este artículo.

* _**Borrowing**_: Préstamo.
* _**Borrows**_: Pedir prestado.
* _**Dereference**_: "Desreferencia". No he encontrado una palabra mejor en español y me he inventado ésta.
* _**Drop**_: Soltar. Pero en este artículo usaré la palabra **liberar** creo que se entiende mejor.
* _**Dropped**_: Caído. Pero en este artículo usaré la palabra **liberado** creo que se entiende mejor.
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

## _Ownership_, _Borrowing_ & _Lifetime_: Propiedad, préstamo y tiempo de vida

Para poder entender cómo se gestiona la memoria en Rust antes es necesario conocer, de una manera muy superficial, cómo funciona la memoría de un ordenador.

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
// dato soportado por el tipo i32)
```

```text
      +----+
Pila  | 10 |
      +----+
```

Ahora veamos un ejemplo de datos almacenado en el montón:

```rust
let mut cadena: &str = "Hola, mundo";
// El dato "Hola, mundo" es almacenado en el montón ya que la variable cadena al
// ser de tipo &str cambiará su tamaño si se cambia su contenido. Es diferente el
// tamaño necesario para almacenar "Hola, mundo" que para almacenar "Hasta luego"
```

La variable `cadena` se guarda en memoria de la siguiente manera: en el montón se guarda el contenido de la variable y en la pila se almacena un puntero \(_pointer_\) a ese espacio en el montón junto con la capacidad de ese espacio y el tamaño del contenido.

```text
            puntero
           /    capacidad
          /    /     tamaño
         /    /     /
       +---+----+----+
Pila   | * | 13 | 12 | cadena
       +-|-+----+----+
         |
      [--|------------capacidad------------------------]
         |
       +-v-+---+---+---+---+---+---+---+---+---+---+---+
Montón | H | o | l | a | , |   | m | u | n | d | o |   |
       +---+---+---+---+---+---+---+---+---+---+---+---+

       [---------------tamaño----------------------]
```

TO DO: Aquí falta hablar de cómo se borran los datos de la pila y el montón

Y es según la manera en cómo se almacena y se borran datos en el montón la que determina, principalmente, cómo se gestiona la memoria en un lenguaje de programación.

### Gestión de la memoria

Todos los lenguajes de programación transfieren al programador, en mayor o menor medida, la responsabilidad de gestionar la memoria. Con esto me refiero principalmente a almacenar datos \(ocupando memoria libre\) y liberar la memoria ocupada cuando los datos que almacena ya no son necesarios.

Esa gestión puede ser de dos maneras:

* mediante un **recolector de basura** \(_garbage collector_\), donde el programador no tiene que pensar ni preocuparse dónde ni cómo los datos son almacenados ni de liberar la memoria. De eso se encarga el propio entorno de ejecución \(_runtime_\) del lenguaje. Lenguajes como PHP, Python, Javascript o Java entre muchos funcionan de esta manera.
* mediante la **asignación manual de memoria** \(_Manual memory allocation_\), en la que la gestión completa de la memoria recae sobre el programador. Lenguajes como C y C++ funcionan de esta manera.

El recolector de basura facilita la vida al desarrollador, a costa de una pérdida de rendimiento y control. Mediante la asignacion manual de memoria tienes rendimiento y control, pero a cambio tienes una mayor complejidad.

Pero hay una tercera manera, que es como lo hace Rust, que ni usa un recolector de basura ni una asignación manual.

#### Bibliografía

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

