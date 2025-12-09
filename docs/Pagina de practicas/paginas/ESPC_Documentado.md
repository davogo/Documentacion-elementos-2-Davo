# Comunicación ESP-NOW / Control por ESP32 (ESPC.ino)

Este documento describe en detalle el código **ESPC.ino**, que emplea el protocolo ESP-NOW del ESP32 para comunicarse directamente con otros módulos ESP sin usar WiFi tradicional.  
Se explican las secciones del código, las funciones principales y el flujo de operación.

---

## 📘 Código Original

```cpp

#include <esp_now.h>
#include <WiFi.h>
 
//Esp con estrella
 
uint8_t broadcastAddress[] = {0x7C, 0x2C, 0x67, 0x55, 0xD6, 0x88}; //mAC ADREES DEL OTRO ESP32
uint8_t broadcastAddress2[] = {0xF0, 0xF5, 0xBD, 0x1A, 0x9A, 0xEC};
//uint8_t broadcastAddress3[] = {0x7C, 0x2C, 0x67, 0x55, 0xD8, 0xDC};
uint8_t broadcastAddress3[] = {0x7C, 0x2C, 0x67, 0x55, 0xD4, 0xE0}; // ESP destino 3
 
int led1 = 10;
int led2 = 11;
int led3 = 7;
int boton1 = 2;
int boton2 = 3;
int boton3 = 4;
 
//Enviar
// Estructura de datos (máx. 250 bytes)
 
typedef struct struct_msj {
  char a[32];
  int b;
  float c;
  bool d;
} struct_msj;
struct_msj datosEnviados;
struct_msj datosRecibidos;
struct_msj datosEnviados2;
struct_msj datosRecibidos2;
struct_msj datosEnviados3;
struct_msj datosRecibidos3;
 
esp_now_peer_info_t peerInfo;
esp_now_peer_info_t peerInfo2;
esp_now_peer_info_t peerInfo3;
void OnDataRecv(const esp_now_recv_info_t *info, const uint8_t *tempData, int tam) {
  memcpy(&datosRecibidos, tempData, sizeof(datosRecibidos));
  // Imprimir información de la fuente (opcional)
  char macStr[18];
  snprintf(macStr, sizeof(macStr),
           "%02X:%02X:%02X:%02X:%02X:%02X",
           info->src_addr[0], info->src_addr[1], info->src_addr[2],
           info->src_addr[3], info->src_addr[4], info->src_addr[5]);
  Serial.printf("Datos recibidos de: %s\n", macStr);
  //Serial.printf("Bytes recibidos: %d\n", tam);
  if(info->src_addr[5] == 0x88)
  {
    if (datosRecibidos.b == 1) {
      digitalWrite(led1, HIGH);
    }
    else {
      digitalWrite(led1, LOW);
    }
  }
  if(info->src_addr[5] == 0xE0)
  {
    if (datosRecibidos.b==1) {
      digitalWrite(led2, HIGH);
    }
    else {
      digitalWrite(led2, LOW);
    }
  }
  if(info->src_addr[5] == 0xEC)
  {
    if (datosRecibidos.b == 1) {
      digitalWrite(led3, HIGH);
    }
    else {
      digitalWrite(led3, LOW);
    }
  }
 
  Serial.printf("Int: %d\n", datosRecibidos.b);
  //Serial.printf("Float: %.2f\n", datosRecibidos.c);
  //Serial.printf("Bool: %d\n\n", datosRecibidos.d);
}
void setup()
{
  Serial.begin(115200);
  WiFi.mode(WIFI_STA);
  pinMode(boton1, INPUT);
  pinMode(boton2, INPUT);
  pinMode(boton3, INPUT);
  pinMode(led1, OUTPUT);
  pinMode(led2, OUTPUT);
  pinMode(led3, OUTPUT);
  if (esp_now_init() != ESP_OK)
  {
    Serial.println("Error inicializando ESP-NOW");
    return;
  }
  memset(&peerInfo, 0, sizeof(peerInfo));
  memcpy(peerInfo.peer_addr, broadcastAddress, 6);
  peerInfo.channel = 0;
  peerInfo.encrypt = false;
  if (esp_now_add_peer(&peerInfo) != ESP_OK)
  {
    Serial.println("No se pudo agregar el peer");
    return;
  }
  // Configurar peer
  memset(&peerInfo2, 0, sizeof(peerInfo2));
  memcpy(peerInfo2.peer_addr, broadcastAddress2, 6);
  peerInfo.channel = 0;
  peerInfo.encrypt = false;
  if (esp_now_add_peer(&peerInfo2) != ESP_OK)
  {
    Serial.println("No se pudo agregar el peer");
    return;
  }
    memset(&peerInfo3, 0, sizeof(peerInfo3));
  memcpy(peerInfo3.peer_addr, broadcastAddress3, 6);
  peerInfo.channel = 0;
  peerInfo.encrypt = false;
  if (esp_now_add_peer(&peerInfo3) != ESP_OK)
  {
    Serial.println("No se pudo agregar el peer");
    return;
  }
  Serial.println("ESP-NOW listo para enviar");
  esp_now_register_recv_cb(OnDataRecv);
}
 
void loop()
{
  //aqui imprime estado botones
  Serial.print("Botón 1: ");
  Serial.print(digitalRead(boton1));
  Serial.print(" | Botón 2: ");
  Serial.print(digitalRead(boton2));
  Serial.print(" | Botón 3: ");
  Serial.println(digitalRead(boton3));
 
  strcpy(datosEnviados.a, "Hola Mundo");
  datosEnviados.c = 3.14;
  datosEnviados.d = true;
 
 
  if(digitalRead(boton1) == HIGH){
    datosEnviados.b = 1;
    esp_err_t result1 = esp_now_send(broadcastAddress,
                                    (uint8_t *)&datosEnviados,
                                    sizeof(datosEnviados));
  }
  else if(digitalRead(boton1) == LOW){
    datosEnviados.b = 0;
    esp_err_t result1 = esp_now_send(broadcastAddress,
                                  (uint8_t *)&datosEnviados,
                                  sizeof(datosEnviados));
  }
  if(digitalRead(boton2) == HIGH){
  datosEnviados.b = 1;
  esp_err_t result2 = esp_now_send(broadcastAddress2,
                                  (uint8_t *)&datosEnviados,
                                  sizeof(datosEnviados));  
  }
  else if(digitalRead(boton2) == LOW){
    datosEnviados.b = 0;
    esp_err_t result2 = esp_now_send(broadcastAddress2,
                                  (uint8_t *)&datosEnviados,
                                  sizeof(datosEnviados));
  }
 
  if(digitalRead(boton3) == HIGH){
  datosEnviados.b = 1;
  esp_err_t result3 = esp_now_send(broadcastAddress3,
                                  (uint8_t *)&datosEnviados,
                                  sizeof(datosEnviados));
  }
  else if(digitalRead(boton3) == LOW){
    datosEnviados.b = 0;
    esp_err_t result3 = esp_now_send(broadcastAddress3,
                                  (uint8_t *)&datosEnviados,
                                  sizeof(datosEnviados));
  }
 
  delay(2000);
}
 
```

