+++
title = "Borrowing: o cómo se gestiona la memoria en Rust (parte II)"
summary = "En esta segunda entrega de una serie de anotaciones que estoy haciendo respecto la gestión de la memoria en Rust trato de explicar y entender cómo funciona el Copiar, el Mover y los Préstamos."
slug = "borrowing-gestion-de-memoria-rust"
date = "2021-03-06"
draft = true

+++

> Este apunte es la segunda entrega de una serie de anotaciones que estoy haciendo al respecto de la gestión de la memoria en Rust.
>
> * [Parte 1: Ownership](../ownership-gestion-de-memoria-rust)

> Puesto que no soy ningún experto en la materia aviso que este escrito puede contener errores. Como sigo estudiando y aprendiendo lo iré corrigiendo y ampliando. También aviso que he simplificado algunas secciones para facilitar alguna explicación y mi propio entendimiento.

Esta es una lista de palabras inglesas muy utilizadas en el ámbito de la gestión de la memoria en Rust con la traducción que he utilizado para este documento.

* ***Borrowing***: Préstamo.
* ***Borrows***: Pedir prestado.
* ***Derived***: Derivado.
* ***Owner***: Propietario.
* ***Ownership***: Propiedad.
* ***Reference***: Referencia.
* ***Struct***: Estructura.
* ***Trait***: Rasgo.

## Copiar y mover

En el [anterior apunte](../ownership-gestion-de-memoria-rust) comenté que cuando asignamos una variable a otra pueden suceder dos cosas:

1. si el tipo del dato **implementa** el rasgo (*trait*) *Copy*, se crea una copia en memoria del dato original y se enlaza a la nueva variable.
2. si el tipo del dato **no implementa** el rasgo *Copy*, se rompe el enlace con la actual variable y se crea un enlace con la nueva, haciendo que la propiedad (*ownership*) se mueva de una variable a la otra.

En el primer caso podemos seguir accediendo al dato original mediante la variable original, pero en el segundo caso no podemos acceder al dato original mediante la variable original, ya que perdió su propiedad, y solo podemos hacerlo a través de la segunda variable (la nueva propietaria).

En este escenario nos puede surgir una pregunta, ¿hay alguna manera de seguir usando el dato y la variable original tras una asignación?. La respuesta es, sí, **implementando el rasgo *Copy*** o mediante los **préstamos** (*borrowing*).

## Implementar el rasgo Copy

Implementar el rasgo *Copy* significa que a partir de ese momento cuando se realize una asginación, en vez de que se mueva la propiedad se hagan copias. ¿Fácil?. Sí, con un pero. Ya que solo podemos hacer que implementen el rasgo *Copy* los tipos creados mediante estructuras (structs). Si el tipo *Vector* de Rust no implementa el rasgo *Copy*, no hay manera de hacer que lo implemente.

```rust
struct Persona {
    edad: i32
}

fn main () {
    let mut amparo = Persona { edad: 40 };
    
    mostrar_edad(amparo);
    
    amparo.edad = 41;
    
    mostrar_edad(amparo);
}

fn mostrar_edad(persona: Persona) {
    println!("Esta persona tiene {} años", persona.edad);
}

// Al ejecutar este código obtendremos el, ya conocido, resultado:

let mut amparo = Persona { edad: 40 };
      ---------- move occurs because `amparo` has type `Persona`, which does not implement the `Copy` trait

mostrar_edad(amparo);
             ------ value moved here

amparo.edad = 41;
^^^^^^^^^^^^^^^^ value partially assigned here after move
```

Implementemos ahora el rasgo *Copy* de la siguiente manera:

```rust
#[derive(Clone, Copy)]
struct Persona {
    edad: i32
}

fn main () {
    let mut amparo = Persona { edad: 40 };
    
    mostrar_edad(amparo);
    
    amparo.edad = 41;
    
    mostrar_edad(amparo);
}

fn mostrar_edad(persona: Persona) {
    println!("Esta persona tiene {} años", persona.edad);
}

// Al ejecutar este código obtendremos el siguiente resultado:

Esta persona tiene 40 años
Esta persona tiene 41 años
```

Es muy sencillo, simplemente añadiendo *#[derive(Clone, Copy)]* antes de la declaración de la estructura, implementamos el rasgo *Copy* y el dato pasa de moverse (junto con la propiedad) a copiarse.

Y ahora nos puede surgir otra pregunta, ¿por qué no hacemos que todo se copie?. Primero que no podemos hacer que los tipos que proporciona Rust que no implementan el rasgo *Copy* pasen a implementarlo, y segundo, copiando estamos gastando tiempo y memoria. Para qué has de invertir el tiempo del procesador y la memoria en copiar un dato cada vez que hay un cambio de asignación cuando, por ejemplo, simplemente quieres que una función lea ese dato, como en el anterior *mostrar_edad()*.

Para estas y otras razones existen los préstamos (borrowing).

## Préstamos (*Borrowing*)

```rust
fn main () {
    let nombre: String = String::from("Amparo");
    // `nombre` es la propietaria del dato "Amparo".

    saludar(nombre);  
    // Al pasar la variable `nombre` como parámetro de una función
    // estamos de nuevo asignándola a otra variable, en este
    // caso la variable `persona` de la función `saludar()`.
    
    despedir(nombre);
    // No podemos usar `nombre` ya que anteriormente se
    // asignó a otra variable.
}

fn saludar (persona: String) {
    println!("Hola, {}", persona);
}

fn despedir (persona: String) {
    println!("Adiós, {}", persona);
}
```

## 






















