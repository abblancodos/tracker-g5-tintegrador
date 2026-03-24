# Firmware para Módulo Tracker LoRa/APRS

**Grupo 5** — Taller Integrador  
Instituto Tecnológico de Costa Rica, Escuela de Ingeniería Electrónica  
Autores: Bonilla Andrés ([@abblancodos](https://github.com/abblancodos)), Chassoul Daniel ([@DanielCh16](https://github.com/DanielCh16))  
Última actualización: 17 de marzo de 2026

---

## Descripción del proyecto

Este repositorio contiene el firmware desarrollado para un módulo tracker LoRa/APRS basado en el hardware TTGO T-Beam (ESP32 + SX1278). El tracker genera y transmite paquetes de posición GPS en formato APRS sobre la red LoRa-APRS, operando en la frecuencia 433.775 MHz conforme a la legislación costarricense (PNAF, Decreto N° 44010-MICITT).

El firmware de referencia utilizado para pruebas base es [CA2RXU LoRa APRS Tracker](https://github.com/richonguzman/LoRa_APRS_Tracker). El firmware propio se desarrolla desde cero según los requerimientos del curso.

El objetivo es que los paquetes transmitidos sean recibidos por iGates locales (Grupos 1 y 2), retransmitidos a la red APRS-IS, y visibles en plataformas de monitoreo como [aprs.fi](https://aprs.fi) y [aprsdirect.de](https://aprsdirect.de).

---

## Hardware

| Componente         | Descripción                               |
|--------------------|-------------------------------------------|
| Plataforma         | TTGO T-Beam v1.1                          |
| Microcontrolador   | ESP32 (Xtensa LX6 dual-core)              |
| Transceiver RF     | SX1278 (LoRa 433 MHz)                     |
| GPS                | NEO-6M / NEO-M8N (según versión de placa) |
| Gestión de batería | AXP192 PMIC                               |

### Conexiones de pines

> Pendiente — se completará en la Semana 13. Ver [#18](https://github.com/abblancodos/tracker-g5-tintegrador/issues/18).

---

## Parámetros de configuración LoRa

| Parámetro        | Valor            |
|------------------|------------------|
| Frecuencia       | 433.775 MHz      |
| Spreading Factor | SF12             |
| Bandwidth        | 125 kHz          |
| Coding Rate      | 4/5              |
| Sync Word        | 0x12 (LoRa-APRS) |

---

## Estructura del repositorio

```
.
├── src/
│   ├── main.cpp               # Punto de entrada del firmware
│   ├── gps.cpp / gps.h        # Lectura y parseo de datos GPS
│   ├── lora.cpp / lora.h      # Configuración y transmisión LoRa
│   ├── aprs.cpp / aprs.h      # Codificación de tramas APRS
│   └── power.cpp / power.h    # Gestión de energía (AXP192)
├── include/
│   └── config.h               # Parámetros configurables (callsign, frecuencia, SSID)
├── docs/
│   ├── diagrama_bloques.png
│   ├── maquina_estados.png
│   └── esquematico.pdf
├── platformio.ini
└── README.md
```

---

## Dependencias

- [RadioLib](https://github.com/jgromes/RadioLib) — Comunicación inalámbrica con SX1278
- [TinyGPS++](https://github.com/mikalhart/TinyGPSPlus) — Parseo de sentencias NMEA
- [AXP202X_Library](https://github.com/lewisxhe/AXP202X_Library) — Control del PMIC AXP192
- Firmware de referencia: [CA2RXU LoRa APRS Tracker](https://github.com/richonguzman/LoRa_APRS_Tracker)

Entorno de desarrollo: [PlatformIO](https://platformio.org/) sobre VSCode.

---

## Configuración

Antes de compilar, editar `include/config.h`:

```cpp
#define CALLSIGN        "TU_INDICATIVO"
#define SSID            9           // SSID APRS (0-15)
#define APRS_SYMBOL     '>'         // Símbolo APRS (vehículo)
#define TX_INTERVAL_MS  60000       // Intervalo de transmisión en ms
#define LORA_FREQ       433.775     // Frecuencia en MHz
```

> **Nota legal:** La operación en 433.775 MHz en Costa Rica requiere licencia de radioaficionado
> y licencia de estación ante SUTEL. Esta frecuencia no es de uso libre según el PNAF
> (Decreto N° 44010-MICITT).

---

## Compilación y carga

```bash
git clone https://github.com/abblancodos/tracker-g5-tintegrador.git
cd tracker-g5-tintegrador

pio run
pio run --target upload
pio device monitor --baud 115200
```

---

## Verificación de funcionamiento

- [https://aprs.fi](https://aprs.fi) — Buscar por callsign
- [https://aprsdirect.de](https://aprsdirect.de) — Red LoRa-APRS global

---

## Marco legal (Costa Rica)

El sistema opera bajo el **Plan Nacional de Atribución de Frecuencias (PNAF)**, Decreto Ejecutivo N° 44010-MICITT, Alcance N° 99 a la Gaceta N° 95 del 30 de mayo de 2023.

- La banda 433 MHz corresponde a la banda UHF de radioaficionados (70 cm) en Costa Rica.
- Se requiere licencia de radioaficionado y licencia de estación ante SUTEL.
- Los trámites ante SUTEL pueden tardar entre 10 y 21 semanas.

---

## Cronograma

> Estado actual: **Semana 5**

### Semanas 4–5 — Pruebas base y arquitectura

| # | Responsable | Tarea | Estado |
|---|-------------|-------|--------|
| [#2](https://github.com/abblancodos/tracker-g5-tintegrador/issues/2) | Daniel | Cargar firmware de referencia (CA2RXU) y verificar GPS fix + TX LoRa | ✅ Completado |
| [#3](https://github.com/abblancodos/tracker-g5-tintegrador/issues/3) | Daniel | Verificar aparición de paquetes en aprs.fi con firmware de referencia | ✅ Completado |
| [#4](https://github.com/abblancodos/tracker-g5-tintegrador/issues/4) | Andrés | Diseñar diagrama de bloques del firmware propio | ✅ Completado |
| [#5](https://github.com/abblancodos/tracker-g5-tintegrador/issues/5) | Andrés | Diseñar máquina de estados del firmware | ✅ Completado |

### Semanas 6–8 — Desarrollo del firmware propio

| # | Responsable | Tarea |
|---|-------------|-------|
| [#7](https://github.com/abblancodos/tracker-g5-tintegrador/issues/7) | Andrés | Implementar módulo GPS (`gps.cpp/h`) con TinyGPS++ |
| [#8](https://github.com/abblancodos/tracker-g5-tintegrador/issues/8) | Andrés | Implementar módulo LoRa (`lora.cpp/h`) con RadioLib |
| [#9](https://github.com/abblancodos/tracker-g5-tintegrador/issues/9) | Andrés | Implementar módulo de gestión de energía (`power.cpp/h`) con AXP192 |
| [#10](https://github.com/abblancodos/tracker-g5-tintegrador/issues/10) | Andrés | Diseñar esquemático eléctrico del hardware |

### Semana 8 — Hito: Informe parcial (40%)

| # | Responsable | Tarea |
|---|-------------|-------|
| [#11](https://github.com/abblancodos/tracker-g5-tintegrador/issues/11) | Andrés | Redactar informe parcial (diagrama eléctrico, pseudocódigo, tramas, presupuesto) |

### Semana 9 — Presentación parcial (40%)

| # | Responsable | Tarea |
|---|-------------|-------|
| [#12](https://github.com/abblancodos/tracker-g5-tintegrador/issues/12) | Daniel | Demostración en clase: GPS fix + TX LoRa con firmware propio |

### Semanas 10–12 — Protocolo APRS e integración

| # | Responsable | Tarea |
|---|-------------|-------|
| [#13](https://github.com/abblancodos/tracker-g5-tintegrador/issues/13) | Andrés | Implementar módulo APRS (`aprs.cpp/h`): construcción de trama con datos GPS |
| [#14](https://github.com/abblancodos/tracker-g5-tintegrador/issues/14) | Daniel | Coordinación con iGates (Grupos 1 y 2): verificar recepción y decodificación |
| [#15](https://github.com/abblancodos/tracker-g5-tintegrador/issues/15) | Daniel | Publicación semanal verificable en aprs.fi / aprsdirect.de (semanas 10–15) |

### Semanas 13–15 — Refinamiento y pruebas de campo

| # | Responsable | Tarea |
|---|-------------|-------|
| [#16](https://github.com/abblancodos/tracker-g5-tintegrador/issues/16) | Andrés | Implementar Smart Beaconing (TX adaptativo por velocidad/dirección) |
| [#17](https://github.com/abblancodos/tracker-g5-tintegrador/issues/17) | Daniel | Pruebas de campo: mover tracker por el campus y verificar alcance hacia iGates |
| [#18](https://github.com/abblancodos/tracker-g5-tintegrador/issues/18) | Andrés | Completar sección de pines y cableado en README.md |

### Semana 16 — Defensa final (50%)

| # | Responsable | Tarea |
|---|-------------|-------|
| [#19](https://github.com/abblancodos/tracker-g5-tintegrador/issues/19) | Andrés + Daniel | Redactar informe final con resultados de pruebas de campo |
| [#20](https://github.com/abblancodos/tracker-g5-tintegrador/issues/20) | Daniel | Defensa final: demostrar tracker en movimiento recibido por iGates |

---

## Evidencias de avances y problemas 

### Semana 5 - Programación inicial con repositorio de Richonguzman
- Se realiza la configuración inicial en el dispositivo tracker según el repositorio de Richonguzman y se verifica su funcionamiento. Al cargar el firmware y el filesystem image se realiza la carga inicial del tracker donde se observa que recibe paquetes correctamente, sin embargo, en primera instancia no se logra que la ubicación se actualice mediante el GPS integrado al tracker al salir por un tiempo considerable al aire libre.
- Se realiza la configuración del tracker desde cero, verificando su modelo exacto, y se logra corregir el problema de notificar la ubicación mediante el GPS integrado al dispositivo. Se verifica en aprs.fi que la actualización del tracker Ti0Tec-7 se realizó de manera exitosa.  

---

## Referencias

- A. Maleki et al., "A Tutorial on Chirp Spread Spectrum Modulation for LoRaWAN," *IEEE Open Journal of the Communications Society*, vol. 5, 2024.
- B. Bruninga (WB4APR), "Automatic Packet Reporting System," aprs.org, 2015.
- MICITT, "Plan Nacional de Atribución de Frecuencias," Decreto N° 44010-MICITT, 2023.
- LilyGO, "TTGO T-Beam v1.1 Datasheet," GitHub, 2023.
- CA2RXU, "LoRa APRS Tracker Reference Implementation," GitHub, 2026.

---

## Licencia

MIT License — ver archivo `LICENSE` para detalles.
