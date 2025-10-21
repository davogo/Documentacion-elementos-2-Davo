# Lectura y eco de cadenas por Serial

## Introducción
Este código realiza la lectura de texto ingresado por el monitor serial y lo devuelve como respuesta. Es una forma simple de probar comunicación serial bidireccional.

## Código fuente
```cpp
String cmd = "";

void setup() {
  Serial.begin(115200);
}

void loop() {
  if (Serial.available() > 0) {
    cmd = Serial.readStringUntil('\n');
    Serial.println(cmd);
  }
}
```

## Explicación extendida
- **Serial.available():** Detecta si hay bytes listos para leerse.  
- **readStringUntil('\n'):** Lee hasta un salto de línea.  
- **println(cmd):** Devuelve el texto recibido, actuando como eco.  
- Útil para pruebas de comandos o comunicación básica.
