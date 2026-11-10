# Registro de verificaciones de la infraestructura

**Fecha:** 10 de noviembre de 2026  
**Plataforma:** Proxmox VE / GNS3 / Windows Server / Zabbix / FortiGate.

## 1. SNMP en R1 y SW1

Desde ZABBIX-SRV, las consultas a `1.3.6.1.2.1.1.1.0` fueron exitosas:

```bash
snmpwalk -v2c -c 'COMUNIDAD_PRIVADA' 10.25.13.1 1.3.6.1.2.1.1.1.0
snmpwalk -v2c -c 'COMUNIDAD_PRIVADA' 10.25.13.2 1.3.6.1.2.1.1.1.0
```

En Zabbix, los hosts `R1-CISCO` y `SW1-CISCO` mostraron el indicador SNMP verde. La comunidad no se publica en el repositorio.

**Evidencias:** terminal SNMP y vista Zabbix > Recopilación de datos > Equipos / Monitorización > Últimos datos.

## 2. AAA mediante Active Directory y NPS

```text
R1-CISCO# show running-config | include ^aaa
R1-CISCO# show running-config | section line
R1-CISCO# show aaa servers
R1-CISCO# show privilege
R1-CISCO# show users
```

Comprobaciones obtenidas: la prueba de autenticación RADIUS fue aceptada; la consola mostró `cisco.admin`, privilegio 15; una sesión SSH simultánea mostró `admin1331`. Las líneas de consola y auxiliar tienen `authorization exec CON-AUX` y `login authentication CON-AUX`. La VTY conserva el método local por defecto.

En Windows, la consulta de eventos permite comprobar las autorizaciones de NPS, después de las pruebas:

```powershell
Get-WinEvent -FilterHashtable @{ LogName='Security'; Id=6272; StartTime=(Get-Date).AddMinutes(-30) } -MaxEvents 5 |
Format-List TimeCreated,Id,Message
```

**Evidencias:** prueba RADIUS aceptada (sin contraseña visible), línea AAA, consola con `show privilege` y `show users`, y evento NPS 6272.

## 3. DHCP y conectividad LAN

```text
PC1> ip dhcp
PC1> show ip
PC1> ping 10.25.13.1
PC1> ping 10.25.13.2

R1-CISCO# show ip dhcp binding
R1-CISCO# show ip dhcp pool
```

PC1 recibió `10.25.13.150/24` desde `10.25.13.1` y respondió a los pings de la LAN.

## 4. Hardening y SSH del switch

```text
SW1-CISCO# show ip ssh
SW1-CISCO# show running-config | section line vty
SW1-CISCO# show running-config | include ip http|ip source-route
SW1-CISCO# show interfaces status
```

Comprobaciones confirmadas: SSHv2, RSA 2048 bits, VTY únicamente SSH, cierre por inactividad a 5 minutos, no HTTP/HTTPS de IOS, no IP source routing. Se logró acceso por SSH desde `10.25.13.105`. Los puertos TCP/23 de R1 y SW1 rechazaron conexiones.

Pruebas desde Zabbix:

```bash
nc -vz -w 3 10.25.13.1 23
nc -vz -w 3 10.25.13.2 23
```

**Evidencias:** `show ip ssh`, `show users`, salida VTY, puertos administrativamente apagados cuando se comprueben, y rechazo Telnet.

## 5. VPN IPsec y pérdida de comunicación

```text
R1-CISCO# show crypto isakmp sa
R1-CISCO# show crypto ipsec sa
R1-CISCO# show crypto map
R1-CISCO# show access-lists VPN-TRAFFIC

PC1> ping 10.25.14.10
PC2> ping 10.25.14.1
```

Se obtuvo `QM_IDLE`, SPI saliente asignado, 4 paquetes cifrados y 4 descifrados, ACL con coincidencias y ping inter-LAN exitoso. Desde la GUI de FortiGate se realizó una desactivación temporal del túnel para comprobar la pérdida de comunicación, seguida de restauración.

**Evidencias:** estado del túnel en FortiGate, salida `show crypto ipsec sa`, ping exitoso, túnel desactivado y ping fallido.

## 6. Exportación de configuraciones individuales

En el router y en el switch, usar `terminal length 0` y luego `show running-config` para copiar la configuración completa a un archivo individual. También exportar el backup de configuración de FortiGate desde su GUI, en lugar de reconstruirlo de memoria.

Antes de publicar los archivos, deben redactarse únicamente los secretos: contraseñas `secret`, claves de RADIUS, clave precompartida IPsec, cadenas de comunidad SNMP y otros datos de autenticación. Conservar interfaces, redes, políticas y comandos de seguridad para permitir que la documentación sea auditada.

## 7. Archivos de evidencia

Los nombres recomendados para organizar las capturas son:

| Carpeta | Contenido |
| --- | --- |
| `evidencias/01-snmp/` | SNMP en Zabbix y consultas al router/switch |
| `evidencias/02-aaa/` | NPS, AAA consola y configuración AUX |
| `evidencias/03-dhcp/` | Obtención de DHCP de PC1 |
| `evidencias/04-hardening/` | Controles de seguridad del switch y puertos |
| `evidencias/05-vpn/` | VPN activa, IPsec y prueba desactivada |
| `evidencias/06-ssh/` | Acceso remoto y bloqueo de Telnet |
| `evidencias/07-direccionamiento/` | Interfaces, diagrama y conectividad |


## 8. Archivos de configuración publicados y verificaciones adicionales

Las configuraciones se extrajeron de los equipos y se publicaron en el repositorio después de anonimizar credenciales:

- [R1-CISCO — running-config real](../configuraciones/R1-CISCO.txt)
- [SW1-CISCO — running-config real](../configuraciones/SW1-CISCO.txt)
- [FortiGate — extracto anonimizado del respaldo de configuración GUI](../configuraciones/FGT-01.conf)

El running-config de SW1 demuestra cinco puertos apagados administrativamente: `GigabitEthernet0/3` y `GigabitEthernet1/0` a `GigabitEthernet1/3`. Todos incluyen `description UNUSED-PORT-DISABLED` y el comando `shutdown`. Este hallazgo amplía los controles documentados de hardening.

La exportación de FortiGate confirma `VPN-R1-ITLA`, `VPN-R1-ITLA-P2`, ambas políticas de firewall y la ruta estática `10.25.13.0/24` a través de la interfaz del túnel. El extracto público omitió certificados, llaves privadas y configuraciones predeterminadas ajenas a esta infraestructura.
