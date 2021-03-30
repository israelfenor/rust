+++
title = "Borrowing: o cómo se gestiona la memoria en Rust (parte II)"
summary = "En este apunte trato del préstamo (borrowing) y es la segunda entrega de una serie de anotaciones que estoy haciendo respecto la gestión de la memoria en Rust."
slug = "borrowing-gestion-de-memoria-rust"
date = "2021-03-06"
draft = true

+++

> Este apunte es la segunda entrega de una serie de anotaciones que estoy haciendo al respecto de la gestión de la memoria en Rust.
>
> * [Parte 1: Ownership](../ownership-gestion-de-memoria-rust)

> Puesto que no soy ningún experto en la materia aviso que este escrito puede contener errores. Como sigo estudiando y aprendiendo lo iré corrigiendo y ampliando. También aviso que he simplificado algunas secciones para facilitar alguna explicación.

Esta es una lista de palabras inglesas muy utilizadas en el ámbito de la gestión de la memoria en Rust con la traducción que he utilizado para este documento.

* ***Borrowing***: Préstamo.
* ***Borrows***: Pedir prestado.
* ***Derived***: Derivado.
* ***Owner***: Propietario.
* ***Ownership***: Propiedad.
* ***Reference***: Referencia.
* ***Struct***: Estructura.

## Copiar y mover

En el [anterior apunte](../ownership-gestion-de-memoria-rust) comenté que cuando asignamos una variable a otra pueden suceder dos cosas:

1. si el dato **implementa** el rasgo (*trait*) *Copy* el dato es copiado y enlazado a la nueva variable
2. si el dato **no implementa** el rasgo *Copy* el dato es movido de una variable a la otra, comportando un cambio de propiedad (*ownership*)

En el primer caso podemos seguir accediendo al dato original mediante la variable original, pero en el segundo caso no podemos acceder al dato original mediante la variable original (perdió su propiedad), solo podemos hacerlo a través de la segunda variable (la nueva propietaria).

Pero, ¿qué hacemos si queremos seguir utilizando la variable original?. Pues tenemos dos cosas a hacer. Hacer que se copie el dato en vez de que se mueva. Prestar

## Implementar el rasgo Copy

Existe una maner Podemos reutilizar una variable que enlazaba a un dato cuyo tipo esté basado en una estructura (*struct*).

