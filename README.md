# Guía breve de programación en ensamblador x86-64
*Prof. Jose Francisco Ruiz Muñoz*<br>
*Programación Avanzada 2026-II*<br>
*Universidad Nacional de Colombia - Sede de La Paz*<br>


A diferencia de los lenguajes de alto nivel, el ensamblador está orientado al funcionamiento del hardware sobre el que se ejecuta, y no al razonamiento lógico de quien programa: no hay variables con nombre, ni tipos, ni estructuras de control, solo registros, memoria e instrucciones. Estudiarlo permite comprender cómo los lenguajes de alto nivel (como C) se traducen en instrucciones concretas: movimientos de datos, operaciones aritméticas, comparaciones y saltos. Este documento usa ensamblador **x86-64**, ensamblado con **GCC**, en sintaxis **AT&T** (la que GCC usa por defecto en Linux). GCC es el compilador de GNU y, además de compilar C, puede ensamblar archivos `.s` hasta convertirlos en un ejecutable.

---

## 1. El lenguaje ensamblador y el modelo del procesador

Ensamblador es un lenguaje:

* De muy bajo nivel (casi una representación textual de las instrucciones binarias del procesador)
* Imperativo (una instrucción a la vez, en secuencia)
* Sin tipos: todo son bits en registros o en memoria
* No portable: depende de la arquitectura (aquí, x86-64)

Un programa en ensamblador se organiza en:

1. Instrucciones ejecutadas en secuencia
2. Comparaciones y saltos (para selección e iteración)
3. Llamadas a funciones (`call`/`ret`)

No existen `if`, `for` ni `switch`: todos estos se **construyen manualmente** con comparaciones y saltos.

---

## 2. Registros

### 2.1 Registros de propósito general (64 bits)

x86-64 tiene 16 registros de propósito general:

```
rax  rbx  rcx  rdx  rsi  rdi  rbp  rsp
r8   r9   r10  r11  r12  r13  r14  r15
```

Cada registro de 64 bits tiene "sub-registros" que representan sus 32, 16 y 8 bits menos significativos:

| 64 bits | 32 bits | 16 bits | 8 bits |
| ------- | ------- | ------- | ------ |
| `rax`   | `eax`   | `ax`    | `al`   |
| `rbx`   | `ebx`   | `bx`    | `bl`   |

Un registro no tiene "tipo": el mismo registro puede contener un entero, una dirección de memoria o cualquier patrón de bits.

### 2.1.1 Los sub-registros no se mezclan libremente con `mov`

Cada sub-registro tiene el tamaño que su nombre indica, y eso trae dos restricciones:

* Un literal debe **caber** en el tamaño del destino. `%al` es de 8 bits, así que solo admite valores de 0 a 255:

  ```asm
  mov $257, %al    # error de ensamblado: 257 no cabe en 8 bits
  ```

* `mov` exige que **origen y destino tengan el mismo tamaño**. No se puede copiar directamente un registro de 8 bits a uno de 64 bits:

  ```asm
  mov %al, %rax    # error: tamaños distintos (8 bits vs 64 bits)
  ```

Para cruzar tamaños a propósito —por ejemplo, tomar solo el byte bajo de un registro grande y ponerlo en uno de 64 bits— existe `movzbq` (*move with zero-extend, byte to quad*), que rellena con ceros los bits que faltan:

```asm
mov    $257, %rcx     # rcx = 257 (cabe sin problema en 64 bits)
movzbq %cl, %rax       # rax = byte bajo de rcx extendido con ceros -> rax = 1 (257 mod 256)
```

Este es el mismo truncamiento a 8 bits que se vio en la sección 7.3 con el código de salida del proceso, pero aquí ocurre explícitamente sobre un registro, al leer solo su sub-registro de 8 bits.

---

### 2.2 Movimiento de datos con `mov`

```asm
mov $5, %rax      # rax = 5   (constante -> registro)
mov %rax, %rbx     # rbx = rax (registro -> registro)
```

En sintaxis AT&T:

