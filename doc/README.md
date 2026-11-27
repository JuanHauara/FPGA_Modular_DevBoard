# FPGA Modular DevBoard

## Memoria RAM

La memoria prevista para el producto final es la **ISSI IS61WV5128BLL-10TLI**, por su menor precio en la comparación realizada. Para esta placa de desarrollo se utilizará la **Infineon CY7C1049G30-10ZSXI**: al seleccionar los componentes (2 de octubre de 2026), de estas dos referencias solo la Infineon estaba disponible en LCSC.

También se contempla la **ISSI IS61WV5128EDBLL-10TLI** como alternativa: incorpora **ECC interno que corrige un error de un bit por byte**, transparente para la FPGA.

Las tres referencias son **compatibles pin a pin** y pueden compartir el mismo footprint. Se montarán **dos chips de 512K × 8**, para un total de **1 MiB de RAM**, trabajando a **3,3 V**. En la ISSI con ECC, elegir el sufijo **`-10TLI`** para conservar el encapsulado y la velocidad indicados.

| Característica | ISSI elegida para el producto final | ISSI alternativa con ECC | Infineon alternativa |
|---|---|---|---|
| Modelo | IS61WV5128BLL-10TLI | IS61WV5128EDBLL-10TLI | CY7C1049G30-10ZSXI |
| Organización | 512K × 8 bits | 512K × 8 bits | 512K × 8 bits |
| Capacidad por chip | 512 KiB (4 Mbit) | 512 KiB (4 Mbit) | 512 KiB (4 Mbit) |
| Interfaz | SRAM asíncrona, direcciones y datos en paralelo | SRAM asíncrona, direcciones y datos en paralelo | SRAM asíncrona, direcciones y datos en paralelo |
| Encapsulado | TSOP-II, 44 pines, paso de 0,8 mm | TSOP-II, 44 pines, paso de 0,8 mm | TSOP-II, 44 pines, paso de 0,8 mm |
| Alimentación | 2,4–3,6 V | 2,4–3,6 V | 2,2–3,6 V |
| Tiempo de acceso | 10 ns | 10 ns | 10 ns |
| Temperatura de operación | −40 a +85 °C | −40 a +85 °C | −40 a +85 °C |
| ECC interno | No | Sí, transparente para la FPGA | Sí, transparente para la FPGA |

Para permitir el intercambio entre las tres referencias, el controlador debe contemplar hasta **5 ns para liberar el bus** después de desactivar `/OE` o `/CE`. La Infineon también requiere **100 µs de espera tras estabilizar la alimentación** antes de operar.

