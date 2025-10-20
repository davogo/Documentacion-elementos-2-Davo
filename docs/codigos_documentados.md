
# Bitácora de Desarrollo del Proyecto

Este documento explica el funcionamiento de cada bloque de código contenido en la bitácora del proyecto.  
Cada sección está documentada según el encabezado original (`///Neopixel`, `///Ble-char`, `///ble Write`, `///Wifi`, `///ESPnow`, `///String 1`).  
Las explicaciones incluyen detalles sobre las funciones, su propósito y el flujo del programa.

---

## ///Neopixel

### Encendido Fijo del LED NeoPixel

```cpp
#include <Adafruit_NeoPixel.h>  // Librería para controlar tiras LED NeoPixel
#ifdef __AVR__
#include <avr/power.h>          // Librería usada solo en placas AVR
#endif

#define PIN 8                   // Pin digital donde se conecta el LED
#define NUMPIXELS 1             // Número de LEDs en la tira

// Se crea un objeto 'pixels' de tipo Adafruit_NeoPixel con los parámetros anteriores
Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

void setup(){
  Serial.begin(115200);         // Inicializa la comunicación serial
  pixels.begin();               // Inicializa la librería NeoPixel
}

void loop(){
  pixels.clear();               // Apaga todos los LEDs antes de actualizar
  pixels.setPixelColor(0, pixels.Color(122, 0, 122)); // LED índice 0 en color magenta
  pixels.show();                // Envía el color al LED
  delay(700);                   // Espera 700 ms
}
```

**Explicación general:**  
Este programa enciende un único LED NeoPixel con un color fijo.  
`pixels.begin()` inicializa la tira, `pixels.setPixelColor()` define el color y `pixels.show()` actualiza el LED físicamente.

---

## ///Ble-char

### Creación de Servicios y Características BLE

```cpp
#include <BLEDevice.h>   // Control del dispositivo BLE
#include <BLEServer.h>   // Permite crear un servidor BLE
#include <BLEUtils.h>    // Funciones auxiliares BLE
#include <BLE2902.h>     // Permite agregar descriptores BLE

#define SERVICE_UUIDA "12345678-1234-1234-1234-1234567890ac"
#define CHARACTERISTIC_UUIDA "abcd1234-ab12-cd34-ef56-abcdef123456"
#define SERVICE_UUIDB "12345678-1234-1234-1234-1234567890ad"
#define CHARACTERISTIC_UUIDB "abcd1234-ab12-cd34-ef56-abcdef123455"
#define CHARACTERISTIC_UUIDC "abcd1234-ab12-cd34-ef56-abcdef123457"

void setup() {
  Serial.begin(115200);              // Inicializa el monitor serial
  BLEDevice::init("ESP32_BLee");     // Crea un dispositivo BLE con nombre "ESP32_BLee"

  BLEServer *pServer = BLEDevice::createServer(); // Crea el servidor BLE

  // Primer servicio BLE
  BLEService *pService = pServer->createService(SERVICE_UUIDA);
  BLECharacteristic *pCharacteristic = pService->createCharacteristic(
      CHARACTERISTIC_UUIDA,
      BLECharacteristic::PROPERTY_READ | BLECharacteristic::PROPERTY_WRITE);
  pCharacteristic->setValue("Buenas nuevas nos trae el señor");
  pService->start();

  // Segundo servicio BLE con dos características
  BLEService *pServiceB = pServer->createService(SERVICE_UUIDB);
  BLECharacteristic *pCharacteristicB = pServiceB->createCharacteristic(
      CHARACTERISTIC_UUIDB,
      BLECharacteristic::PROPERTY_READ | BLECharacteristic::PROPERTY_WRITE);
  pCharacteristicB->setValue("Silksong mañana");

  BLECharacteristic *pCharacteristicC = pServiceB->createCharacteristic(
      CHARACTERISTIC_UUIDC,
      BLECharacteristic::PROPERTY_READ | BLECharacteristic::PROPERTY_WRITE);
  pCharacteristicC->setValue("Prepara la cartera");

  pServiceB->start();

  BLEAdvertising *pAdvertising = BLEDevice::getAdvertising();
  pAdvertising->start();              // Comienza la difusión del dispositivo

  Serial.println("Dispositivo BLE listo");
}

void loop() {
  delay(2000); // No hay lógica activa en el bucle principal
}
```

**Explicación detallada:**  
Este código crea dos servicios BLE con características de lectura y escritura.  
Cada característica tiene un UUID único y un valor inicial que puede verse desde una app BLE.  
`BLEAdvertising` hace que el dispositivo sea detectable desde un teléfono u otro cliente BLE.

---

## ///ble Write

