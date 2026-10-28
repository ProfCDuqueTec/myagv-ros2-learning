# myagv-ros2-learning — Guía de implementación 1.0

**Plataforma:** Elephant Robotics myAGV 2023 JN (Jetson Nano)  
**Entorno de referencia:** Ubuntu MATE 20.04 / ROS 2 Galactic, rama oficial `galactic-JN`  
**Destinatarios:** estudiantes principiantes e intermedios de ingeniería  
**Propósito:** construir un repositorio GitHub educativo, ejecutar un paquete ROS 2 propio en Python y desarrollar nueve prácticas (00–08), desde el diagnóstico hasta la navegación.

> **Estado:** tutorial de implementación preparado a partir de documentación y código oficial. Los programas propuestos **no se han probado en un myAGV físico**. Validar siempre la imagen instalada, las interfaces, los signos de ejes y las medidas de seguridad antes del movimiento.

## 1. Qué vamos a construir

El repositorio **`myagv-ros2-learning`** tendrá tres áreas:

1. `docs/`: referencias transversales, diagnóstico y seguridad.
2. `practicas/`: nueve guías para alumnos, cada una con objetivo, comandos, reto y evidencias.
3. `ros2_ws/src/myagv_learning/`: paquete ROS 2 Python con nodos educativos ejecutables.

**No modificar los drivers de Elephant Robotics.** Se utilizarán los paquetes oficiales instalados en `~/myagv_ros2` y las interfaces ROS 2 (`/cmd_vel`, `/odom`, `/scan`, `/imu`). El repositorio docente será un proyecto independiente.

### Plan de prácticas

| Práctica | Nivel | Tema | Evidencia mínima |
|---|---|---|---|
| 00 | Inicial | Preparación y diagnóstico | Registro del SO, ROS 2 y tópicos |
| 01 | Inicial | Primer nodo Python | Nodo que imprime mensajes periódicos |
| 02 | Básico | Control de movimiento | Lectura de `/cmd_vel` y prueba de consigna segura |
| 03 | Básico | Secuencias y trayectorias | Secuencia de movimientos, primero sin contacto con el suelo |
| 04 | Básico | LiDAR e IMU | Monitor de datos y captura de consola |
| 05 | Intermedio | Odometría | Registro y gráfica de `x`, `y` y orientación |
| 06 | Intermedio | Obstáculos | Monitor pasivo por sector frontal del LiDAR |
| 07 | Intermedio | SLAM | Archivos `.pgm` y `.yaml` del mapa |
| 08 | Intermedio | Navegación autónoma | Ruta por objetivos en RViz/Nav2 y registro de resultados |

## 2. Seguridad antes de cualquier movimiento

1. La primera prueba de movimiento se hace **con el robot sobre un soporte estable y las ruedas suspendidas**, sin personas junto a las ruedas.
2. Disponer de un procedimiento de **corte físico de movimiento/alimentación conocido, accesible y comprobado**; los comandos de software no constituyen una parada de emergencia.
3. Una sola aplicación debe controlar `/cmd_vel` durante cada prueba. No mezclar teleoperación, scripts y Nav2.
4. Empezar con consignas pequeñas y tiempos breves. Los valores de `Twist` representan velocidades por convención ROS, pero el driver oficial del myAGV los transmite mediante una codificación escalada y limitada; **no asumir que una consigna equivale exactamente a la velocidad física** sin calibración.
5. **Riesgo confirmado del código oficial:** el nodo de odometría conserva las últimas velocidades y las reutiliza en su bucle; no observamos en esa ruta un watchdog por ausencia de mensajes. Apagar el programa, perder la conexión o pulsar `Ctrl+C` no garantiza inmovilización. Antes de prácticas sobre suelo, integrar y validar una capa de seguridad independiente.
6. El programa de ejemplo de movimiento publica ceros en `finally`, pero no protege ante un bloqueo del proceso, pérdida de comunicaciones o apagado inesperado.
7. No ejecutar código de navegación sin mapa, localización, transformaciones TF y área delimitada previamente verificadas.

## 3. Requisitos previos