* El **origen** va primero, el **destino** va segundo (orden inverso a Intel).
* Las constantes literales se escriben con `$`.
* Los registros se escriben con `%`.

Por defecto, un literal como `$42` se interpreta en **decimal**. También pueden escribirse en otras bases, con los mismos prefijos que en C:

```asm
mov $42, %rax        # decimal
mov $0x2A, %rax       # hexadecimal (= 42)
mov $052, %rax        # octal (= 42)
mov $0b101010, %rax   # binario (= 42)
```

Esto es solo una forma de **escribir** el valor: dentro del registro no hay ninguna base asociada, solo bits.

---

## 3. Operaciones aritméticas

### 3.1 Suma y resta

```asm
mov $8, %rax
mov $3, %rbx
add %rbx, %rax     # rax = rax + rbx  ->  rax = 11
sub %rbx, %rax     # rax = rax - rbx
```

`add` y `sub` modifican el **primer operando indicado como destino** (el segundo operando en la instrucción). El resultado siempre queda sobre el destino; si se necesita conservar el valor original hay que copiarlo antes a otro registro.

---

### 3.2 Multiplicación con `imul`

```asm
mov $6, %rax
mov $7, %rbx
imul %rbx, %rax    # rax = rax * rbx  ->  rax = 42
```

`imul` en su forma de dos operandos multiplica y guarda el resultado en el destino, truncando el resultado al tamaño del registro si es necesario.

---

## 4. Comparaciones y saltos

### 4.1 La instrucción `cmp`

```asm
cmp %rbx, %rax     # calcula rax - rbx (sin guardar el resultado) y actualiza flags
```

`cmp` no modifica ningún registro: solo actualiza las **flags** del procesador (cero, signo, acarreo, overflow) según el resultado de la resta.

---

### 4.2 Saltos condicionales

Después de un `cmp`, se puede saltar según el resultado de la comparación:

| Instrucción | Salta si...                    |
| ----------- | ------------------------------- |
| `je`        | Iguales                         |
| `jne`       | Distintos                       |
| `jl`        | Menor (con signo)                |
| `jle`       | Menor o igual (con signo)        |
| `jg`        | Mayor (con signo)                 |
| `jge`       | Mayor o igual (con signo)         |

Ejemplo: dejar en `rdx` el valor 1 si `rax != rbx`, o 0 en caso contrario.

```asm
    cmp   %rbx, %rax
    jne   son_distintos
    mov   $0, %rdx
    jmp   fin
son_distintos:
    mov   $1, %rdx
fin:
```

Esta estructura es exactamente la traducción manual de un `if / else`.

---

## 5. Ciclos (iteración)

Un ciclo se construye con una etiqueta, una condición de salida y un salto de regreso. Por ejemplo, contar de `rcx = 5` hasta 0 y acumular cuántas iteraciones se hicieron en `rax`:

```asm
    mov   $5, %rcx     # contador
    mov   $0, %rax     # acumulador de iteraciones
ciclo:
    cmp   $0, %rcx
    je    fin_ciclo
    inc   %rax
    dec   %rcx
    jmp   ciclo
fin_ciclo:
```

Todo ciclo, sin importar el lenguaje, puede descomponerse en:

* Inicialización
* Evaluación de condición
* Cuerpo del ciclo
* Actualización
* Salto de regreso a la condición

---

## 6. Direccionamiento de memoria

### 6.1 Acceso a memoria con paréntesis

```asm
mov (%rsi), %rax       # rax = valor apuntado por rsi
mov %rax, (%rdi)       # guarda rax en la dirección apuntada por rdi
```

El paréntesis `( )` en AT&T indica **desreferenciación**, similar al `*p` de C: en lugar de operar sobre el registro, se opera sobre el valor almacenado en la dirección que ese registro contiene.

---

### 6.2 Ejemplo paso a paso: leer y modificar un valor en memoria

