# Guion de demostración del parcial

**Duración objetivo:** 9 minutos y 30 segundos (máximo permitido: 10 minutos).  
**Formato:** mostrar fecha y hora actual del equipo, el rostro y la voz del estudiante. Grabar únicamente las verificaciones y cambios de estado de los requisitos. Subir solamente a YouTube.

| Tiempo | Prueba que se muestra |
| --- | --- |
| 00:00–00:25 | Fecha, hora, rostro, identificación breve y topología |
| 00:25–01:30 | Zabbix: R1 y SW1 disponibles por SNMP y lectura de métricas; consulta SNMP |
| 01:30–02:20 | PC1 obtiene IP por DHCP; ping a R1 y SW1 |
| 02:20–03:45 | Consola Cisco con `cisco.admin`, privilegio 15; NPS autorizado; `show running-config | section line` |
| 03:45–04:20 | AUX: demostrar acceso real, si la imagen permite conectar al puerto; no simular acceso AUX usando consola |
| 04:20–05:45 | Switch: `show ip ssh`, VTY sin Telnet, `no ip http server`, `no ip source-route`, `exec-timeout 5 0`, SNMP RO/ACL y puertos no utilizados si están desactivados |
| 05:45–06:35 | SSH a R1 y SW1 con usuario local; comprobar Telnet rechazado |
| 06:35–08:30 | VPN: túnel activo por GUI FortiGate, `show crypto ipsec sa`, ping PC1→PC2; desactivar por GUI, repetir ping fallido, reactivar |
| 08:30–09:10 | Dirección de equipos, conectividad de LAN y diagramas |
| 09:10–09:30 | GitHub: README, imágenes incrustadas y archivos individuales de configuración |

## Capturas para respaldar la demostración

La documentación requiere imágenes originales que muestren el resultado real de cada requisito. Tomar la captura cuando se vea la información clave, evitando grabar contraseñas, secretos RADIUS, claves de IKE o el contenido sensible del dominio.

## Entrega

Subir el video a YouTube. Insertar el enlace como primera sección del `README.md`. Crear un archivo con nombre `ElianSterlingSeverinoMejia_20251331_P2.txt` si el examen corresponde efectivamente al segundo parcial; confirmar el número de parcial antes de finalizar. El archivo debe contener exactamente dos líneas: enlace GitHub y enlace YouTube.
