# Práctica: Control de Servomotores mediante Interfaz Web con ESP32

**Equipo / Autor(es):** David Gonzales, Ximena Verdi y Héctor Noriega  
**Curso / Asignatura:** Elementos Programables 2  
**Fecha:** 21/10/2025  

---

## 🧩 Resumen

En esta práctica se desarrolló un sistema de control remoto para **dos servomotores** utilizando una **placa de desarrollo ESP32**.  
Se implementó un **servidor web embebido** que aloja una interfaz HTML con **controles deslizantes (sliders)**.  
La comunicación entre el cliente (navegador web) y el servidor (ESP32) se realiza **de forma asíncrona mediante JavaScript (AJAX)**, permitiendo ajustar la posición de los servos en tiempo real sin recargar la página.  
El sistema convierte los valores de los sliders en señales **PWM** adecuadas para el posicionamiento angular de los servomotores.

---

## 🎯 Objetivos

### General  
Desarrollar un sistema embebido capaz de controlar la posición de servomotores de forma remota a través de una red local Wi-Fi, utilizando una interfaz web intuitiva.

### Específicos
- Configurar el ESP32 como punto de acceso Wi-Fi o estación.  
- Implementar un servidor web en el ESP32 para servir una página HTML de control.  
- Diseñar una interfaz de usuario con HTML y JavaScript que incluya sliders para el control preciso de los servos.  
- Programar la lógica en el ESP32 para recibir datos y traducirlos a señales PWM.  
- Validar el funcionamiento del sistema, asegurando respuesta fluida y en tiempo real.

---

## ⚙️ Alcance y Exclusiones

**Incluye:**
- Uso del ESP32 como servidor web.  
- Control independiente de **dos servomotores**.  
- Interfaz HTML/JavaScript con sliders.  
- Comunicación asíncrona mediante **AJAX**.  
- Generación de PWM por hardware con **ledc**.

**No incluye:**
- Control a través de Internet (solo red local).  
- Autenticación o cifrado.  
- Almacenamiento persistente de posiciones.  
- Uso de frameworks web avanzados (React, Vue, WebSockets, etc.).

---

## 📘 Código Original (Versión Corregida y Funcional)

```cpp
#include <WiFi.h>
#include <WebServer.h>

// --- CONFIGURACIÓN DE RED ---
const char* ssid = "HECTOR 15";
const char* password = "12345abc";

// --- CONFIGURACIÓN DE PINES Y SERVOS ---
const int servoPin1 = 15; // Pin para el Servo 1
const int servoPin2 = 23; // Pin para el Servo 2

// --- VARIABLES GLOBALES ---
WebServer servidor(80);
int sliderValue1 = 90; // Valor inicial del slider 1 (0-180)
int sliderValue2 = 90; // Valor inicial del slider 2 (0-180)

// --- PLANTILLA HTML ---
const char* htmlTemplate = R"rawliteral(
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Control de Servos ESP32</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
    <h1>Control de Servomotores con ESP32</h1>

    <h2>Servo 1</h2>
    <input type="range" min="0" max="180" value="%SLIDER_VALUE1%" id="slider1" oninput="updateSlider(1, this.value)">
    <span id="sliderValue1">%SLIDER_VALUE1%</span>&deg;

    <h2>Servo 2</h2>
    <input type="range" min="0" max="180" value="%SLIDER_VALUE2%" id="slider2" oninput="updateSlider(2, this.value)">
    <span id="sliderValue2">%SLIDER_VALUE2%</span>&deg;

    <script>
      function updateSlider(servoNum, value) {
        document.getElementById('sliderValue' + servoNum).innerText = value;
        var xhr = new XMLHttpRequest();
        xhr.open("GET", "/slider?servo=" + servoNum + "&value=" + value, true);
        xhr.send();
      }
    </script>
</body>
</html>
)rawliteral";

// --- FUNCIONES DEL SERVIDOR WEB ---
void setServoPosition(int servoChannel, int degrees) {
  int dutyCycle = map(degrees, 0, 180, 205, 410);
  ledcWrite(servoChannel, dutyCycle);
}

void handleRoot() {
  String html = String(htmlTemplate);
  html.replace("%SLIDER_VALUE1%", String(sliderValue1));
  html.replace("%SLIDER_VALUE2%", String(sliderValue2));
  servidor.send(200, "text/html", html);
}

void handleSlider() {
  if (servidor.hasArg("servo") && servidor.hasArg("value")) {
    int servoNum = servidor.arg("servo").toInt();
    int value = servidor.arg("value").toInt();

    if (servoNum == 1) {
      sliderValue1 = value;
      setServoPosition(0, sliderValue1);
      Serial.println("Servo 1 movido a: " + String(sliderValue1) + " grados");
    } else if (servoNum == 2) {
      sliderValue2 = value;
      setServoPosition(1, sliderValue2);
      Serial.println("Servo 2 movido a: " + String(sliderValue2) + " grados");
    }
  }
  servidor.send(200, "text/plain", "OK");
}

void setup() {
  Serial.begin(115200);

  // --- PWM PARA SERVOS ---
  ledcAttachPin(servoPin1, 0);
  ledcSetup(0, 50, 12);
  ledcAttachPin(servoPin2, 1);
  ledcSetup(1, 50, 12);

  setServoPosition(0, sliderValue1);
  setServoPosition(1, sliderValue2);

  // --- CONEXIÓN WIFI ---
  WiFi.begin(ssid, password);
  Serial.print("Conectando a WiFi...");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi conectado!");
  Serial.print("IP del servidor: ");
  Serial.println(WiFi.localIP());

  // --- RUTAS DEL SERVIDOR ---
  servidor.on("/", handleRoot);
  servidor.on("/slider", handleSlider);
  servidor.begin();
  Serial.println("Servidor web iniciado.");
}

void loop() {
  servidor.handleClient();
}
```

