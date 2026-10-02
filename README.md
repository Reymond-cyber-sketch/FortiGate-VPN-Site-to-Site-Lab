# FortiGate VPN Site-to-Site Lab

Laboratorio de infraestructura de red desarrollado en **GNS3**, utilizando dos firewalls FortiGate, un MikroTik como ISP, una red de usuarios y un servidor WEB HTTPS.

El objetivo principal del laboratorio es establecer una **VPN IPsec Site-to-Site** entre los dos FortiGate y demostrar que la comunicación entre el usuario y el servidor WEB depende del túnel VPN.

---

## 🎥 Video demostrativo

📺 **Video de la práctica:**  
`[PENDIENTE - AGREGAR ENLACE DE YOUTUBE](https://www.youtube.com/watch?v=Xh6HYaw4Sq8)`

> El video demuestra el funcionamiento de la infraestructura, la VPN activa e inactiva y las pruebas de conectividad.

---

## 🎯 Objetivo del laboratorio

Implementar una infraestructura donde:

- Una red de usuarios se encuentre detrás de `FGT-USER`.
- Un servidor WEB HTTPS se encuentre detrás de `FGT-WEB`.
- Ambos FortiGate estén comunicados a través de un ISP MikroTik.
- Se configure NAT para acceso a Internet.
- Los usuarios reciban direccionamiento mediante DHCP.
- Se establezca una VPN IPsec Site-to-Site.
- El USER-PC pueda acceder al WEB-SERVER solamente cuando la VPN esté activa.
- Se realicen pruebas mediante Ping, HTTPS y Traceroute.

---

## 🖥️ Topología

La infraestructura está compuesta por:

- 2 FortiGate.
- 1 MikroTik CHR funcionando como ISP.
- 1 switch virtual.
- 1 USER-PC.
- 1 WEB-SERVER.
- 1 nodo NAT para acceso a Internet.
- GNS3 como plataforma de virtualización de red.

---

## 🌐 Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Función |
|---|---|---|---|
| FGT-USER | WAN-ISP / port1 | `198.51.100.98/30` | Conexión hacia ISP |
| ISP | ether1 | `198.51.100.97/30` | Gateway de FGT-USER |
| FGT-USER | USERS / port2 | `10.24.96.1/25` | Gateway de usuarios |
| FGT-USER | port3 | `192.168.178.2/24` | Administración |
| USER-PC | eth0 | `10.24.96.10/25` (DHCP) | Equipo de usuario |
| FGT-WEB | WAN-ISP / port1 | `203.0.113.98/30` | Conexión hacia ISP |
| ISP | ether2 | `203.0.113.97/30` | Gateway de FGT-WEB |
| FGT-WEB | WEB / port2 | `10.24.96.129/28` | Gateway WEB |
| FGT-WEB | port3 | `192.168.178.3/24` | Administración |
| WEB-SERVER | eth0 | `10.24.96.130/28` | Servidor HTTPS |

---

## 👤 Red de usuarios

La red de usuarios utiliza:

```text
Red: 10.24.96.0/25
Gateway: 10.24.96.1
VLAN: 10
```

El FortiGate entrega las direcciones IP mediante DHCP.

Rango configurado:

```text
10.24.96.10 - 10.24.96.120
```

DNS:

```text
8.8.8.8
1.1.1.1
```

---

## 🌐 Red WEB

La red del servidor utiliza:

```text
Red: 10.24.96.128/28
Gateway: 10.24.96.129
```

El servidor tiene una dirección IP estática:

```text
IP: 10.24.96.130
Máscara: 255.255.255.240
Gateway: 10.24.96.129
```

El servicio WEB utiliza:

```text
HTTPS / TCP 443
```

---

## 🔥 NAT

Se configuraron políticas de NAT para permitir que ambas redes puedan tener acceso a Internet.

### FGT-USER

```text
USERS → WAN-ISP
NAT: Enabled
```

### FGT-WEB

```text
WEB → WAN-ISP
NAT: Enabled
```

Se comprobó el acceso a Internet mediante:

```text
ping 8.8.8.8
ping google.com
```

---

## 🔐 VPN IPsec Site-to-Site

Se configuró una VPN entre los dos FortiGate.

### FGT-USER

```text
Nombre: VPN-TO-WEB
WAN Local: 198.51.100.98
Remote Gateway: 203.0.113.98

Red Local:
10.24.96.0/25

Red Remota:
10.24.96.128/28
```

### FGT-WEB

```text
Nombre: VPN-TO-USER
WAN Local: 203.0.113.98
Remote Gateway: 198.51.100.98

Red Local:
10.24.96.128/28

Red Remota:
10.24.96.0/25
```

La VPN utiliza IKEv2 y autenticación mediante una clave precompartida.

> La clave PSK real fue eliminada de los archivos publicados en el repositorio por motivos de seguridad.

---

## ✅ VPN activa

La VPN fue comprobada desde el FortiGate y aparece en estado:

```text
VPN-TO-WEB → Up
```

También se verificó mediante CLI:

