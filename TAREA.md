# Tarea

## Parte 1

## Ejercicio 1 — Registros y aritmética básica

Escriba un programa que:

1. Cargue dos constantes en los registros `rax` y `rbx`.
2. Calcule `rax + rbx` y `rax - rbx`, dejando cada resultado en un registro distinto (sin sobrescribir `rax` ni `rbx`).
3. Discuta qué ocurriría si la suma o la resta produjera **overflow** en un registro de 64 bits.

---

## Ejercicio 2 — Condicional simple

Escriba un programa que:

1. Compare los registros `rax` y `rbx` usando `cmp`.
2. Si `rax > rbx`, deje `rcx = 1`; en caso contrario, deje `rcx = 0`.
3. Implemente la lógica únicamente con `cmp` y saltos condicionales (`jg`, `jle`, etc.), sin usar instrucciones de alto nivel.

---

## Ejercicio 3 — Máximo de dos enteros

Escriba un programa que:

1. Reciba dos enteros en registros.
2. Reutilice la lógica condicional del Ejercicio 2 (`cmp` + saltos) para determinar cuál es mayor.
3. Deje el valor **mayor** de los dos en `rax`.

---

## Ejercicio 4 — Suma con ciclo

Escriba un programa que:

1. Defina una constante `N`.
2. Calcule `1 + 2 + ... + N` usando un ciclo (etiquetas, comparación y salto de regreso).
3. Deje el resultado final en `rax`.

---

## Ejercicio 5 — Factorial

Escriba un programa que:

1. Calcule `N!` para un valor pequeño de `N` (por ejemplo, `N = 10`), usando multiplicación (`imul`) dentro de un ciclo.
2. Deje el resultado final en `rax`.
3. Discuta a partir de qué valor de `N` el resultado dejaría de caber en un registro de 64 bits (overflow).

---

## Parte 2

## Ejercicio 6 — Lectura y escritura en memoria

Escriba un programa que:

1. Declare una variable entera en la sección `.data`.
2. Obtenga su dirección con `lea` y guárdela en un registro.
3. Lea su valor en `rax` usando direccionamiento con paréntesis (por ejemplo, `mov (%rsi), %rax`).
4. Modifique el valor directamente en memoria (sin pasar por un registro intermedio) y vuelva a leerlo para confirmar el cambio.

---

## Ejercicio 7 — Recorrido de un arreglo

Escriba un programa que:

1. Declare un arreglo de al menos 5 enteros en `.data`.
2. Recorra el arreglo con un ciclo, usando direccionamiento indexado (`desplazamiento(base, índice, escala)`).
3. Calcule el valor **máximo** del arreglo.
4. Deje el resultado en `rax`.
5. Discuta qué pasaría si el ciclo recorriera un índice más allá del último elemento del arreglo.

---

## Ejercicio 8 — Inversión de un arreglo

Escriba un programa que:

1. Declare un arreglo de `N` enteros en `.data` (`N` constante, por ejemplo `N = 5`).
2. Invierta el contenido del arreglo **en el mismo arreglo**, usando direccionamiento indexado tanto para leer como para escribir. Por ejemplo, `[10, 20, 30, 40, 50]` debe quedar como `[50, 40, 30, 20, 10]`.
3. Verifique con el depurador (sección 6.5 del README) el contenido del arreglo antes y después de la inversión.

---

## Ejercicio 9 — Funciones

Escriba un programa que:

1. Defina una función que reciba dos enteros como parámetros, siguiendo la convención de llamada de Linux x86-64 (System V AMD64).
2. La función debe retornar el mayor de los dos valores (puede reutilizar la lógica del Ejercicio 3).
3. Llame a la función desde `main` usando `call`, con dos valores concretos como argumentos.
4. Deje el valor retornado en `rax`.

---

## Ejercicio 10 — Función que recorre un arreglo

Escriba un programa que:

1. Defina una función que reciba dos parámetros: la dirección base de un arreglo y su longitud.
2. La función debe recorrer el arreglo (usando direccionamiento indexado) y calcular la suma de sus elementos.
3. Retorne el resultado en `rax`.
4. Llame a la función desde `main`, pasándole un arreglo declarado en `.data` y su longitud como argumentos.

---