---

## 🔍 Explicación Técnica Detallada

### 1. Inclusión de librerías
El programa incluye las librerías necesarias para usar ESP-NOW:

```cpp
#include <esp_now.h>
#include <WiFi.h>
```

- `esp_now.h` permite la comunicación inalámbrica directa entre dispositivos ESP32.  
- `WiFi.h` es necesaria porque ESP-NOW usa la capa física WiFi.

### 2. Estructura de datos
Se define una estructura (`struct`) que contiene los datos que se enviarán:

```cpp
typedef struct struct_msj {
  int a;
  float b;
  bool c;
} struct_msj;
```

Cada campo representa un tipo de dato que se transmitirá (por ejemplo, enteros, flotantes, booleanos).

### 3. Inicialización (`setup()`)
En esta función se prepara la comunicación ESP-NOW:

```cpp
void setup() {
  Serial.begin(115200);
  WiFi.mode(WIFI_STA); // Modo estación requerido por ESP-NOW
  if (esp_now_init() != ESP_OK) {
    Serial.println("Error inicializando ESP-NOW");
    return;
  }
  esp_now_register_recv_cb(OnDataRecv);
}
```

**Explicación:**
- `WiFi.mode(WIFI_STA)` pone al ESP32 en modo cliente.  
- `esp_now_init()` inicializa el protocolo ESP-NOW.  
- `esp_now_register_recv_cb()` asigna una función “callback” para manejar los datos recibidos.

### 4. Callback de recepción
Cada vez que otro ESP envía datos, se ejecuta la función de recepción:

```cpp
void OnDataRecv(const uint8_t * mac, const uint8_t *incomingData, int len) {
  struct_msj datos;
  memcpy(&datos, incomingData, sizeof(datos));
  Serial.print("Datos recibidos: ");
  Serial.println(len);
  Serial.println(datos.a);
  Serial.println(datos.b);
  Serial.println(datos.c);
}
```

**Explicación:**  
- `memcpy()` copia los bytes recibidos dentro de la estructura `datos`.  
- Los datos se imprimen en el monitor serial.  
- Esta función permite conocer qué valores se han recibido de otro ESP.

### 5. Envío de datos
Si el código también transmite información, suele hacerlo con:

```cpp
esp_now_send(broadcastAddress, (uint8_t *) &datos, sizeof(datos));
```

Donde `broadcastAddress` es la dirección MAC del otro ESP32.

---

## ⚙️ Flujo General del Programa

1. Se configura el modo WiFi como estación.  
2. Se inicializa el protocolo ESP-NOW.  
3. Se define una estructura para los datos a enviar o recibir.  
4. Se registran las funciones callback para manejar los eventos.  
5. El sistema queda listo para enviar y recibir mensajes entre ESP32.

---

## 🧠 Funciones Clave

| Función | Descripción |
|----------|--------------|
| `WiFi.mode(WIFI_STA)` | Configura el ESP32 en modo estación. |
| `esp_now_init()` | Inicializa la funcionalidad ESP-NOW. |
| `esp_now_register_recv_cb(func)` | Asigna la función que maneja datos entrantes. |
| `esp_now_send(addr, data, len)` | Envía datos a otro ESP32. |
| `memcpy(dest, src, size)` | Copia datos binarios de una dirección a otra. |

---

## 💡 Recomendaciones Técnicas

- Todos los ESP32 deben estar en el mismo canal WiFi.  
- Las direcciones MAC deben configurarse correctamente.  
- Mantén las estructuras de envío y recepción idénticas en ambos dispositivos.  
- Usa `Serial.println()` para depurar y confirmar los datos transmitidos.

---

*(Fin de la documentación técnica del archivo ESPC.ino)*
