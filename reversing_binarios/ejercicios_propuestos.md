# Ejercicios de reversing

## Ejercicio 1

Compila el código fuente `hello.c` y ejecuta el programa. ¿Qué hace? ¿Qué salida produce?

Analiza el ejecutable con `objdump` y `strings`. ¿Qué instrucciones se utilizan para imprimir la cadena? ¿Qué cadenas aparecen en el ejecutable?

Analiza el ejecutable con radare2, obten las direcciones de las funciones usadas.

Analiza la funcion `main`, ¿que puedes obtener a partir de la misma?

## Ejercicio 2

Analiza el ejecutable `serial-check` y obtén acceso a la aplicación. 

## Ejercicio 3

Modifica el ejecutable de `serial-check` para que acepte el serial `pass1234`.

## Ejercicio 4

Analiza los ejecutables `mindreader` y `mindreader2`. ¿Eres capaz de saber cual está compilada desde C, y cual desde python? ¿Por qué?

## Ejercicio 5

Ambos ejecutables tienen una flag, intenta hacer que el programa te de la flag. **NO** sirve encontrar la flag como un string, debes provocar que el ejecutable lo devuelva en tiempo de ejecución.

