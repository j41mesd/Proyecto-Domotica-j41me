# Comparativa Técnica y Diferenciación del Proyecto

En esta sección se analiza la propuesta de Proyecto-Domotica-j41me, frente a referencias existentes en la comunidad, específicamente el proyecto **"X10 Control with ScadaBR" de Electron Hacks**, con el fin de destacar las innovaciones y mejoras técnicas implementadas.
Este es el enlace de la libreria Modbus (La dirección del X10, no aparece en github, asi que voy a poner un enlace que usa Electron Hacks, para la comunicación modbus-arduino: https://github.com/andresarmento/modbus-arduino)

## 1. Referencia de Contraste
* **Proyecto:** X10 Home Automation with ScadaBR and Arduino.
* **Fuente:** Electron Hacks (Protocolo X10).
* **Uso:** Utilizado como base para entender la integración de ScadaBR con microcontroladores, aunque con una arquitectura de comunicación distinta.

---

## 2. Diferenciación Técnica Detallada

### Protocolo de Comunicación y Robustez
Mientras que la referencia utiliza el protocolo **X10** (señales sobre red eléctrica), propenso a interferencias y ruidos, este proyecto implementa **Modbus RTU/Directo**. Esta elección garantiza una comunicación serie industrial, más rápida y fiable para sistemas críticos.

### Evolución del Rol de Arduino
En la referencia, el Arduino actúa solo como una **pasarela (gateway)** que traduce comandos. En este proyecto, el Arduino funciona como un **controlador local inteligente**: procesa la lógica de aerotermia y actúa sobre los relés de forma autónoma, asegurando el funcionamiento incluso sin conexión al servidor ScadaBR.

### Monitorización de Variables vs. Control Binario
A diferencia del control básico **ON/OFF** de la referencia, este sistema realiza una **monitorización precisa de variables analógicas** (temperatura y humedad). Esto permite una gestión de datos en tiempo real y un control de clima dinámico.

### Seguridad Industrial y Normativa
Este proyecto se diferencia por su enfoque profesional, habiendo sido diseñado bajo la **norma UNE-EN 60204-1**. Esto asegura el aislamiento físico de las señales de control y el cumplimiento de estándares de seguridad, a diferencia de los montajes domésticos estándar.

### Optimización de Costes y Escalabilidad
Se mantiene un presupuesto estrictamente optimizado de **31,26 €**. Al evitar módulos X10 obsoletos y usar el estándar Modbus, el sistema permite añadir **nodos esclavos** de forma económica y sencilla, algo inviable en la arquitectura de referencia.

### Interfaz de Gestión en ScadaBR
La interfaz no es un simple mando a distancia. Se ha desarrollado un **Dashboard de Gestión** profesional en ScadaBR que permite supervisar gráficas de rendimiento, estados de seguridad y ajustar consignas de temperatura para el sistema de aerotermia.