- myAGV 2023 JN encendido, batería suficiente y acceso a su Jetson Nano (teclado/pantalla o SSH).
- ROS 2 Galactic instalado en el robot y workspace oficial `~/myagv_ros2` compilado, **si la imagen del robot ya lo incluye**.
- Git y `colcon` disponibles (`git --version`, `colcon --help`).
- Conocimientos básicos de terminal, Python y archivos.
- Conectividad a GitHub únicamente para publicar el repositorio; **no es necesaria para las prácticas locales** si el entorno ya está instalado.

> ROS 2 Galactic llegó al fin de soporte. Evitar instalar paquetes desde repositorios antiguos sin verificar fuentes y compatibilidad. Si el entorno oficial no está listo, consultar primero la guía del fabricante; no reinstalar el sistema a ciegas.

### 3.1. Diagnóstico inicial (práctica 00)

En una terminal del robot:

```bash
cat /etc/os-release
cat /etc/version_log 2>/dev/null || true
ls /opt/ros
source /opt/ros/galactic/setup.bash
echo "$ROS_DISTRO"
which ros2
which colcon
```

Si existe el workspace oficial ya instalado:

```bash
source ~/myagv_ros2/install/local_setup.bash
ros2 pkg list | grep -E 'myagv|ydlidar|slam_gmapping'
```

**Resultado esperado:** ROS_DISTRO = `galactic`, paquetes `myagv_odometry` y `myagv_navigation2` disponibles. Si no aparecen, detener aquí la implementación en hardware y revisar la documentación del fabricante.

> Algunas imágenes del fabricante permiten seleccionar ROS 1 o ROS 2 al abrir cada terminal. Confirmar que **todas las terminales de la práctica usan ROS 2**, no ROS 1.

## 4. Crear el repositorio local

```bash
mkdir -p ~/myagv-ros2-learning
cd ~/myagv-ros2-learning
mkdir -p docs ros2_ws/src

for carpeta in \
  00_preparacion \
  01_primer_nodo \
  02_movimiento \
  03_trayectorias \
  04_sensores \
  05_odometria \
  06_obstaculos \
  07_slam \
  08_navegacion
do
  mkdir -p "practicas/$carpeta"
  touch "practicas/$carpeta/README.md"
done
```

Estructura inicial:

```text
myagv-ros2-learning/
├── README.md
├── .gitignore
├── docs/
│   ├── SEGURIDAD.md
│   └── REFERENCIAS.md
├── practicas/
│   ├── 00_preparacion/README.md
│   ├── 01_primer_nodo/README.md
│   ├── 02_movimiento/README.md
│   ├── 03_trayectorias/README.md
│   ├── 04_sensores/README.md
│   ├── 05_odometria/README.md
│   ├── 06_obstaculos/README.md
│   ├── 07_slam/README.md
│   └── 08_navegacion/README.md
└── ros2_ws/
    └── src/
        └── myagv_learning/   # Lo crea el paso siguiente
```

### 4.1. Archivo `.gitignore`

Dentro de la raíz del repositorio:

```gitignore
ros2_ws/build/
ros2_ws/install/
ros2_ws/log/
__pycache__/
*.py[cod]
.pytest_cache/
.vscode/
.DS_Store
*.db3
```

> Los mapas y resultados pequeños pueden guardarse intencionalmente en las carpetas de evidencias; evitar subir rosbag voluminosos, imágenes de sistema, credenciales o archivos personales.

### 4.2. README principal

Crear `README.md` con: objetivo, modelo exacto, versiones de referencia, advertencias de seguridad, tabla de prácticas, requisitos, procedimiento de instalación y enlaces a la documentación. Puede comenzar copiando las secciones 1–3 de esta guía.

## 5. Crear el primer paquete ROS 2 Python (práctica 01)

**Concepto:** un paquete agrupa código ROS 2; un **nodo** es un programa que se comunica por tópicos, servicios o acciones. Utilizaremos `ament_python` para mantener los ejemplos accesibles a estudiantes que ya conocen Python.

Con ROS 2 Galactic activado:

```bash
cd ~/myagv-ros2-learning/ros2_ws/src
ros2 pkg create myagv_learning \
  --build-type ament_python --node-name hola_ros2 \
  --dependencies rclpy std_msgs geometry_msgs sensor_msgs nav_msgs
```

El asistente crea `package.xml`, `setup.py`, `setup.cfg`, `resource/`, `test/` y el módulo `myagv_learning/`. No confundir **la carpeta del paquete** con **el módulo Python** de igual nombre.

