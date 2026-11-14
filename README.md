# Anycubic ACE Pro en Creality Spark i7

Integración no oficial del **Anycubic ACE Pro (4 filamentos)** con la **Creality Spark i7**, utilizando Klipper y Tina Linux (OpenWrt).

El proyecto permite realizar impresiones multicolor mediante cambios automáticos de filamento, aprovechando el hardware original de la impresora.

## Características

- Control de los 4 canales del ACE Pro.
- Cambios automáticos mediante `T0`, `T1`, `T2` y `T3`.
- Corte, carga y descarga automática de filamento.
- Integración con el sensor de filamento.
- Purga y limpieza utilizando las funciones originales de Creality.
- Compatibilidad con PrusaSlicer en modo multimaterial.

## Comunicación con el ACE Pro

### Primer intento: libusb

Inicialmente se utilizó `libusb` para comunicarse con el ACE Pro, debido a que Tina Linux no reconocía el dispositivo como puerto serial.

Aunque se conseguía intercambiar información con el AMS, la implementación de `ace.py` mediante `libusb` no funcionaba correctamente.

Por este motivo, se decidió utilizar comunicación USB Serial.

### Compilación del módulo cdc_acm.ko

La Spark i7 utiliza Tina Linux, basado en OpenWrt, con un kernel Linux **5.4.61** y arquitectura ARMv7.

El kernel original tenía soporte USB, pero no incluía habilitado el controlador CDC ACM necesario para reconocer el ACE Pro como puerto serial.

Para solucionar esto:

1. Se preparó un entorno de compilación cruzada en Linux, utilizando una BIQU Hurakan con CB1.
2. Se utilizó un árbol de fuentes Linux 5.4.61 de OpenCentauri como base.
3. Se habilitó `CONFIG_USB_ACM=m` en la configuración del kernel.
4. Se compiló el controlador CDC ACM para arquitectura ARM.
5. Se transfirió el módulo compilado a la Spark i7 y se cargó en Tina Linux.



**Importante:**  Para compilar es necesario preparar previamente las fuentes y la configuración compatibles con el kernel de la impresora.

Una vez instalado el módulo, Tina Linux consiguió reconocer el ACE Pro:

```text
cdc_acm 1-1.3:1.0: ttyACM0: USB ACM device
```

Esto permitió utilizar `/dev/ttyACM0` para establecer comunicación serial con el AMS.


## Bug conocido del ACE Pro

El ACE Pro presenta un problema de desconexiones aproximadamente cada **3 segundos cuando permanece en reposo**.

Este comportamiento corresponde al propio AMS y no se consiguió corregir.

Durante las pruebas se comprobó que no impedía realizar cambios automáticos de filamento ni completar impresiones multicolor.

Este problema es independiente de las dificultades que presentó nuestra implementación inicial de `ace.py` mediante `libusb`.

## Integración con Klipper

Se adaptó una extensión de Klipper (`ace.py`) y se desarrollaron macros para controlar el ACE Pro.

Los comandos originales `T0`–`T3` de Creality se reemplazaron mediante `rename_existing`, conservando los originales con otros nombres.

Ejemplo:

```ini
[gcode_macro T0]
rename_existing: T800
gcode:
    ACE_CHANGEFILAMENT TOOL=0
```

El sistema aprovecha el cortador, extrusor, sensor de filamento y rutinas de purga originales de la Spark i7.

## Estado del proyecto
**Instalación  la impresora no cuenta con github así que tuvimos que hacer nuestro propio modulo Ace para que sea facil
** incluye los macros y variables y printer.cfg * ace.py
**No nos hacemos responsable de la manipulación de la maquina en caso de problemas todo queda bajo tu responsabilidad .


**Funcional — impresión multicolor probada correctamente.**

- [x] Compilación del módulo CDC ACM para Tina Linux.
- [x] Reconocimiento del ACE Pro mediante USB Serial.
- [x] Comunicación serial funcional.
- [x] Control de los cuatro filamentos.
- [x] Cambios automáticos durante la impresión.
- [x] Integración con PrusaSlicer.
- [ ] Optimización del tiempo de cambio de filamento.
- [ ] Purga variable según los colores.
- [ ] Mejoras en la recuperación ante errores.

## Compatibilidad

- **Impresora:** Creality Sparkx i7
- **AMS:** Anycubic ACE Pro
- **Sistema operativo:** Tina Linux / OpenWrt
- **Kernel:** Linux 5.4.61
- **Arquitectura:** ARMv7
- **Firmware:** Klipper modificado por Creality
- **Comunicación:** USB CDC ACM
- **Slicer:** cualquiera

## Aviso

Proyecto experimental y no oficial, sin relación con Creality ni Anycubic.

Se recomienda realizar copias de seguridad antes de modificar el firmware o instalar componentes del kernel.
