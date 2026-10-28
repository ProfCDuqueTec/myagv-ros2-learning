# Guía práctica de comandos de teclado — myAGV 2023 JN (ROS 2 Galactic)

> **Manual didáctico para alumnos principiantes e intermedios**  
> Plataforma: **Elephant Robotics myAGV 2023 JN (Jetson Nano, ruedas mecanum)**  
> Referencia técnica: repositorio del fabricante, rama [`galactic-JN`](https://github.com/elephantrobotics/myagv_ros2/tree/galactic-JN)  
> Versión del tutorial: 1.0 — octubre de 2026

## Objetivos de aprendizaje

Al terminar esta guía, el alumno podrá:

1. Iniciar el controlador del myAGV y el programa de teleoperación con ROS 2.
2. Identificar y utilizar **todas las teclas** que reconoce la versión de `teleop_twist_keyboard` incluida por Elephant Robotics en la rama `galactic-JN`.
3. Distinguir giro, desplazamiento longitudinal y desplazamiento lateral mecanum.
4. Ajustar velocidades desde el teclado.
5. Inspeccionar tópicos y datos del robot desde la terminal ROS 2.
6. Ejecutar los comandos básicos de mapeo y navegación, cuando el sistema ya esté preparado.

---

## 1. Dos maneras de usar el teclado

**A. Teclas interactivas de teleoperación:** presionar `i`, `j`, `k` o `l` mientras se ejecuta `teleop_twist_keyboard` para enviar movimientos al robot.

**B. Instrucciones escritas en la terminal:** escribir `ros2 node list`, `ros2 topic echo /odom` o `ros2 launch ...` para iniciar aplicaciones y consultar sensores.

> **No confundir con AGV_UI ni con ROS 1.** Elephant Robotics también documenta una interfaz `AGV_UI` con un mapa de teclas diferente: ahí `j/l` pueden representar desplazamiento lateral y `u/o` giros. En **este manual**, específicamente para **ROS 2 Galactic + `teleop_twist_keyboard`**, `j/l` hacen girar y `J/L` mayúsculas desplazan lateralmente. Consultar las fuentes al final.

---

## 2. Seguridad y preparación

Antes del primer movimiento:

- Cargar la batería y comprobar que el equipo está correctamente montado.
- Despejar el entorno y retirar personas/objetos de la trayectoria.
- **Realizar la primera prueba con las cuatro ruedas libres del suelo, en un soporte firme.**
- Identificar y comprobar una forma de **parada física independiente** de ROS 2.
- No ejecutar varias aplicaciones que envíen simultáneamente órdenes al tópico `/cmd_vel`.
- No permitir que el robot se mueva sin supervisión.

**Precaución técnica importante:** el controlador `myagv_odometry` conserva el último comando de velocidad recibido y el `teleop_twist_keyboard.py` incluido en la rama `galactic-JN` **no implementa un watchdog basado en tiempo de inactividad**. Si se interrumpe la conexión o un programa falla, no debe suponerse que el robot se detendrá. `k` publica una orden de parada, **no es un botón de emergencia**. La parada física y un watchdog validado son necesarios antes del uso autónomo en piso.

### 2.1. Abrir una terminal

En Ubuntu: **Ctrl + Alt + T**.

Si necesita cargar manualmente el entorno ROS 2 Galactic:

```bash
source /opt/ros/galactic/setup.bash
```

Si el workspace oficial existe y ya fue compilado, cargarlo (ajustar la ruta a la instalación del equipo):

```bash
source ~/myagv_ros2/install/setup.bash
```

Para comprobar el entorno:

```bash
echo $ROS_DISTRO
ros2 --help
ros2 pkg list | grep -E 'myagv|teleop_twist|ydlidar'
```

### 2.2. Terminal 1 — Iniciar el chasis y sensores

```bash
ros2 launch myagv_odometry myagv_active.launch.py
```

Mantener abierta esta terminal; revisar errores de comunicación, puerto serie o sensores.

### 2.3. Terminal 2 — Iniciar la teleoperación

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

**Hacer clic en la terminal de teleoperación** para que reciba las pulsaciones. El programa interpreta **caracteres**; utiliza mejor una distribución de teclado estadounidense (US).

Para comenzar a velocidad reducida:

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard \
  --ros-args -p speed:=0.05 -p turn:=0.15
```

**No iniciar dos programas de teleoperación al mismo tiempo.** La línea con parámetros sustituye a la línea básica, no se suma a ella.

---

## 3. Teclas de movimiento normal (minúsculas)

Distribución que muestra el programa:

```text
     u     i     o
     j     k     l
     m     ,     .
```

| Tecla | Acción física | Componentes ROS 2 enviados (signos) | Ejemplo |
|:---:|---|---|---|
| `i` | Avanzar | `linear.x = +` | Acercarse a una estación |
| `,` | Retroceder | `linear.x = -` | Alejarse de una pared |
| `j` | Girar sobre el eje hacia la izquierda | `angular.z = +` | Cambiar de orientación |
| `l` | Girar sobre el eje hacia la derecha | `angular.z = -` | Reorientarse |
| `u` | Avanzar **y** girar a la izquierda | `linear.x = +`, `angular.z = +` | Curva a la izquierda |
| `o` | Avanzar **y** girar a la derecha | `linear.x = +`, `angular.z = -` | Curva a la derecha |
| `m` | Retroceder **y** girar a la derecha | `linear.x = -`, `angular.z = -` | Maniobra de reversa |
| `.` | Retroceder **y** girar a la izquierda | `linear.x = -`, `angular.z = +` | Reversa en curva |
| `k` | Enviar velocidades cero | `linear.x/y/z = 0`, `angular.z = 0` | Terminar una maniobra |

**Concepto clave:** una curva con `u`/`o` combina avanzar y rotar. No es un desplazamiento lateral puro. Cuando el robot está retrocediendo, la descripción visual de la trayectoria puede resultar poco intuitiva; por eso en la tabla se especifica el **sentido del giro del chasis**.

---

## 4. Teclas para movimiento mecanum (mayúsculas / Shift)

Al introducir las letras mayúsculas (habitualmente **Shift + letra**), el programa activa movimientos holonómicos, con desplazamiento lateral y diagonal **sin ordenar un giro del chasis**.

```text
     U     I     O
     J     K     L
     M     <     >
```

| Tecla recibida | Cómo producirla normalmente | Acción | Componentes de velocidad |
|:---:|---|---|---|
| `U` | `Shift + u` | Adelante + izquierda | `x+`, `y+` |
| `I` | `Shift + i` | Avanzar | `x+` |
| `O` | `Shift + o` | Adelante + derecha | `x+`, `y-` |
| `J` | `Shift + j` | Desplazamiento lateral izquierdo | `y+` |
| `L` | `Shift + l` | Desplazamiento lateral derecho | `y-` |
| `M` | `Shift + m` | Atrás + izquierda | `x-`, `y+` |
| `<` | `Shift + ,` (teclado US) | Retroceder | `x-` |
| `>` | `Shift + .` (teclado US) | Atrás + derecha | `x-`, `y-` |
| `K` | `Shift + k` | Parar: tecla no asignada, publica ceros | Todo en cero |

**Ojo con el teclado en español:** `<` y `>` pueden encontrarse en otra tecla. La versión del fabricante reconoce los **caracteres `<` y `>`**, no necesariamente la combinación física `Shift + ,` / `Shift + .`.

**Comprensión didáctica:** `j` (minúscula) = gira a la izquierda; `J` (mayúscula) = se traslada a la izquierda. Es la diferencia más importante de todo el mapa.

---

## 5. Teclas para modificar las velocidades

| Tecla | Cambio al pulsarla | Afecta a |
|:---:|---|---|
| `q` | Multiplica por `1.1` (+10%) | Lineal y angular |
| `z` | Multiplica por `0.9` (-10%) | Lineal y angular |
| `w` | Multiplica por `1.1` (+10%) | Solo lineal |
| `x` | Multiplica por `0.9` (-10%) | Solo lineal |
| `e` | Multiplica por `1.1` (+10%) | Solo angular |
| `c` | Multiplica por `0.9` (-10%) | Solo angular |

En **la implementación integrada por el fabricante** los parámetros iniciales son:

- `speed = 0.25` → velocidad lineal inicial en m/s.
- `turn = 0.5` → velocidad angular inicial en rad/s.

Otras versiones de `teleop_twist_keyboard` pueden tener diferentes valores predeterminados.

**Ejemplo A — Reducción doble:** partiendo de `0.25 m/s`, dos pulsaciones `z` producen `0.25 × 0.9 × 0.9 = 0.2025 m/s`; el valor angular disminuye de modo análogo.

**Ejemplo B — Aumentar solo la rotación:** con `w` la velocidad angular no cambia; con `e`, la velocidad angular se incrementa un 10% sin cambiar la lineal.

**Advertencia:** las teclas de incremento pueden elevar repetidamente los valores de `speed` y `turn`. La aplicación no establece por sí misma un límite didáctico de seguridad. Conviene iniciar con `-p speed:=0.05 -p turn:=0.15` y no aumentar durante la primera sesión.

---

## 6. Resto de teclas y comportamientos

| Tecla / combinación | Efecto |
|---|---|
| `t` | Publica `linear.z` positivo (movimiento vertical genérico) |
| `b` | Publica `linear.z` negativo (movimiento vertical genérico) |
| `k` | Publica orden de velocidad cero |
| `Espacio` | Tecla no asignada: publica orden de velocidad cero |
| `Ctrl + C` | Solicita salir; el programa intenta publicar una última orden de ceros |
| Otras teclas no reconocidas | Publican orden de velocidad cero |

**Las teclas `t` y `b` NO elevan el myAGV.** La lógica de locomoción del chasis utiliza `linear.x`, `linear.y` y `angular.z`; `linear.z` no se usa para desplazar físicamente este robot terrestre.

**No forman parte de este programa:** flechas de dirección, WASD o barra espaciadora como avance. `Espacio` puede servir para publicar un cero, pero el atajo de parada que enseña el fabricante es `k`.

### 6.1. Atajos de Ubuntu útiles para la práctica

| Atajo | Uso |
|---|---|
| `Ctrl + Alt + T` | Abrir otra terminal |
| `Tab` | Completar comandos y nombres de archivo, cuando Bash lo permite |
| `↑` / `↓` | Recuperar instrucciones previas **en Bash**, no en la teleoperación |
| `Ctrl + L` | Limpiar la pantalla de la terminal Bash |
| `Ctrl + C` | Interrumpir un proceso de terminal |

**Distinción:** cuando `teleop_twist_keyboard` está activo, no se están escribiendo comandos Bash; cada tecla puede convertirse en una instrucción de movimiento.

---

## 7. Prácticas guiadas (primero con ruedas elevadas)

**Práctica 1 — Adelante y parada**  
Teclas: `i` → `k`.  
Observar giro de las ruedas en el sentido de avance y comprobar la parada.

**Práctica 2 — Girar sin avanzar**  
Teclas: `j` → `k` → `l` → `k`.  
Identificar rotación del chasis y comparar ambos sentidos.

**Práctica 3 — Mecanum lateral**  
Teclas: `J` → `k` → `L` → `k`.  
Mantener `Shift` para producir `J` y `L`. Comprobar que se ordena desplazamiento lateral sin giro de orientación.

**Práctica 4 — Curvas**  
Teclas: `u` → `k` → `o` → `k`.  
Comparar movimiento longitudinal más giro con movimiento lateral.

**Práctica 5 — Diagonal**  
Teclas: `U` → `k` → `O` → `k`.  
Comparar diagonal mecanum con curva. Repetir solo si las anteriores se hicieron correctamente.

**Práctica 6 — Escalado de velocidad**  
Teclas: `z` → `z` → `i` → `k`.  
Observar en pantalla cómo cambian los valores de velocidad.

**Evaluación sugerida:** explique en sus palabras la diferencia entre `j` y `J`, entre `u` y `U`, y entre la velocidad **comandada** y la velocidad **medida**.

---

## 8. Comandos ROS 2 que se escriben desde la terminal

Los siguientes comandos son para ejecutar **en Bash**, no como pulsaciones dentro de `teleop_twist_keyboard`.

### 8.1. Diagnóstico y exploración

```bash
# Ver la distribución activa
echo $ROS_DISTRO

# Ayuda general
ros2 --help

# Lista de paquetes disponibles
ros2 pkg list

# Nodos activos
ros2 node list

# Ver información del nodo del chasis, si está activo
ros2 node info /myagv_odometry_node

# Tópicos, con sus tipos
ros2 topic list -t

# Información del tópico de velocidades
ros2 topic info /cmd_vel

# Definición del mensaje de velocidades
ros2 interface show geometry_msgs/msg/Twist
```

### 8.2. Datos de los sensores

```bash
# Odometría estimada
ros2 topic echo /odom

# Datos inerciales
ros2 topic echo /imu

# Lecturas del LiDAR (muchos valores)
ros2 topic echo /scan

# Frecuencia de lecturas del LiDAR
ros2 topic hz /scan

# Voltaje reportado por el chasis
ros2 topic echo /voltage

# Voltaje de respaldo reportado
ros2 topic echo /voltage_backup
```

Los tópicos se publican solo si los nodos y sensores correspondientes están activos; confirmar siempre con `ros2 topic list -t`.

### 8.3. Ejemplo: visualizar lo que envía el teclado

Abrir una **tercera terminal** y ejecutar:

```bash
ros2 topic echo /cmd_vel
```

Volver a la terminal de teleoperación y pulsar `i`. Con los valores predeterminados de la implementación del fabricante, deberá aparecer un mensaje **similar a**:

```yaml
linear:
  x: 0.25
  y: 0.0
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: 0.0
```

Pulsar `k` y comprobar que los campos pasan a cero. **Esto muestra la orden enviada, no confirma que el movimiento físico tenga exactamente esa velocidad.**

### 8.4. Publicar un mensaje de parada de prueba

```bash
ros2 topic pub --once /cmd_vel geometry_msgs/msg/Twist \
  "{linear: {x: 0.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}"
```

Este comando publica **una sola** orden de velocidad cero. No constituye un mecanismo de parada de emergencia: otra aplicación puede publicar inmediatamente otra velocidad y sustituirla. Por seguridad, en esta guía no se propone controlar el movimiento físico mediante publicaciones indefinidas de la CLI.

---

## 9. Comandos para pasar del teclado al mapeo y la navegación

> Requieren un ROS 2 funcional, sensores operativos y una configuración comprobada. No iniciar navegación y teleoperación simultáneas sobre el mismo tópico de movimiento.

**Controlador del robot (terminal 1)**

```bash
ros2 launch myagv_odometry myagv_active.launch.py
```

**Crear un mapa con Gmapping (terminal 2)**

```bash
ros2 launch slam_gmapping slam_gmapping.launch.py
```

**Conducir durante el mapeo (terminal 3)**

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard \
  --ros-args -p speed:=0.05 -p turn:=0.15
```

**Guardar el mapa (terminal 4)**

```bash
ros2 run nav2_map_server map_saver_cli -f ~/map_laboratorio
```

Deberían generarse los archivos `map_laboratorio.pgm` y `map_laboratorio.yaml`, si el servidor de mapas/SLAM está publicando el mapa correctamente.

**Iniciar navegación Nav2 (en una sesión distinta, sin control manual en conflicto)**

```bash
ros2 launch myagv_navigation2 navigation2_active.launch.py \
  map:=$HOME/map_laboratorio.yaml
```

En RViz 2, localizar el robot con **2D Pose Estimate** y seleccionar objetivos mediante **Nav2 Goal**. Las rutas solo deberán ejecutarse después de haber validado mapeo, localización y seguridad.

---

## 10. Solución de problemas comunes

| Síntoma | Diagnóstico inicial |
|---|---|
| Pulso una tecla y no ocurre nada | ¿Está seleccionada la terminal donde corre `teleop_twist_keyboard`? |
| Sale `ros2: command not found` | Revisar `source /opt/ros/galactic/setup.bash` y la instalación ROS 2 |
| Sale `Package not found` | Comprobar el workspace compilado y su `install/setup.bash` |
| Se imprimen comandos pero el chasis no responde | Revisar errores del controlador, puerto serie, alimentación y estado del hardware |
| `j` mueve en vez de girar | Posiblemente está usando **AGV_UI/ROS 1** u otro programa, no este nodo ROS 2 |
| `Shift+,` no retrocede | Comprobar si su teclado realmente produce el carácter `<` |
| No aparece `/scan` | Verificar que está activo el driver LiDAR y que el sensor está alimentado |
| No aparece `/odom` | Verificar el nodo `myagv_odometry_node` |
| El robot sigue moviéndose cuando no pulso teclas | El programa no tiene vigilancia por inactividad; ordenar parada y usar la parada física si es necesario |
| No se genera mapa | Revisar LiDAR, transformaciones, tópicos y el nodo SLAM |

---

## 11. Chuleta de referencia rápida

```text
myAGV 2023 JN  |  teleop_twist_keyboard (ROS 2 Galactic)

MINÚSCULAS                    MAYÚSCULAS / MECANUM
 u    i    o                   U    I    O
 j    k    l                   J    K    L
 m    ,    .                   M    <    >

 i  Adelante                  J  Lateral izquierda
 ,  Atrás                     L  Lateral derecha
 j  Girar izquierda           U  Adelante-izquierda
 l  Girar derecha             O  Adelante-derecha
 u  Curva adelante-izq.       M  Atrás-izquierda
 o  Curva adelante-der.       >  Atrás-derecha
 m  Reversa + giro der.       I  Adelante
 .  Reversa + giro izq.       <  Atrás
 k  Parar                     K  Parar

VELOCIDAD                     TERMINAL
 q / z  Ambas ±10%            Ctrl+Alt+T  Nueva terminal
 w / x  Lineal ±10%           Ctrl+C      Salir/interrumpir
 e / c  Angular ±10%          ros2 topic list  Ver tópicos

 t / b  Comando Z genérico: NO levanta este AMR.
 ATENCIÓN: 'k' NO sustituye una parada de emergencia.
```

---

## 12. Fuentes técnicas

1. Elephant Robotics. [**myAGV 2023 JN — Gmapping y teleoperación ROS 2**](https://docs.elephantrobotics.com/docs/myagv_jn23_en/6-SDKDevelopment/6.3-ApplicationBaseROS2/6.3.2-Real-time_Mapping_with_Gmapping.html).
2. Elephant Robotics. [**Código fuente del teclado `teleop_twist_keyboard.py` en la rama `galactic-JN`**](https://github.com/elephantrobotics/myagv_ros2/blob/galactic-JN/teleop_twist_keyboard/teleop_twist_keyboard.py). **Fuente principal para las asignaciones exactas y parámetros iniciales**.
3. Elephant Robotics. [**Driver `myAGVSub.cpp` que recibe `/cmd_vel`**](https://github.com/elephantrobotics/myagv_ros2/blob/galactic-JN/myagv_odometry/src/myAGVSub.cpp).
4. Elephant Robotics. [**Repositorio oficial ROS 2 del myAGV**](https://github.com/elephantrobotics/myagv_ros2/tree/galactic-JN).
5. Elephant Robotics. [**Navegación Nav2**](https://docs.elephantrobotics.com/docs/myagv_jn23_en/6-SDKDevelopment/6.3-ApplicationBaseROS2/6.3.3-Navigation2.html).
6. Elephant Robotics. [**AGV_UI: distribución de teclas diferente**](https://docs.elephantrobotics.com/docs/myagv_jn23_en/MYAGV_JN_2023_EN/5-BasicApplication/5.2-ApplicationUse/5.2.3-myagv_UI/user_manual.html).
7. ROS 2. [**Conceptos de tópicos, echo y pub (Galactic)**](https://repo.test.ros2.org/en/galactic/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics.html).

---

**Nota de alcance:** las instrucciones se refieren a la implementación de `teleop_twist_keyboard` incluida por Elephant Robotics en la rama `galactic-JN`. No deben asumirse idénticas para ROS 1, AGV_UI ni otras versiones del paquete. Los comandos fueron verificados contra la documentación y el código publicados, pero **no se ejecutaron sobre el robot físico del laboratorio**.
