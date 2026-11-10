## 🎬 Video demostrativo — Segundo Parcial

**▶️ [Ver demostración completa en YouTube](https://youtu.be/m7rV6heYFqU)**

[![Ver video demostrativo del Segundo Parcial](https://img.youtube.com/vi/m7rV6heYFqU/hqdefault.jpg)](https://youtu.be/m7rV6heYFqU)

---

<p align="center">
  <img src="assets/portada-seguridad-redes.svg" alt="Portada del Segundo Parcial de Seguridad de Redes" width="100%"/>
</p>

<h1 align="center">Segundo Parcial · Seguridad de Redes</h1>

<p align="center"><strong>Elian Sterling Severino Mejía</strong> · Matrícula <strong>2025-1331</strong><br/>
Laboratorio de infraestructura, monitoreo, autenticación y comunicaciones seguras</p>

<p align="center">
  <a href="#-topología-y-direccionamiento">Topología</a> ·
  <a href="#-monitoreo-snmp-con-zabbix">SNMP y Zabbix</a> ·
  <a href="#-autenticación-aaa--active-directory--nps">AAA / NPS</a> ·
  <a href="#-hardening-del-switch-cisco">Hardening</a> ·
  <a href="#-vpn-ipsec-cisco--fortigate">VPN IPsec</a> ·
  <a href="evidencias/README.md">Galería completa</a>
</p>

---

## 📌 Presentación del proyecto

Este proyecto reproduce una infraestructura de dos sedes sobre **Proxmox VE y GNS3**. La sede Cisco integra un router IOSv, un switch IOSvL2, una PC cliente, un servidor Windows con Active Directory y NPS, y un servidor Zabbix. La segunda sede emplea un firewall FortiGate y una PC remota.

Se configuraron y verificaron el direccionamiento IP, DHCP, monitoreo SNMP de solo lectura, administración SSH, autenticación y autorización AAA con RADIUS, medidas de hardening y una VPN IPsec sitio a sitio. Las pruebas incluyen conectividad LAN, autenticación de dominio y tráfico cifrado entre las dos redes.

| Módulo | Implementación | Evidencia |
|:--|:--|:--|
| **LAN / DHCP** | Router Cisco como gateway y servidor DHCP | PC1 recibe parámetros y se comunica con la LAN |
| **SNMP + Zabbix** | Router y switch supervisados con comunidad RO y ACL | Respuestas SNMP y hosts disponibles |
| **AAA / NPS** | Active Directory, política NPS y método CON-AUX | Autenticación RADIUS y acceso real por consola |
| **Hardening** | SSHv2, VTY restringidas y servicios inseguros desactivados | Comandos y pruebas sobre IOS |
| **Acceso remoto** | Usuario local y SSH; Telnet bloqueado | Acceso SSH y rechazo del puerto 23 |
| **VPN IPsec** | Cisco IOSv ↔ FortiGate FortiOS 7.0.3 | IKE, ESP y ping PC1 ↔ PC2 |

> **Alcance de la demostración:** la línea AUX tiene AAA configurado, pero el acceso real por AUX requiere una prueba adicional. Las capturas permiten verificar las configuraciones y los resultados que aparecen en ellas; no sustituyen pruebas que no estén registradas.

## 🗺️ Topología y direccionamiento

<p align="center"><a href="evidencias/01-topologia-11-23-48.png"><img src="evidencias/01-topologia-11-23-48.png" alt="Topología de GNS3: router y switch Cisco, servicios auxiliares, enlace hacia FortiGate y redes de usuarios." width="88%"/></a><br/><sub>Topología de GNS3: router y switch Cisco, servicios auxiliares, enlace hacia FortiGate y redes de usuarios.</sub></p>

<details>
<summary><strong>Ver imágenes adicionales de la plataforma y del diseño inicial</strong></summary>

<table><tr>
<td width="50%" valign="top"><a href="evidencias/01-infraestructura-09-08-59.png"><img src="evidencias/01-infraestructura-09-08-59.png" alt="Infraestructura virtualizada y equipos de laboratorio." width="100%"/></a><p align="center"><sub>Infraestructura virtualizada y equipos de laboratorio.</sub></p></td>
<td width="50%" valign="top"><a href="evidencias/01-topologia-09-15-48.png"><img src="evidencias/01-topologia-09-15-48.png" alt="Diseño inicial de los enlaces en GNS3." width="100%"/></a><p align="center"><sub>Diseño inicial de los enlaces en GNS3.</sub></p></td>
</tr></table>



</details>

| Dispositivo / función | Dirección IP | Descripción |
|:--|:--|:--|
| R1-CISCO — LAN | `10.25.13.1/24` | Gateway y DHCP |
| SW1-CISCO | `10.25.13.2/24` | Switch administrable |
| PC1 | `10.25.13.150/24` | Cliente por DHCP |
| SRV-AD-NPS | `10.25.13.10/24` | Windows Server 2019, AD DS, DNS y NPS |
| ZABBIX-SRV | `10.25.13.105/24` | Monitoreo SNMP (IP observada durante las pruebas) |
| Cisco WAN | `172.31.13.1/30` | Peer IPsec de Cisco |
| FortiGate WAN — port2 | `172.31.13.2/30` | Peer IPsec de FortiGate |
| FortiGate LAN — port3 | `10.25.14.1/24` | Gateway LAN remota |
| PC2 | `10.25.14.10/24` | Cliente de la sede FortiGate |

**Puentes Proxmox:** `vmbr1` (LAN Cisco), `vmbr2` (enlace WAN), `vmbr3` (LAN remota). `vmbr0` se utiliza para administración de máquinas y contenedores.

## 🔌 Direccionamiento, DHCP y conectividad

El router Cisco distribuye direcciones en `10.25.13.0/24` a través de DHCP. La PC1 obtuvo la dirección `10.25.13.150/24`, gateway `10.25.13.1` y servidores DNS `10.25.13.10` y `8.8.8.8`. Se verificó ICMP hacia el router y el switch.

<table><tr>
<td width="50%" valign="top"><a href="evidencias/02-conectividad-09-23-49.png"><img src="evidencias/02-conectividad-09-23-49.png" alt="Pruebas de conectividad LAN en la consola de la PC." width="100%"/></a><p align="center"><sub>Pruebas de conectividad LAN en la consola de la PC.</sub></p></td>
<td width="50%" valign="top"><a href="evidencias/07-dhcp-11-24-40.png"><img src="evidencias/07-dhcp-11-24-40.png" alt="Parámetros IP obtenidos por DHCP y pruebas de la LAN remota." width="100%"/></a><p align="center"><sub>Parámetros IP obtenidos por DHCP y pruebas de la LAN remota.</sub></p></td>
</tr></table>



**Comandos para repetir la comprobación:**
```text
PC1> show ip
PC1> ping 10.25.13.1
PC1> ping 10.25.13.2
R1-CISCO# show ip dhcp binding
```

## 📡 Monitoreo SNMP con Zabbix

Se habilitó SNMPv2c de solo lectura en R1 y SW1 con una ACL que limita las consultas al servidor Zabbix. Se comprobaron las respuestas a `sysDescr.0` mediante `snmpwalk`. Ambos dispositivos fueron incorporados a Zabbix usando la plantilla **Cisco IOS by SNMP**.

<table><tr>
<td width="50%" valign="top"><a href="evidencias/03-snmp-09-47-58.png"><img src="evidencias/03-snmp-09-47-58.png" alt="Panel de Zabbix con los hosts Cisco y sus indicadores SNMP." width="100%"/></a><p align="center"><sub>Panel de Zabbix con los hosts Cisco y sus indicadores SNMP.</sub></p></td>
<td width="50%" valign="top"><a href="evidencias/03-snmp-09-43-10.png"><img src="evidencias/03-snmp-09-43-10.png" alt="Verificación desde consola de SNMP y conectividad." width="100%"/></a><p align="center"><sub>Verificación desde consola de SNMP y conectividad.</sub></p></td>
</tr></table>



<details>
<summary><strong>Ver registros de alta de los dispositivos y pruebas adicionales</strong></summary>

<table><tr>
<td width="50%" valign="top"><a href="evidencias/03-snmp-09-45-02.png"><img src="evidencias/03-snmp-09-45-02.png" alt="Configuración de host Cisco en Zabbix." width="100%"/></a><p align="center"><sub>Configuración de host Cisco en Zabbix.</sub></p></td>
<td width="50%" valign="top"><a href="evidencias/03-snmp-09-46-12.png"><img src="evidencias/03-snmp-09-46-12.png" alt="Configuración del segundo host Cisco en Zabbix." width="100%"/></a><p align="center"><sub>Configuración del segundo host Cisco en Zabbix.</sub></p></td>
</tr></table>

<table><tr>
<td width="50%" valign="top"><a href="evidencias/03-snmp-09-50-33.png"><img src="evidencias/03-snmp-09-50-33.png" alt="Comprobaciones adicionales del servicio y del switch." width="100%"/></a><p align="center"><sub>Comprobaciones adicionales del servicio y del switch.</sub></p></td>
<td width="50%"></td>
</tr></table>



</details>

**Verificación técnica:** disponibilidad SNMP en Zabbix, consultas de solo lectura autorizadas y bloqueo de consultas desde orígenes no contemplados en la ACL.

## 🔐 Autenticación AAA · Active Directory + NPS

Se creó una política de autorización de red en **Windows Server 2019 NPS** para la cuenta administrativa del dominio `itla.test`, con grupo `GG-CISCO-ADMINS`. R1 utiliza el grupo RADIUS `NPS-GRP`; los métodos `CON-AUX` se asignaron a **console 0** y **aux 0**, manteniendo el acceso VTY mediante autenticación local.

Las pruebas realizadas incluyen `test aaa group NPS-GRP` con respuesta **User successfully authenticated**, comprobación del privilegio 15 y sesión real en consola mediante `cisco.admin`.

<table><tr>
<td width="50%" valign="top"><a href="evidencias/05-aaa-nps-10-40-55.png"><img src="evidencias/05-aaa-nps-10-40-55.png" alt="Creación de la cuenta de administración en Active Directory." width="100%"/></a><p align="center"><sub>Creación de la cuenta de administración en Active Directory.</sub></p></td>
<td width="50%" valign="top"><a href="evidencias/05-aaa-nps-10-42-19.png"><img src="evidencias/05-aaa-nps-10-42-19.png" alt="Configuración del grupo de seguridad para los administradores." width="100%"/></a><p align="center"><sub>Configuración del grupo de seguridad para los administradores.</sub></p></td>
</tr></table>



<table><tr>
<td width="50%" valign="top"><a href="evidencias/05-aaa-nps-10-50-18.png"><img src="evidencias/05-aaa-nps-10-50-18.png" alt="Política de red en NPS y condiciones de autorización." width="100%"/></a><p align="center"><sub>Política de red en NPS y condiciones de autorización.</sub></p></td>
<td width="50%" valign="top"><a href="evidencias/05-aaa-nps-10-51-14.png"><img src="evidencias/05-aaa-nps-10-51-14.png" alt="Administración y verificación del servicio NPS." width="100%"/></a><p align="center"><sub>Administración y verificación del servicio NPS.</sub></p></td>
</tr></table>



<details>
<summary><strong>Ver la secuencia de comprobación AAA y las consolas del router</strong></summary>

<table><tr>
<td width="50%" valign="top"><a href="evidencias/05-aaa-nps-10-52-04.png"><img src="evidencias/05-aaa-nps-10-52-04.png" alt="Pruebas y salida en consola del router." width="100%"/></a><p align="center"><sub>Pruebas y salida en consola del router.</sub></p></td>
<td width="50%" valign="top"><a href="evidencias/05-aaa-nps-11-02-18.png"><img src="evidencias/05-aaa-nps-11-02-18.png" alt="Resultado de una verificación de AAA en Cisco." width="100%"/></a><p align="center"><sub>Resultado de una verificación de AAA en Cisco.</sub></p></td>
</tr></table>

<table><tr>
<td width="50%" valign="top"><a href="evidencias/05-aaa-nps-11-03-54.png"><img src="evidencias/05-aaa-nps-11-03-54.png" alt="Sesión o prueba administrativa en Cisco." width="100%"/></a><p align="center"><sub>Sesión o prueba administrativa en Cisco.</sub></p></td>
<td width="50%" valign="top"><a href="evidencias/05-aaa-nps-11-05-15.png"><img src="evidencias/05-aaa-nps-11-05-15.png" alt="Comprobaciones de consola y privilegios." width="100%"/></a><p align="center"><sub>Comprobaciones de consola y privilegios.</sub></p></td>
</tr></table>

<table><tr>
<td width="50%" valign="top"><a href="evidencias/05-aaa-nps-11-06-03.png"><img src="evidencias/05-aaa-nps-11-06-03.png" alt="Verificación adicional de acceso y autenticación AAA." width="100%"/></a><p align="center"><sub>Verificación adicional de acceso y autenticación AAA.</sub></p></td>
<td width="50%"></td>
</tr></table>



</details>

```cisco
show running-config | section line con
show running-config | section line aux
show running-config | section line vty
show users
show privilege
```

**Nota de verificación:** se documentó acceso exitoso mediante consola; la línea AUX tiene autenticación y autorización configuradas, pero la prueba real de conexión por AUX sigue siendo un elemento independiente del requisito.

## 🛡️ Hardening del switch Cisco

Se implementaron medidas de endurecimiento alineadas con recomendaciones de seguridad de Cisco IOS/CIS. Los controles incluyen SSHv2 y claves RSA 2048, prohibición de Telnet en VTY, deshabilitación de los servidores web de IOS, deshabilitación de IP source routing, expiración de sesiones administrativas y restricción SNMP.

| Control | Configuración | Cómo verificar |
|:--|:--|:--|
| SSHv2 | `ip ssh version 2` | `show ip ssh` |
| Solo SSH en VTY | `transport input ssh` | `show running-config \| section line vty` |
| HTTP desactivado | `no ip http server` | `show running-config all \| include ip http` |
| HTTPS desactivado | `no ip http secure-server` | `show running-config all \| include ip http` |
| Sin IP source routing | `no ip source-route` | `show running-config all \| include source-route` |
| Timeout administrativo | `exec-timeout 5 0` | `show running-config \| section line vty` |
| SNMP con privilegio mínimo | Comunidad RO + ACL | `show running-config \| include snmp-server` |

**La atribución a controles CIS concretos debe contrastarse con la edición del Benchmark usada en la práctica.** No se afirma cumplimiento formal del benchmark completo.

<table><tr>
<td width="50%" valign="top"><a href="evidencias/04-hardening-09-56-24.png"><img src="evidencias/04-hardening-09-56-24.png" alt="Configuraciones de seguridad aplicadas al switch Cisco." width="100%"/></a><p align="center"><sub>Configuraciones de seguridad aplicadas al switch Cisco.</sub></p></td>
<td width="50%" valign="top"><a href="evidencias/04-hardening-10-01-11.png"><img src="evidencias/04-hardening-10-01-11.png" alt="Verificación del endurecimiento y servicios administrativos." width="100%"/></a><p align="center"><sub>Verificación del endurecimiento y servicios administrativos.</sub></p></td>
</tr></table>



## 🔑 SSH administrativo y Telnet deshabilitado

Se comprobó el acceso remoto con la cuenta local `admin1331` y SSHv2 en el router y en el switch. Desde el servidor de monitoreo se ejecutaron pruebas TCP hacia el puerto 23 en ambos dispositivos y las conexiones Telnet fueron rechazadas.

```bash
nc -vz -w 3 10.25.13.1 23
nc -vz -w 3 10.25.13.2 23
```

La evidencia del acceso administrativo y los comandos de VTY también se encuentran en las secciones anteriores de **AAA** y **Hardening**.

## 🔒 VPN IPsec Cisco ↔ FortiGate

Se configuró una VPN IPsec sitio a sitio entre **R1-CISCO (`172.31.13.1`)** y **FGT-01 (`172.31.13.2`)**. Los selectores protegen `10.25.13.0/24` y `10.25.14.0/24`.

La negociación IKEv1 se observó en estado `QM_IDLE`. Después del ping de PC1 a PC2, el router mostró una asociación ESP activa con **4 paquetes cifrados y 4 descifrados**, sin errores. Se probó también la desactivación y posterior activación del túnel desde la GUI de FortiGate.

<table><tr>
<td width="50%" valign="top"><a href="evidencias/06-vpn-11-13-03.png"><img src="evidencias/06-vpn-11-13-03.png" alt="Configuración IKE y parámetros del túnel en FortiGate." width="100%"/></a><p align="center"><sub>Configuración IKE y parámetros del túnel en FortiGate.</sub></p></td>
<td width="50%" valign="top"><a href="evidencias/06-vpn-11-26-28.png"><img src="evidencias/06-vpn-11-26-28.png" alt="Visualización de la VPN IPsec desde la interfaz de FortiGate." width="100%"/></a><p align="center"><sub>Visualización de la VPN IPsec desde la interfaz de FortiGate.</sub></p></td>
</tr></table>



<table><tr>
<td width="50%" valign="top"><a href="evidencias/06-vpn-11-25-28.png"><img src="evidencias/06-vpn-11-25-28.png" alt="Prueba de conectividad desde la LAN Cisco hacia la LAN FortiGate." width="100%"/></a><p align="center"><sub>Prueba de conectividad desde la LAN Cisco hacia la LAN FortiGate.</sub></p></td>
<td width="50%" valign="top"><a href="evidencias/06-vpn-11-26-59.png"><img src="evidencias/06-vpn-11-26-59.png" alt="Salida de Cisco con la asociación IPsec y los contadores de cifrado." width="100%"/></a><p align="center"><sub>Salida de Cisco con la asociación IPsec y los contadores de cifrado.</sub></p></td>
</tr></table>



<table><tr>
<td width="50%" valign="top"><a href="evidencias/06-vpn-11-28-27.png"><img src="evidencias/06-vpn-11-28-27.png" alt="Operación sobre el túnel en la GUI FortiGate." width="100%"/></a><p align="center"><sub>Operación sobre el túnel en la GUI FortiGate.</sub></p></td>
<td width="50%" valign="top"><a href="evidencias/06-vpn-11-28-53.png"><img src="evidencias/06-vpn-11-28-53.png" alt="Verificación de comunicación entre redes y recuperación del servicio." width="100%"/></a><p align="center"><sub>Verificación de comunicación entre redes y recuperación del servicio.</sub></p></td>
</tr></table>



<details>
<summary><strong>Ver la secuencia restante de pruebas y cambios de estado de la VPN</strong></summary>

<table><tr>
<td width="50%" valign="top"><a href="evidencias/06-vpn-11-28-06.png"><img src="evidencias/06-vpn-11-28-06.png" alt="Comprobación desde PC1 durante la prueba VPN." width="100%"/></a><p align="center"><sub>Comprobación desde PC1 durante la prueba VPN.</sub></p></td>
<td width="50%" valign="top"><a href="evidencias/06-vpn-11-28-38.png"><img src="evidencias/06-vpn-11-28-38.png" alt="Estado del túnel después de una operación de la GUI." width="100%"/></a><p align="center"><sub>Estado del túnel después de una operación de la GUI.</sub></p></td>
</tr></table>



</details>

```cisco
show crypto isakmp sa
show crypto ipsec sa
show crypto map
show access-lists VPN-TRAFFIC
```

> **Seguridad del laboratorio:** por restricciones de la licencia de evaluación FortiGate se utilizó una propuesta heredada DES/SHA1/DH2. Estos algoritmos no son adecuados para producción.

## 📂 Organización del repositorio

```text
Parcial-Redes-Seguridad-2025-1331/
├── README.md
├── assets/
│   └── portada-seguridad-redes.svg
├── diagramas/
│   └── topologia.svg
├── documentacion/
│   ├── verificaciones.md
│   └── guion-video.md
└── evidencias/
    ├── README.md
    ├── 01-*.png              # Infraestructura y topología
    ├── 02-*.png              # Conectividad
    ├── 03-snmp-*.png         # Monitoreo y SNMP
    ├── 04-hardening-*.png    # Endurecimiento Cisco
    ├── 05-aaa-nps-*.png      # AD, NPS, consola y AAA
    ├── 06-vpn-*.png          # VPN Cisco / FortiGate
    └── 07-dhcp-*.png         # Direccionamiento DHCP
```

**[Abrir galería con las 29 capturas](evidencias/README.md)** · **[Abrir el guion del video](documentacion/guion-video.md)** · **[Comandos y verificaciones](documentacion/verificaciones.md)**

### Configuraciones originales

Los running-configs individuales de **R1-CISCO** y **SW1-CISCO**, y la exportación GUI de **FGT-01**, deben incorporarse en archivos independientes, con contraseñas y claves redactadas. No se incluye una configuración completa reconstruida de memoria como si fuera una exportación del equipo.

---

<p align="center"><strong>Segundo Parcial · Seguridad de Redes · 2025-1331</strong><br/><sub>Elian Sterling Severino Mejía · 10 de noviembre de 2026</sub></p>
