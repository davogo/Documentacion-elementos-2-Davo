# Creación de Servicios BLE en ESP32

## Introducción
Este código implementa un **servidor BLE (Bluetooth Low Energy)** en el ESP32 con dos servicios y tres características. Cada característica tiene un UUID único y puede ser leída o escrita desde una aplicación BLE.

## Código fuente
```cpp
#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>

#define SERVICE_UUIDA "12345678-1234-1234-1234-1234567890ac"
#define CHARACTERISTIC_UUIDA "abcd1234-ab12-cd34-ef56-abcdef123456"
#define SERVICE_UUIDB "12345678-1234-1234-1234-1234567890ad"
#define CHARACTERISTIC_UUIDB "abcd1234-ab12-cd34-ef56-abcdef123455"
#define CHARACTERISTIC_UUIDC "abcd1234-ab12-cd34-ef56-abcdef123457"
```
...
