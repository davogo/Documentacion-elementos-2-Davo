# Control PID y Control Manual por Mano de Plataforma Stewart con ESP32 + OpenCV + Bluetooth

Este documento describe el sistema completo de control de una plataforma tipo Stewart de 3 grados de libertad, controlada mediante servomotores y un ESP32. El sistema tiene **dos modos de operación principales**:

- **Modo Manual:** La cámara detecta la mano y los servos replican el movimiento en tiempo real.
- **Modo Automático PID:** La cámara detecta una pelota y un controlador PID equilibra la plataforma para mantenerla en el centro.

Este documento incluye la explicación, códigos completos, arquitectura, comunicación Bluetooth, diseño mecánico y análisis de control.

---

# 🧩 Objetivos del Proyecto

- Implementar un sistema de control basado en visión por computadora.  
- Manipular una plataforma física en tiempo real mediante movimientos de mano.  
- Aplicar un controlador PID para equilibrar una pelota en la plataforma.  
- Crear una comunicación robusta entre Python ↔ ESP32 vía Bluetooth.  

---

# 🖥️ Componentes Principales del Sistema

## 🔹 Cámara y Visión por Computadora

Tecnologías utilizadas:

- OpenCV  
- MediaPipe (detección de mano)  
- Filtros y procesamiento geométrico  
- Detección de posición de pelota (modo PID)

> ✍️ La mano se detecta usando MediaPipe, que analiza cada imagen de la cámara y reconoce automáticamente 21 puntos clave en la mano. Con esos puntos se calcula la orientación (pitch y roll), se filtra y se convierte en ángulos para servomotores. Luego, los datos se envían por Bluetooth al ESP32.

---

## 🔹 Plataforma Stewart de 3 Servos

Servomotores: **MG995**

- Pin 4 → Servo Izquierdo  
- Pin 15 → Servo Superior  
- Pin 5 → Servo Derecho  

### 📁 Modelos 3D

[Descargar fixedhorn55.stl](../archivos/fixedhorn55.stl)  
[Descargar 1CUADRITOSBASEFINAL.stl](../archivos/1CUADRITOSBASEFINAL.stl)  
[Descargar BASEFINALIMPR.stl](../archivos/BASEFINALIMPR.stl)

> La plataforma inicialmente se iba a imprimir en 3D, pero por el tamaño requerido no fue posible. Luego se intentó fabricarla en MDF, pero era demasiado pesada para los servos, así que finalmente se construyó en cartón.

---

# 🔷 Código Completo del Modo Manual (Python)

Este script detecta la mano, calcula **pitch** y **roll**, suaviza movimientos, calcula ángulos para los 3 servos y los envía por Bluetooth al ESP32.

---

```python
import cv2
import mediapipe as mp
import time
import bluetooth


PORT = 1
ESP32_MAC = "14:33:5C:02:4D:2A"
sock = bluetooth.BluetoothSocket()
sock.settimeout(20)

print("Intentando conectar al ESP32...")

while True:
    try:
        sock.connect((ESP32_MAC, PORT))
        print("¡Conectado al ESP32!")
        break
    except Exception as e:
        print("Error en conexión... reintentando:", e)
        time.sleep(1)

def send_bt(message: str):
    try:
        sock.send(message.encode())
        print("Enviado:", message.strip())
    except Exception as e:
        print("Error enviando datos:", e)


mp_hands = mp.solutions.hands
hands = mp_hands.Hands(
    max_num_hands=1,
    min_detection_confidence=0.6,
    min_tracking_confidence=0.5
)
mp_draw = mp.solutions.drawing_utils


cap = cv2.VideoCapture(0)


pitch_filtrado = 0.0
roll_filtrado = 0.0
alpha = 0.3

ultimo_envio = time.time()
intervalo_envio = 0.05

print("=== PLATAFORMA STEWART ===")
print("Pin 14 = Servo Izquierda")
print("Pin 26 = Servo Arriba (medio)")
print("Pin 27 = Servo Derecha")
print("==========================\n")


K_orient = 8.0
K_roll = 0.02
Kp = 40.0
Kr = 90.0

while cap.isOpened():
    success, img = cap.read()
    if not success:
        break

    img = cv2.flip(img, 1)
    img_rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    results = hands.process(img_rgb)

    frame_height, frame_width, _ = img.shape
    mano_detectada = False

    if results.multi_hand_landmarks:
        for hand_landmarks in results.multi_hand_landmarks:
            mano_detectada = True
            mp_draw.draw_landmarks(img, hand_landmarks, mp_hands.HAND_CONNECTIONS)

            
            y_wrist = hand_landmarks.landmark[0].y
            y_fingers = (
                hand_landmarks.landmark[8].y +
                hand_landmarks.landmark[12].y +
                hand_landmarks.landmark[16].y +
                hand_landmarks.landmark[20].y
            ) / 4

            
            orient_raw = y_wrist - y_fingers
            pitch_norm = max(-1.0, min(1.0, orient_raw * K_orient))

            \
            x_index = hand_landmarks.landmark[8].x
            x_pinky = hand_landmarks.landmark[20].x
            roll_raw = x_index - x_pinky
            roll_norm = max(-1.0, min(1.0, roll_raw * K_roll))

            \
            pitch_filtrado = (1 - alpha) * pitch_filtrado + alpha * pitch_norm
            roll_filtrado = (1 - alpha) * roll_filtrado + alpha * roll_norm

            if abs(pitch_filtrado) < 0.05:
                pitch_filtrado = 0.0
            if abs(roll_filtrado) < 0.05:
                roll_filtrado = 0.0

            
            delta_arriba = Kp * pitch_filtrado + abs(Kr * roll_filtrado) * 0.5
            delta_izq = -Kp * pitch_filtrado - Kr * roll_filtrado
            delta_der = -Kp * pitch_filtrado + Kr * roll_filtrado

            a_arriba = int(max(0, min(180, 90 + delta_arriba)))
            a_izq = int(max(0, min(180, 90 + delta_izq)))
            a_der = int(max(0, min(180, 90 + delta_der)))

            
            tiempo_actual = time.time()
            if tiempo_actual - ultimo_envio >= intervalo_envio:
                mensaje = f"A1:{a_izq},A2:{a_arriba},A3:{a_der}\n"
                send_bt(mensaje)
                ultimo_envio = tiempo_actual

    if not mano_detectada:
        cv2.putText(img, "No se detecta mano", (10, 30),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 0, 255), 2)

    cv2.imshow("Plataforma Stewart", img)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

sock.close()
cap.release()
cv2.destroyAllWindows()
print("Programa terminado")
