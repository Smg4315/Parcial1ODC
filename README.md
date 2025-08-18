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

## Restricciones de implementación

- El Hardware simulator de Nand2Tteris no permite la unión de buses (en este caso, unir dos buses de 16 a uno de 32 para dar una salida unificada), Aún así las salidas individaules son consistentes y cumplen con las características de una ALU de 32.
- En la IDE online, la opción de crear chips complementarios no esta implementada, por lo que se dificulta la implementación de las operaciones y de esta manera del desarrollo general

## Conclusión

La **ALU32** implementa:

- Operaciones lógicas y aritméticas de 32 bits.
- Manejo correcto de **complemento a 2** para números con signo.
- Negaciones condicionales de entrada y salida.
- Banderas de **zero, negativo y overflow**.
- Propagación correcta de **carry** entre parte baja y alta.

Este diseño es **modular y escalable**, permitiendo futuras extensiones sin reescribir la lógica central.  
Funciona de manera consistente con el **simulador Nand2Tetris**, respetando las restricciones de hardware y señales de control.

# Preguntas de Pensamiento Crítico sobre la ALU32

## 1. Modularidad: ventajas y desventajas de usar dos ALU16 vs. una ALU32 monolítica

**ALU16 doble (modular):**  
**Ventajas:**  
- Reutilización de diseño: Los bloques de 16 bits pueden emplearse en otros proyectos o para construir ALU más grandes.  
- Facilidad de depuración: Es más fácil aislar errores en módulos más pequeños.  
- Claridad y mantenimiento: El diseño es más modular y comprensible.  

**Desventajas:**  
- Latencia por carry: La propagación de carry entre la ALU baja y la alta introduce un retraso adicional.  
- Overhead de interconexión: Se necesita lógica adicional para unir las salidas y manejar flags entre módulos.  

**ALU32 monolítica:**  
**Ventajas:**  
- Menor latencia: No hay retraso de propagación entre bloques.  
- Integración directa: Todo está en un solo módulo, sin necesidad de unir bloques.  

**Desventajas:**  
- Difícil de depurar: Errores en un módulo grande son más difíciles de aislar.  
- Menos reutilizable: No se puede aprovechar en diseños más pequeños sin cambios.  
- Complejidad de mantenimiento: Cambiar una parte implica revisar todo el módulo.  

---

## 2. Signed vs. unsigned: cambios necesarios para soportar ambos tipos de operaciones

- Para **unsigned**, el overflow se detecta con el carry-out del bit más significativo.  
- Para **signed**, el overflow se calcula comparando el carry-in y carry-out del bit más significativo (bit 31).  
- Para soportar ambos tipos, se deben **agregar flags o señales de control** que seleccionen el método de detección de overflow según el tipo de operación.  
- Las entradas se interpretan de manera diferente: signed usa **complemento a 2**, mientras que unsigned trata todos los bits como valor positivo puro.  

---

## 3. Carry propagation: implementación de un carry-lookahead

- Un **carry-lookahead** permite calcular los carries de manera anticipada usando las señales **propagate** y **generate** de cada bit.  
- **Ventajas:**  
  - Reduce la latencia de propagación del carry, mejorando la velocidad de la ALU.  
- **Desventajas:**  
  - Mayor complejidad de compuertas, incrementando el uso de hardware y el costo del diseño.  

---

## 4. Optimización: reducir consumo de compuertas lógicas

Técnicas posibles:  
- **Multiplexores compartidos:** Reutilizar la misma lógica para operaciones distintas en lugar de duplicar circuitos.  
- **Simplificación de puertas:** Combinar NOT, AND y OR cuando sea posible.  
- **Propagación condicional:** Solo calcular carry o flags cuando sea estrictamente necesario.  
- **Uso de buses parciales:** Procesar datos en bloques (16 bits) y unirlos solo al final.  

---

## 5. Escalabilidad: extender a 64 o 128 bits

- Estrategia: **encadenar más ALU16** en lugar de crear una ALU64 o ALU128 monolítica.  
- Cada bloque de 16 bits calcula sus operaciones locales y el carry se propaga al siguiente bloque.  
- Flags de zero, negativo y overflow se mantienen mediante **unión jerárquica** de las banderas de cada bloque.  
- Ventaja: no se reescribe la lógica central, manteniendo modularidad y facilidad de mantenimiento.

# 📽️ Video
https://www.youtube.com/watch?v=aFyRhFw0onk
