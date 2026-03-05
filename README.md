# Firmware para Modulo Tracker LoRa/APRS

**Grupo 5** — Taller Integrador  
Instituto Tecnológico de Costa Rica, Escuela de Ingeniería Electrónica  
Autores: Bonilla Andrés, Chassoul Daniel

---

## Descripción del proyecto

Este repositorio contiene el firmware desarrollado para un módulo tracker LoRa/APRS basado en el hardware TTGO T-Beam (ESP32 + SX1278). El tracker genera y transmite paquetes de posición GPS en formato APRS sobre la red LoRa-APRS, operando en la frecuencia 433.775 MHz conforme a la legislación costarricense (PNAF, Decreto N° 44010-MICITT).

El objetivo es lograr que los paquetes transmitidos sean recibidos por iGates locales, retransmitidos a la red APRS-IS, y visibles en plataformas de monitoreo como [aprs.fi](https://aprs.fi) y [aprsdirect.de](https://aprsdirect.de).

---

## Hardware

| Componente | Descripción |
|---|---|
| Plataforma | TTGO T-Beam v1.1 |
| Microcontrolador | ESP32 (Xtensa LX6 dual-core) |
| Transceiver RF | SX1278 (LoRa 433 MHz) |
| GPS | NEO-6M / NEO-M8N (según versión de la placa) |
| Gestión de batería | AXP192 PMIC |

---

## Parámetros de configuración LoRa

| Parámetro | Valor |
|---|---|
| Frecuencia | 433.775 MHz |
| Spreading Factor | SF12 |
| Bandwidth | 125 kHz |
| Coding Rate | 4/5 |
| Sync Word | 0x12 (LoRa-APRS) |

Esta configuración prioriza cobertura y confiabilidad sobre tasa de datos, apropiado para la transmisión periódica de paquetes cortos de posición.

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

- [RadioLib](https://github.com/jgromes/RadioLib) — Biblioteca de comunicación inalámbrica universal para Arduino/ESP32
- [TinyGPS++](https://github.com/mikalhart/TinyGPSPlus) — Parseo de sentencias NMEA
- [AXP202X_Library](https://github.com/lewisxhe/AXP202X_Library) — Control del PMIC AXP192
- Referencia de implementación: [LoRa APRS Tracker (OE5BPA)](https://github.com/lora-aprs/LoRa_APRS_Tracker)

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

> **Nota legal:** La operación en 433.775 MHz en Costa Rica requiere licencia de radioaficionado y licencia de estación ante SUTEL. Esta frecuencia no es de uso libre según el PNAF (Decreto N° 44010-MICITT).

---

## Compilación y carga

```bash
# Clonar el repositorio
git clone https://github.com/grupo5-taller/lora-aprs-tracker.git
cd lora-aprs-tracker

# Compilar con PlatformIO
pio run

# Cargar al TTGO T-Beam
pio run --target upload

# Monitor serial
pio device monitor --baud 115200
```

---

## Verificación de funcionamiento

Una vez en operación, los paquetes deben ser visibles en:

- [https://aprs.fi](https://aprs.fi) — Buscar por callsign
- [https://aprsdirect.de](https://aprsdirect.de) — Red LoRa-APRS global

---

## Marco legal (Costa Rica)

El sistema opera bajo el marco del **Plan Nacional de Atribución de Frecuencias (PNAF)**, Decreto Ejecutivo N° 44010-MICITT, Alcance N° 99 a la Gaceta N° 95 del 30 de mayo de 2023.

- La banda 433 MHz corresponde a la banda UHF de radioaficionados (70 cm) en Costa Rica.
- Se requiere licencia de radioaficionado y licencia de estación ante SUTEL.
- Los trámites ante SUTEL pueden tardar entre 10 y 21 semanas.
- La PIRE permitida para esta banda está definida en el PNAF según la clase de emisión utilizada.

---

## Cronograma (Semanas 3–15)

```
Tarea                                    S3  S4  S5  S6  S7  S8  S9  S10 S11 S12 S13 S14 S15
─────────────────────────────────────────────────────────────────────────────────────────────
DISEÑO E INVESTIGACION
  Revisión de hardware (T-Beam/SX1278)   [===]
  Estudio de protocolo APRS/AX.25            [===]
  Estudio de librería RadioLib                   [=]
  Diagrama de bloques del sistema         [===]
  Máquina de estados del firmware             [===]
  Esquemático y conexiones eléctricas         [===]

INFORME PARCIAL (Semana 8)
  Definición de tramas de datos                   [===]
  Pseudocódigo del sistema                        [===]
  Redacción del informe parcial                       [=======]

IMPLEMENTACION
  Configuración entorno PlatformIO                        [=]
  Módulo GPS (lectura NMEA, TinyGPS++)                    [===]
  Módulo LoRa (RadioLib, SX1278)                              [===]
  Codificador de tramas APRS                                  [===]
  Integración GPS + LoRa + APRS                                   [===]
  Gestión de energía (AXP192)                                     [===]

PRUEBAS Y AJUSTE
  Pruebas de transmisión RF local                                     [===]
  Verificación en aprs.fi y aprsdirect.de                             [=======]
  Ajuste de parámetros LoRa (SF/BW/CR)                                    [===]
  Prueba de integración con iGate y servidor                                  [===]

DOCUMENTACION FINAL
  Documentación del código (comentarios)                  [================]
  Commits regulares en GitHub                 [===============================]
  Redacción informe final                                                 [===]
─────────────────────────────────────────────────────────────────────────────────────────────
```

### Hitos principales

| Semana | Hito | Peso |
|--------|------|------|
| 3 | Presentación inicial e informe de investigación (APRS, LoRa, PNAF) | Evaluado |
| 8 | Informe parcial: diagramas, pseudocódigo, esquemático, tramas | 40% |
| 10–15 | Aparición semanal verificable en aprs.fi / aprsdirect.de | 1 pt/semana |
| 16 | Informe final + presentación + código final documentado | 50% |

---

## Referencias

- A. Maleki et al., "A Tutorial on Chirp Spread Spectrum Modulation for LoRaWAN," *IEEE Open Journal of the Communications Society*, vol. 5, 2024.
- B. Bruninga (WB4APR), "Automatic Packet Reporting System," aprs.org, 2015.
- MICITT, "Plan Nacional de Atribución de Frecuencias," Decreto N° 44010-MICITT, 2023.
- LilyGO, "TTGO T-Beam v1.1 Datasheet," GitHub, 2023.
- OE5BPA, "LoRa APRS Tracker Reference Implementation," GitHub, 2024.

---

## Licencia

MIT License — ver archivo `LICENSE` para detalles.
