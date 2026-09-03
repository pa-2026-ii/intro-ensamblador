# Tarea

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
