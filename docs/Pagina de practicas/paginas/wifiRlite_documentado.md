# Control WiFi con ESP32 (wifiRlite.ino)

Este documento explica detalladamente el funcionamiento del código **wifiRlite.ino**, el cual utiliza el módulo WiFi del ESP32 para establecer comunicación o control remoto.  
Se detalla cada parte del código, las funciones empleadas y el flujo general de operación.

---

## 📘 Código Original

```cpp
#include <WiFi.h>
#include <WebServer.h>

// Reemplaza con tus credenciales de red
const char* ssid = "AndroidAP";
const char* password = "kkcl99113";

// Crea un servidor web en el puerto 80
WebServer servidor(80);

const int ledPin = LED_BUILTIN;  // LED integrado en muchas placas ESP32
String ledState = "OFF";

// Plantilla HTML definida como raw string literal
const char* htmlTemplate = R"rawliteral(
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Control de LED ESP32</title>
  </head>
  <body>
    <h1>Control de LED ESP32</h1>
    <p>El LED está %LED_STATE%</p>
    <a href="/%LINK%"><button>%BUTTON_TEXT%</button></a>
  </body>
</html>
)rawliteral";

// Manejador para la página principal
void handleroot() {
  String html = String(htmlTemplate);
  
  // Reemplaza el marcador %LED_STATE% con el estado actual del LED
  html.replace("%LED_STATE%", ledState);
  
  // Configura el enlace y el texto del botón según el estado del LED
  if (ledState == "OFF") {
    html.replace("%LINK%", "ON");
    html.replace("%BUTTON_TEXT%", "ON");
  } else {
    html.replace("%LINK%", "OFF");
    html.replace("%BUTTON_TEXT%", "OFF");
  }
  
  servidor.send(200, "text/html", html);
}


void handleOn() {
  digitalWrite(ledPin, HIGH);
  ledState = "ON";
  handleroot();  // Muestra la página actualizada
}

// Manejador para apagar el LED
void handleOff() {
  digitalWrite(ledPin, LOW);
  ledState = "OFF";
  handleroot();  // Muestra la página actualizada
}

void setup() {
  Serial.begin(115200);
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);
  
  // Conectar a WiFi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi conectado");
  Serial.println(WiFi.localIP());
  
  // Definir rutas
  servidor.on("/", handleroot);
  servidor.on("/ON", handleOn);
  servidor.on("/OFF", handleOff);
  
  // Iniciar el servidor
  servidor.begin();
  Serial.println("Server started");
}

void loop() {
  // Manejar solicitudes entrantes
  servidor.handleClient();
}

```

---

## 🔍 Explicación Técnica Detallada

### 1. Inclusión de librerías
El código utiliza la librería principal de WiFi del ESP32:

```cpp
#include <WiFi.h>
```

Esta librería permite conectar el ESP32 a una red inalámbrica y usar funciones como `WiFi.begin()`, `WiFi.status()` o `WiFi.localIP()`.

### 2. Configuración inicial
Normalmente el programa define las credenciales de red:

```cpp
const char* ssid = "NombreRed";
const char* password = "Contraseña";
```

Estas variables contienen el SSID (nombre de la red) y la contraseña de la red WiFi a la que el ESP32 se conectará.

### 3. Función `setup()`
En esta sección se inicializa el monitor serial y se intenta establecer la conexión WiFi:

```cpp
void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("Conectado a WiFi");
  Serial.println(WiFi.localIP());
}
```

**Explicación:**
- `WiFi.begin()` inicia la conexión con la red indicada.  
- `WiFi.status()` devuelve el estado actual (espera hasta `WL_CONNECTED`).  
- `WiFi.localIP()` imprime la dirección IP asignada al ESP32.

### 4. Función `loop()`
Una vez conectado, el código puede ejecutar funciones periódicas o manejar comunicación con un cliente (HTTP, UDP, etc.).  
Por ejemplo:

```cpp
void loop() {
  // Ejemplo: enviar datos, recibir comandos o verificar conexión
}
```

El contenido exacto depende de si el ESP32 actúa como servidor o cliente.

---

## ⚙️ Flujo General del Programa

1. Se inicializa la comunicación serial.  
2. Se configuran las credenciales WiFi.  
3. El ESP32 se conecta a la red y muestra su IP.  
4. Una vez conectado, puede ejecutar otras tareas o esperar comandos.

---

## 🧠 Funciones Clave

| Función | Descripción |
|----------|--------------|
| `WiFi.begin(ssid, password)` | Inicia la conexión WiFi con las credenciales proporcionadas. |
| `WiFi.status()` | Devuelve el estado actual de la conexión WiFi. |
| `WiFi.localIP()` | Muestra la dirección IP asignada al ESP32. |
| `Serial.begin(baud)` | Inicializa la comunicación serial. |
| `delay(ms)` | Pausa la ejecución durante el tiempo indicado. |

---

## 💡 Recomendaciones Técnicas

- Asegúrate de usar una fuente de energía estable para el ESP32.  
- Verifica que la red WiFi tenga suficiente cobertura.  
- Evita colocar `delay()` prolongados en el `loop()` si el dispositivo necesita responder rápido.  
- Puedes usar `WiFi.reconnect()` para restablecer la conexión en caso de desconexión.

---

*(Fin de la documentación técnica del archivo wifiRlite.ino)*
