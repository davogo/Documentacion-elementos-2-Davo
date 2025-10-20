# Control de NeoPixel con ESP32

## Introducción
Este programa demuestra cómo inicializar y controlar un LED **NeoPixel** (WS2812 o similar) desde un ESP32. Se utiliza la librería oficial de Adafruit para manejar LEDs RGB direccionables.

## Código fuente
```cpp
#include <Adafruit_NeoPixel.h>
#ifdef __AVR__
#include <avr/power.h>
#endif

#define PIN 8
#define NUMPIXELS 1

Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

void setup(){
  Serial.begin(115200);
  pixels.begin();
}

void loop(){
  pixels.clear();
  pixels.setPixelColor(0, pixels.Color(122, 0, 122));
  pixels.show();
  delay(700);
}
```

## Explicación detallada
- **Librerías:** `Adafruit_NeoPixel` permite crear y controlar una o varias luces RGB direccionables.  
- **Configuración:** Se define el pin de salida y el número de LEDs (`NUMPIXELS`).  
- **Inicio:** `pixels.begin()` configura el pin y la señal.  
- **Lógica:** En cada iteración se limpia el estado anterior, se define un color magenta (122,0,122) y se muestra.  
- **Retardo:** `delay(700)` genera una pausa visible antes de volver a ejecutar el ciclo.
