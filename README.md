# ALU32 – Unidad Aritmético-Lógica de 32 bits

## Descripción general

La **ALU32** es una unidad aritmético-lógica de 32 bits diseñada para procesar operaciones sobre números binarios en **complemento a 2**.  
Permite:

- Sumas y restas con detección de **overflow**.
- Operaciones lógicas **AND**.
- Negación condicional de entradas y salida.
- Detección de bandera de **cero (`zr`)** y **signo (`ng`)**.

El diseño utiliza **chips de 16 bits** para las partes baja y alta, combinados para formar los 32 bits de salida. Está optimizado para el simulador **Nand2Tetris**.

---

## Señales de entrada

| Señal | Tamaño | Descripción |
|-------|--------|------------|
| `x1`  | 16 bits | Parte baja del primer operando de 32 bits |
| `x2`  | 16 bits | Parte alta del primer operando de 32 bits |
| `y1`  | 16 bits | Parte baja del segundo operando de 32 bits |
| `y2`  | 16 bits | Parte alta del segundo operando de 32 bits |
| `zx`  | 1 bit   | Si es 1, pone a cero la entrada X antes de cualquier operación |
| `nx`  | 1 bit   | Si es 1, niega (bit a bit) la entrada X después de aplicar `zx` |
| `zy`  | 1 bit   | Si es 1, pone a cero la entrada Y antes de cualquier operación |
| `ny`  | 1 bit   | Si es 1, niega (bit a bit) la entrada Y después de aplicar `zy` |
| `f`   | 1 bit   | Selecciona la operación: 0 → AND, 1 → ADD |
| `no`  | 1 bit   | Si es 1, niega la salida final después de aplicar `f` |

---

## Señales de salida

| Señal | Tamaño | Descripción |
|-------|--------|------------|
| `out` | 32 bits | Resultado de la operación seleccionada, considerando negaciones de entrada y salida |
| `zr`  | 1 bit   | Bandera de cero: 1 si `out == 0`, 0 en cualquier otro caso |
| `ng`  | 1 bit   | Bandera de signo: 1 si `out` es negativo (MSB = 1), 0 si positivo |
| `overflow` | 1 bit | Bandera de overflow: 1 si se produce desbordamiento aritmético en suma/resta con signo |

---

## Diseño y decisiones clave

### 1. Manejo de entradas de 32 bits

- Se utilizan dos bloques de 16 bits (parte baja y alta) para operaciones de 32 bits.
- Cada bloque aplica **negaciones condicionales** (`nx`, `ny`) y **cero condicional** (`zx`, `zy`).
- Se usan `Add16Carry` para propagar correctamente el carry de la parte baja a la alta.

### 2. Selección de operaciones (`f`)

- Se utiliza un **multiplexor (Mux16)** para elegir entre:
  - AND (`f = 0`)
  - ADD (`f = 1`) con propagación de carry.

### 3. Negación de salida (`no`)

- Después de aplicar la operación `f`, la salida puede ser negada condicionalmente mediante otro Mux16.
- Permite implementar operaciones como **x - y** o **NOT(x AND y)**.

### 4. Banderas de estado

- **`zr` (zero):**  
  - Detecta si todos los bits son cero mediante `Or16Way` seguido de `Not`.
- **`ng` (negativo):**  
  - Se toma el **bit más significativo** de la parte alta (`MSB`) como indicador de signo.
- **`overflow`:**  
  - Se detecta comparando el **carry-in y carry-out del bit 31**.
  - Regla: si operandos del mismo signo dan resultado de signo distinto → overflow = 1.

### 5. Manejo de carry

- `Add16Carry` devuelve `cout` que se propaga de la parte baja a la alta.
- La señal de **overflow** se calcula usando reglas del complemento a 2:
  ```text
  if (signo_operando1 == signo_operando2) && (signo_resultado != signo_operando1)
      overflow = 1
  else
      overflow = 0

## 6. Consideraciones de implementación

- Se utiliza `Join32` para unir las dos mitades de salida.
- Modularización: reutiliza bloques de 16 bits (`And16`, `Add16Carry`, `Not16`, `Mux16`).
- El diseño evita ciclos lógicos y permite propagar correctamente **carry** y banderas de estado.
- Preparado para el simulador **Nand2Tetris** versión 2.0 o superior.

---

## Referencias de hardware

- Basado en la arquitectura **Hack Computer (Nand2Tetris)**.
- Chips usados:
  - `Mux16`
  - `Not16`
  - `And16`
  - `Add16Carry`
  - `Or16Way`
  - `Join32`
- Estos chips permiten construir operaciones aritméticas y lógicas de manera modular y reutilizable.

---

## Conclusión

La **ALU32** implementa:

- Operaciones lógicas y aritméticas de 32 bits.
- Manejo correcto de **complemento a 2** para números con signo.
- Negaciones condicionales de entrada y salida.
- Banderas de **zero, negativo y overflow**.
- Propagación correcta de **carry** entre parte baja y alta.

Este diseño es **modular y escalable**, permitiendo futuras extensiones sin reescribir la lógica central.  
Funciona de manera consistente con el **simulador Nand2Tetris**, respetando las restricciones de hardware y señales de control.
