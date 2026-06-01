#  Sercomm Entry 34T – Technical Wiki

## Introducción

El **Sercomm Entry 34T** es un módem/router residencial diseñado para conexión a internet y distribución de red local. Es comúnmente proporcionado por INFINITUM como CPE (Customer Premises Equipment).  
El dispositivo corre un **Linux embebido**, con un bootloader U-Boot y almacenamiento SPI-NAND. Está pensado para uso cerrado, con el firmware controlado por el proveedor de servicio.

Este wiki documenta información de **arranque**, **modo de recuperación**, y **acceso UART**, para propósitos educativos y de diagnóstico de hardware.

---

## Inicio / Boot

El arranque del Entry 34T sigue estas etapas:

1. **Preloader**
   - Inicializa la memoria RAM y periféricos básicos
   - Realiza calibración de DRAM y NAND

2. **U-Boot**
   - Inicializa almacenamiento y verifica particiones
   - Decide si arrancar desde firmware activo o entrar en modo recuperación
Warning: Nuestra version de uboot es prod por lo que no permite detener el arranque

3. **Kernel Linux**
   - Monta el sistema de archivos (SquashFS/UBI)
   - Inicia los servicios del router (LAN, WiFi, provisioning)

4. **User space**
   - Web UI del proveedor
   - TR-069 / provisioning
   - Servicios de red y WiFi

---

## Modo Download / Recovery

El dispositivo puede entrar en **modo download/recovery** para restauración de firmware. Esto se activa cuando:

- Se presiona el **botón RESET** durante el encendido, o
- El bootloader detecta corrupción en la imagen principal

**Señales típicas:**

- LEDs de **DSL** y **WiFi** parpadeando en patrón específico
- UART muestra mensajes como:
```
***************************************************
    Sercomm Boot Version 3.1.2.0, at Apr 22 2019, 12:54:49

***************************************************

Entering Firmware : Everything is OK.
Ethernet is closed!
[rtl8676_halt(63)] Halt
can Not find parammeter dual-flag
DEBUG_INF:===================================================
DEBUG_INF:Sercomm Upgrade(Module Ver 2.16.04.34) Start!
DEBUG_INF:===================================================
DEBUG_INF:ecc bytes 40
DEBUG_DBG:<BurnedLoop          > has been read before erased!
DEBUG_DBG:<MT_Flag             > has been read before erased!
DEBUG_DBG:<boot-version        > has been read before erased!
DEBUG_DBG:<boot-pid            > has been read before erased!
DEBUG_ERR:Partition 2 has 0 good blocks, while Param dual-flag should be in good block !
DEBUG_DBG:init state, listening
```
- El dispositivo espera recibir firmware válido a través de protocolo de recuperación
- Solo permite interacciones definidas por el bootloader
- Acceso completo a Linux no está disponible, pero UART sigue mostrando logs de arranque

---

## UART Console

El Entry 34T incluye **puerto UART** accesible mediante soldaduras a la placa. Esto permite monitorear el arranque y ver el modo recovery.

**Configuración típica de UART:**

| Parámetro          | Valor             |
|-------------------|-----------------|
| Baudrate           | 115200           |
| Bits de datos      | 8                |
| Paridad            | Ninguna          |
| Stop bits          | 1                |
| Flow control       | Ninguno          |

## Conexión física del UART

El UART del Sercomm Entry 34T se encuentra en un **header de 4 pines** en la placa. Normalmente los pines están etiquetados como:

| Pin | Señal |
|-----|-------|
| 1   | GND   |
| 2   | TX    |
| 3   | RX    |
| 4   | VCC (opcional, no conectar al adaptador USB-UART) |

> ⚠️ Solo conectar GND, TX y RX al adaptador USB-UART. No aplicar voltaje externo al VCC del UART.

**Conexión**

- **GND** es el cable blanco en la imagen.
- **TX (del router)** cable amarillo  
- **RX (del router)** cable naranja

<img width="400" height="500" alt="2026-05-31-19-07-24-690" src="https://github.com/user-attachments/assets/94d49caf-2800-49da-a0a2-087341e0dea3" />

**Consejos:**
Mantener la soldadura lo más limpia posible para evitar cortocircuitos  
Intenta quitar todo el flux recidual que puedas. si dos conexiones hacen contacto no podras leer UART

Una vez conectado, abrir un terminal serie con los parámetros indicados anteriormente (115200 8N1) para monitorear el arranque.
## Usuario root
Hasta ahora lo unico que se conoce acerca del usuario root es que el username es TELMEX. se deberia ejecutar un bruteforce para saber la contrasenya
## Logs de inicio
[bootlog.txt](https://github.com/user-attachments/files/28448573/bootlog.txt)