### 5.1. Escribir el nodo `hola_ros2.py`

Guardar en `ros2_ws/src/myagv_learning/myagv_learning/hola_ros2.py`:

```python
import rclpy
from rclpy.node import Node


class HolaRobot(Node):
    def __init__(self):
        super().__init__('hola_robot')
        self.contador = 0
        self.temporizador = self.create_timer(1.0, self.saludar)

    def saludar(self):
        self.contador += 1
        self.get_logger().info(
            f'Hola desde myAGV: mensaje {self.contador}'
        )


def main(args=None):
    rclpy.init(args=args)
    nodo = HolaRobot()
    try:
        rclpy.spin(nodo)
    except KeyboardInterrupt:
        pass
    finally:
        nodo.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### 5.2. Compilar y ejecutar

Desde el workspace **docente** (no el oficial):

```bash
cd ~/myagv-ros2-learning/ros2_ws
source /opt/ros/galactic/setup.bash
# Si también se necesitan los paquetes oficiales:
source ~/myagv_ros2/install/local_setup.bash

colcon build --packages-select myagv_learning --symlink-install \
  --parallel-workers 2
source install/local_setup.bash
ros2 run myagv_learning hola_ros2
```

La terminal deberá mostrar `Hola desde myAGV: mensaje 1`, `...2`, etc. Finalizar con `Ctrl+C`.

**Reto:** cambiar el saludo y la periodicidad a dos segundos.  
**Evidencia:** captura de la salida y explicar qué es un nodo y qué es un temporizador.

## 6. Agregar programas al paquete

A partir de ahora añadiremos otros archivos `.py` al módulo `myagv_learning/`. En el archivo `ros2_ws/src/myagv_learning/setup.py`, ubicar `entry_points` y mantener **un único bloque** `console_scripts` con las siguientes entradas:

```python
entry_points={
    'console_scripts': [
        'hola_ros2 = myagv_learning.hola_ros2:main',
        'monitor_odom = myagv_learning.monitor_odom:main',
        'monitor_lidar = myagv_learning.monitor_lidar:main',
        'monitor_obstaculos = myagv_learning.monitor_obstaculos:main',
        'mover_temporizado = myagv_learning.mover_temporizado:main',
    ],
},
```

**Importante:** esta lista supone que los archivos de las secciones siguientes ya se han creado; si se compila antes, incluir solo los existentes. Cada vez que se modifique `setup.py`, volver a ejecutar `colcon build --packages-select myagv_learning --symlink-install` y cargar `install/local_setup.bash`.

## 7. Práctica 02 — Comprender y emitir comandos de movimiento

### 7.1. Aprender primero a observar

El driver oficial recibe `geometry_msgs/msg/Twist` por el tópico `cmd_vel`. Iniciar la comunicación del robot en la **terminal A**, solo después del diagnóstico:

```bash
source /opt/ros/galactic/setup.bash
source ~/myagv_ros2/install/local_setup.bash
ros2 launch myagv_odometry myagv_active.launch.py
```

En la **terminal B**:

```bash
source /opt/ros/galactic/setup.bash
source ~/myagv_ros2/install/local_setup.bash
ros2 topic list
ros2 topic type /cmd_vel
ros2 topic info /cmd_vel
ros2 topic echo /cmd_vel
```

Si se utiliza la teleoperación del fabricante en una tercera terminal, las teclas `i`, `j`, `k`, `l`, `J` y `L` permiten observar cómo cambia `cmd_vel`. Usar el tutorial de teclado y **no ejecutar un nodo propio que también publique movimiento** durante la teleoperación.

Interpretación de `Twist`:

| Campo | Significado en ROS | Observación para myAGV |
|---|---|---|
| `linear.x` | Desplazamiento longitudinal | Signo positivo: avanzar, si se conserva la convención base |
| `linear.y` | Desplazamiento lateral | Signo positivo: lateral izquierda, validar ejes |
| `angular.z` | Giro | Signo positivo: giro antihorario, validar ejes |
| `linear.z` | Movimiento vertical | Sin función de movimiento terrestre en este driver |

**Advertencia de unidades:** aunque el mensaje ROS usa m/s y rad/s por convención, el método `writeSpeed` del driver transforma las tres consignas en valores codificados y las limita a ±1. Por ello se debe **calibrar** la relación consigna–velocidad real en la práctica 05.

### 7.2. Primer nodo de movimiento temporizado (solo ruedas suspendidas)

Guardar como `ros2_ws/src/myagv_learning/myagv_learning/mover_temporizado.py`:

```python
import time