---

## 🔍 Explicación Técnica Detallada

### 1. Librerías y configuración de red  
Se incluyen las librerías `WiFi.h` y `WebServer.h` que permiten la conexión del ESP32 a una red inalámbrica y la creación de un servidor HTTP local.

### 2. Control PWM de los servomotores  
El ESP32 utiliza los canales **ledc** para generar señales PWM a 50 Hz con resolución de 12 bits.  
El rango de 205 a 410 equivale aproximadamente a un pulso de 1–2 ms, que representa 0–180 ° en los servos.

### 3. Interfaz Web  
La página HTML incluye dos sliders para el control de los servos.  
Cada cambio genera una solicitud AJAX al ESP32, sin recargar la página.

### 4. Servidor HTTP  
Maneja las rutas:
- `/` → Página principal  
- `/slider` → Actualiza la posición del servo según los parámetros recibidos.

### 5. Flujo del sistema  
1. Inicialización de PWM y red.  
2. Conexión Wi-Fi.  
3. Ejecución del servidor web.  
4. Recepción de comandos desde la interfaz.  
5. Ajuste de posición de servos en tiempo real.

---

## 🧠 Funciones Principales

| Función | Descripción |
|----------|-------------|
| `WiFi.begin()` | Inicia la conexión Wi-Fi. |
| `WebServer servidor(80)` | Crea el servidor HTTP. |
| `ledcSetup()` | Configura la frecuencia y resolución PWM. |
| `ledcAttachPin()` | Asocia un pin al canal PWM. |
| `ledcWrite()` | Ajusta el ciclo de trabajo PWM. |
| `servidor.on()` | Define las rutas HTTP. |
| `servidor.handleClient()` | Atiende las solicitudes entrantes. |

---

## 💡 Recomendaciones Técnicas

- Utilizar fuente de alimentación externa para los servos (mínimo 1 A).  
- Conectar **GND común** entre la fuente y el ESP32.  
- Agregar **condensador de 100 µF** para evitar caídas de tensión.  
- Considerar **WebSockets** para control en tiempo real bidireccional.  
- Añadir **CSS** para mejorar la interfaz web.

---

## 🧾 Conclusiones

Se logró implementar con éxito un sistema de control remoto de servomotores usando ESP32 y una interfaz web.  
El uso de AJAX permitió una comunicación eficiente sin recarga de página, y las funciones **ledc** garantizaron un control PWM estable.  

El proyecto demuestra la integración entre sistemas embebidos y tecnologías web, aplicable en **IoT, robótica** y **domótica**.  
Futuras mejoras incluyen interfaz visual mejorada, comunicación WebSocket y autenticación segura.
