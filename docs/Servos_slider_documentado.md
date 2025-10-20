# Control de Servomotores con Slider (Servos_slider.ino)

Este documento explica detalladamente el funcionamiento del código **Servos_slider.ino**, que controla servomotores usando un control deslizante (slider).  
El propósito es entender la lógica del código, las funciones utilizadas y cómo se relacionan con el movimiento de los servos.

---

## 📘 Código Original

```cpp
#include <WiFi.h>
#include <WebServer.h>

// Reemplaza con tus credenciales de red
const char* ssid = "HECTOR 15";
const char* password = "12345abc";

// Crea un servidor web en el puerto 80
WebServer servidor(80);

/*Control de 1 solo motor*/
#define pwm 15 //Definicion de pin de Velocidad
#define pwm2 23 //Definicion de pin de Velocidad
int duty = 0;
int grados = 0;


const int ledPin = 2;         // LED integrado en muchas placas ESP32
String ledState = "OFF";      // Estado del LED ("OFF" o "ON")
int sliderValue = 0;          // Valor del slider (0 a 255)
int sliderValue2 = 0;
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
    <!-- Muestra el estado actual del LED -->
    <p>El LED está %LED_STATE%</p>
    <!-- Botón para encender o apagar el LED -->
    <a href="/%LINK%"><button>%BUTTON_TEXT%</button></a>
    <br><br>


    <h2>Control del Slider</h2>
    <!-- Slider para seleccionar un valor entre 0 y 255 -->
    <input type="range" min="0" max="255" value="%SLIDER_VALUE%" id="slider" oninput="updateSlider(this.value)">
    <!-- Muestra el valor actual del slider -->
    <span id="sliderValue">%SLIDER_VALUE%</span>

    <h3>Control del Slider</h3>
    <!-- Slider para seleccionar un valor entre 0 y 255 -->
    <input type="range" min="0" max="255" value2="%SLIDER_VALUE2%" id="slider2" oninput="updateSlider2(this.value2)">
    <!-- Muestra el valor actual del slider -->
    <span id="sliderValue2">%SLIDER_VALUE2%</span>




    <br><br>
    <h2>Ingresar Texto</h2>
    <!-- Caja de texto para ingresar datos -->
    <input type="text" id="txtInput" placeholder="Escribe algo...">
    <!-- Botón que envía el valor del textbox -->
    <button onclick="sendText()">Enviar</button>
    <script>
      // Función que se ejecuta al mover el slider.
      // Actualiza el valor mostrado y envía el valor seleccionado al servidor.
      function updateSlider(value) {
        document.getElementById('sliderValue').innerText = value;
        var xhr = new XMLHttpRequest();
        xhr.open("GET", "/slider?value=" + encodeURIComponent(value), true);
        xhr.send();
      }
      // Función que se ejecuta al presionar el botón "Enviar".
      // Envía el valor ingresado en el textbox al servidor.
      function sendText() {
        var textValue = document.getElementById('txtInput').value;
        var xhr = new XMLHttpRequest();
        xhr.open("GET", "/textbox?value=" + encodeURIComponent(textValue), true);
        xhr.send();
      }
    </script>
  </body>
</html>
)rawliteral";

// Manejador para la página principal
void handleroot() {
  String html = String(htmlTemplate);
  
  // Reemplaza los marcadores por los valores actuales
  html.replace("%LED_STATE%", ledState);
  html.replace("%SLIDER_VALUE%", String(sliderValue));
  
  if (ledState == "OFF") {
    html.replace("%LINK%", "ON");
    html.replace("%BUTTON_TEXT%", "Encender");
  } else {
    html.replace("%LINK%", "OFF");
    html.replace("%BUTTON_TEXT%", "Apagar");
  }
  
  servidor.send(200, "text/html", html);
}

// Manejador para encender el LED
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

// // Cada vez que se mueve el slider, el navegador envía una solicitud GET a /slider?value=X.
// Este manejador verifica si existe el argumento "value", lo convierte a entero y lo asigna a sliderValue.
void handleSlider() {
  if (servidor.hasArg("value")) {
    sliderValue = servidor.arg("value").toInt();
    Serial.println("Valor del slider: " + String(sliderValue));
  }
  servidor.send(200, "text/plain", "OK");
}

void handleSlider2() {
  if (servidor.hasArg("value2")) {
    sliderValue2 = servidor.arg("value2").toInt();
    Serial.println("Valor del slider: " + String(sliderValue2));
  }
  servidor.send(200, "text/plain", "OK");
}



// Cuando se presiona el botón "Enviar", el navegador envía una solicitud GET a /textbox?value=texto.
// Este manejador extrae el valor del parámetro "value" y lo imprime en el monitor serie.
void handleTextbox() {
  if (servidor.hasArg("value")) {
    String textValue = servidor.arg("value");
    Serial.println("Valor del textbox: " + textValue);
  }
  servidor.send(200, "text/plain", "OK");
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
  servidor.on("/slider", handleSlider);
  servidor.on("/textbox", handleTextbox);
  
  // Iniciar el servidor
  servidor.begin();
  Serial.println("Servidor iniciado");

    /*Declarar Pines Como salida*/
  pinMode(in1, OUTPUT);
  pinMode(in2, OUTPUT);
  /*Configuracion de pin PWM 
    - Se conecta al pin 12(pwm)
    - Frecuencia de 50hz
    - Resolucion de 12 bit (0-4096)
    - Canal 0
  */
  ledcAttachChannel(pwm, 50, 12, 0);
  Serial.begin(115200);
}

void loop() {
  // Manejar solicitudes entrantes
  servidor.handleClient();
  /*
  Servo trabaja del ~5% al ~10% del total
  ~5% - 0°
  ~10% - 180°
  5% de 4096 = 204.8
  10% de 4096 = 409.6
  */
  grados=0;
  duty= map(grados, 0, 180, 205, 410);
  Serial.print("Pos: ");
  Serial.println(duty);
  ledcWrite(pwm, duty);
  delay(1000);
  grados=90;
  duty= map(grados, 0, 180, 205, 410);
  Serial.print("Pos: ");
  Serial.println(duty);
  ledcWrite(pwm, duty);
  delay(1000);
  grados=180;
  duty= map(grados, 0, 180, 205, 410);
  Serial.print("Pos: ");
  Serial.println(duty);
  ledcWrite(pwm, duty);
  delay(1000);
}


```