import rclpy
from rclpy.node import Node
from geometry_msgs.msg import Twist


class MovimientoCorto(Node):
    def __init__(self):
        super().__init__('movimiento_corto')
        self.pub = self.create_publisher(Twist, '/cmd_vel', 10)

    def enviar(self, vx=0.0, vy=0.0, wz=0.0):
        msg = Twist()
        msg.linear.x = float(vx)
        msg.linear.y = float(vy)
        msg.angular.z = float(wz)
        self.pub.publish(msg)


def main(args=None):
    rclpy.init(args=args)
    nodo = MovimientoCorto()
    try:
        # Aguardar al driver: esta comprobación no sustituye una prueba física.
        limite = time.monotonic() + 3.0
        while nodo.pub.get_subscription_count() == 0:
            if time.monotonic() > limite:
                raise RuntimeError('No se detecta suscriptor de /cmd_vel')
            rclpy.spin_once(nodo, timeout_sec=0.1)

        # Publicar primero consigna cero durante un segundo.
        for _ in range(10):
            nodo.enviar()
            rclpy.spin_once(nodo, timeout_sec=0.1)

        # Prueba muy breve; consigna NO calibrada en velocidad física.
        inicio = time.monotonic()
        while time.monotonic() - inicio < 0.4:
            nodo.enviar(vx=0.04)
            rclpy.spin_once(nodo, timeout_sec=0.08)

    finally:
        # Se intenta detener incluso si hubo una excepción Python.
        # No sustituye el corte físico ni protege frente a un cierre abrupto.
        for _ in range(15):
            nodo.enviar()
            rclpy.spin_once(nodo, timeout_sec=0.05)
        nodo.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

**Antes de ejecutarlo:** confirmar que las cuatro ruedas están libres, que se ha probado el sistema de detención física, que `/cmd_vel` tiene un solo controlador y que no están en marcha teleoperación ni Nav2.

```bash
cd ~/myagv-ros2-learning/ros2_ws
colcon build --packages-select myagv_learning --symlink-install \
  --parallel-workers 2
source install/local_setup.bash
ros2 run myagv_learning mover_temporizado
```

**Resultado esperado:** impulso pequeño de las ruedas y después orden cero; no se promete distancia ni velocidad física exacta. Si hay movimiento inesperado, interrumpir físicamente según el procedimiento validado.

**Reto:** cambiar `vx` por `vy` para observar movimiento lateral, siempre con las ruedas suspendidas.  
**Evidencia:** archivo Python y salida de `ros2 topic echo /cmd_vel`.

## 8. Práctica 03 — Diseñar trayectorias

**Concepto:** una trayectoria es una secuencia de consignas a lo largo del tiempo. No es lo mismo una trayectoria **por tiempo** que un desplazamiento **medido por odometría**.

### 8.1. Secuencia conceptual

| Paso | Consigna (vx, vy, wz) | Duración de ejemplo | Efecto esperado |
|---|---|---|---|
| 1 | (0, 0, 0) | 1.0 s | Estado detenido |
| 2 | (0.04, 0, 0) | 0.4 s | Impulso longitudinal |
| 3 | (0, 0, 0) | 1.0 s | Parada |
| 4 | (0, 0.04, 0) | 0.4 s | Impulso lateral |
| 5 | (0, 0, 0) | 1.0 s | Parada |

Implementar primero con el chasis suspendido **modificando el ejemplo de la práctica 02** y encapsulando una función `ejecutar_paso(vx, vy, wz, duracion)`. Tras cada paso, publicar ceros repetidos. No usar esta secuencia para dibujar cuadrados sobre el suelo sin calibración, control por odometría y seguridad adicional.

**Reto:** explicar por qué dos segundos a una misma consigna no garantizan recorrer siempre la misma distancia.  
**Evidencia:** tabla de secuencia y el código comentado.

## 9. Práctica 04 — Leer el LiDAR y la IMU

### 9.1. Inspección por terminal

Con el launch de odometría activo:

