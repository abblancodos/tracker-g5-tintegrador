# Firmware para Módulo Tracker LoRa/APRS

**Grupo 5** — Taller Integrador  
Instituto Tecnológico de Costa Rica, Escuela de Ingeniería Electrónica  
Autores: Bonilla Andrés ([@abblancodos](https://github.com/abblancodos)), Chassoul Daniel ([@DanielCh16](https://github.com/DanielCh16))  
Última actualización: junio de 2026

---

## Descripción del proyecto

Este repositorio contiene el firmware desarrollado para un módulo tracker LoRa/APRS basado en el hardware TTGO T-Beam v1.2 (ESP32 + SX1278 + NEO-6M/8M + AXP2101). El tracker genera y transmite paquetes de posición GPS en formato APRS Base91 sobre la red LoRa-APRS, operando en la frecuencia 433.775 MHz conforme a la legislación costarricense (PNAF, Decreto N° 44010-MICITT).

El firmware está basado en el proyecto de referencia [CA2RXU LoRa APRS Tracker](https://github.com/richonguzman/LoRa_APRS_Tracker) de Ricardo Guzmán, empleando las bibliotecas RadioLib, TinyGPS++ y APRSPacketLib. El desarrollo del grupo consistió en la integración, configuración y validación del sistema sobre el hardware asignado.

El objetivo es que los paquetes transmitidos sean recibidos por iGates LoRa-APRS compatibles, retransmitidos a la red APRS-IS, y visibles en plataformas de monitoreo como [aprs.fi](https://aprs.fi) y [aprsdirect.de](https://aprsdirect.de).

---

## Hardware

| Componente         | Descripción                                      |
|--------------------|--------------------------------------------------|
| Plataforma         | TTGO T-Beam v1.2                                 |
| Microcontrolador   | ESP32 (Xtensa LX6 dual-core, 240 MHz)            |
| Transceiver RF     | SX1278 (LoRa 433 MHz, hasta 20 dBm)             |
| GPS                | NEO-6M / NEO-M8N (según revisión de placa)       |
| Gestión de batería | AXP2101 PMIC (I²C)                               |
| Batería            | Li-Po 18650, 2 600 mAh                           |

### Conexiones de pines

| Señal         | GPIO ESP32 | Interfaz  |
|---------------|-----------|-----------|
| GPS TX → ESP  | GPIO 34   | UART2 RX  |
| GPS RX ← ESP  | GPIO 12   | UART2 TX  |
| LoRa CS       | GPIO 18   | SPI CS    |
| LoRa SCK      | GPIO 5    | SPI CLK   |
| LoRa MOSI     | GPIO 27   | SPI MOSI  |
| LoRa MISO     | GPIO 19   | SPI MISO  |
| LoRa RST      | GPIO 23   | GPIO      |
| LoRa DIO0     | GPIO 26   | IRQ TX done |
| AXP2101 SDA   | GPIO 21   | I²C SDA   |
| AXP2101 SCL   | GPIO 22   | I²C SCL   |
| Botón USER    | GPIO 38   | GPIO (pull-up) |

---

## Parámetros de configuración LoRa

| Parámetro        | Valor            |
|------------------|------------------|
| Frecuencia       | 433.775 MHz      |
| Spreading Factor | SF12             |
| Bandwidth        | 125 kHz          |
| Coding Rate      | 4/5              |
| Sync Word        | 0x12 (LoRa-APRS) |
| Potencia TX      | 20 dBm (máx.)   |
| CRC              | Habilitado       |

---

## Estructura del repositorio

```
.
├── src/
│   ├── LoRa_APRS_Tracker.cpp   # Punto de entrada: setup() y loop() de eventos
│   ├── lora_utils.cpp/h        # Configuración SX1278 y TX/RX con RadioLib
│   ├── gps_utils.cpp/h         # Lectura GPS con TinyGPS++ y smart beaconing
│   ├── station_utils.cpp/h     # Construcción de paquetes APRS con APRSPacketLib
│   ├── power_utils.cpp/h       # Gestión de energía AXP2101 (LORA_VCC, GPS_VDD)
│   ├── smartbeacon_utils.cpp/h # Lógica de smart beaconing (distancia, rumbo)
│   ├── sleep_utils.cpp/h       # GPS eco mode (gpsShouldSleep)
│   ├── configuration.cpp/h     # Carga de config desde SPIFFS
│   └── ...                     # Módulos auxiliares (BT, display, WiFi, etc.)
├── data/
│   └── tracker_conf.json       # Configuración: callsign, frecuencia, beaconing
├── docs/
│   ├── diagrama_bloques.png
│   ├── maquina_estados.png
│   ├── diagrama_flujo.png
│   └── informe_final.pdf
├── Imágenes/
│   ├── estados.png
│   ├── flujo.png
│   ├── diagramasistema.png
│   └── recorrido.jpeg
├── platformio.ini
└── README.md
```

---

## Dependencias

| Biblioteca | Versión | Uso |
|---|---|---|
| [RadioLib](https://github.com/jgromes/RadioLib) | ≥ 6.x | Control SX1278: TX, RX, RSSI/SNR, DIO0 ISR |
| [TinyGPS++](https://github.com/mikalhart/TinyGPSPlus) | ≥ 1.0 | Parseo NMEA — `gps.encode()`, `gps.location`, `gps.speed` |
| [APRSPacketLib](https://github.com/richonguzman/APRSPacketLib) | latest | Generación de paquetes APRS Base91, validación de callsign |
| [CA2RXU LoRa APRS Tracker](https://github.com/richonguzman/LoRa_APRS_Tracker) | v2.4.2 | Firmware base completo |

Entorno de desarrollo: [PlatformIO](https://platformio.org/) sobre VSCode.

> **Nota:** El PMIC AXP2101 se controla directamente mediante los métodos de `POWER_Utils`
> incluidos en el firmware base. No se requiere una biblioteca externa adicional.

---

## Configuración

El tracker se configura mediante el archivo `data/tracker_conf.json`, que se carga a la
memoria SPIFFS del ESP32. Los parámetros principales son:

```json
{
  "callsign": "TU_INDICATIVO",
  "ssid": 9,
  "symbol": ">",
  "overlay": "/",
  "comment": "TEC-GR5 v1.0",
  "tx_interval_s": 60,
  "smart_beaconing": true,
  "gps_eco_mode": true
}
```

La frecuencia y los parámetros LoRa se configuran en el mismo archivo bajo la clave `loraTypes`.

> **Nota legal:** La operación en 433.775 MHz en Costa Rica requiere licencia de radioaficionado
> y licencia de estación ante SUTEL. Esta frecuencia no es de uso libre según el PNAF
> (Decreto N° 44010-MICITT). La operación en el marco de este curso se realiza bajo
> **permiso experimental institucional del ITCR**.

---

## Compilación y carga

```bash
git clone https://github.com/abblancodos/tracker-g5-tintegrador.git
cd tracker-g5-tintegrador

# Compilar y cargar firmware
pio run --target upload

# Cargar configuración (SPIFFS)
pio run --target uploadfs

# Monitor serial
pio device monitor --baud 115200
```

---

## Formato del frame LoRa-APRS

El frame transmitido sigue el estándar LoRa-APRS con posición codificada en Base91:

```
[0x3C 0xFF 0x01] + <CALLSIGN>APLRT1,WIDE1-1,WIDE2-1:!<base91pos> comentario
```

| Campo | Valor / descripción |
|---|---|
| Header | `0x3C 0xFF 0x01` — identificador LoRa-APRS, prepended por `sendNewPacket()` |
| Destino | `APLRT1` — identificador estándar para trackers LoRa-APRS |
| Path | `WIDE1-1,WIDE2-1` — rutas de digipeating |
| Posición | Base91 generado por `APRSPacketLib::generateBase91GPSBeaconPacket()` |
| Comentario | String libre, incluye voltaje de batería si está habilitado: `Bat=X.XXV (YYmA)` |

Ejemplo:
```
<TI5GR5-9>APLRT1,WIDE1-1,WIDE2-1:!<base91pos> Bat=3.92V (45mA)
```

---

## Arquitectura del firmware

El firmware implementa un **bucle de eventos cooperativo** (`loop()`) sin FreeRTOS.
La transmisión se dispara mediante la bandera `sendUpdate`, que se activa por:

- **Timer fijo** (intervalo configurado, por defecto 60 s)
- **Smart beaconing** — `calculateDistanceTraveled()`: distancia desde última TX > umbral
- **Smart beaconing** — `calculateHeadingDelta()`: cambio de rumbo > ángulo mínimo
- **Botón USER** (GPIO 38) — transmisión forzada inmediata

### Máquina de estados conceptual

```
INIT → GPS_WAIT → BUILD_PKT → TX_LORA → CONFIRMA → SLEEP → (GPS_WAIT)
                                              ↓ FAIL
                                           ERROR → BUILD_PKT (max 3 reintentos)
                                              ↓ agotado
                                           SLEEP
```

---

## Verificación de funcionamiento

Las pruebas de campo se realizaron de forma independiente en el campus del ITCR:

- Receptor LoRa de referencia (segundo T-Beam en modo RX) verificó la recepción
- RSSI y SNR monitoreados por serial en cada recepción
- Paquetes decodificados correctamente en campo abierto y con obstrucción parcial
- Smart beaconing y GPS Eco Mode operaron conforme al diseño
- Botón de TX forzada respondió en < 1 s en todos los casos de prueba

Para verificar en red APRS-IS (cuando hay iGate disponible):

- [https://aprs.fi](https://aprs.fi) — buscar por callsign
- [https://aprsdirect.de](https://aprsdirect.de) — red LoRa-APRS global

---

## Marco legal (Costa Rica)

El sistema opera bajo el **Plan Nacional de Atribución de Frecuencias (PNAF)**,
Decreto Ejecutivo N° 44010-MICITT, Alcance N° 99 a la Gaceta N° 95 del 30 de mayo de 2023.

- La banda 430–440 MHz está atribuida al servicio de **Aficionados** como servicio primario (nota CR-013).
- Se requiere licencia de radioaficionado (categoría mínima Intermedio) y licencia de estación ante SUTEL.
- El segmento 430–440 MHz **no** aparece en el Adendum VII (frecuencias de uso libre).
- La clase de emisión más aproximada para LoRa/CSS es **F1D** (FM digital, datos).
- Los trámites ante SUTEL pueden tardar entre 10 y 21 semanas.
- La operación en el marco de este curso se realiza bajo **permiso de estación experimental**
  gestionado por el ITCR ante SUTEL.

---

## Cronograma

> Estado: **Semana 16 — Proyecto completado** ✅

### Semanas 4–5 — Pruebas base y arquitectura

| # | Responsable | Tarea | Estado |
|---|-------------|-------|--------|
| [#2](https://github.com/abblancodos/tracker-g5-tintegrador/issues/2) | Daniel | Cargar firmware de referencia (CA2RXU) y verificar GPS fix + TX LoRa | ✅ |
| [#3](https://github.com/abblancodos/tracker-g5-tintegrador/issues/3) | Daniel | Verificar aparición de paquetes en aprs.fi con firmware de referencia | ✅ |
| [#4](https://github.com/abblancodos/tracker-g5-tintegrador/issues/4) | Andrés | Diseñar diagrama de bloques del firmware propio | ✅ |
| [#5](https://github.com/abblancodos/tracker-g5-tintegrador/issues/5) | Andrés | Diseñar máquina de estados del firmware | ✅ |

### Semanas 6–8 — Desarrollo del firmware propio

| # | Responsable | Tarea | Estado |
|---|-------------|-------|--------|
| [#7](https://github.com/abblancodos/tracker-g5-tintegrador/issues/7) | Andrés | Integrar módulo GPS con TinyGPS++ | ✅ |
| [#8](https://github.com/abblancodos/tracker-g5-tintegrador/issues/8) | Andrés | Integrar módulo LoRa con RadioLib | ✅ |
| [#9](https://github.com/abblancodos/tracker-g5-tintegrador/issues/9) | Andrés | Integrar gestión de energía AXP2101 (`POWER_Utils`) | ✅ |
| [#10](https://github.com/abblancodos/tracker-g5-tintegrador/issues/10) | Andrés | Diseñar esquemático eléctrico del hardware | ✅ |

### Semana 8 — Hito: Informe parcial (40%)

| # | Responsable | Tarea | Estado |
|---|-------------|-------|--------|
| [#11](https://github.com/abblancodos/tracker-g5-tintegrador/issues/11) | Andrés | Redactar informe parcial (diagrama eléctrico, pseudocódigo, tramas, presupuesto) | ✅ |

### Semana 9 — Presentación parcial (40%)

| # | Responsable | Tarea | Estado |
|---|-------------|-------|--------|
| [#12](https://github.com/abblancodos/tracker-g5-tintegrador/issues/12) | Daniel | Demostración en clase: GPS fix + TX LoRa con firmware configurado | ✅ |

### Semanas 10–12 — Protocolo APRS e integración

| # | Responsable | Tarea | Estado |
|---|-------------|-------|--------|
| [#13](https://github.com/abblancodos/tracker-g5-tintegrador/issues/13) | Andrés | Implementar construcción de trama APRS Base91 con APRSPacketLib | ✅ |
| [#14](https://github.com/abblancodos/tracker-g5-tintegrador/issues/14) | Daniel | Pruebas de recepción con receptor LoRa de referencia | ✅ |
| [#15](https://github.com/abblancodos/tracker-g5-tintegrador/issues/15) | Daniel | Pruebas de cobertura semanales en campus ITCR (semanas 10–15) | ✅ |

### Semanas 13–15 — Refinamiento y pruebas de campo

| # | Responsable | Tarea | Estado |
|---|-------------|-------|--------|
| [#16](https://github.com/abblancodos/tracker-g5-tintegrador/issues/16) | Andrés | Smart Beaconing (TX adaptativo por distancia/rumbo) | ✅ |
| [#17](https://github.com/abblancodos/tracker-g5-tintegrador/issues/17) | Daniel | Pruebas de campo: recorrido campus ITCR, verificar alcance y cobertura | ✅ |
| [#18](https://github.com/abblancodos/tracker-g5-tintegrador/issues/18) | Andrés | Completar sección de pines y cableado en README.md | ✅ |

### Semana 16 — Defensa final (50%)

| # | Responsable | Tarea | Estado |
|---|-------------|-------|--------|
| [#19](https://github.com/abblancodos/tracker-g5-tintegrador/issues/19) | Andrés + Daniel | Redactar informe final con resultados de pruebas de campo | ✅ |
| [#20](https://github.com/abblancodos/tracker-g5-tintegrador/issues/20) | Daniel | Defensa final del proyecto | ✅ |

---

## Resultados

Las pruebas de campo se realizaron de forma independiente en el campus del ITCR durante
las semanas 10–14, portando el tracker en recorridos a pie alrededor del edificio de
Electrónica, parqueo central y accesos externos.

- Los paquetes fueron recibidos y decodificados correctamente en todos los puntos
  evaluados en campo abierto.
- En zonas con obstrucción parcial (pasillos, bajo aleros) se observó reducción del
  RSSI sin pérdida de paquetes.
- El Smart Beaconing redujo la tasa de balizamiento en reposo y la aumentó durante
  el movimiento, comportamiento verificado por los intervalos entre paquetes en el
  monitor serial del receptor.
- La función de TX forzada por botón respondió en menos de 1 segundo en todos los
  casos de prueba.
- La batería sostuvo la operación completa de todas las sesiones sin alcanzar voltaje
  de corte, con margen al finalizar.

El recorrido de prueba está documentado en `Imágenes/recorrido.jpeg`.

---

## Evidencias de avances

### Semana 5 — Programación inicial con firmware de referencia

Se carga el firmware CA2RXU en el tracker T-Beam v1.2 y se verifica su funcionamiento.
Al cargar el firmware y el filesystem image se realiza la configuración inicial donde
se observa que el tracker recibe paquetes correctamente; en primera instancia el GPS
no actualiza la ubicación al salir al aire libre. Tras verificar el modelo exacto de
placa (AXP2101, no AXP192) y ajustar la inicialización del PMIC, se corrige el problema.
Se verifica en aprs.fi que la posición del tracker Ti0Tec-7 se actualiza exitosamente.

### Semanas 6–9 — Arquitectura, diseño y documentación

- Identificación y documentación del hardware exacto: T-Beam v1.2 con AXP2101, SX1278,
  NEO-6M/8M.
- Diseño del diagrama eléctrico: GPS por UART2, SX1278 por SPI, AXP2101 por I²C.
- Justificación técnica de protocolos y selección de periféricos.
- Elaboración del diagrama de bloques, máquina de estados, diagrama de flujo y presupuesto.
- Pseudocódigo completo de las rutinas GPS, LoRa y APRS.
- Definición de la trama APRS: formato Base91, destino APLRT1, header `0x3C 0xFF 0x01`.
- Código base subido al repositorio de GitHub con comentarios explicativos.

### Semanas 10–12 — Integración y pruebas de cobertura

- Integración de APRSPacketLib para construcción de paquetes Base91.
- Identificación y corrección del bug crítico: el AXP2101 debe habilitar `LORA_VCC`
  y `GPS_VDD` antes de cualquier otro módulo; de lo contrario RadioLib bloquea en
  `while(true)`.
- Pruebas de recepción con receptor LoRa de referencia portado por el segundo integrante.
- Verificación de correcta decodificación del callsign, posición, velocidad y rumbo.
- Smart Beaconing (`calculateDistanceTraveled` + `calculateHeadingDelta`) validado
  comparando los intervalos entre paquetes en el monitor serial.

### Semanas 13–15 — Pruebas de campo y refinamiento

- Recorridos a pie en el campus del ITCR (edificio de Electrónica, parqueo, accesos).
- Verificación de cobertura en campo abierto y con obstrucción parcial.
- GPS Eco Mode (`gpsShouldSleep`) verificado: mantiene el fix GPS entre transmisiones.
- Botón de TX forzada (GPIO 38) probado en campo: respuesta < 1 s en todos los casos.
- Ajuste fino de parámetros de smart beaconing para el entorno de campus.

### Semana 16 — Informe final y defensa

- Redacción del informe final en formato IEEE con todas las secciones completadas:
  fundamentos APRS/LoRa, marco regulatorio PNAF, descripción de la aplicación,
  planteamiento del desarrollo de software, resultados cualitativos de campo.
- Elaboración de la presentación del proyecto (12 slides).
- Defensa final del proyecto ante el profesor.

---

## Referencias

- A. Maleki et al., "A Tutorial on Chirp Spread Spectrum Modulation for LoRaWAN,"
  *IEEE Open Journal of the Communications Society*, vol. 5, 2024.
- B. Bruninga (WB4APR), "Automatic Packet Reporting System," aprs.org, 2015.
- J. Mottern (DV7GDL), "APRS Demystified," how.aprs.works, 2025.
- MICITT, "Plan Nacional de Atribución de Frecuencias," Decreto N° 44010-MICITT, 2023.
- LilyGO, "TTGO T-Beam v1.2 Datasheet," GitHub, 2023.
- R. Guzmán (CA2RXU), "LoRa APRS Tracker," GitHub, 2026.
- J. Jirák (jgromes), "RadioLib," GitHub, 2024.
- RAKwireless, "RAK5146 Concentrator Datasheet," docs.rakwireless.com, 2024.
- M. Arora et al., "PyroGuardian: IoT-Enabled System for Firefighting Environments,"
  arXiv:2411.03654, 2025.

---

## Licencia

MIT License — ver archivo `LICENSE` para detalles.