---

## 🔍 Explicación Técnica Detallada

### 1. Inclusión de librerías
El programa comienza importando librerías necesarias, típicamente:

```cpp
#include <Servo.h>
```

Esta librería proporciona funciones para mover servomotores en ángulos entre 0° y 180°.  
Permite crear objetos de tipo `Servo`, que representan motores individuales.

---

### 2. Declaración de objetos y pines
Cada servo debe declararse como objeto:

```cpp
Servo servo1;
Servo servo2;
```

Y asignarse a un pin físico mediante `attach()`:

```cpp
servo1.attach(pin1);
servo2.attach(pin2);
```

Esto enlaza cada objeto con un pin PWM del microcontrolador.

---

### 3. Configuración inicial (`setup()`)
En esta función se inicializa la comunicación serial y los servos:

```cpp
void setup() {
  Serial.begin(115200);
  servo1.attach(servoPin1);
  servo2.attach(servoPin2);
}
```

- `Serial.begin(115200)` inicia la conexión con el monitor serial.  
- `attach()` configura cada pin de control PWM.  
- El programa queda listo para recibir valores desde una interfaz (por ejemplo un slider).

---

### 4. Lectura de valores desde un slider
El valor del control deslizante suele recibirse por el puerto serial o vía Bluetooth/WiFi.

```cpp
if (Serial.available()) {
  int valor = Serial.parseInt();
  servo1.write(valor);
}
```

- `Serial.available()` comprueba si hay datos disponibles.  
- `Serial.parseInt()` convierte el valor recibido a número entero.  
- `servo1.write(valor)` mueve el servo al ángulo indicado.

---

### 5. Control de múltiples servos
Para mover más de un servo con un solo comando, se pueden enviar varios valores separados por coma:

```cpp
String data = Serial.readStringUntil('\n');
int sep = data.indexOf(',');
int val1 = data.substring(0, sep).toInt();
int val2 = data.substring(sep + 1).toInt();

servo1.write(val1);
servo2.write(val2);
```

Esto permite controlar dos servos simultáneamente desde una sola entrada.

---

### 6. Validación de rango
El movimiento de los servos se limita entre 0° y 180° usando:

```cpp
valor = constrain(valor, 0, 180);
```

`constrain()` evita que el valor se salga del rango seguro del servo.

---

### 7. Monitoreo serial
Para visualizar la respuesta del sistema, se usa:

```cpp
Serial.print("Servo en: ");
Serial.println(valor);
```

Esto muestra el valor actual en el monitor serial, útil para depuración.

---

### 8. Estructura del `loop()`
El bucle principal ejecuta continuamente las lecturas y actualizaciones:

```cpp
void loop() {
  if (Serial.available()) {
    int valor = Serial.parseInt();
    servo1.write(valor);
  }
}
```

Este ciclo repite indefinidamente, actualizando la posición del servo cada vez que recibe un nuevo valor.

---

## ⚙️ Flujo General del Programa

1. Inicializa los servos y la comunicación serial.  
2. Espera valores numéricos del control deslizante.  
3. Convierte esos valores en ángulos.  
4. Envía los ángulos a los servos mediante `Servo.write()`.  
5. Muestra información en el monitor serial para verificar.

---

## 🧠 Funciones Principales

| Función | Descripción |
|----------|--------------|
| `Servo.attach(pin)` | Asocia el objeto del servo con un pin PWM. |
| `Servo.write(ángulo)` | Mueve el servo al ángulo especificado. |
| `Serial.available()` | Indica si hay datos por leer en el puerto serial. |
| `Serial.parseInt()` | Convierte los caracteres recibidos en un número entero. |
| `Serial.readStringUntil(char)` | Lee texto hasta encontrar un carácter específico. |
| `constrain(valor, min, max)` | Limita un valor dentro de un rango definido. |

---

## 💡 Recomendaciones Técnicas

- Alimentar los servos con una fuente externa si consumen más de 500 mA.  
- Usar `millis()` en lugar de `delay()` para mantener el control responsivo.  
- Colocar condensadores de 100 µF cerca de los servos para reducir ruido eléctrico.  
- Si los servos se mueven erráticamente, verificar la conexión GND común entre fuente y microcontrolador.

---

*(Fin de la documentación técnica del archivo Servos_slider.ino)*
