Configuración para el desafío de obstáculos
===

## Configuración del hardware:

```
import cv2
import numpy as np
from gpiozero import Motor, AngularServo, Button
from time import sleep
import board
import busio
from adafruit_vl53l0x import VL53L0X

# Hardware configuration
servo = AngularServo(18, min_angle=-90, max_angle=90,
                     min_pulse_width=0.5/1000, max_pulse_width=2.5/1000)
motor = Motor(forward=11, backward=9)
i2c = busio.I2C(board.SCL, board.SDA)
vl53 = VL53L0X(i2c)
boton = Button(12)
```
Explicación: Se importan las librerías necesarias y se configuran los componentes:

* Servomotor (GPIO 18): Controla la dirección del vehículo

* Motor (GPIO 11 y 9): Controla el movimiento hacia adelante/atrás

* Sensor VL53L0X: Sensor de distancia por láser

* Botón (GPIO 12): Da inicio al código

## Configuración de la cámara

```
cap = cv2.VideoCapture(0)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)
```
 Configura la cámara con resolución 640x480 píxeles para el procesamiento de imágenes.

## DETECCIÓN DE COLORES

```
COLOR1_LOWER = np.array([67, 140, 88])  # Green
COLOR1_UPPER = np.array([179, 255, 255])

COLOR2_LOWER = np.array([158, 120, 40])  # Red (low range)
COLOR2_UPPER = np.array([175, 209, 71])
COLOR2B_LOWER = np.array([170, 100, 100])  # Red (high range)
COLOR2B_UPPER = np.array([179, 255, 255])
```
 Define los rangos HSV para detectar:

* Verde: Objetos a esquivar maniobrando a la izquierda

* Rojo: Objetos a esquivar maniobrando a la derecha

El rojo tiene dos rangos porque en la escala HSV el rojo aparece en ambos extremos del espectro

## PARÁMETROS DE NAVEGACIÓN

```
GREEN_EVASION_DISTANCE = 22  # 22cm
RED_EVASION_DISTANCE = 22  # 22cm
OBSTACLE_DISTANCE = 23  # 40cm
PARKING_DISTANCE = 6
```
Distancias en centímetros para:

* Evadir verde/rojo: 22cm

* Detectar obstáculos: 23cm

* Detectar estacionamiento: 6cm

## Máscara rectangular de la cámara

```
def apply_rectangular_mask(frame, rect_width, rect_height, y_offset):
```
Limita la visión a un área rectangular en la parte inferior de la imagen, enfocándose en lo que está frente al vehículo. Con esto se evita que el auto detecte frecuencias más allá de la zona de juego.

## SECUENCIAS DE MOVIMIENTO
```
Secuencia Inicial
def execute_initial_sequence():


Secuencia de Evasión
def evade_green():  # Gira a izquierda
def evade_red():    # Gira a derecha  
def avoid_obstacle(): # Evasión estándar


Secuencia de Estacionamiento
def execute_parking_sequence():
```

## Bucle Principal
```
while True:
    ret, frame = cap.read()
    distancia = read_constrained_distance_cm()
```
  ## Prioridad de Detección
```
  if color1_obj is not None:  # Verde - prioridad máxima
if color2_obj is not None:  # Rojo - prioridad media  
if color3_obj is not None:  # Estacionamiento - prioridad baja
```
## Lógica de Decisión
```
if distancia <= GREEN_EVASION_DISTANCE and objeto_detectado:
    evade_green()
elif distancia <= RED_EVASION_DISTANCE and objeto_detectado2:
    evade_red()
elif distancia > OBSTACLE_DISTANCE:
    motor.forward(0.25)  # Avanzar
elif distancia < OBSTACLE_DISTANCE:
    avoid_obstacle()  # Esquivar
    ```
