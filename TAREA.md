# Tarea

## Objetivo

Afianzar la comprensión de los **conceptos fundamentales del ensamblador x86-64** (sintaxis AT&T, GCC) y su relación con el modelo real de ejecución de un programa, mediante ejercicios que enfatizan el uso explícito de registros, aritmética, comparaciones, saltos y ciclos.

---

## Instrucciones de entrega

La solución de los ejercicios debe presentarse en formato de diapositivas.

Las diapositivas deben:

* Incluir explicaciones claras y concisas de cada ejercicio.

* Incorporar recursos gráficos adecuados (diagramas, esquemas, fragmentos de código comentados) que faciliten la comprensión de los conceptos desarrollados.

* Presentar el código fuente de manera legible, destacando las partes relevantes para la explicación.

* Mantener una estructura ordenada y coherente entre las diferentes secciones.

Todos los programas deben escribirse en **ensamblador x86-64**, ensamblados con **GCC** y sintaxis **AT&T**.

---

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
