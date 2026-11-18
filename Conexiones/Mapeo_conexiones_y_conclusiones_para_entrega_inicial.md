# Robot seguidor de línea

**Documento de avance — Arduino Nano, driver TB6612FNG (HW-166) y arreglo de sensores IR**

Este documento resume el estado actual del proyecto: qué componentes se están usando, cómo están conectados, qué código se ha probado hasta ahora y qué conclusiones sacamos de las pruebas realizadas. La idea es dejar registro de cómo está armado todo en este momento antes de seguir avanzando con la lógica final del programa.

---

## 1. Componentes utilizados

| Componente | Detalle |
| --- | --- |
| Microcontrolador | Arduino Nano |
| Driver de motores | TB6612FNG, módulo HW-166 |
| Motores | 2 motorreductores de 5V |
| Sensores de línea | Arreglo de 8 sensores infrarrojos (2 quedaron dañados, ver sección 4) |
| Alimentación de los motores (VM) | Batería de 3.7V conectada directo al driver |
| Alimentación de la lógica del driver (VCC) | 5V tomados del propio Arduino Nano |
| Alimentación de pruebas iniciales | USB del PC (antes de tener batería) |

---

## 2. Mapeo de conexiones actual

### 2.1 Sensores de línea

Este mapeo no se ha vuelto a tocar desde que se detectaron los sensores dañados; se dejó fijo para que el resto de las conexiones del driver se acomodaran alrededor de este.

| Función | Pin Arduino Nano |
| --- | --- |
| IR (emisor infrarrojo) | D5 |
| Sensor 1 | A0 (dañado, siempre da 1) |
| Sensor 2 | A1 (dañado, siempre da 1) |
| Sensor 3 | A2 |
| Sensor 4 | A3 (sensor de línea media) |
| Sensor 5 | A4 (sensor de línea media) |
| Sensor 6 | A5 |
| Sensor 7 | D4 |
| Sensor 8 | D3 |

### 2.2 Motor 1 — Canal A del driver

| TB6612FNG | Arduino Nano / destino |
| --- | --- |
| AIN1 | D7 |
| AIN2 | D8 |
| PWMA | D9 |
| AO1 | Cable 1 del motor 1 |
| AO2 | Cable 2 del motor 1 |

### 2.3 Motor 2 — Canal B del driver

| TB6612FNG | Arduino Nano / destino |
| --- | --- |
| BIN1 | D12 |
| BIN2 | D11 |
| PWMB | D10 |
| BO1 | Cable 1 del motor 2 |
| BO2 | Cable 2 del motor 2 |

### 2.4 Control y alimentación del driver

| TB6612FNG | Conectado a |
| --- | --- |
| STBY | D2 |
| VM (potencia de los motores) | Positivo de la batería de 3.7V |
| VCC (lógica del driver) | 5V del Arduino Nano |
| GND (los 3 pines del driver son el mismo nodo interno) | GND del Nano + negativo de la batería, todo compartiendo la misma tierra |

### 2.5 Resumen de pines del Nano en uso

| Pin | Uso |
| --- | --- |
| D2 | STBY (driver) |
| D3 | Sensor 8 |
| D4 | Sensor 7 |
| D5 | IR (emisor) |
| D7 | AIN1 |
| D8 | AIN2 |
| D9 | PWMA |
| D10 | PWMB |
| D11 | BIN2 |
| D12 | BIN1 |
| A0 – A5 | Sensores 1 a 6 |
| D6, D13 | Libres, sin uso por ahora |

> **Nota importante sobre el GND:** el TB6612FNG trae 3 pines de tierra en el módulo HW-166, pero internamente los 3 son un mismo nodo eléctrico (misma pista de cobre). No es necesario cablear los tres, con que al menos uno del driver quede unido al GND del Nano y al negativo de la batería es suficiente. Aun así, conectar más de uno da un margen extra de seguridad frente a picos de corriente cuando el motor arranca o frena.

---

## 3. Código de prueba actual

Este es el código con el que se está validando que los sensores y los dos motores reaccionen juntos correctamente. La lógica final del programa (cómo debe comportarse el robot en la pista real) todavía está por definirse; esto es solo la prueba de funcionamiento.

