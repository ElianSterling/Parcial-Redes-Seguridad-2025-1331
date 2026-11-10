# Archivos de configuración

Estos archivos provienen de **exportaciones reales realizadas el 10 de noviembre de 2026**, no de configuraciones construidas a partir de ejemplos.

| Dispositivo | Archivo | Procedencia | Alcance |
|---|---|---|---|
| R1-CISCO | [R1-CISCO.txt](R1-CISCO.txt) | Captura de `show running-config` en Cisco IOSv 15.9 | Running-config completo, exceptuando banners de licencia y secretos protegidos |
| SW1-CISCO | [SW1-CISCO.txt](SW1-CISCO.txt) | Captura de `show running-config` en Cisco IOSvL2 15.2 | Running-config completo, exceptuando banners de licencia y secretos protegidos |
| FGT-01 | [FGT-01.conf](FGT-01.conf) | Respaldo original desde GUI de FortiOS 7.0.3 | **Extracto del respaldo real**: interfaces, objetos de red, VPN fase 1/2, políticas y rutas |

## Tratamiento de la información

- Se reemplazaron los **hashes de usuario**, las **claves RADIUS**, la **clave IPsec** y la **cadena de comunidad SNMP** por `<REDACTED>`. Los valores reales no están presentes en estos archivos.
- Los registros Cisco se limpiaron de códigos de control de terminal y de los avisos de licencia repetidos en los banners.
- La exportación del FortiGate original contiene más de diez mil líneas de configuración, incluidas claves privadas, certificados y perfiles predeterminados del sistema. La versión publicada incluye exclusivamente las secciones del **laboratorio** y no expone ese material sensible.
- Los archivos con marcadores de anonimización son **evidencia para auditoría académica**. **No son respaldos restaurables** sin sustituir las credenciales.
- El respaldo original sin anonimizar debe permanecer **fuera del repositorio público**.

## Controles y pruebas de interés

**R1-CISCO:** pool DHCP `LAN-PARCIAL`, `NPS-GRP` y método AAA `CON-AUX`, `line con 0` / `line aux 0`, SNMP RO con ACL, administración SSH, ACL `VPN-TRAFFIC`, mapa IPsec `VPN-FORTIGATE` en Gi0/1 y ruta hacia `10.25.14.0/24`.

**SW1-CISCO:** SVI `Vlan1`, acceso SSHv2, usuarios locales, administración VTY solo por SSH, desactivación HTTP/HTTPS, `no ip source-route`, SNMP RO con ACL y cinco puertos no utilizados configurados en `shutdown` (Gi0/3, Gi1/0, Gi1/1, Gi1/2, Gi1/3).

**FGT-01:** `port2` como WAN Cisco, `port3` como LAN remota, interfaz IPsec `VPN-R1-ITLA`, selectores de fase 2, políticas `LAN-TO-VPN` / `VPN-TO-LAN` y ruta estática por el túnel.

[Volver al README principal](../README.md)
