# Recepción de Comandos por Comunicación Serial (String_comandos_documentado.md)

Este documento explica detalladamente el funcionamiento del código correspondiente a la **Práctica 2: Recepción de Comandos por Comunicación Serial**, el cual implementa la lectura y el eco de cadenas de texto (`String`) enviadas desde el Monitor Serial hacia una placa Arduino o ESP32.

---

## 📘 Código Original

```cpp
// Variable global para almacenar el comando recibido
String cmd = "";

void setup() {
  // Inicializa la comunicación serial a 115200 baudios
  Serial.begin(115200); 
}

void loop() {
  // Verifica si hay datos disponibles para leer en el buffer serial
  if (Serial.available() > 0) {            
    // Lee todos los caracteres hasta encontrar un salto de línea ('\n')
    // y los almacena en la variable 'cmd'
    cmd = Serial.readStringUntil('\n');
    
    // Imprime la cadena completa recibida en el monitor serial
    Serial.println(cmd);                   
  }
}

```

---

## 🔍 Explicación Técnica Detallada

### 1. Declaración de variables globales

```cpp
String cmd = "";
```

Se declara una variable global llamada `cmd` de tipo `String`, la cual servirá para almacenar temporalmente el texto recibido desde el puerto serial.  
Esta variable permite manejar mensajes de longitud variable (palabras, frases o comandos).

---

### 2. Función `setup()`

```cpp
void setup() {
  Serial.begin(115200); 
}
```

**Explicación:**
- `Serial.begin(115200)` inicia la comunicación serial entre la placa y el ordenador a una velocidad de **115200 baudios** (bits por segundo).  
- Este paso es indispensable para habilitar el intercambio de datos mediante el **Monitor Serial** del IDE de Arduino.

---

### 3. Función `loop()`

El ciclo principal del programa se ejecuta continuamente. En él se realiza la lectura del puerto serial y el reenvío del texto recibido (eco).

```cpp
void loop() {
  if (Serial.available() > 0) {
    cmd = Serial.readStringUntil('\n');
    Serial.println(cmd);
  }
}
```

**Desglose paso a paso:**

1. `Serial.available() > 0`  
   Comprueba si existen datos disponibles en el **buffer serial**.  
   Si el valor es mayor que 0, significa que el usuario ha enviado texto desde el Monitor Serial.

2. `Serial.readStringUntil('\n')`  
   Lee todos los caracteres recibidos **hasta encontrar un salto de línea (`\n`)**, el cual indica el final del mensaje.  
   Todo el texto anterior a este carácter se guarda en la variable `cmd`.  
   Esta función simplifica la lectura de cadenas completas, evitando el uso de bucles que lean carácter por carácter.

3. `Serial.println(cmd)`  
   Imprime nuevamente el texto almacenado en `cmd`.  
   Este comportamiento implementa un **“eco de línea”**, confirmando que los datos fueron correctamente recibidos.

---

## ⚙️ Flujo General del Programa

1. Se inicializa la comunicación serial.  
2. El programa entra en un bucle continuo.  
3. Si hay datos en el buffer, se leen hasta un salto de línea (`\n`).  
4. El texto recibido se guarda en `cmd`.  
5. Se imprime el mismo texto de vuelta en el Monitor Serial.

---

## 🧠 Funciones Clave

| Función | Descripción |
|----------|--------------|
| `Serial.begin(baud)` | Inicia la comunicación serial con la velocidad indicada. |
| `Serial.available()` | Devuelve el número de bytes disponibles para leer en el buffer serial. |
| `Serial.readStringUntil(char)` | Lee los caracteres recibidos hasta encontrar el carácter delimitador especificado. |
| `Serial.println(variable)` | Envía el valor de la variable al Monitor Serial seguido de un salto de línea. |

---

## 📈 Resultados Observados

- **Recepción correcta:** El microcontrolador recibe la cadena completa enviada desde el Monitor Serial.  
- **Eco funcional:** El mismo texto se reenvía y se muestra en pantalla, verificando que la transmisión es exitosa.  
- **Uso de `Serial.readStringUntil()`:** Se confirmó que la función simplifica la lectura de mensajes al detenerse automáticamente al detectar `\n`.  
- **Flexibilidad de `String`:** La variable `cmd` maneja cadenas de longitud variable sin requerir un tamaño de buffer fijo.

---

## 💡 Conclusiones Técnicas

Esta práctica permite comprender cómo gestionar la **comunicación serial basada en texto completo**, un paso esencial para implementar sistemas controlados por comandos.  
El método `Serial.readStringUntil()` proporciona una forma sencilla y eficiente de recibir mensajes legibles por humanos, eliminando la necesidad de reconstruir cadenas carácter por carácter.  

Aunque el uso de la clase `String` facilita la programación, se debe tener precaución en proyectos complejos, ya que puede consumir memoria dinámica. En sistemas con recursos limitados, se recomienda el uso de arreglos de caracteres (`char[]`) cuando la estabilidad de memoria sea prioritaria.  

Este programa constituye la base para sistemas más avanzados de interpretación de comandos, donde funciones como `cmd.startsWith()` o `cmd.equals()` pueden utilizarse para ejecutar acciones específicas según el contenido del texto recibido.

---

*(Fin de la documentación técnica del archivo String_comandos_documentado.md)*
