---
description: O cómo se gestiona la memoria en Rust
---

# Ownership, borrowing y lifetimes

Para empezar voy a listar una serie de palabras inglesas muy utilizadas en el ámbito de la gestión de la memoria en Rust con la traducción que he utilizado para este artículo.

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
* _**Runtime**_: Entorno de ejecución
* _**Scope**_: Ámbito.
* _**Size**_: Tamaño.
* _**Stack**_: Pila.

## _Ownership_, _Borrowing_ & _Lifetime_: Propiedad, préstamo y tiempo de vida

### Gestión de la memoria

Todos los lenguajes de programación transfieren al programador, en mayor o menor medida, la responsabilidad de gestionar la memoria. Con esto me refiero principalmente a almacenar datos \(ocupando memoria libre\) y liberar la memoria ocupada cuando los datos que almacena ya no son necesarios.

Esa gestión puede ser de dos maneras:

* mediante un **recolector de basura** \(_garbage collector_\), donde el programador no tiene que pensar ni preocuparse dónde ni cómo los datos son almacenados ni de liberar la memoria. De eso se encarga el propio entorno de ejecución \(_runtime_\) del lenguaje. Y es una comodidad para el programador. Lenguajes como PHP, Python, Ruby, Javascript, Go o Java entre muchos funcionan de esta manera.
* mediante la **asignación manual de memoria** \(_Manual memory allocation_\), en la que la gestión completa de la memoria recae sobre el programador y por tanto requiere un mayor esfuerzo. Lenguajes como C y C++ funcionan de esta manera.

Pero hay una tercera manera, que es como lo hace Rust, que ni usa un recolector de basura ni una asignación manual.

#### _Stack_ y _Heap_: Pila y montón

La pila y el montón son partes de la memoria donde se pueden almacenar datos.

Como su nombre ya nos indica, en la pila se guardan los datos "uno encima del otro" y se quitan de uno en uno empezando por el último que se puso. A esto se le llama [LIFO](https://es.wikipedia.org/wiki/Last_in%2C_first_out), _Last In, First Out_ _y_ lo que viene a decir es que el último en entrar es el primero en salir. Pensemos en una pila de libros, complicado quitar el libro que hay más abajo sin quitar antes los que hay encima.

El montón no tiene una estructura fija tan "estricta" como la pila, es más un espacio, pero podríamos decir que es algo parecido a una lista. Siguiendo con la analogía anterior, podríamos decir que es una estantería dónde vamos poniendo los libros.

Qué va en la pila y qué va en el montón depende del tipo del dato que queremos almacenar. Generalizando diremos que todo dato que tenga un tamaño \(_size_\) fijo o su tamaño sea conocido en tiempo de compilación se almacena en la pila, y los datos que no tengan un tamaño fijo o éste sea desconocido, se almacenan en el montón.

Cuando hablamos de tamaño nos referimos a la cantidad de bytes necesarios para almacenar el dato.

* Se guardan en la pila, por ejemplo: enteros, flotantes, booleanos, caracteres, punteros...
* Se guardan en el montón, por ejemplo: cadenas de texto, listas, vectores...

Por detalles técnicos que quedan fuera del ámbito de este artículo, almacenar datos en el montón requiere más tiempo que en la pila, ya que el sistema debe encontrar un espacio de memoria libre suficientemente grande para almacenar esos datos.

El programa no puede tener un acceso directo a los datos almacenados en el montón, como sí lo tiene a los de la pila. Cada vez que se almacena un dato en el montón, el programa se guarda un puntero \(_pointer_\) a ese espacio en la pila.

```rust
let i: i32 = 10;
// El dato 10 es almacenado en la pila ya que conocemos la cantidad de bytes
// necesarios para almacenar ese dato
```

```text
      +----+
Pila  | 10 |
      +----+
```

```rust
let cadena: String = String::from("Hola, mundo"); // Este dato (Hola, mundo) es almacenado en el montón
```

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
Montón | H | o | l | a |   | , | m | u | n | d | o |   |
       +---+---+---+---+---+---+---+---+---+---+---+---+

       [---------------tamaño----------------------]
```

La variable `cadena` se guarda en memoria de la siguiente manera: en el montón se guarda el contenido de la variable y en la pila se almacena un puntero al espacio reservado en el montón para guardar el contenido de la variable, junto con la capacidad de ese espacio y el tamaño del contenido.

#### Bibliografía

Un listado de todo aquello de lo que me he servido para aprender y poder escribir este documento. Sincero agradecimiento a cada uno de sus autores.

* [https://blog.adrianistan.eu/2017/07/03/referencias-prestamos-en-rust/](https://blog.adrianistan.eu/2017/07/03/referencias-prestamos-en-rust/)
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