Los ejemplos anteriores son fragmentos sueltos, útiles para ilustrar una instrucción a la vez, pero no se pueden ejecutar tal cual: `mov (%rsi), %rax` solo tiene sentido si `rsi` ya contiene una dirección de memoria válida. Para probar direccionamiento de verdad hace falta un programa completo, con una variable real declarada en memoria a la cual apuntar. En **GAS** (*GNU Assembler*, el ensamblador que usa GCC internamente) esa declaración se hace en la sección `.data`, la zona de memoria reservada para variables — en contraste con `.text`, donde van las instrucciones:

```asm
.data
valor:  .quad 99          # reserva 8 bytes en memoria, inicializados en 99

.text
.global main
main:
    lea   valor(%rip), %rsi   # rsi = dirección de 'valor'
    mov   (%rsi), %rax        # rax = *rsi  ->  rax = 99
    ret
```

Paso a paso:

1. `.data` marca el inicio de una zona de memoria para variables inicializadas.
2. `valor: .quad 99` crea una etiqueta `valor` que representa una dirección, y reserva ahí un *quad word* (8 bytes) con el valor 99.
3. `.text` regresa a la sección de código. Es un paso obligatorio: `.data` deja al ensamblador ubicando todo lo que sigue en la zona de datos, así que sin este cambio explícito, `main` terminaría colocado ahí también — y esa zona normalmente no tiene permiso de ejecución, lo que produce una violación de segmento (`SIGSEGV`, código de salida 139) en cuanto el programa intenta correr.
4. `lea valor(%rip), %rsi` calcula la **dirección** de `valor` (no su contenido) y la guarda en `rsi`. `lea` (*load effective address*) nunca accede a memoria: solo hace aritmética de direcciones. El sufijo `(%rip)` indica que la dirección se calcula relativa a la instrucción actual, que es como GCC genera código de posición independiente en Linux x86-64.
5. `mov (%rsi), %rax` sí accede a memoria: lee los 8 bytes que hay en la dirección guardada en `rsi` y los copia a `rax`. Al terminar, `rax = 99`.

Para **modificar** ese valor en memoria, el destino de un `mov` también puede ser una dirección:

```asm
mov $77, (%rsi)     # escribe 77 directamente en memoria, en la dirección de 'valor'
```

Note que `lea` obtiene una dirección y `mov (%reg)` obtiene o modifica el **contenido** de esa dirección — es la misma distinción que hay en C entre `&x` y `*p`.

---

### 6.3 Direccionamiento con desplazamiento e índice

```asm
mov 8(%rsi), %rax          # rax = *(rsi + 8)
mov (%rsi, %rcx, 8), %rax  # rax = *(rsi + rcx*8)   -> recorrer un arreglo de 8 bytes
```

Esta forma general `desplazamiento(base, índice, escala)` es la que usa el compilador para traducir el acceso a arreglos (`arr[i]`): `base` es la dirección de inicio del arreglo, `índice` es la posición `i`, y `escala` es el tamaño en bytes de cada elemento (1, 2, 4 u 8).

---

### 6.4 Ejemplo paso a paso: recorrer un arreglo

Combinando el direccionamiento indexado con el esquema de ciclo de la sección 5, se puede sumar los elementos de un arreglo:

```asm
.data
arreglo: .quad 10, 20, 30, 40   # 4 elementos de 8 bytes cada uno

.text
.global main
main:
    lea   arreglo(%rip), %rsi   # rsi = dirección base del arreglo
    mov   $0, %rcx              # rcx = índice i = 0
    mov   $0, %rax              # rax = acumulador de la suma

sumar:
    cmp   $4, %rcx               # ¿i == 4 (longitud del arreglo)?
    je    fin_sumar
    add   (%rsi, %rcx, 8), %rax  # rax += arreglo[i]   (dirección = rsi + i*8)
    inc   %rcx                   # i++
    jmp   sumar

fin_sumar:
    ret                          # rax queda con la suma total
```

Recorrido del ciclo, iteración por iteración:

