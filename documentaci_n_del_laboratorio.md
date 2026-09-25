# Laboratorio de Seguridad de Redes: Implementación y Defensa con FortiGate

## 📹 Video Demostrativo
**[INSERTA AQUÍ EL ENLACE A TU VIDEO DE YOUTUBE / ONEDRIVE]**

*(Nota: En el video se demuestra la configuración vía GUI y el funcionamiento en vivo de las políticas de seguridad, prevención de intrusos, control de descargas y protección DoS).*

---

## 🎯 Propósito del Laboratorio
El propósito de esta práctica es diseñar y desplegar una infraestructura de red segura y segmentada, utilizando un firewall FortiGate como núcleo de enrutamiento y seguridad (configurado íntegramente mediante GUI). Se busca implementar y demostrar controles de acceso estrictos (Zero Trust), inspección profunda de paquetes (DPI), prevención de intrusiones (bloqueo y cuarentena ante SQL Injection), mitigación de ataques de denegación de servicio (DoS) y control de tráfico a nivel de aplicación (bloqueo de ejecutables), garantizando la protección de una granja de servidores frente a la red de usuarios.

---

## 🏗️ Diagrama de Topología
**[INSERTA AQUÍ LA IMAGEN DE TU DIAGRAMA DE GNS3]**

*(Diagrama que muestra la interconexión entre el FortiGate, el Switch Core, la red de Usuarios y la red de Servidores).*

---

## 🖥️ Infraestructura de Red y Segmentación

La topología está segmentada utilizando el Switch Cisco, aplicando seguridad básica de puertos y redes:

*   **VLAN 10 - Red de Usuarios (/25)**
    *   Los usuarios reciben su direccionamiento IP de forma dinámica mediante el servicio **DHCP** configurado en el FortiGate.
*   **VLAN 20 - Red de Servidores (/28)**
    *   **WEB-Server:** Expuesto de forma segura para tráfico HTTPS.
    *   **DB-Server (Base de Datos):** Aislado de los usuarios finales, accesible únicamente por el Web Server.

---

## ⚙️ Configuraciones de Seguridad (FortiGate)

Toda la administración del equipo perimetral se realizó mediante la Interfaz Gráfica (GUI), aplicando las siguientes directivas:

### 1. Enrutamiento y Conectividad Base
*   **Ruta por Defecto (Default Route):** Configurada para garantizar el acceso a Internet de los recursos permitidos.
*   **NAT:** Habilitado en las políticas de salida correspondientes para la traducción de direcciones hacia la red externa.

### 2. Políticas de Control de Acceso (Firewall Policies)
Se aplicó el principio de mínimo privilegio mediante la creación de las siguientes reglas explícitas:
*   ✅ **Política 1:** Se permite el tráfico desde la red de Usuarios (VLAN 10) hacia el WEB-Server **exclusivamente** a través del puerto 443 (HTTPS).
*   🚫 **Política 2:** Se bloquea de forma explícita y total cualquier intento de acceso desde la red de Usuarios (VLAN 10) hacia el DB-Server en el puerto 3306 (MySQL).
*   ✅ **Política de Servidores:** El WEB-Server tiene permitido comunicarse con el DB-Server **única y exclusivamente** por el puerto 3306. Cualquier otro tipo de tráfico interno entre ellos está denegado.

### 3. Inspección Profunda y Seguridad Avanzada
*   **DPI (Deep Packet Inspection):** Activado en las políticas clave para inspeccionar el tráfico cifrado y aplicar perfiles de seguridad avanzados.
*   **Prevención de Intrusos (IPS / WAF) y Cuarentena:** Se creó una regla específica (sensor IPS) que detecta intentos de **SQL Injection** en el tráfico dirigido al WEB-Server.
    *   *Demostración:* Al inyectar payloads maliciosos desde el cliente, el FortiGate **bloquea** el ataque, genera el **log** de seguridad y coloca la IP del atacante en **cuarentena** (Banned IP) temporalmente.
    *   **[INSERTA AQUÍ IMAGEN/CAPTURA DE LOS LOGS DEL FORTIGATE MOSTRANDO EL BLOQUEO DEL SQLi Y LA CUARENTENA]**
*   **Control de Aplicaciones (File Filter):** Se agregó un filtro que intercepta y **bloquea la descarga de archivos ejecutables (`.exe`)** desde la web hacia los usuarios.
    *   **[INSERTA AQUÍ IMAGEN/CAPTURA DEL BLOQUEO DE DESCARGA .EXE]**
*   **Protección DoS (Rate Limiting):** Se implementó una política IPv4 DoS con umbrales (thresholds) definidos para limitar la tasa de paquetes y evitar ataques de denegación de servicio (ej. TCP SYN Flood o ICMP Flood) hacia los recursos internos.

---

## 📂 Repositorio de Scripts y Configuraciones

En cumplimiento con los requerimientos del laboratorio, **absolutamente todos los scripts utilizados y los backups de las configuraciones (Running-Configs)** se encuentran alojados en la raíz de este repositorio.

*   `SW-Core_Config.txt`: Configuración completa de seguridad básica y VLANs del Switch.
*   `FortiGate_Config.conf`: Respaldo completo de la configuración del firewall (Políticas, IPS, DoS, etc.).
*   `Scripts_Pruebas.txt`: Scripts utilizados para las simulaciones de ataque (SQL Injection, descarga de .exe, pruebas DoS).