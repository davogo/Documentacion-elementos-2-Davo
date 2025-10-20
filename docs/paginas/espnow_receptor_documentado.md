# Comunicación ESP-NOW: Receptor

## Introducción
El siguiente código convierte el ESP32 en un **receptor ESP-NOW**, una tecnología de comunicación directa sin necesidad de una red WiFi tradicional.

## Código fuente
```cpp
#include <esp_now.h>
#include <WiFi.h>

typedef struct struct_msj {
  char a[32];
  int b;
  float c;
  bool d;
} struct_msj;

struct_msj datosRecibidos;

void OnDataRecv(const uint8_t * mac, const uint8_t *tempData, int tam) {
  memcpy(&datosRecibidos, tempData, sizeof(datosRecibidos));
  Serial.print("Bytes recibidos: "); Serial.println(tam);
  Serial.print("Char: "); Serial.println(datosRecibidos.a);
  Serial.print("Int: "); Serial.println(datosRecibidos.b);
  Serial.print("Float: "); Serial.println(datosRecibidos.c);
  Serial.print("Bool: "); Serial.println(datosRecibidos.d);
  Serial.println();
}

void setup() {
  Serial.begin(115200);
  WiFi.mode(WIFI_STA);
  if (esp_now_init() != ESP_OK) {
    Serial.println("Error inicializando ESP-NOW");
    return;
  }
  esp_now_register_recv_cb(OnDataRecv);
}

void loop() {}
```

## Explicación detallada
- **WiFi.mode(WIFI_STA):** El ESP32 debe estar en modo estación.  
- **esp_now_init():** Inicializa el protocolo ESP-NOW.  
- **OnDataRecv:** Se ejecuta automáticamente al recibir datos.  
- **Estructura struct_msj:** Permite recibir datos de distintos tipos (texto, número, flotante y booleano).  
- Esta técnica se usa para redes rápidas entre varios ESP32 sin router.