| `rcx` (i) | Dirección accedida | `arreglo[i]` | `rax` después de sumar |
| --------- | ------------------- | ------------ | ----------------------- |
| 0         | `rsi + 0`           | 10           | 10                       |
| 1         | `rsi + 8`           | 20           | 30                       |
| 2         | `rsi + 16`          | 30           | 60                       |
| 3         | `rsi + 24`          | 40           | 100                      |
| 4         | —                    | —            | (se sale del ciclo)      |

El registro `rcx` nunca contiene una dirección: es un índice entero. Es `(%rsi, %rcx, 8)` quien hace el cálculo `rsi + rcx*8` en cada iteración, exactamente como haría el compilador con `arreglo[i]` en C.

---

### 6.5 Cómo inspeccionar varios registros a la vez

Terminar el programa con `ret` solo permite ver **un** valor (el de `rax`, como código de salida). Para revisar `rsi`, `rcx` y `rax` al mismo tiempo —por ejemplo, en medio del ciclo del ejemplo anterior— hace falta un depurador (*debugger*):

1. En OnlineGDB, usa el botón **Debug** en lugar de **Run**.
2. Pon un *breakpoint* haciendo clic junto al número de línea donde quieras detener la ejecución (por ejemplo, en `cmp $4, %rcx`, dentro del ciclo).
3. Al llegar al breakpoint aparece un panel de **Registers**, con `rax`, `rbx`, `rcx`, `rdx`, `rsi`, `rdi`, etc. mostrados simultáneamente.
4. Con **Step Over** se avanza instrucción por instrucción, viendo cómo cambian los registros en cada vuelta del ciclo — útil para comparar contra la tabla anterior.

Esto funciona porque OnlineGDB usa **GDB** (*GNU Debugger*) por debajo, que internamente hace algo equivalente al comando `info registers`.

---

## 7. Llamadas a funciones

Hasta ahora todos los ejemplos viven dentro de una sola función (`main`). Un programa real casi siempre se divide en varias funciones, y esas funciones necesitan un **acuerdo común** sobre en qué registro va cada argumento y en cuál se deja el resultado — sin eso, ninguna función podría llamar a otra de forma confiable (ni siquiera a una compilada por separado, como una función de biblioteca). Esta sección describe ese acuerdo y cómo usarlo para definir y llamar una función propia.

### 7.1 Convención de llamada (System V AMD64, Linux)

Los primeros seis argumentos enteros se pasan en registros, en este orden:

```
rdi, rsi, rdx, rcx, r8, r9
```

El valor de retorno se deja en `rax`. Por ejemplo, una función que suma sus dos argumentos:

```asm
suma:
    mov   %rdi, %rax   # rax = primer argumento
    add   %rsi, %rax   # rax += segundo argumento
    ret
```

---

### 7.2 Ejemplo paso a paso: definir y llamar una función

El fragmento anterior define `suma`, pero por sí solo no hace nada visible: nadie la llama. Para probarla hace falta un `main` que la invoque con `call`, pasándole argumentos concretos en `rdi` y `rsi`:

```asm
.text
.global main

suma:
    mov   %rdi, %rax   # rax = primer argumento
    add   %rsi, %rax   # rax += segundo argumento
    ret

main:
    mov   $7, %rdi      # primer argumento de suma
    mov   $35, %rsi     # segundo argumento de suma
    call  suma           # rax = suma(7, 35)
    ret                   # exit code = rax = 42
```

Paso a paso:

1. Antes de llamar a `suma`, `main` coloca los argumentos en los registros que exige la convención: `7` en `rdi`, `35` en `rsi`.
2. `call suma` guarda en la pila la dirección de la instrucción siguiente (para saber a dónde volver) y salta a la etiqueta `suma` — es el análogo en ensamblador de invocar `suma(7, 35)` en C.
3. Dentro de `suma`, `rdi` y `rsi` ya traen los argumentos, así que el cuerpo de la función es el mismo fragmento de la sección 7.1: calcula `rax = rdi + rsi = 42`.
4. El `ret` de `suma` saca de la pila la dirección guardada por `call` y regresa la ejecución justo después del `call`, dentro de `main` — con `rax` todavía en 42.
5. El `ret` de `main` usa ese mismo `rax` como código de salida del programa: `42`.

