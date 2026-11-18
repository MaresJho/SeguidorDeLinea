# AVANTE — Robot Seguidor de Línea

Robot seguidor de línea autónomo desarrollado para **InnovaTecNM 2026**, categoría *Robots Seguidores de Línea*.

- **País:** Colombia
- **Universidad:** Corporación Universitaria Americana

---

## Descripción del proyecto

AVANTE combina fabricación aditiva personalizada con electrónica comercial de fácil acceso, buscando un equilibrio entre tamaño compacto, funcionalidad y capacidad de adaptación para el seguimiento autónomo de una trayectoria.

Un Arduino Nano procesa las señales de un módulo seguidor de línea de ocho sensores y genera las señales de control para dos micromotores reductores N20 de 300 RPM mediante un puente H TB6612FNG. La alimentación usa dos celdas Li-ion de 3.7 V en serie y un regulador DC-DC LM2596.

El robot se desplaza de forma autónoma sobre una pista con una línea de referencia. Los sensores determinan la posición de la línea y un algoritmo de control PD calcula correcciones sobre la velocidad de los motores para mantenerlo sobre la trayectoria.

---

## Hardware

| Componente | Función |
| --- | --- |
| Arduino Nano | Procesa las lecturas de los sensores y ejecuta el algoritmo de control |
| Módulo seguidor de línea de 8 sensores | Detecta la posición de la línea |
| TB6612FNG | Controla los dos motores de forma independiente (dirección y PWM) |
| Micromotores reductores N20 de 300 RPM (x2) | Tracción diferencial |
| Baterías Li-ion 3.7 V (x2) en serie | Alimentación (~7.4 V nominales) |
| LM2596 DC-DC | Regulador para adecuar la tensión del sistema |
| Ruedas | Contacto con la superficie (conjunto seleccionado para las pruebas) |
| Cableado y conexiones | Interconexión entre microcontrolador, sensores, driver y alimentación |

**Chasis:** diseño propio modelado en SolidWorks, preparado en Bambu Studio e impreso en PLA con una Bambu Lab P1S.

---

## Software

| Herramienta | Uso |
| --- | --- |
| SolidWorks | Modelado 3D del chasis |
| Bambu Studio | Preparación de la impresión 3D (PLA) |
| Arduino IDE (C++) | Programación del robot |

**Algoritmo:** el programa lee los ocho sensores, calcula la posición relativa de la línea y aplica un control **PD** para obtener la corrección. Con esa corrección se ajustan las señales PWM de los motores izquierdo y derecho a través del TB6612FNG, haciendo correcciones continuas de trayectoria.

```
corrección = VEL_BASE · (KP·e + KP_CURVA·e·|e| + KD·de/dt)
PWM_izq = VEL_BASE + corrección
PWM_der = VEL_BASE − corrección
```

El desarrollo fue progresivo: primero se validaron sensores y motores por separado, luego se integraron con el controlador y finalmente se hicieron pruebas de seguimiento para ajustar los parámetros.

---

## Innovación

- **Diseño mecánico propio** impreso en 3D, adaptado a los componentes disponibles y fácil de modificar y reimprimir sin depender de un chasis comercial.
- **Integración entre fabricación digital y control autónomo:** el diseño físico, la electrónica y el software se desarrollan en conjunto.
- **Evolución iterativa:** las pruebas generan cambios en el diseño mecánico, la electrónica o los parámetros del algoritmo, que luego se verifican de nuevo con fabricación y pruebas.

---

## Pruebas de funcionamiento

Los resultados de recorrido se registran con pruebas cronometradas.

| Prueba | Tiempo | Resultado | Observaciones |
| --- | --- | --- | --- |
| Prueba 1 | 40 seg | Aprobada | Velocidad configurada a 90. El robot completó el recorrido de forma estable. |
| Prueba 2 | 38 seg | Aprobada | Velocidad aumentada a 110. Presentó 2 salidas, pero logró reincorporarse y completar el recorrido. |
| Prueba 3 | 26 seg | Aprobada | Velocidad aumentada a 160. Solo un pequeño desvío en una curva y completó el recorrido. |

---

## Modelo construido

*Pendiente: fotografías del prototipo final (diferentes vistas, montaje de componentes y configuración para las pruebas).*

---

## Equipo

### Estudiantes — Ingeniería de Sistemas

| Nombre | Semestre | Correo institucional |
| --- | --- | --- |
| Melissa Lucia Iriarte Montero | 7 | melissa_iriarte6037@americana.edu.co |
| Ivan Andres Coronado Hamburger | 4 | ivan_coronado1142@americana.edu.co |
| Jhonier De Jesus Mariano Escorcia | 8 | jhonier_mariano1728@americana.edu.co |
| Juan Jose Barrera Hurtado | 5 | juanbarrera@americana.edu.co |

### Asesor

| Nombre | Departamento | Correo institucional |
| --- | --- | --- |
| Emanuel Elias Gutierrez Paez | Facultad de Ingeniería – Ingeniería Industrial | emanuelgutierrez@americana.edu.co |

---

*AVANTE — Seguidor de Línea · InnovaTecNM 2026*