```cpp
// --- Sensores ---
//Sensores de la linea media son A3 y A4
const int IR = 5;
const int sensores[8] = {
  A0, A1, A2, A3, A4, A5, 4, 3
};
const int UMBRAL = 6; // minimo de sensores en 1 para avanzar

// --- Motor 1 (canal A) ---
const int AIN1 = 7;
const int AIN2 = 8;
const int PWMA = 9;

// --- Motor 2 (canal B) ---
const int BIN1 = 12;
const int BIN2 = 11;
const int PWMB = 10;

// --- Control driver ---
const int STBY = 2;
const int VELOCIDAD = 120;

void setup() {
  Serial.begin(9600);
  pinMode(IR, OUTPUT);
  digitalWrite(IR, HIGH);

  for (int i = 0; i < 8; i++) {
    pinMode(sensores[i], INPUT);
  }

  pinMode(STBY, OUTPUT);
  pinMode(AIN1, OUTPUT);
  pinMode(AIN2, OUTPUT);
  pinMode(PWMA, OUTPUT);
  pinMode(BIN1, OUTPUT);
  pinMode(BIN2, OUTPUT);
  pinMode(PWMB, OUTPUT);

  digitalWrite(STBY, HIGH);
}

void avanzar() {
  digitalWrite(AIN1, HIGH);
  digitalWrite(AIN2, LOW);
  analogWrite(PWMA, VELOCIDAD);

  digitalWrite(BIN1, HIGH);
  digitalWrite(BIN2, LOW);
  analogWrite(PWMB, VELOCIDAD);
}

void detener() {
  analogWrite(PWMA, 0);
  analogWrite(PWMB, 0);
}

void loop() {
  int conteo = 0;

  for (int i = 0; i < 8; i++) {
    int lectura = digitalRead(sensores[i]);
    Serial.print(lectura);
    Serial.print(" ");
    if (lectura == 1) conteo++;
  }

  Serial.print("| Conteo: ");
  Serial.print(conteo);

  if (conteo >= UMBRAL) {
    Serial.println(" -> Avanzando");
    avanzar();
  } else {
    Serial.println(" -> Detenido");
    detener();
  }

  delay(100);
}
```

**Sobre el umbral:** como 2 de los 8 sensores (A0 y A1) quedaron dañados fijos en 1, esos dos siempre suman al conteo sin representar una lectura real. Con `UMBRAL` en 6, en la práctica se necesita que 4 de los 6 sensores funcionales detecten línea (valor 1) para que ambos motores avancen a la misma velocidad. Esto es válido para la prueba de funcionamiento actual, pero si más adelante se reemplazan los sensores dañados, el umbral debería recalcularse.

---

## 4. Pruebas realizadas

### Lectura individual de sensores

Se probó el arreglo de 8 sensores IR de forma aislada, imprimiendo por Serial el valor de cada uno (1 = negro, 0 = blanco). Se detectó que los sensores en las posiciones A0 y A1 quedaron arrojando siempre 1, sin importar la superficie.

### Movimiento de motores por separado

Con el driver TB6612FNG alimentado desde el propio Arduino Nano (sin batería), se probó mover el motor 1 y luego el motor 2 de forma alternada, con mensajes por Serial indicando cuál motor estaba activo en cada momento. El movimiento alternado funcionó correctamente.

### Detección de choque de pines

Al combinar el código de sensores con el del driver, se identificó que varios pines se repetían entre ambos circuitos. Se reacomodó el mapeo del driver para dejar intacto el código de sensores y evitar cualquier cruce.

### Conexión de batería 3.7V al driver

Se conectó una batería de 3.7V al pin VM del driver para dejar de depender del USB del PC. Después de esta conexión, se detectó que 2 sensores adicionales del arreglo quedaron dañados (fijos en 1).

### Diagnóstico del daño en los sensores

Se revisó la conexión de tierra usada durante la prueba con batería. Se identificó que se utilizaron columnas cortas de la protoboard (de 5 puntos) para compartir el GND entre la batería, el Nano y los sensores. Esas columnas no están unidas entre sí como una línea continua, a diferencia de las líneas largas de alimentación de los bordes de la protoboard. Esto se identificó como la causa más probable de que el retorno de corriente del motor encontrara un camino inadecuado y dañara los sensores.

---

## 5. Conclusiones y estado actual

- El driver TB6612FNG puede alimentarse en su lógica (VCC) desde el mismo Arduino Nano sin ningún problema, ya que ese consumo es mínimo. La alimentación de potencia (VM) sí debe separarse a una fuente externa cuando se van a mover motores reales, porque el USB del PC no siempre entrega suficiente corriente.
- Sin importar cuántos pines de GND tenga el driver, todos deben terminar compartiendo un mismo nodo de tierra junto con el Arduino y la fuente de los motores. Sin esa tierra común, las señales de control pueden comportarse de forma errática o generar retornos de corriente por caminos no previstos.
- De los 8 sensores del arreglo original, 2 quedaron dañados (A0 y A1) tras conectar la batería, muy probablemente por usar columnas de protoboard no continuas como tierra común. La lección para las próximas conexiones es usar siempre las líneas largas de alimentación de la protoboard (o verificar con multímetro la continuidad) antes de dar por hecho que una fila comparte el mismo nodo eléctrico.
- Con el mapeo de pines actual no hay cruces entre el arreglo de sensores y el driver de motores; ambos circuitos pueden estar conectados al Arduino Nano al mismo tiempo sin interferencia.
- Los pines D9 y D10 (PWMA y PWMB) quedaron libres e independientes entre sí, lo que permite en el futuro asignar velocidades distintas a cada motor (por ejemplo, para hacer giros), algo que no hubiera sido posible con un mapeo anterior donde ambos PWM debían compartir un mismo pin.
- El código actual es únicamente una prueba de funcionamiento conjunto (sensores + motores), no la lógica final del robot. Falta definir y documentar el programa con el que el robot va a operar en la pista real.

---

*Documento generado como registro del avance del proyecto, con base en las conexiones y pruebas realizadas hasta el momento. Los componentes se dejan tal como están descritos aquí hasta que se retome el trabajo sobre la lógica final del programa.*