### Control del LED integrado por Bluetooth BLE

```cpp
#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>

#define SERVICE_UUID "12345678-1234-1234-1234-1234567890ab"
#define CHARACTERISTIC_UUID "abcd1234-ab12-cd34-ef56-abcdef123456"

// Clase callback: maneja eventos cuando se escribe sobre la característica BLE
class MyCallbacks : public BLECharacteristicCallbacks {
  void onWrite(BLECharacteristic *pCharacteristic) {
    String rxValue = pCharacteristic->getValue(); // Obtiene el valor recibido

    if (rxValue.length() > 0) {
      Serial.print("Valor recibido: ");
      Serial.println(rxValue.c_str());

      if (rxValue == "0") {
        digitalWrite(LED_BUILTIN, HIGH); // Enciende LED
        Serial.println("LED ENCENDIDO");
      } else if (rxValue == "1") {
        digitalWrite(LED_BUILTIN, LOW);  // Apaga LED
        Serial.println("LED APAGADO");
      } else {
        Serial.println("Usa '0' para prender y '1' para apagar.");
      }
    }
  }
};

void setup() {
  Serial.begin(115200);
  pinMode(LED_BUILTIN, OUTPUT);
  digitalWrite(LED_BUILTIN, LOW);

  BLEDevice::init("ESP32_BLE_LED");          // Inicia el dispositivo BLE
  BLEServer *pServer = BLEDevice::createServer();
  BLEService *pService = pServer->createService(SERVICE_UUID);

  BLECharacteristic *pCharacteristic = pService->createCharacteristic(
      CHARACTERISTIC_UUID,
      BLECharacteristic::PROPERTY_READ | BLECharacteristic::PROPERTY_WRITE);

  pCharacteristic->setValue("1");
  pCharacteristic->setCallbacks(new MyCallbacks()); // Asigna la clase de manejo

  pService->start();
  BLEAdvertising *pAdvertising = BLEDevice::getAdvertising();
  pAdvertising->start();

  Serial.println("Esperando conexión BLE...");
}

void loop() {
  delay(1000);
}
```

**Explicación:**  
Cada vez que se recibe un valor BLE, se ejecuta el método `onWrite()`.  
Según el valor (“0” o “1”), el LED se enciende o apaga.

---

## ///Wifi

### Configuración del modo WiFi estación

```cpp
#include <WiFi.h>

void setup() {
  Serial.begin(115200);
  WiFi.mode(WIFI_STA);     // Configura el ESP32 como estación (no punto de acceso)
  Serial.println("WiFi en modo estación listo");
}

void loop() {
  delay(2000);
}
```

**Explicación:**  
`WiFi.mode(WIFI_STA)` permite que el ESP32 se comunique con otros dispositivos o redes.  
Es necesario para tecnologías como ESP-NOW.

---

## ///ESPnow

### Comunicación entre ESP32 (Receptor)

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

// Función callback: se ejecuta cuando llegan datos
void OnDataRecv(const uint8_t * mac, const uint8_t *tempData, int tam) {
  memcpy(&datosRecibidos, tempData, sizeof(datosRecibidos)); // Copia los datos
  Serial.print("Bytes recibidos: "); Serial.println(tam);
  Serial.print("Char: "); Serial.println(datosRecibidos.a);
  Serial.print("Int: "); Serial.println(datosRecibidos.b);
  Serial.print("Float: "); Serial.println(datosRecibidos.c);
  Serial.print("Bool: "); Serial.println(datosRecibidos.d);
  Serial.println();
}

void setup() {
  Serial.begin(115200);
  WiFi.mode(WIFI_STA);  // Requerido para ESP-NOW

  if (esp_now_init() != ESP_OK) {
    Serial.println("Error inicializando ESP-NOW");
    return;
  }

  esp_now_register_recv_cb(OnDataRecv); // Asigna la función callback
}

void loop() {}
```

**Explicación:**  
El ESP32 actúa como receptor de mensajes. Cada vez que otro dispositivo envía datos, la función `OnDataRecv` se activa automáticamente.

---

## ///String 1

### Lectura de cadenas desde el puerto serial

```cpp
String cmd = "";

void setup() {
  Serial.begin(115200); // Inicia el monitor serial
}

void loop() {
  if (Serial.available() > 0) {            // Si hay datos disponibles
    cmd = Serial.readStringUntil('\n');   // Lee hasta salto de línea
    Serial.println(cmd);                   // Imprime el texto recibido
  }
}
```

**Explicación línea por línea:**  
- `Serial.available()` devuelve cuántos bytes están listos para leerse.  
- `readStringUntil('\n')` lee hasta el final de la línea.  
- `println()` imprime la cadena recibida.  

---