```bash
ros2 topic list -t
ros2 topic echo /imu --once
ros2 topic echo /scan --once
ros2 topic hz /scan
```

En algunos entornos puede necesitarse ajustar la política QoS al consultar `LaserScan`:

```bash
ros2 topic echo /scan --qos-reliability best_effort
```

### 9.2. Nodo lector del LiDAR

Archivo `ros2_ws/src/myagv_learning/myagv_learning/monitor_lidar.py`:

```python
import math

import rclpy
from rclpy.node import Node
from rclpy.qos import qos_profile_sensor_data
from sensor_msgs.msg import LaserScan


class MonitorLidar(Node):
    def __init__(self):
        super().__init__('monitor_lidar')
        self.contador = 0
        self.sub = self.create_subscription(
            LaserScan, '/scan', self.recibir, qos_profile_sensor_data
        )

    def recibir(self, msg):
        validas = [d for d in msg.ranges
                   if math.isfinite(d) and msg.range_min <= d <= msg.range_max]
        self.contador += 1
        if self.contador % 10 == 0:
            if validas:
                self.get_logger().info(
                    f'Lecturas válidas: {len(validas)} | '
                    f'mínima distancia: {min(validas):.2f} m'
                )
            else:
                self.get_logger().warn('Sin distancias válidas')


def main(args=None):
    rclpy.init(args=args)
    nodo = MonitorLidar()
    try:
        rclpy.spin(nodo)
    except KeyboardInterrupt:
        pass
    finally:
        nodo.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

Compilar y ejecutar:

```bash
cd ~/myagv-ros2-learning/ros2_ws
colcon build --packages-select myagv_learning --symlink-install \
  --parallel-workers 2
source install/local_setup.bash
ros2 run myagv_learning monitor_lidar
```

**Interpretación:** el valor mínimo es la distancia mínima observada **en cualquier dirección**, no necesariamente frente al robot.  
**Reto:** dibujar un histograma de distancias y explicar qué valores se descartan.  
**Evidencia:** lectura real del LiDAR junto a un obstáculo estático.

## 10. Práctica 05 — Consultar la odometría

El paquete oficial publica `nav_msgs/msg/Odometry` en `/odom` y datos inerciales en `/imu`. La odometría es una **estimación** que puede acumular errores por patinamiento y movimiento mecanum.

Archivo `ros2_ws/src/myagv_learning/myagv_learning/monitor_odom.py`:

```python
import rclpy
from rclpy.node import Node
from nav_msgs.msg import Odometry


class MonitorOdom(Node):
    def __init__(self):
        super().__init__('monitor_odom')
        self.contador = 0
        self.sub = self.create_subscription(
            Odometry, '/odom', self.recibir, 10
        )

    def recibir(self, msg):
        self.contador += 1
        if self.contador % 10 != 0:
            return
        p = msg.pose.pose.position
        v = msg.twist.twist.linear
        self.get_logger().info(
            f'x={p.x:.3f} m, y={p.y:.3f} m | '
            f'vx={v.x:.3f}, vy={v.y:.3f}'
        )


def main(args=None):
    rclpy.init(args=args)
    nodo = MonitorOdom()
    try:
        rclpy.spin(nodo)
    except KeyboardInterrupt:
        pass
    finally:
        nodo.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

```bash
ros2 run myagv_learning monitor_odom
```

**Reto:** registrar `x` e `y` antes y después de un desplazamiento manual corto, una vez que el movimiento en suelo esté autorizado; comparar contra una medición externa.  
**Evidencia:** tabla de valores y discusión del error. No presentar la lectura de `/odom` como verdad absoluta.

## 11. Práctica 06 — Monitor pasivo de obstáculos

**Objetivo:** construir un detector **informativo**, sin emitir movimiento ni pretender que es un mecanismo de seguridad.

Archivo `ros2_ws/src/myagv_learning/myagv_learning/monitor_obstaculos.py`:

```python
import math

import rclpy
from rclpy.node import Node
from rclpy.qos import qos_profile_sensor_data
from sensor_msgs.msg import LaserScan


class MonitorObstaculos(Node):
    def __init__(self):
        super().__init__('monitor_obstaculos')
        self.contador = 0
        self.sub = self.create_subscription(
            LaserScan, '/scan', self.recibir, qos_profile_sensor_data
        )

    def recibir(self, msg):
        # Sector cercano a 0 rad EN EL MARCO DEL LIDAR.
        sector = []
        for i, distancia in enumerate(msg.ranges):
            angulo = msg.angle_min + i * msg.angle_increment
            if (abs(angulo) <= 0.35 and math.isfinite(distancia)
                    and msg.range_min <= distancia <= msg.range_max):
                sector.append(distancia)

        self.contador += 1
        if self.contador % 10 != 0:
            return
        if not sector:
            self.get_logger().warn('Sin datos válidos en sector 0 rad')
            return
        minima = min(sector)
        texto = 'OBJETO CERCANO' if minima < 0.5 else 'Sector libre según LiDAR'
        self.get_logger().info(f'{texto} | mínimo={minima:.2f} m')


def main(args=None):
    rclpy.init(args=args)
    nodo = MonitorObstaculos()
    try:
        rclpy.spin(nodo)
    except KeyboardInterrupt:
        pass
    finally:
        nodo.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

**Precisión crítica:** el sector angular de ±0.35 rad se define respecto a **0 rad del marco LiDAR**. Verificar en RViz/TF que coincide con el frente del chasis antes de llamarlo «frontal». Si no coincide, ajustar el ángulo central documentando el cambio.

**Reto:** modificar el umbral a 0.7 m y analizar falsos positivos/ausencia de datos.  
**Evidencia:** captura de advertencias con un objeto estático acercado manualmente al sensor, sin mover el robot.

## 12. Práctica 07 — Crear un mapa SLAM del laboratorio

Con seguridad física, espacio preparado, LiDAR operativo y teleoperación validada:

**Terminal A:**

```bash
source /opt/ros/galactic/setup.bash
source ~/myagv_ros2/install/local_setup.bash
ros2 launch myagv_odometry myagv_active.launch.py
```

**Terminal B:**

```bash
source /opt/ros/galactic/setup.bash
source ~/myagv_ros2/install/local_setup.bash
ros2 launch slam_gmapping slam_gmapping.launch.py
```

**Terminal C:**

```bash
source /opt/ros/galactic/setup.bash
source ~/myagv_ros2/install/local_setup.bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard \
  --ros-args -p speed:=0.05 -p turn:=0.15
```

**Durante el recorrido:** supervisar visualmente el robot, moverlo despacio y presionar `k` al terminar cada maniobra; `k` es un comando de parada, **no un paro de emergencia**.

Guardar mapa (en otra terminal, una vez mapeado):

```bash
mkdir -p ~/myagv-ros2-learning/evidencias/mapas
ros2 run nav2_map_server map_saver_cli \
  -f ~/myagv-ros2-learning/evidencias/mapas/laboratorio
```

Resultado: `laboratorio.pgm` y `laboratorio.yaml` (si el guardado tiene éxito). Guardar también una captura de RViz y una breve descripción de la zona de prueba. Revisar que `evidencias/mapas/` se incluya en Git solo cuando la publicación del mapa sea permitida.

**Reto:** comparar mapa con croquis del laboratorio e identificar tres diferencias.  
**Evidencia:** mapa y reporte corto.

## 13. Práctica 08 — Navegación con Nav2

**Prerrequisitos obligatorios:** mapa legible, TF consistente, odometría plausible, LiDAR operativo, zona despejada, control de movimiento exclusivo para Nav2 y parada física comprobada.

Terminal A:

```bash
source /opt/ros/galactic/setup.bash
source ~/myagv_ros2/install/local_setup.bash
ros2 launch myagv_odometry myagv_active.launch.py
```

Terminal B:

```bash
source /opt/ros/galactic/setup.bash
source ~/myagv_ros2/install/local_setup.bash
ros2 launch myagv_navigation2 navigation2_active.launch.py \
  map:=$HOME/myagv-ros2-learning/evidencias/mapas/laboratorio.yaml
