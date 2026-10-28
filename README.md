# myAGV ROS 2 Learning

**Tutoriales y prácticas para aprender robótica móvil con ROS 2 y Python utilizando el Elephant Robotics myAGV 2023 JN.**

Este repositorio es un recurso educativo para estudiantes de ingeniería de nivel **principiante e intermedio**. Parte de los conceptos básicos de ROS 2 y avanza hasta el control de movimiento, la lectura de sensores, el mapeo y la navegación autónoma.

## Robot utilizado

| Característica | Descripción |
|---|---|
| **Fabricante / modelo** | Elephant Robotics **myAGV 2023 JN** |
| **Computadora a bordo** | NVIDIA Jetson Nano (versión JN) |
| **Locomoción** | Cuatro ruedas mecanum; desplazamiento omnidireccional |
| **Sensores relevantes** | LiDAR 2D, IMU y cámara |
| **Entorno de referencia** | Ubuntu 20.04, JetPack 4.6 y **ROS 2 Galactic**, según la rama oficial `galactic-JN` |
| **Lenguaje del curso** | Python (`rclpy`) |

> **Compatibilidad:** ROS 2 Galactic está fuera de soporte. Este material toma como referencia la configuración original del fabricante; comprueba la versión de sistema, firmware y drivers instalada en tu robot antes de ejecutar prácticas.

## Contenido del curso

| Práctica | Tema |
|---|---|
| 00 | Preparación del entorno ROS 2 |
| 01 | Primer nodo Python |
| 02 | Control de movimiento con `/cmd_vel` |
| 03 | Programación de trayectorias |
| 04 | Lectura de sensores |
| 05 | Odometría |
| 06 | Detección de obstáculos |
| 07 | Mapeo con SLAM |
| 08 | Navegación autónoma con Nav2 |


## 📚 Tutoriales y documentación

Este repositorio incluye dos tutoriales para aprender a utilizar y programar el robot móvil **Elephant Robotics myAGV 2023 JN** mediante ROS 2.

### 1. Tutorial de control por teclado

Guía práctica para controlar el movimiento del myAGV, utilizar los comandos de ROS 2, consultar sensores y realizar ejercicios básicos.

📖 [Ver tutorial de comandos de teclado](docs/tutorial_myAGV_2023_JN_teclado_ROS2.md)

### 2. Tutorial de implementación ROS 2 Learning

Guía paso a paso para construir un entorno educativo de programación del myAGV, desde los primeros nodos Python hasta SLAM y navegación autónoma.

📖 [Ver tutorial de implementación ROS 2](docs/tutorial_implementacion_myagv_ros2_learning.md)


La organización propuesta es:

```text
myagv-ros2-learning/
├── README.md
├── docs/
│   └── GUIA_IMPLEMENTACION.md
├── practicas/
│   ├── 00_preparacion/
│   ├── 01_primer_nodo/
│   └── ... hasta 08_navegacion/
└── ros2_ws/
    └── src/
        └── myagv_learning/
```

## Repositorio oficial recomendado

**Elephant Robotics — [`myagv_ros2`, rama `galactic-JN`](https://github.com/elephantrobotics/myagv_ros2/tree/galactic-JN)**

Es la referencia principal para trabajar con el **myAGV 2023 JN**: incluye controladores, teleoperación por teclado, odometría, integración con LiDAR, SLAM y navegación. **Este repositorio educativo lo complementa, pero no lo sustituye ni modifica.**

Otros recursos útiles:

- [Manual oficial del myAGV 2023 JN](https://docs.elephantrobotics.com/docs/myagv_jn23_en/)
- [Ejemplos oficiales ROS 2 en Python, rama Galactic](https://github.com/ros2/examples/tree/galactic/rclpy)

## Primeros pasos

Con el entorno oficial instalado y configurado en la Jetson Nano:

```bash
source /opt/ros/galactic/setup.bash
source ~/myagv_ros2/install/local_setup.bash
ros2 pkg list | grep myagv
```

Para iniciar el controlador del robot (solo después de verificar la conexión y las condiciones de seguridad):

```bash
ros2 launch myagv_odometry myagv_active.launch.py
```

Sigue la **[guía de implementación](docs/GUIA_IMPLEMENTACION.md)** para crear el paquete educativo y ejecutar las prácticas paso a paso.

## Seguridad

Antes de las primeras pruebas de movimiento, coloca el robot sobre un soporte estable con las ruedas libres. Prueba un procedimiento de parada física independiente y trabaja en un área despejada y supervisada. **Cerrar la terminal o interrumpir un programa no garantiza la detención del robot**: el controlador oficial puede conservar la última consigna recibida. No utilices los ejemplos educativos como sistema de seguridad o anticolisión.

## Alcance

Material **educativo y no oficial**, basado en documentación y software públicos de Elephant Robotics. Los ejemplos requieren validación en el equipo físico y pueden necesitar ajustes según la imagen de sistema y el firmware.
