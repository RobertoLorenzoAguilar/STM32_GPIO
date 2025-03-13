# Proyecto de Control de Motor Paso a Paso con Interrupciones

## INTRODUCCIÓN

### Objetivo General
El objetivo principal de esta práctica es combinar los conocimientos adquiridos en clase sobre GPIO (General Purpose Input/Output) digital con aplicaciones prácticas. Se desarrollará un sistema que controle un proceso mediante un servicio de interrupción, lo que ayudará a comprender cómo administrar los componentes de hardware y sus interacciones de manera efectiva.

### Objetivos Específicos
Para completar con éxito esta práctica, se deben alcanzar los siguientes objetivos específicos:

1. **Controlar un motor paso a paso**
   - **Microstepping**: Controlar un motor paso a paso utilizando técnicas de micropasos, específicamente usando 1/8 de paso o cualquier otra opción de micropaso disponible.
   - **Botones**: Implementar dos botones:
     - **Botón de control de dirección (Botón 1)**: Permite cambiar la dirección del motor (en el sentido de las agujas del reloj o en el sentido contrario).
     - **Botón de latch (Botón 2)**: Se utiliza para bloquear la posición del motor.

2. **Diseñar un sistema de conteo**
   - **Pantalla de 7 segmentos**: Crear un sistema que utilice un display de 7 segmentos para contar el número de pasos que hace el motor.
   - **Dirección de conteo**: Asegurar que la pantalla pueda contar tanto en el sentido de las agujas del reloj (CW) como en el sentido contrario (CCW) en función del movimiento del motor.
   - **Funcionalidad de latch**: El display también debe ser capaz de mostrar el conteo actual cuando se presiona el botón de latch.

3. **Implementación de un servicio de interrupción**
   - **Interrupción generada por botón**: Utilizar el botón 3 para generar un servicio de interrupción que activará un relé.
   - **Activación del relé**: Cuando el relé está activado, debe arrancar un motor de CD. Esto demostrará cómo las interrupciones pueden controlar otros procesos en el sistema.

4. **Funcionamiento del sistema**
   - **Funciones principales y de interrupción**: Asegurar que el motor y la pantalla funcionen como las funciones principales del sistema, mientras que el control del motor y la activación del relé se ejecutan como funciones de interrupción. Esto ayudará a comprender la importancia de priorizar las tareas en los sistemas integrados.

## MARCO TEÓRICO

### Motores a Pasos
Los motores a pasos son dispositivos electromecánicos que convierten pulsos eléctricos en movimientos mecánicos discretos. Su funcionamiento se basa en la energización secuencial de sus bobinas, lo que permite un control preciso de la posición del rotor sin necesidad de sensores adicionales. Se utilizan ampliamente en aplicaciones que requieren posicionamiento exacto, como impresoras 3D, robótica y máquinas CNC.

En esta práctica se trabaja con el motor paso a paso 28BYJ-48, el cual es un motor de imán permanente de 5 fases que se controla mediante un driver como el ULN2003.

### Microstepping
El microstepping es una técnica de control de motores a pasos que permite incrementar la resolución del movimiento al dividir cada paso completo en fracciones más pequeñas. Esto se logra aplicando voltajes intermedios a las bobinas del motor, reduciendo las vibraciones y mejorando la suavidad del movimiento. En este proyecto se emplea la modalidad 1/8 de paso, lo que permite obtener una mejor precisión en el giro del motor.

### GPIO y Control de Hardware
Los pines GPIO (General Purpose Input/Output) de un microcontrolador permiten la comunicación entre el software y los dispositivos externos. En este caso, se utilizan para enviar señales de control al motor, leer el estado de los botones y manejar la pantalla de 7 segmentos.

**Entradas y salidas utilizadas:**
- **Salidas (Outputs)**: Control de fases del motor y activación del relevador para el motor de corriente directa (DC).
- **Entradas (Inputs)**: Lectura de los botones para control de dirección, parada y activación del relevador.

### Pantallas de 7 Segmentos
Las pantallas de 7 segmentos son dispositivos utilizados para mostrar caracteres numéricos. Se componen de siete LEDs organizados en forma de "8" que pueden encenderse de manera selectiva para representar diferentes números. En esta práctica, la pantalla se emplea para contar y visualizar los pasos que realiza el motor.

### Interrupciones en Microcontroladores
Las interrupciones son eventos que permiten que el microcontrolador ejecute una tarea específica sin afectar el flujo principal del programa. Se activan mediante señales externas, como la presión de un botón, y pueden utilizarse para responder a eventos de manera rápida y eficiente.

**Ejemplo de interrupción en esta práctica:**
- Un botón (BTN3) genera una interrupción que activa un relevador, permitiendo el encendido de un motor de corriente directa.

