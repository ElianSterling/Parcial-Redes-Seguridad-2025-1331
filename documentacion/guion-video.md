# Guion definitivo del video — Segundo Parcial de Seguridad de Redes

**Estudiante:** Elian Sterling Severino Mejía  
**Matrícula:** 2025-1331  
**Duración objetivo:** 09:25 (máximo 10:00)  
**Video:** rostro y voz del estudiante, fecha y hora visibles. Subir únicamente a YouTube y colocar el enlace al principio del README.

> Recomendación: dejar abiertas en ventanas o pestañas GNS3, PC1, R1, SW1, FortiGate GUI, Zabbix, Windows NPS y GitHub. Mostrar únicamente pruebas reales; ocultar contraseñas, secretos RADIUS, comunidad SNMP y clave precompartida.

## 00:00–00:20 | Presentación

**Pantalla:** webcam y terminal con `date`.

**Narración:** «Buenos días. Mi nombre es Elian Sterling Severino Mejía, matrícula 2025-1331. Presentaré la demostración práctica de mi Segundo Parcial de Seguridad de Redes, desarrollado en GNS3 y Proxmox con Cisco, Zabbix, Windows Server, NPS y FortiGate».

## 00:20–00:55 | Topología

**Pantalla:** GNS3, diagrama completo.

**Narración:** «La sede Cisco utiliza 10.25.13.0/24, con R1 en 10.25.13.1 y SW1 en 10.25.13.2. El servidor NPS tiene 10.25.13.10. La sede FortiGate utiliza 10.25.14.0/24. Los extremos WAN 172.31.13.1 y 172.31.13.2 se comunican mediante una VPN IPsec».

## 00:55–01:45 | DHCP y conectividad

**Pantalla:** PC1 y R1.

```text
PC1> show ip
PC1> ping 10.25.13.1
PC1> ping 10.25.13.2
R1-CISCO# show ip dhcp binding
```

**Narración:** «El router entrega direcciones por DHCP. PC1 recibió 10.25.13.150/24 con gateway 10.25.13.1. Confirmo conectividad con el router y el switch».

## 01:45–02:40 | Zabbix y SNMP

**Pantalla:** Zabbix, hosts R1/SW1, disponibilidad verde, Monitoring → Latest data.

**Narración:** «Configuré SNMPv2c de solo lectura, restringido mediante ACL a la IP del servidor Zabbix. Ambos dispositivos aparecen disponibles y muestran métricas recopiladas por SNMP».

## 02:40–04:20 | AAA con NPS, consola y AUX

**Pantalla:** NPS / AD (grupo GG-CISCO-ADMINS y política CISCO-AAA-ADMINS), R1-CISCO.

```cisco
show running-config | section line con
show running-config | section line aux
show running-config | section line vty
show aaa servers
show privilege
show users
```

**Narración:** «Active Directory contiene el grupo autorizado de administradores. NPS permite autenticación RADIUS y asigna privilegio administrativo 15. En el router configuré autenticación y autorización para consola y AUX con el método CON-AUX. VTY mantiene el acceso local por SSH. Aquí se observa la sesión cisco.admin por consola, privilegio 15, y otra sesión administrativa local por SSH».

**Nota crítica:** la configuración de AUX NO equivale a demostrar un inicio de sesión real por AUX. Si no se puede establecer una sesión física/virtual AUX, presentar la configuración con transparencia, sin afirmar que su inicio de sesión se probó.

## 04:20–05:50 | Hardening Cisco alineado con CIS

**Pantalla:** SW1-CISCO.

```cisco
show ip ssh
show running-config | section line vty
show running-config | include ^no ip http
show running-config | include ^no ip source-route
show access-lists ZABBIX-SNMP-ACL
show interfaces status
```

**Narración:** «Implementé SSHv2 con RSA de 2048 bits, permití únicamente SSH en VTY para eliminar Telnet, deshabilité los servidores HTTP/HTTPS, desactivé IP source routing y configuré cierre de sesión tras cinco minutos de inactividad. Como control adicional, SNMP es de solo lectura y la ACL restringe las consultas. Estos controles se comprueban en las salidas del dispositivo».

**Precaución:** no afirmar que los puertos no utilizados están apagados sin verificar `administratively down` o `disabled`.

## 05:50–06:35 | SSH y Telnet

**Pantalla:** terminal Linux con sesión SSH administrativa en SW1 o R1 y verificación de puertos.

```bash
nc -vz -w 3 10.25.13.1 23
nc -vz -w 3 10.25.13.2 23
```

**Narración:** «El acceso remoto administrativo funciona por SSH con una cuenta local. En ambos dispositivos Telnet está bloqueado, tal como confirma el rechazo de conexiones al puerto 23».

## 06:35–08:55 | VPN IPsec Cisco–FortiGate

**Pantalla:** FortiGate GUI (VPN activa), PC1, R1.

```text
PC1> ping 10.25.14.10
```

```cisco
show crypto isakmp sa
show crypto ipsec sa
```

**Narración:** «La VPN IPsec sitio a sitio protege las redes 10.25.13.0/24 y 10.25.14.0/24. El router muestra QM_IDLE; los contadores de cifrado y descifrado incrementan y PC1 alcanza PC2 a través del túnel».

**Acción:** desde la GUI de FortiGate deshabilitar el túnel (no solo desconectarlo, porque podría renegociarse automáticamente), repetir ping PC1→PC2 y demostrar timeout, luego habilitar nuevamente.

**Narración:** «Ahora desactivo la VPN desde FortiGate; sin el túnel, PC1 pierde comunicación con PC2. La habilito nuevamente y recupero el servicio».

## 08:55–09:25 | GitHub y cierre

**Pantalla:** README con portada, diagrama, galería, carpetas de configuración exportada.

**Narración:** «La documentación, los diagramas, las imágenes de evidencia y las configuraciones de cada equipo están organizadas en el repositorio público. Esto concluye la demostración del Segundo Parcial de Seguridad de Redes».

Si los running-configs todavía no están subidos, **no afirmar que sí lo están**. Mostrar lo que esté realmente publicado.

## Lista antes de grabar

- [ ] Webcam y micrófono activos; fecha y hora actuales visibles
- [ ] GNS3, Zabbix, PC1, Cisco, FortiGate y GitHub ya abiertos
- [ ] VPN activa y pings preparados
- [ ] No mostrar contraseñas ni claves RADIUS/IPsec/SNMP
- [ ] Cinco controles de hardening identificados y comprobables
- [ ] Diferenciar configuración AUX de prueba real AUX
- [ ] Running-configs individuales exportados y secretos anonimizados
- [ ] Grabar 09:25 o menos; subir únicamente a YouTube
- [ ] Pegar URL de YouTube al **principio del README**
- [ ] Crear TXT con exactamente dos líneas: URL GitHub y URL YouTube
