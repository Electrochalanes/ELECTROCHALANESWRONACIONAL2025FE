Código para el Desafío Abierto
===

## Modos de Operación
El robot tiene tres modos configurables que se encuentran dentro de una matriz que decide las configuraciones según el modo:

```
config_modos = {
    'inner': {  # Para distancias cercanas
        'velocidad_giro': 0.35,
        'velocidad_recto': 0.3,
        'tiempo_giro': 1.8,
        'rango_setpoint': (60, 80),
        'condicion_sensor': (160, 100),
        'pid': {'Kp': 2.5, 'Ki': 0, 'Kd': 9.5},
        'sleep': 0
    },
    'middle': {  # Para distancias medias
        'velocidad_giro': 0.4,
        'velocidad_recto': 0.4,
        'tiempo_giro': 1.8,
        'rango_setpoint': (35, 60),
        'condicion_sensor': (80, 80),
        'pid': {'Kp': 3, 'Ki': 0, 'Kd': 8},
        'sleep': 0.5
    },
    'outter': {  # Para distancias lejanas
        'velocidad_giro': 0.35,
        'velocidad_recto': 0.4,
        'tiempo_giro': 1.4,
        'rango_setpoint': (0, 25),
        'condicion_sensor': (50, 80),
        'pid': {'Kp': 5.0, 'Ki': 0.35, 'Kd': 3.2},
        'sleep': 0.5
    }
}
```

Cada modo tiene parámetros específicos para:

* Velocidades en recto y giro
* Tiempos de giro
* Rangos de distancia objetivo (setpoint)
* Condiciones de sensor para detectar esquinas
* Constantes PID
* Tiempos de espera

### Inicio con botón
```
boton_inicio.wait_for_press()
SETPOINT = sensor.distance * 100  # Distancia inicial en cm
SETPOINT2 = sensor2.distance * 100
```
el robot espera la señal del botón y después establece la distancia inicial como referencia.

### Bucle principal (se detiene al ejecutarse 12 veces):
```
while turn_count < max_turns:
    distancia = sensor2.distance * 100
    distancia2 = sensor3.distance * 100
    
    # Selección de modo basado en SETPOINT
    for modo, config in config_modos.items():
        min_sp, max_sp = config['rango_setpoint']
        if min_sp <= SETPOINT <= max_sp:
            mfo = ejecutar_modo(modo)
            break
```

### Función ejecutar_modo()
Esta función indica cuando debe girar el carro, ajusta los parámetros de PID y se encarga de la condición de finalización.
```
def ejecutar_modo(modo):
    global turn_count, Kp, Ki, Kd
    config = config_modos[modo]
    
    if distancia <= distancia_max and distancia2 > distancia2_min:
        # Realizar giro
        motor.forward(config['velocidad_giro'])
        servo.angle = 31  # Ángulo para girar
        time.sleep(config['tiempo_giro'])
        servo.angle = 80.5  # Volver a posición central
        motor.forward(config['velocidad_recto'])
        time.sleep(config['sleep'])
        turn_count += 1  # Incrementar contador de segmentos
        
        if turn_count == max_turns:
            # Comportamiento final
            if distancia < SETPOINT2:
                motor.forward(0.4)
            else:
                motor.stop()
    
    # Actualizar parámetros PID
    Kp = config['pid']['Kp']
    Ki = config['pid']['Ki']
    Kd = config['pid']['Kd']
    
    return config['velocidad_recto']
```
### PID
Esta función ajusta el ángulo del servomotor para mantener al robot en la distancia indicada
```
# Calcular error
error = SETPOINT - distance

# Términos PID
P = Kp * error  # Proporcional
integral += error * elapsed
I = Ki * integral  # Integral
derivative = (error - previous_error) / elapsed
D = Kd * derivative  # Derivativo

pid_output = P + I + D