### Relevadores
Un relevador es un interruptor electromagnético que permite activar o desactivar circuitos de alta corriente utilizando una señal de baja corriente. Se emplea en esta práctica para controlar un motor de DC de 5V, demostrando la aplicación de interrupciones en la gestión de procesos industriales.

### Importancia de la Priorización de Tareas en Sistemas Embebidos
Los sistemas embebidos manejan diversas tareas simultáneamente, por lo que es crucial asignar prioridades adecuadas para garantizar un funcionamiento eficiente. En este caso, se ejecutan tareas principales como el control del motor y la visualización en la pantalla, mientras que los eventos críticos, como la activación del relevador, se manejan mediante interrupciones.

## PROCEDIMIENTO

### MATERIALES
1. Núcleo STM32F411RET6.
2. Motor a pasos 28BYJ-48.
3. Un controlador para el motor a pasos ARD-300.
4. 4 resistencias de 220-200 Ω.
5. 3 Push buttons.
6. Pantalla led de siete segmentos.
7. Jumpers.
8. Fuente externa mayor igual a 5V (Recomendable).
9. Motor 5V CD.
10. Un relevador.
11. Fuente de Corriente directa externa.

### DESARROLLO

**Paso 1:** Abrir un nuevo proyecto para empezar a configurar el microcontrolador.

**Paso 2:** Buscar el microcontrolador correspondiente, en este caso el STM32F411RET6, seleccionarlo y darle clic en siguiente, después darle el nombre al proyecto, darle “terminar”.

**Paso 3:** Configurar las entradas y salidas del microcontrolador.

**Salidas (Outputs):**
- Etiquetas F1-F4 (son las fases del motor).
- SEG0-SEG7 (Segmentos utilizados para mostrar numeración de los pasos del motor).
- M1 (para enviar la señal al relevador conectado en el motor DC).

**Entradas (Inputs):**
- Etiquetas BTN1, BTN2 para controlar el inicio de giro del motor, paro y dirección, giro del motor a pasos.
- BTN3 solo para activar o desactivar el motor CD por medio del relevador.

**Declaración de variables:**
```c
int contador; // variable contadora de pasos.
int velocidad = 1; // velocidad giro motor.
int giro; // variable botón1. “inicialización maquina de estados o proceso”.
int latch; // variable botón2. “paro giro dirección”.
int paro; // variable botón3. “relay motor cd”.
int inicio; // variable para de asignación inicialización estado.
int Fases_motor[4][4] = { { 0, 0, 0, 1 }, { 0, 0, 1, 0 }, { 0, 1, 0, 0 }, { 1,0, 0, 0 } }; // Fases Motor bvn v m n.
```
**Funcionalidad principal en el bucle while:**
```c
while (1) {
  giro = HAL_GPIO_ReadPin(GPIOC, BTN1_Pin); // Leemos el giro para comenzar el proceso de funcionalidad del motor
  if (giro) {
    inicio = 1;
  }
  while (inicio == 1) { // Si hay un giro cambia la variable inicio uno y entra en un ciclo
    latch = HAL_GPIO_ReadPin(GPIOC, BTN2_Pin); // Si el botón latch fue presionado pausa y cambia la dirección.
    paro = HAL_GPIO_ReadPin(GPIOC, BTN3_Pin); // Si el botón paro fue presionado manda señal al relay motor CD.
    if (latch) {
      HAL_Delay(200);
      giro = !giro; // Cambia de dirección
    }
    if (paro) {
      // Lee el estado actual del pin
      GPIO_PinState pinState = HAL_GPIO_ReadPin(GPIOA, M2_Pin);
      // Si el pin está encendido (HIGH), lo apagas (LOW)
      if (pinState == GPIO_PIN_SET) {
        HAL_GPIO_WritePin(GPIOA, M2_Pin, GPIO_PIN_RESET);
      }
      // Si el pin está apagado (LOW), lo enciendes (HIGH)
      else {
        HAL_GPIO_WritePin(GPIOA, M2_Pin, GPIO_PIN_SET);
      }
    }
    if (!latch) {
      if (giro) {
        contador--;
        if (contador < 0) {
          contador = 3;
        }
      } else {
        contador++;
        if (contador >= 4) {
          contador = 0;
        }
      }
      Display(contador); // muestra los pasos del 0 al 3.
      HAL_GPIO_WritePin(GPIOC, F1_Pin, Fases_motor[contador][0]); // excita las fases del motor.
      HAL_GPIO_WritePin(GPIOC, F2_Pin, Fases_motor[contador][1]);
      HAL_GPIO_WritePin(GPIOC, F3_Pin, Fases_motor[contador][2]);
      HAL_GPIO_WritePin(GPIOC, F4_Pin, Fases_motor[contador][3]);
      HAL_Delay(velocidad); // realiza un delay para evitar problemas de activación en el movimiento.
    }
  }
}
```
