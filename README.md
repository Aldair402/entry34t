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

# LADIES AND GENTLEEMAN. WE GOT HIM
el enry34t solo es una version modificada del vodafone h300s
Mismo procesador mips, mismo almacenamiento y esquema de particiones. con el mismo uboot de sercomm.
Gracias a dios este modem tiene firmware backups y esta mas docuementado https://openwrt.org/toh/vodafone/vodafoneh300s

# Primera imagen de uboot funcional!
Utilizando el h300s y otros modems de referencia se consiguio el primer boot a uboot real.
No es un parche de la imagen original porque no pude dumpear el firmware de telmex lamentablemente

```
Preloader 4.0 RV:0000029d BD:19042212 TK:msdk-4.8.5-1004k-EB-3.18-u0.9.33-m32t-150818
II: SNAF version check OK!
TLB initial done:
    .ro section works!
    .text and .ro sections work!
Power Led turn on
II: SPIF 13MHz
II: LDO setup...
II: DRAM init...

******* DRAM calibration version: v3.0 (r92780-0xd3ecc72d) *******
AK: TP=0x3 VD=0x1 CC=0x2 OT=0x0 EP=0x0
AK: ENABLE: Triple SYNC
AK: DRAM initialization procedure, wait done bit...Done!
PLL: MEMPLL31 = 0x10407f
PLL: MEMPLL63 = 0xc00001ef
PLL: MEMPLL95 = 0x2ea23980
PLL: MEMPLL127 = 0x2d0f0000
PLL: MEMPLL159 = 0xfff
PLL: MEMPLL191 = 0x0
PLL: MEMPLL223 = 0x0
AK: Start DDR3 Reset Procedure
System FREQ. info:OCP0: 0x3e8 MHz, OCP1: 0x1f4 MHz, DRAM: 0x258 MHz, LX: 0xc8 MHz
Slow bits configuration:
0xb8000380: 0x80000070
0xb8000390: 0x80000070
0xb8000388: 0x22200
0xb8000398: 0x200
0xb80003a0: 0xf000f
SCATS settings:
0xb80040f8: 0xc5
ZQ: Clock GROUP
ZQ: ODT/OCD= 3c/22
ZQ: zq_zctrl_prog=0x808037f  zq_zctrl_status=0x20a198
ZQ: Address GROUP
ZQ: ODT/OCD= 3c/35
ZQ: zq_zctrl_prog=0x18080370  zq_zctrl_status=0x20a154
ZQ: DQ GROUP
ZQ: ODT/OCD= 3c/30
ZQ: zq_zctrl_prog=0x2808033e  zq_zctrl_status=0x20a154
ZQ: DQS GROUP
ZQ: ODT/OCD= 3c/30
ZQ: zq_zctrl_prog=0x3808033e  zq_zctrl_status=0x20a154
ZQ: ZQ Calibration PASS!
ZQ: ZQCL done & ZQCS Enable
MRS: mr[0] = 101a50
MRS: mr[1] = 110004
MRS: mr[2] = 120010
MRS: mr[3] = 130000
AK: Start DDR_Calibration...
AK: Reduce DDR calibration
AK: Bit:0 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:1 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:2 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:3 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:4 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:5 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:6 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:7 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:8 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1d 
AK: Bit:9 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:a max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1b 
AK: Bit:b max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1b 
AK: Bit:c max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1b 
AK: Bit:d max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1b 
AK: Bit:e max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1b 
AK: Bit:f max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1d 
AK: Bit:10 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:11 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:12 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:13 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:14 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:15 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:16 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:17 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:18 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1d 
AK: Bit:19 max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
AK: Bit:1a max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1d 
AK: Bit:1b max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1d 
AK: Bit:1c max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1b 
AK: Bit:1d max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1d 
AK: Bit:1e max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1d 
AK: Bit:1f max_r_s:0 max_r_l:1f max_w_s:0 max_w_l:1f 
ENABLE: DQ output enable delay SYNC
AK: Finish DDR Calibration
DCR: 0x21320000
DTR: tRFC[DRAM_capacity_index]=a0
DTR: DRAM_capacity_index=2
DTR: dram_freq_mhz=258
DTR: tRFC=17700
DTR: DTR0=98644828, DTR1=808051f, DTR2=6516000
AK: Disable Rd_after_Wr, Enable Parallel_Bank/R-Buf Patch
II: Change Stack from 0x9f001edc to 0x81bfffe0
II: U-boot Magic Number is 0x27051956
II: Inflating U-Boot (0x81c00040 -> 0x83c00000)... OK
II: Starting U-boot... 


U-Boot 2011.12.NA-svn29 (Apr 22 2019 - 12:54:04)

Board: VD2, CPU:1000MHz, LX:200MHz, DDR3:600MHz
DRAM: 256 MB
SPI-NAND Flash: C8D1/Mode0 1x128MB
Create bbt:    5    6    7   12   13   14   15   38   39   40   41   42   43   44   45   46   47   48   49   50   51   52   53   54   55   56   57   58   59   60   61   62   63   64   65   66   67   68   69   70   71   72   73   74   75   76   77   78   79   80   81   82   83   84   85   86   87   88   89   90   91   92   93   94   95   96   97   98   99  100  101  102  103  104  105  106  107  108  109  110  111  112  113  114  115  116  117  118  119  120  121  122  123  124  125  126  127  128  129  130  131  132  133  134  135  136  137  138  139  140  141  142  143  144  145  146  147  148  149  150  151  152  153  154  155  156  157  158  159  160  161  162  163  164  165  166  167  168  169  170  171  172  173  174  175  176  177  178  179  180  181  182  183  184  185  186  187  188  189  190  191  192  193  194  195  196  197  198  199  200  201  202  203  204  205  206  207  208  209  210  211  212  213  214  215  216  217  218  219  220  221  222  223  224  225  226  227  228  229  230  231  232  233  234  235  236  237  238  239  240  241  242  243  244  245  246  247  248  249  250  251  252  253  254  255  256  257  258  259  260  261  262  263  264  265  266  267  268  269  270  271  272  273  274  275  276  277  278  279  280  281  282  283  284  285  286  287  288  289  290  291  292  293  294  295  296  297  298  299  300  301  302  303  304  305  306  307  308  309  310  311  312  313  314  315  316  317  318  319  320  321  322  323  324  325  326  327  328  329  330  331  332  333  334  335  336  337  338  339  340  341  342  343  344  345  346  347  348  349  350  351  352  353  354  355  356  357  358  359  360  361  362  363  364  365  366  367  368  369  370  371  372  373  374  375  376  377  378  379  380  381  382  383  384  385  386  387  388  389  390  391  392  393  394  395  396  397  398  399  400  401  402  403  404  405  406  407  408  409  410  411  412  413  414  415  416  417  418  419  420  421 1023 
Don't use bad block skip table!
Loading 16384B env. variables from offset 0x200000
Net:   Ext. phy is not found. 
RTL SwCore
Warning: eth device name has a space!
SC_DEBUG: Nand Partition Table Magic Found at 100000.
reading at Bi 0x8
reading at Bi 0x376
70 f8 2b a1 88 11 

Load Standalone Code from flash to buffer address in DRAM 2...
VD2# 
```