```text
get vpn ipsec tunnel summary
```

Resultado:

```text
selectors(total,up): 1/1
```

---

## 🧪 Prueba con VPN activa

Desde el USER-PC se realizaron las siguientes pruebas:

```bash
ping -c 4 10.24.96.130
```

Resultado:

```text
4 packets transmitted
4 packets received
0% packet loss
```

También se comprobó HTTPS:

```bash
curl -k https://10.24.96.130/
```

El servidor respondió correctamente:

```text
WEB SERVER - Laboratorio FortiGate
Servidor HTTPS funcionando correctamente.
IP: 10.24.96.130
```

Finalmente se utilizó traceroute:

```bash
traceroute 10.24.96.130
```

La comunicación llegó correctamente al servidor remoto a través de la infraestructura.

---

## ❌ Prueba con VPN inactiva

Para demostrar que la comunicación depende del túnel IPsec, se deshabilitó temporalmente la interfaz VPN en FGT-USER.

Comando utilizado:

```text
config system interface
edit "VPN-TO-WEB"
set status down
next
end
```

Se verificó el estado utilizando:

```text
diagnose vpn tunnel list name VPN-TO-WEB
```

El túnel mostró:

```text
status=down
accept_traffic=0
```

Posteriormente se realizaron nuevamente las pruebas desde el USER-PC:

```bash
ping -c 4 10.24.96.130
```

```bash
curl --connect-timeout 5 -k https://10.24.96.130/
```

La comunicación con el servidor dejó de funcionar.

Esto demuestra que el tráfico entre la red USERS y la red WEB depende de la VPN Site-to-Site.

---

## 🔄 Restauración de la VPN

Luego de finalizar la prueba, el túnel se habilitó nuevamente:

```text
config system interface
edit "VPN-TO-WEB"
set status up
next
end
```

Se comprobó nuevamente:

```text
get vpn ipsec tunnel summary
```

Resultado:

```text
selectors(total,up): 1/1
```

La comunicación con `10.24.96.130` volvió a funcionar correctamente.

---

## 📊 Resultado final

| Prueba | VPN UP | VPN DOWN |
|---|---|---|
| USER-PC → WEB-SERVER | ✅ Funciona | ❌ No funciona |
| Ping `10.24.96.130` | ✅ | ❌ |
| HTTPS `443` | ✅ | ❌ |
| Traceroute | ✅ | ❌ |
| Internet mediante NAT | ✅ | ✅ |

El comportamiento obtenido cumple con el objetivo de que la comunicación entre las dos redes privadas se realice mediante el túnel VPN.

---

## 📂 Archivos de configuración

Las configuraciones utilizadas durante el laboratorio se encuentran disponibles en:

### FortiGate USER

[`configs/FGT-USER-running-config.txt`](configs/FGT-USER-running-config.txt)

### FortiGate WEB

[`configs/FGT-WEB-running-config.txt`](configs/FGT-WEB-running-config.txt)

### MikroTik ISP

[`configs/ISP-MikroTik-running-config.txt`](configs/ISP-MikroTik-running-config.txt)

---

## 🧰 Comandos de verificación

Los comandos utilizados para comprobar la infraestructura se encuentran en:

[`verification-command.txt`](verification-command.txt)

Entre ellos:

```bash
ping -c 4 10.24.96.130
curl -k https://10.24.96.130/
traceroute 10.24.96.130
```

Para comprobar la VPN en FortiGate:

```text
get vpn ipsec tunnel summary
diagnose vpn tunnel list name VPN-TO-WEB
```

---

## 📁 Estructura del repositorio

```text
FortiGate-VPN-Site-to-Site-Lab/
│
├── README.md
├── verification-command.txt
│
├── configs/
│   ├── FGT-USER-running-config.txt
│   ├── FGT-WEB-running-config.txt
│   └── ISP-MikroTik-running-config.txt
│
└── images/
    ├── topology.png
    ├── fgt-user-interfaces.png
    ├── fgt-web-interfaces.png
    ├── vpn-up.png
    ├── vpn-test-success.png
    └── vpn-test-failed.png
```

---

## 📝 Conclusión

En este laboratorio se implementó correctamente una infraestructura de comunicación segura utilizando una VPN IPsec Site-to-Site entre dos FortiGate.

Se configuraron redes independientes para usuarios y servidores, direccionamiento IP, DHCP, NAT, rutas estáticas y políticas de firewall.

Las pruebas demostraron que el USER-PC puede acceder al WEB-SERVER mediante HTTPS cuando el túnel VPN está activo. Al desactivar la VPN, la comunicación entre ambas redes deja de funcionar, demostrando que el acceso entre las dos infraestructuras depende del túnel IPsec configurado.

Finalmente, la VPN fue restaurada y se comprobó nuevamente la comunicación entre ambos extremos.

---

## 👨‍💻 Autor

**Reymond Daniel Guerrero Cruz**  
Matrícula: **2024-0963**

Laboratorio realizado en **GNS3** utilizando FortiGate VM y MikroTik CHR.
