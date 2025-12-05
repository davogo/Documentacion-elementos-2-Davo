# Configuración WiFi en modo estación

## Introducción
Este programa configura el ESP32 para operar como **estación WiFi (WIFI_STA)**, permitiendo conectarse a una red existente o habilitar otras funciones como ESP-NOW.

## Código fuente
```cpp
#include <WiFi.h>

void setup() {
  Serial.begin(115200);
  WiFi.mode(WIFI_STA);
  Serial.println("WiFi en modo estación listo");
}

void loop() {
  delay(2000);
}
```

## Explicación
- `WiFi.mode(WIFI_STA)` convierte el ESP32 en cliente WiFi.  
- Es necesario para protocolos como ESP-NOW o conexión a routers.  
- Ideal para módulos que actúan como nodos dentro de una red.