Referencias: [datasheet ISSI BLL](https://www.issi.com/WW/pdf/61-64WV5128Axx-Bxx.pdf), [datasheet ISSI EDBLL con ECC](https://www.issi.com/WW/pdf/61-64WV5128EDBLL.pdf), [datasheet Infineon](https://www.infineon.com/assets/row/public/documents/10/49/infineon-cy7c1049g-and-cy7c1049ge-are-high-performance-cmos-fast-static-ram-devices-with-embedded-ecc-datasheet-en.pdf) e [Infineon en LCSC — C2950234](https://www.lcsc.com/product-detail/C2950234.html).

## Entradas de reloj de la FPGA (MachXO3LF)

### Qué pin elegir

| Necesidad | Pin recomendado | Función |
|---|---|---|
| Distribuir un reloj externo sin cambiar su frecuencia | `PCLKTx_y` | Entrada a la **red global de reloj**, que distribuye el clock a la lógica secuencial de toda la FPGA. No requiere PLL. |
| Generar otra frecuencia a partir de un reloj externo | `L_GPLLT_IN` o `R_GPLLT_IN` | Entrada dedicada de referencia a la **PLL izquierda o derecha**. La PLL puede multiplicar/dividir la frecuencia y alimentar la red global con el reloj resultante. |
| Realimentar externamente una PLL | `L_GPLL[T/C]_FB` o `R_GPLL[T/C]_FB` | **Entrada de feedback** para configuraciones que requieren realimentación externa; no es la entrada de referencia `_IN` ni una salida de reloj. Con realimentación interna no requiere conexión externa. |

Los pines `GPLL..._IN` son la ruta recomendada hacia la PLL, aunque existen otras rutas internas posibles. Las frecuencias de entrada y salida deben cumplir los límites de la PLL; configurar y comprobarlas en Diamond. [Guía sysCLOCK, §§4 y 11.3](https://www.latticesemi.com/view_document?document_id=50124).

### Cómo leer los nombres

- **`T` = True (+), `C` = Complement (−):** forman un par para reloj diferencial. Para un reloj de una sola señal (*single-ended*), elegir `PCLKT` o `GPLLT_IN`, según el uso. `PCLKC` no es una segunda entrada independiente de reloj single-ended.
- **`PCLK` = Primary Clock.** En `PCLKT4_0`, `4` indica el banco y `0` el índice de entrada de reloj. El reloj global puede llegar a toda la FPGA; no queda limitado a ese banco.
- **`L` / `R` = Left / Right:** identifican la PLL izquierda/derecha. **`IN`** es referencia y **`FB`** es realimentación.
- **Uso como GPIO:** si no se usa su función de reloj/PLL, estos pines pueden ser entradas o salidas normales, incluidos los `_FB`. En ese modo, `C` no invierte la señal ni obliga a usar su pareja `T`.

### Referencia rápida en J3

| Contacto de J3 | Bola de la FPGA | Función |
|---|---|---|
| — | — | **J3 no expone pines dedicados `PCLK` ni `GPLL`.** |

J3 pertenece al banco 0, pero sus entradas dedicadas de reloj no llegan a este conector. El reloj de **12 MHz de la placa** entra por **C8 (`PCLKT0_1`)**, fuera de J3. Para conectar un reloj externo a una entrada dedicada, usar J4, J6 o J8.

### Referencia rápida en J4

| Contacto de J4 | Bola de la FPGA | Función |
|---|---|---|
| 19 | H14 | `PCLKT1_0`: entrada de reloj global |
| 35 | D16 | `R_GPLLT_IN`: referencia externa de la PLL derecha |

Ambos pertenecen al **banco 1**. Para reloj diferencial, sus parejas `C` son, respectivamente, **J4-25** (H16, `PCLKC1_0`) y **J4-34** (E14, `R_GPLLC_IN`). La realimentación externa de la PLL derecha está en **J4-37** (D14, `R_GPLLT_FB`), con su pareja **J4-30** (E15, `R_GPLLC_FB`).

### Referencia rápida en J6

| Contacto de J6 | Bola de la FPGA | Función |
|---|---|---|
| 17 | T9 | `PCLKT2_1`: entrada de reloj global |
| 30 | T7 | `PCLKT2_0`: entrada de reloj global |

Ambos pertenecen al **banco 2**. Para reloj diferencial, sus parejas `C` son, respectivamente, **J6-19** (P9, `PCLKC2_1`) y **J6-28** (R8, `PCLKC2_0`).

### Referencia rápida en J8

| Contacto de J8 | Bola de la FPGA | Función |
|---|---|---|
| 7 | L2 | `PCLKT3_0`: entrada de reloj global |
| 14 | J1 | `PCLKT4_0`: entrada de reloj global |
| 28 | E1 | `PCLKT5_0`: entrada de reloj global |
| 27 | E2 | `L_GPLLT_IN`: referencia externa de la PLL izquierda |

Para reloj diferencial, las parejas `C` de esos cuatro pines son, respectivamente, **J8-6, J8-17, J8-25 y J8-37**. Usar niveles eléctricos compatibles con el banco y configurar el estándar de E/S correspondiente en Diamond.

Referencias: [Hardware Checklist, §13](https://www.latticesemi.com/view_document?document_id=50516), [datasheet MachXO3, §§2.4 y 4](https://www.latticesemi.com/view_document?document_id=50121) y [guía local del Starter Kit](Datasheet/FPGA-CPLD/MachXO3LF_starter_kit/FPGA-EB-02036-1-4-MachXO2-MachXO3-Starter-Kit-User-Guide.pdf), tablas 7.2–7.5 (páginas 18–21, columna **-6900C**) y esquemas del banco 0 y banco 1 (páginas 32–33).

## Programación de la DevEbox mediante ST-Link V2

La **MCUDev DevEBox STM32H7XX_M con STM32H750VBT6** se puede programar y depurar con un **ST-Link V2 por SWD**, a través del conector **J1 de la DevEbox**.

### Pinout de J1

| Pin de J1 | Señal en la DevEbox | Conexión / función |
|---|---|---|
| 1 | `BOOT0` | Selección de arranque; no se conecta al ST-Link. Mantener en `0` para el arranque normal desde Flash con la configuración de fábrica. La placa incorpora un pull-down de 10 kΩ (R7). |
| 2 | `3V3` | Tensión de la placa. En el ST-Link V2 original, conectar a `VAPP` como referencia de tensión. |
| 3 | `GND` | Masa común con el ST-Link. |
| 4 | `PA13 / SWDIO` | Conectar a `SWDIO` del ST-Link. |
| 5 | `PA14 / SWCLK` | Conectar a `SWCLK` del ST-Link. |

La numeración corresponde al esquema de la DevEbox; identificar el pin 1 antes de conectar.

- **Alimentación:** alimentar la DevEbox por su entrada prevista, por ejemplo USB. `VAPP` del ST-Link V2 original es una entrada de referencia, no una salida de alimentación. Si se utiliza un clon cuyo pin `3V3` es una salida de alimentación, dejar esa salida desconectada cuando la placa ya esté alimentada.
- **Reset:** J1 no incluye `NRST`. Se puede programar normalmente sin él; para usar *connect under reset* hace falta una conexión adicional a `NRST`.
- **Configuración:** seleccionar **SWD** en STM32CubeProgrammer o STM32CubeIDE. No hace falta poner `BOOT0` en `1` para programar por SWD.

Referencias: [esquema de la DevEbox, J1](Datasheet/Placa_MCUDev_DevEBox_STM32H7XX_M/MCUDEV_DEVEBOX_H7XX_M/docs/STM32H7XX_M_schematics.pdf), [datasheet STM32H750, interfaz de depuración](https://www.st.com/resource/en/datasheet/stm32h750vb.pdf) y [manual ST-Link V2, UM1075, §4.2 y tabla 4](https://www.st.com/resource/en/user_manual/dm00026748-st-link-v2-in-circuit-debugger-programmer-for-stm8-and-stm32-stmicroelectronics.pdf).

## Selección de conexión de la memoria SRAM mediante jumpers

Las conexiones previstas entre la FPGA, la DevEbox y la SRAM son **mutuamente excluyentes**: solo se puede seleccionar una a la vez.

| Conexión seleccionada | Elementos conectados |
|---|---|
| FPGA ↔ DevEbox | Bus entre la FPGA y el STM32H750 de la DevEbox. |
| FPGA ↔ SRAM | Bus entre la FPGA y la memoria RAM estática. |
| DevEbox ↔ SRAM | Bus entre el STM32H750 de la DevEbox y la memoria RAM estática. |

La selección se realizará colocando jumpers entre tiras de pines adyacentes. Quedan pendientes las referencias de las tiras y las posiciones de los jumpers para cada conexión.