```

En RViz 2:

1. Cargar el mapa correcto y comprobar que el LiDAR coincide con los obstáculos reales.
2. Utilizar **2D Pose Estimate** para indicar la localización inicial.
3. Elegir **Nav2 Goal** en un punto próximo, despejado y verificable.
4. Supervisar la ruta. Si la localización es errónea o aparece una maniobra peligrosa, utilizar el procedimiento físico seguro.
5. Para nivel intermedio, ensayar una misión de dos objetivos con **Waypoint Following**, después de validar el primer objetivo.

**Reto:** registrar éxito/fallo, desviación final y causa probable de cada intento.  
**Evidencia:** captura de RViz, recorrido y tabla de resultados.

> En esta primera versión del curso, Nav2 se opera desde RViz; la programación mediante clientes de acciones de Nav2 se reserva para una siguiente ampliación, una vez establecida una configuración segura.

## 14. Plantilla de cada práctica para estudiantes

Copiar el siguiente formato en cada `practicas/NN_nombre/README.md`:

```markdown
# Práctica NN — Título

## Objetivo
¿Qué aprenderé y qué conseguiré demostrar?

## Conceptos previos
Nodos, tópicos, mensajes y/o sensores necesarios.

## Requisitos y seguridad
Entorno, herramientas, condiciones para operar sin riesgos.

## Procedimiento
1. Comando o acción, con explicación.
2. Qué observar.
3. Qué significa el resultado.

## Código
Ruta al programa en `ros2_ws/src/myagv_learning/`.

## Reto
Una modificación realizable por el estudiante.

## Evidencias
Código, captura, mediciones y breve reflexión.

## Preguntas de análisis
¿Qué observé? ¿Qué error o limitación encontré?
```

### Rúbrica breve (10 puntos por práctica)

| Criterio | Puntos |
|---|---:|
| Preparación correcta y cumplimiento de seguridad | 2 |
| Ejecución reproducible | 3 |
| Código propio comprensible y comentado | 2 |
| Evidencia verificable | 2 |
| Reflexión técnica | 1 |

Una práctica de hardware **no debe aprobarse** si se omitieron las medidas de seguridad, independientemente de los puntos de funcionalidad.

## 15. Probar el paquete antes de publicar

En `~/myagv-ros2-learning/ros2_ws`:

```bash
source /opt/ros/galactic/setup.bash
source ~/myagv_ros2/install/local_setup.bash
colcon build --packages-select myagv_learning --symlink-install \
  --parallel-workers 2
source install/local_setup.bash
ros2 pkg executables myagv_learning
```

La salida debe listar, según archivos creados y registrados en `setup.py`:

```text
myagv_learning hola_ros2
myagv_learning monitor_odom
myagv_learning monitor_lidar
myagv_learning monitor_obstaculos
myagv_learning mover_temporizado
```

**Validación sin movimiento:** ejecutar `hola_ros2`, `monitor_lidar` y `monitor_odom` antes del programa de movimiento. Solo probar nodos que precisan sensores si se arrancó previamente el driver correspondiente.

## 16. Publicar en GitHub

Primero crear en GitHub un repositorio vacío llamado **`myagv-ros2-learning`** (sin archivos generados automáticamente) en la cuenta personal o de la institución elegida.

Desde la raíz del proyecto:

```bash
cd ~/myagv-ros2-learning
git init
git branch -M main
git add README.md .gitignore docs practicas ros2_ws/src
git commit -m "Crear curso myAGV ROS 2: estructura y ejemplos"