Prueba este ejemplo en OnlineGDB (sección 6.5) y pon un *breakpoint* justo en el `call suma` para ver, en el panel de Registers, cómo `rdi` y `rsi` ya tienen 7 y 35 antes de saltar — y cómo `rax` cambia a 42 apenas se ejecuta el `ret` de `suma`.

---

### 7.3 `ret`

`ret` finaliza la función y retorna el control (y el valor en `rax`) al llamador, análogo al `return` de C.

---

### 7.4 El valor en `rax` frente al código de salida del proceso

Cuando `main` termina con `ret`, el valor de `rax` no llega intacto a la terminal. `rax` es un registro de **64 bits**, pero el código de salida de un proceso en Linux/POSIX **solo tiene 8 bits** (0 a 255), sin importar qué tan grande sea el valor real en el registro.

```asm
.global main
main:
    mov   $257, %rcx
    mov   %rcx, %rax   # rax = 257
    ret                # el sistema operativo solo reporta el byte bajo: 257 mod 256 = 1
```

```
257 = 0b1_0000_0001
        ^ ^^^^^^^^
        |   byte bajo = 1   <- esto es lo que se reporta como código de salida
        bit 8, se descarta
```

Este truncamiento ocurre porque el valor de `main` termina pasando por la syscall `exit`, y el sistema operativo solo conserva el byte menos significativo (es la misma razón por la que en una terminal `echo $?` nunca muestra números fuera de 0–255).

Es importante no confundir esto con el **overflow** de la sección 9: aquí `rax` nunca se desborda —257 cabe sin problema en 64 bits— y no se activa la flag `OF`. Es un truncamiento impuesto por la convención del sistema operativo sobre el tamaño del código de salida, no por el tamaño del registro.

---

## 8. Relación con la programación en C

Las construcciones de C tienen traducción directa a ensamblador:

* Una variable local suele ser un registro o una posición en la pila (`rsp`, `rbp`)
* `if / else` se traduce en `cmp` + salto condicional
* Un ciclo `for` se traduce en `cmp` + salto + etiqueta de regreso
* `return` se traduce en dejar el valor en `rax` y ejecutar `ret`
* `arr[i]` se traduce en direccionamiento con índice y escala

Compilar un programa en C con `gcc -S archivo.c` permite ver exactamente el ensamblador que el compilador genera para cada construcción.

---

## 9. Overflow en aritmética de enteros

Los registros tienen un tamaño fijo (64 bits en `rax`, `rbx`, etc.). Si el resultado de una suma, resta o multiplicación no cabe en ese tamaño, ocurre **overflow**: el valor se trunca (se pierden los bits más significativos) y el resultado ya no representa el cálculo real.

* El procesador señala esta condición con la **flag OF** (overflow), consultable con saltos como `jo` (*jump if overflow*) / `jno`.
* `imul` es especialmente sensible: multiplicar dos números moderados puede desbordar 64 bits rápidamente (por ejemplo, calcular un factorial grande).
* En ensamblador **no hay verificación automática**: es responsabilidad de quien programa detectar y manejar el overflow si es relevante para el problema.

---

## 10. Errores comunes

* Confundir el orden origen/destino de AT&T (`mov %origen, %destino`)
* Olvidar el `$` en constantes o el `%` en registros
* Modificar un registro que todavía se necesita más adelante (por ejemplo, sobrescribir `rax` antes de usarlo)
* Comparar con `cmp` y olvidar que el orden de los operandos afecta el sentido de `jg`/`jl`
* No considerar el overflow al sumar, restar o multiplicar valores grandes
* Olvidar `ret` al final de una función, o desbalancear la pila

## TAREA

La tarea de este tema se encuentra en el archivo [TAREA.md](TAREA.md).

---

**Este repositorio fue desarrollado con apoyo de inteligencia artificial. El contenido fue revisado, validado y editado cuidadosamente por el docente responsable.*
