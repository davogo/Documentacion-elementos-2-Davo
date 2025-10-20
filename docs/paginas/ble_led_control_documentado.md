# Control de LED mediante BLE

## Introducción
Este código permite encender y apagar el LED integrado del ESP32 utilizando **Bluetooth Low Energy (BLE)**. Los comandos “0” y “1” se envían desde una app BLE.

## Código fuente
```cpp
#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>

#define SERVICE_UUID "12345678-1234-1234-1234-1234567890ab"
#define CHARACTERISTIC_UUID "abcd1234-ab12-cd34-ef56-abcdef123456"
```
...