git remote add origin https://github.com/TU_USUARIO/myagv-ros2-learning.git
git push -u origin main
```

Sustituir `TU_USUARIO` por la cuenta real. La autenticación se hará con el mecanismo de GitHub disponible en el equipo (por ejemplo, credencial Git o SSH), **no escribiendo tokens en el repositorio**.

Para actualizar el curso:

```bash
git status
git add practicas ros2_ws/src docs README.md
git commit -m "Completar practica de sensores"
git push
```

### Nota sobre licencias y atribución

- Nuestro código educativo puede publicarse con una licencia elegida expresamente por sus autores/institución; no inventar ni asignar una licencia institucional sin autorización.
- **No copiar código de repositorios ajenos eliminando sus avisos**. Cuando se adapte un ejemplo, incluir origen, enlace, licencia y cambios realizados.
- Preferir enlazar al repositorio oficial de Elephant Robotics a incorporar sus fuentes dentro de nuestro repositorio docente.

## 17. Diagnóstico rápido de problemas

| Situación | Posible causa / acción |
|---|---|
| `ros2: command not found` | Activar `/opt/ros/galactic/setup.bash` |
| `Package 'myagv_learning' not found` | Compilar y cargar `ros2_ws/install/local_setup.bash` |
| `No executable found` | Revisar `console_scripts` y recompilar |
| `myagv_odometry` no aparece | Revisar entorno y workspace del fabricante |
| No llegan mensajes `/scan` | Iniciar LiDAR; comprobar QoS, alimentación, driver y tópico |
| No llega `/odom` | Verificar nodo de odometría y puerto serie `/dev/ttyS0` |
| `cmd_vel` tiene más de un publicador | Parar teleoperación/Nav2/scripts concurrentes |
| El robot no se detiene al cerrar terminal | Riesgo de consigna retenida; usar parada física y corregir arquitectura con watchdog probado |
| Dirección de movimiento invertida | No corregir a ciegas: revisar TF, ejes, signo y configuración física |
| Mapa incorrecto | Revisar calibración, odometría, velocidad, LiDAR y transformaciones |
| Fallo de memoria en Jetson | Compilar solo paquete seleccionado y limitar paralelismo |

## 18. Checklist de entrega del proyecto

- [ ] Imagen del myAGV y versión ROS 2 identificadas.
- [ ] Paquetes oficiales visibles en ROS 2.
- [ ] Procedimiento físico de parada comprobado.
- [ ] Repositorio local con nueve prácticas.
- [ ] Paquete `myagv_learning` compila.
- [ ] `hola_ros2` funciona sin mover el robot.
- [ ] Lecturas de `/scan`, `/imu` y `/odom` comprobadas.
- [ ] Prueba de movimiento solo con ruedas suspendidas realizada y documentada.
- [ ] Advertencias de control retenido/watchdog comunicadas a los alumnos.
- [ ] Condiciones de seguridad autorizan, si procede, movimiento sobre suelo.
- [ ] Mapa SLAM guardado y revisado.
- [ ] Navegación con Nav2 validada.
- [ ] Código y evidencias publicados sin credenciales ni material confidencial.

## 19. Referencias

**Fabricante / robot:**

1. [Repositorio oficial `elephantrobotics/myagv_ros2`, rama `galactic-JN`](https://github.com/elephantrobotics/myagv_ros2/tree/galactic-JN) — paquetes y README de instalación.
2. [Introducción ROS 2 para myAGV 2023 JN](https://docs.elephantrobotics.com/docs/myagv_jn23_en/6-SDKDevelopment/6.3-ApplicationBaseROS2/6.3.1-ROS2_Introduction.html) — preparación y compilación.
3. [Gmapping y teleoperación](https://docs.elephantrobotics.com/docs/myagv_jn23_en/6-SDKDevelopment/6.3-ApplicationBaseROS2/6.3.2-Real-time_Mapping_with_Gmapping.html) — proceso de mapeo.
4. [Nav2 en myAGV 2023 JN](https://docs.elephantrobotics.com/docs/myagv_jn23_en/6-SDKDevelopment/6.3-ApplicationBaseROS2/6.3.3-Navigation2.html) — navegación y waypoints.
5. [Código del suscriptor oficial `cmd_vel`](https://github.com/elephantrobotics/myagv_ros2/blob/galactic-JN/myagv_odometry/src/myAGVSub.cpp) — interfaz a chasis.
6. [Código del driver oficial `myAGV.cpp`](https://github.com/elephantrobotics/myagv_ros2/blob/galactic-JN/myagv_odometry/src/myAGV.cpp) — codificación de consignas y publicación de sensores.

**Formación:**

7. [Ejemplos oficiales ROS 2 para Python, rama Galactic](https://github.com/ros2/examples/tree/galactic/rclpy) — nodos y comunicación.
8. [Tutorial de creación de paquetes, ROS 2 Galactic](https://docs.ros.org/en/galactic/Tutorials/Beginner-Client-Libraries/Creating-Your-First-ROS2-Package.html) — `ament_python` y `colcon`.
9. [Robotics & ROS 2 Essentials](https://github.com/henki-robotics/robotics_essentials_ros2) — referencia pedagógica con simulación, en una distribución distinta.

---

**Siguiente evolución propuesta (v1.1):** añadir tests automatizados de los nodos pasivos, un simulador dedicado para experimentar sin arriesgar el robot real y una capa de supervisión de comandos con watchdog físico/funcional validado, antes de ampliar las misiones con Python y Nav2.
