# Laboratorio de Seguridad de Redes: Arquitectura Segura con FortiGate y Cisco

**Autor:** Jaymel Feliz
**Matrícula:** 20240886
**Plataforma de Virtualización:** GNS3
**Enlace al Video Demostrativo:** https://youtu.be/pNCY_9pFcsY

---

## Tabla de Contenido
1. [Propósito del Laboratorio](#1-propósito-del-laboratorio)
2. [Topología de Red](#2-topología-de-red)
3. [Tabla de Direccionamiento](#3-tabla-de-direccionamiento)
4. [Funcionamiento de la Configuración](#4-funcionamiento-de-la-configuración)
5. [Configuraciones Implementadas](#5-configuraciones-implementadas)
6. [Políticas de Seguridad](#6-políticas-de-seguridad)
7. [Validación y Pruebas](#7-validación-y-pruebas)
8. [Diagramas y Evidencias](#8-diagramas-y-evidencias)
9. [Running Configurations](#9-running-configurations)
10. [Scripts y Archivos Utilizados](#10-scripts-y-archivos-utilizados)
11. [Conclusión](#11-conclusión)

---

## 1. Propósito del Laboratorio
El objetivo principal de esta práctica es diseñar, implementar y auditar una infraestructura de red segura aplicando principios de **Defensa en Profundidad** y **Zero-Trust**. Mediante la integración de un firewall FortiGate (NGFW) y un Switch Cisco (Capa 2), el laboratorio busca aislar los entornos de usuarios y servidores, mitigar ataques de capa de aplicación (como Inyecciones SQL y descargas de malware), y proteger la disponibilidad de los servicios frente a ataques de Denegación de Servicio (DoS).

## 2. Topología de Red
La arquitectura consta de tres zonas principales conectadas a través de un enlace troncal (Trunk 802.1Q) desde un Switch Cisco Core hacia un FortiGate:
* **Zona LAN (Usuarios):** Compuesta por equipos finales en la VLAN 10 que reciben configuración dinámica (DHCP).
* **Zona DMZ/Servidores:** Compuesta por dos servidores Ubuntu en la VLAN 20 (WEB-Server y DB-Server) con direccionamiento estático.
* **Zona WAN (Internet):** Enlace de salida a través de la nube NAT de GNS3.

<img width="585" height="503" alt="image" src="https://github.com/user-attachments/assets/b3a27096-4820-4637-bdf8-21d81053d04d" />

## 3. Tabla de Direccionamiento
El esquema IP ha sido diseñado de manera jerárquica utilizando los últimos dígitos de la matrícula asignada (**20240886**).

| Dispositivo / Zona | VLAN | Red Asignada | Gateway (FortiGate) | IP Asignada / Rango |
| :--- | :---: | :--- | :--- | :--- |
| **Usuarios (VPCS)** | 10 | `10.8.86.0/25` | `10.8.86.1` | Por DHCP (`.2` al `.126`) |
| **WEB-Server (HTTPS)**| 20 | `10.8.86.128/28` | `10.8.86.129` | `10.8.86.130` |
| **DB-Server (MySQL)** | 20 | `10.8.86.128/28` | `10.8.86.129` | `10.8.86.131` |

<img width="383" height="159" alt="image" src="https://github.com/user-attachments/assets/15c83779-ccf0-4184-8fd2-6480cbcbdd27" />


## 4. Funcionamiento de la Configuración
La red opera bajo un modelo de segmentación estricta. Todo el tráfico Inter-VLAN es enrutado y filtrado obligatoriamente por el FortiGate. Los usuarios en la VLAN 10 tienen salida a Internet gracias a una regla de **NAT (Network Address Translation)** y una **Ruta por Defecto (0.0.0.0/0)** hacia el puerto WAN del firewall. El tráfico interno no es nateado, pero sí es sometido a **Inspección Profunda de Paquetes (DPI)** para evaluar amenazas en la capa de aplicación de forma transparente.

## 5. Configuraciones Implementadas
**Switch Cisco (Capa 2):**
* Creación e identificación de VLAN 10 y VLAN 20.
* Configuración de puerto Troncal (Gi0/0) con deshabilitación de DTP (`switchport nonegotiate`) para evitar ataques de VLAN Hopping.
* Seguridad de puertos de acceso mediante la mitigación de bucles y suplantación con `spanning-tree portfast` y `bpduguard enable`.

<img width="100" height="71" alt="image" src="https://github.com/user-attachments/assets/9b30fd2a-28fc-45de-87a0-ca33bedb52d5" />


**FortiGate (Capa 3):**
* Creación de subinterfaces lógicas (VLAN 10 y VLAN 20) sobre el puerto físico conectado al switch.
* Configuración del servicio DHCP Server exclusivo para la interfaz de la VLAN 10.
* Creación de *Address Objects* para el manejo limpio y estandarizado de las políticas de seguridad.

* <img width="105" height="72" alt="image" src="https://github.com/user-attachments/assets/a488f9e5-f056-4a94-a037-dfabc857e4f2" />


## 6. Políticas de Seguridad
Para cumplir estrictamente con los requerimientos, se implementaron las siguientes políticas vía GUI en el FortiGate:

1. **Pol1_Usuarios_a_Web:** Permite el tráfico de la VLAN 10 hacia la IP del WEB-Server únicamente por el puerto `HTTPS (443)`. <img width="990" height="841" alt="image" src="https://github.com/user-attachments/assets/14a4bedd-19aa-498d-b0af-14aa67f3dda3" />

2. **Pol2_Bloqueo_Usuarios_a_DB:** Bloquea explícitamente y registra (Log) cualquier intento de conexión desde la VLAN 10 hacia el DB-Server por el puerto `MYSQL (3306)`. <img width="993" height="637" alt="image" src="https://github.com/user-attachments/assets/ae95e498-2fe5-4c33-9040-9a04b2bf33c4" />

3. **Comunicación de Servidores:** Solo el WEB-Server está autorizado para comunicarse con el DB-Server a través del puerto 3306. <img width="993" height="832" alt="image" src="https://github.com/user-attachments/assets/b4462532-57b2-414d-b775-255ab87a3d45" />

4. **DPI & IPS (SQL Injection):** Se activó un sensor IPS sobre la Política 1 para detectar payloads maliciosos (SQLi). La acción configurada es **Block & Quarantine**, lo que bloquea el tráfico y aísla la IP del atacante. <img width="1001" height="651" alt="image" src="https://github.com/user-attachments/assets/efccc9db-8852-400d-a1a4-b3252935ac1a" />

5. **Application Control (Bloqueo de `.exe`):** Se implementó un perfil de filtrado que identifica a nivel de aplicación y descarta la descarga de archivos ejecutables, previniendo malware. <img width="1001" height="636" alt="image" src="https://github.com/user-attachments/assets/2edce031-39b8-44ef-b2cf-62ca0c0306f0" />

6. **Rate Limiting (DoS Policy):** Se configuraron umbrales de prevención de anomalías (*SYN Flood* y límite de conexiones concurrentes) para proteger al WEB-Server de ataques de denegación de servicio. <img width="992" height="877" alt="image" src="https://github.com/user-attachments/assets/cb4066a4-d04a-4f2b-a1cc-e1ffcca9e98c" />


## 7. Validación y Pruebas
Las pruebas demostradas en el video adjunto incluyen:
1. **Conectividad y DHCP:** La VPCS/UBUNTU PC recibe exitosamente su IP (`10.8.86.X`) y tiene conectividad a Internet. <img width="897" height="674" alt="image" src="https://github.com/user-attachments/assets/0c5145e1-6f41-4ec5-9b55-cfa6bff498f6" />

2. **Control de Acceso Web y Base de Datos:** Conexión exitosa al WEB-Server (443), pero rechazo absoluto al intentar hacer ping o conectar al puerto 3306 del DB-Server desde la VLAN de usuarios. <img width="915" height="87" alt="image" src="https://github.com/user-attachments/assets/b86436ce-cfe0-452c-bc81-f348f1b47eb1" />

3. **Detección SQLi y Cuarentena:** Se inyecta un payload SQL en el tráfico hacia el servidor web. El firewall intercepta el ataque, corta la sesión y coloca la IP origen en cuarentena. <img width="901" height="86" alt="image" src="https://github.com/user-attachments/assets/e0a74696-610d-4680-b1b7-efbda06212d3" />
<img width="1655" height="265" alt="image" src="https://github.com/user-attachments/assets/ef65196e-661b-47b8-812b-3b1c0845e71a" />

4. **Bloqueo de Malware (.exe):** Intento fallido de descargar un archivo ejecutable desde la web, denegado por Application Control.
5. <img width="894" height="206" alt="image" src="https://github.com/user-attachments/assets/bfcaa185-6a86-4fe8-97b1-c72deeb6a350" />


## 8. Diagramas y Evidencias

### 8.1 Diagrama Lógico de la Topología en GNS3
> <img width="393" height="374" alt="image" src="https://github.com/user-attachments/assets/84b20a48-87cc-49e3-a330-c959e283ffb5" />


### 8.2 Evidencia: Política de Bloqueo a la Base de Datos (DB-Server)
> <img width="1644" height="413" alt="image" src="https://github.com/user-attachments/assets/bcd7b58d-87bb-49cc-a425-71a37e0e6549" />

<img width="1216" height="676" alt="image" src="https://github.com/user-attachments/assets/56bd785b-462c-4cb4-93fb-60ecdef8b816" />


### 8.3 Evidencia: Bloqueo y Cuarentena por Ataque de Inyección SQL
> <img width="1639" height="266" alt="image" src="https://github.com/user-attachments/assets/e68cd20a-fe67-497e-acd2-f858e7cc670b" />

<img width="1216" height="676" alt="image" src="https://github.com/user-attachments/assets/7baf9189-9e52-444d-a3cd-c473a6c12bc0" />


### 8.4 Evidencia: Bloqueo de Descargas Ejecutables (App Control)
> <img width="887" height="114" alt="image" src="https://github.com/user-attachments/assets/54f60f7a-5edb-411e-8397-acafb35155d8" />


## 9. Running Configurations

Este es el running config

enable
configure terminal
hostname SW-Core

! Seguridad básica
service password-encryption
enable secret cisco123

! Creación de VLANs
vlan 10
 name Usuarios
vlan 20
 name Servidores
exit

! Configuración del Troncal (Hacia FortiGate)
interface GigabitEthernet0/0
 description ENLACE_TRONCAL_FORTIGATE
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport nonegotiate
 exit

! Configuración Puerto Usuario (VLAN 10)
interface GigabitEthernet0/1
 description VPCS_USUARIO
 switchport mode access
 switchport access vlan 10
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
 exit

! Configuración Puerto WEB-Server (VLAN 20)
interface GigabitEthernet0/2
 description WEB_SERVER
 switchport mode access
 switchport access vlan 20
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
 exit

! Configuración Puerto DB-Server (VLAN 20)
interface GigabitEthernet0/3
 description DB_SERVER
 switchport mode access
 switchport access vlan 20
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
 exit

end
write memory

## 10. Scripts y Archivos Utilizados
Para las auditorías y pruebas de intrusión demostradas en la sección de validación, se utilizaron los siguientes comandos desde la terminal del Usuario:

# CONFIGURACION DEL SWITCH CISCO (NO EL SHOW RUNNING-CONFIG)

enable
configure terminal
hostname SW-Core

! Seguridad básica
service password-encryption
enable secret cisco123

! Creación de VLANs
vlan 10
 name Usuarios
vlan 20
 name Servidores
exit

! Configuración del Troncal (Hacia FortiGate)
interface GigabitEthernet0/0
 description ENLACE_TRONCAL_FORTIGATE
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport nonegotiate
 exit

! Configuración Puerto Usuario (VLAN 10)
interface GigabitEthernet0/1
 description VPCS_USUARIO
 switchport mode access
 switchport access vlan 10
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
 exit

! Configuración Puerto WEB-Server (VLAN 20)
interface GigabitEthernet0/2
 description WEB_SERVER
 switchport mode access
 switchport access vlan 20
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
 exit

! Configuración Puerto DB-Server (VLAN 20)
interface GigabitEthernet0/3
 description DB_SERVER
 switchport mode access
 switchport access vlan 20
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
 exit

end
write memory

# SHOW RUNNING-CONFIG

Building configuration...

Current configuration : 3862 bytes
!
! Last configuration change at 23:43:28 UTC Fri Sep 25 2026
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
service password-encryption
service compress-config
!
hostname SW-Core
!
boot-start-marker
boot-end-marker
!
!
enable secret 5 $1$8Gv6$xUVCaHtu7qI3VKwvMjI4f/
!
no aaa new-model
!
!
!
!
!
!
!
!
ip cef
no ipv6 cef
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
interface GigabitEthernet0/0
 description ENLACE_TRONCAL_FORTIGATE
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport nonegotiate
 negotiation auto
!
interface GigabitEthernet0/1
 description VPCS_USUARIO
 switchport access vlan 10
 switchport mode access
 switchport nonegotiate
 negotiation auto
 spanning-tree portfast edge
 spanning-tree bpduguard enable
!
interface GigabitEthernet0/2
 description WEB_SERVER
 switchport access vlan 20
 switchport mode access
 switchport nonegotiate
 negotiation auto
 spanning-tree portfast edge
 spanning-tree bpduguard enable
!
interface GigabitEthernet0/3
 description DB_SERVER
 switchport access vlan 20
 switchport mode access
 switchport nonegotiate
 negotiation auto
 spanning-tree portfast edge
 spanning-tree bpduguard enable
!
interface GigabitEthernet1/0
 negotiation auto
!
interface GigabitEthernet1/1
 negotiation auto
!
interface GigabitEthernet1/2
 negotiation auto
!
interface GigabitEthernet1/3
 negotiation auto
!
interface GigabitEthernet2/0
 negotiation auto
!
interface GigabitEthernet2/1
 negotiation auto
!
interface GigabitEthernet2/2
 negotiation auto
!
interface GigabitEthernet2/3
 negotiation auto
!
interface GigabitEthernet3/0
 negotiation auto
!
interface GigabitEthernet3/1
 negotiation auto
!
interface GigabitEthernet3/2
 negotiation auto
!
interface GigabitEthernet3/3
 negotiation auto
!
ip forward-protocol nd
!
ip http server
ip http secure-server
!
ip ssh server algorithm encryption aes128-ctr aes192-ctr aes256-ctr
ip ssh client algorithm encryption aes128-ctr aes192-ctr aes256-ctr
!
!
!
!
!
!
control-plane
!
banner exec ^C
IOSv - Cisco Systems Confidential -


Supplemental End User License Restrictions


This IOSv software is provided AS-IS without warranty of any kind. Under no circumstances may this software be used separate from the Cisco Modeling Labs Software that this software was provided with, or deployed or used as part of a production environment.


By using the software, you agree to abide by the terms and conditions of the Cisco End User License Agreement at http://www.cisco.com/go/eula. Unauthorized use or distribution of this software is expressly prohibited.
^C
banner incoming ^C
IOSv - Cisco Systems Confidential -


*Sep 25 23:46:50.720: %PLATFORM-5-SIGNATURE_VERIFIED: Image 'flash0:/vios_l2-adventerprisek9-m' passed code signing verification

Supplemental End User License Restrictions


This IOSv software is provided AS-IS without warranty of any kind. Under no circumstances may this software be used separate from the Cisco Modeling Labs Software that this software was provided with, or deployed or used as part of a production environment.


By using the software, you agree to abide by the terms and conditions of the Cisco End User License Agreement at http://www.cisco.com/go/eula. Unauthorized use or distribution of this software is expressly prohibited.
^C
banner login ^C
IOSv - Cisco Systems Confidential -


Supplemental End User License Restrictions


This IOSv software is provided AS-IS without warranty of any kind. Under no circumstances may this software be used separate from the Cisco Modeling Labs Software that this software was provided with, or deployed or used as part of a production environment.


By using the software, you agree to abide by the terms and conditions of the Cisco End User License Agreement at http://www.cisco.com/go/eula. Unauthorized use or distribution of this software is expressly prohibited.
^C
!
line con 0
line aux 0
line vty 0 4
 login
!
!
end

# 1. Solicitar IP por DHCP (VPCS)
ip dhcp

# 2. Prueba de ping cruzado (Debe fallar hacia la base de datos)
ping 10.8.86.131

# 3. Simulación de ataque Inyección SQL hacia el Web Server
curl -k -G --data-urlencode "user=admin' OR '1'='1" "https://10.8.86.130/login.php"

# 4. Intento de descarga de archivo ejecutable malicioso (App Control)
wget http://nmap.org/dist/nmap-7.95-setup.exe

**Payload para prueba de SQL Injection (Simulación de Inyección):**
```bash
curl -k -G --data-urlencode "user=admin' OR '1'='1" "[https://10.8.86.130/login.php](https://10.8.86.130/login.php)"
