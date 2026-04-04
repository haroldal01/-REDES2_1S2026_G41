# Práctica 2 — La Facultad de Ingeniería y su problema de conectividad en la biblioteca
**Redes de Computadoras 2 | ECYS, FIUSAC**
**Grupo 41**

---

## Integrantes

| Nombre | Carnet | Rol |
|--------|--------|-----|
| Madeline Fabiola Prado Reyes  | 202100039 | 1 |
| Harold Alejandro Sánchez Hernández | 202200100 | 2 |
| Engel Emilio Coc Raxjal | 202200314 | 3 |

---

## 1. Topología propuesta

La red está compuesta por tres edificios interconectados:

- **Piso 1 (Biblioteca de Ingeniería):** Red cableada con VLANs, dos routers con HSRP y un switch de acceso.
- **Piso 2:** Red inalámbrica con dos Wireless Routers conectados a MLS0.
- **Piso 3:** Red inalámbrica con dos Wireless Routers conectados a MLS3.
- **Data Center / Biblioteca Central:** Servidores Web y DHCP, dos routers con HSRP, conectados a MLS2.

Los cuatro edificios se interconectan mediante **enlaces LACP de 4 interfaces** entre MLS4 (Piso 1) y MLS2, MLS3, MLS0. El protocolo de enrutamiento utilizado es **EIGRP** (grupo impar).

### Dispositivos por zona

| Zona | Dispositivos |
|------|-------------|
| Piso 1 | MLS4, Router1, Router0, Switch0, PC1, Laptop1 |
| Piso 2 | MLS0, Wireless Router1, Wireless Router0, PC3 |
| Piso 3 | MLS3, Wireless Router2, Wireless Router0(1), PC2 |
| Data Center | MLS2, Router2, Router3, Switch3, ServerWeb, ServerDHCP |

---

## 2. Subnetting

> **Nota:** Grupo 41 → 4+1 = **X = 5**

### 2.1 VLSM — Piso 1 (192.198.15.0/24)

Se aplica VLSM porque los requerimientos de hosts son distintos.

| VLAN | Nombre | Red | Máscara | Hosts útiles | Rango usable | Broadcast |
|------|--------|-----|---------|--------------|--------------|-----------|
| 25 | ESTUDIANTES | 192.198.15.0 | /26 (255.255.255.192) | 62 | .1 — .62 | .63 |
| 15 | ADMIN | 192.198.15.64 | /28 (255.255.255.240) | 14 | .65 — .78 | .79 |

### 2.2 FLSM — Biblioteca Central (192.198.100.0/24)

Se divide en 2 partes iguales (/25 cada una).

| VLAN | Nombre | Red | Máscara | Hosts útiles | Rango usable | Broadcast |
|------|--------|-----|---------|--------------|--------------|-----------|
| 35 | WEB_SERVERS | 192.198.100.0 | /25 (255.255.255.128) | 126 | .1 — .126 | .127 |
| 45 | DHCP_SERVERS | 192.198.100.128 | /25 (255.255.255.128) | 126 | .129 — .254 | .255 |

### 2.3 FLSM — Piso 2 (192.198.25.0/24)

| Red | Máscara | Uso | Rango usable |
|-----|---------|-----|--------------|
| 192.198.25.0 | /25 | WLAN1 (80 hosts) | .1 — .126 |
| 192.198.25.128 | /25 | WLAN2 (80 hosts) | .129 — .254 |

### 2.4 FLSM — Piso 3 (192.198.35.0/24)

| Red | Máscara | Uso | Rango usable |
|-----|---------|-----|--------------|
| 192.198.35.0 | /25 | WLAN1 (80 hosts) | .1 — .126 |
| 192.198.35.128 | /25 | WLAN2 (80 hosts) | .129 — .254 |

### 2.5 VLSM — Red de enrutamiento (10.2.5.0/24)

Se usan subredes /30 para enlaces punto a punto.

| Enlace | Red | Máscara | IP lado A | IP lado B |
|--------|-----|---------|-----------|-----------|
| MLS4 ↔ MLS2 (Po1 LACP) | 10.2.5.0 | /30 | 10.2.5.1 (MLS4) | 10.2.5.2 (MLS2) |
| MLS4 ↔ MLS3 (Po2 LACP) | 10.2.5.4 | /30 | 10.2.5.5 (MLS4) | 10.2.5.6 (MLS3) |
| MLS4 ↔ MLS0 (Po3 LACP) | 10.2.5.8 | /30 | 10.2.5.9 (MLS4) | 10.2.5.10 (MLS0) |
| MLS4 ↔ Router1 | 10.2.5.12 | /30 | 10.2.5.13 (MLS4) | 10.2.5.14 (R1) |
| MLS4 ↔ Router0 | 10.2.5.16 | /30 | 10.2.5.17 (MLS4) | 10.2.5.18 (R0) |
| MLS2 ↔ Router2 | 10.2.5.20 | /30 | 10.2.5.21 (MLS2) | 10.2.5.22 (R2) |
| MLS2 ↔ Router3 | 10.2.5.24 | /30 | 10.2.5.25 (MLS2) | 10.2.5.26 (R3) |

---

## 3. VLANs

| ID | Nombre | Zona |
|----|--------|------|
| 15 | ADMIN | Piso 1 |
| 25 | ESTUDIANTES | Piso 1 |
| 35 | WEB_SERVERS | Data Center |
| 45 | DHCP_SERVERS | Data Center |

---

## 4. Configuración Persona 1 — Red cableada, VLANs y capa 3

### 4.1 MLS4

```
enable
configure terminal

ip routing

vlan 15
 name ADMIN
vlan 25
 name ESTUDIANTES
vlan 35
 name WEB_SERVERS
vlan 45
 name DHCP_SERVERS
exit

interface range Fa0/1-3, Fa0/24
 no switchport
 channel-group 1 mode active
 no shutdown

interface range Fa0/6-8, Fa0/23
 no switchport
 channel-group 2 mode active
 no shutdown

interface range Fa0/9-11, Fa0/22
 no switchport
 channel-group 3 mode active
 no shutdown

interface Port-channel1
 no switchport
 ip address 10.2.5.1 255.255.255.252
 no shutdown

interface Port-channel2
 no switchport
 ip address 10.2.5.5 255.255.255.252
 no shutdown

interface Port-channel3
 no switchport
 ip address 10.2.5.9 255.255.255.252
 no shutdown

interface Fa0/4
 no switchport
 ip address 10.2.5.13 255.255.255.252
 no shutdown

interface Fa0/5
 no switchport
 ip address 10.2.5.17 255.255.255.252
 no shutdown

router eigrp 100
 network 10.2.5.0 0.0.0.3
 network 10.2.5.4 0.0.0.3
 network 10.2.5.8 0.0.0.3
 network 10.2.5.12 0.0.0.3
 network 10.2.5.16 0.0.0.3
 network 192.198.15.0 0.0.0.63
 network 192.198.15.64 0.0.0.15
 network 192.198.100.0 0.0.0.127
 network 192.198.100.128 0.0.0.127
 no auto-summary

exit
write memory
```

### 4.2 Router1 (HSRP Active)

```
enable
configure terminal

interface GigabitEthernet0/0
 ip address 10.2.5.14 255.255.255.252
 no shutdown

interface GigabitEthernet0/1
 no shutdown

interface GigabitEthernet0/1.15
 encapsulation dot1Q 15
 ip address 192.198.15.66 255.255.255.240
 standby 15 ip 192.198.15.78
 standby 15 priority 110
 standby 15 preempt
 no shutdown

interface GigabitEthernet0/1.25
 encapsulation dot1Q 25
 ip address 192.198.15.2 255.255.255.192
 standby 25 ip 192.198.15.62
 standby 25 priority 110
 standby 25 preempt
 no shutdown

router eigrp 100
 network 10.2.5.12 0.0.0.3
 network 192.198.15.0 0.0.0.63
 network 192.198.15.64 0.0.0.15
 no auto-summary

exit
write memory
```

### 4.3 Router0 (HSRP Standby)

```
enable
configure terminal

interface GigabitEthernet0/0
 ip address 10.2.5.18 255.255.255.252
 no shutdown

interface GigabitEthernet0/1
 no shutdown

interface GigabitEthernet0/1.15
 encapsulation dot1Q 15
 ip address 192.198.15.67 255.255.255.240
 standby 15 ip 192.198.15.78
 standby 15 priority 90
 no shutdown

interface GigabitEthernet0/1.25
 encapsulation dot1Q 25
 ip address 192.198.15.3 255.255.255.192
 standby 25 ip 192.198.15.62
 standby 25 priority 90
 no shutdown

router eigrp 100
 network 10.2.5.16 0.0.0.3
 network 192.198.15.0 0.0.0.63
 network 192.198.15.64 0.0.0.15
 no auto-summary

exit
write memory
```

### 4.4 Switch0

```
enable
configure terminal

vlan 15
 name ADMIN
vlan 25
 name ESTUDIANTES
vlan 35
 name WEB_SERVERS
vlan 45
 name DHCP_SERVERS
exit

interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 15,25
 no shutdown

interface GigabitEthernet0/2
 switchport mode trunk
 switchport trunk allowed vlan 15,25
 no shutdown

interface FastEthernet0/1
 switchport mode access
 switchport access vlan 25
 no shutdown

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 15
 no shutdown

exit
write memory
```

### 4.5 MLS2 (Data Center)

```
enable
configure terminal

ip routing

vlan 15
 name ADMIN
vlan 25
 name ESTUDIANTES
vlan 35
 name WEB_SERVERS
vlan 45
 name DHCP_SERVERS
exit

interface range Fa0/1-3, Fa0/24
 no switchport
 channel-group 1 mode active
 no shutdown

interface Port-channel1
 no switchport
 ip address 10.2.5.2 255.255.255.252
 no shutdown

router eigrp 100
 network 10.2.5.0 0.0.0.3
 network 192.198.100.0 0.0.0.127
 network 192.198.100.128 0.0.0.127
 no auto-summary

exit
write memory
```

### 4.6 IPs virtuales HSRP — Piso 1

| VLAN | IP Virtual HSRP | Uso |
|------|----------------|-----|
| 15 ADMIN | 192.198.15.78 | Gateway hosts admin |
| 25 ESTUDIANTES | 192.198.15.62 | Gateway hosts estudiantes |

---

## 5. Configuración Persona 2 — Inalámbrico, DHCP y LACP

> ** Persona 2 **

---
##  Alcance de lo realizado

Se trabajó la parte correspondiente a **LACP, DHCP e inalámbrico**. Para el grupo 41, se usa **X = 5**, por lo que el direccionamiento aplicado quedó sobre las redes `192.198.15.0/24`, `192.198.25.0/24`, `192.198.35.0/24`, `192.198.100.0/24` y `10.2.5.0/24`. El protocolo de capa 3 utilizado es **EIGRP**, porque el grupo es impar. El enunciado también exige LACP entre edificios, DHCP dinámico para dispositivos finales, y configuración WiFi con SSID, broadcast y WPA2 según piso.  

## 2. Subnetting usado y que debe respetarse

En el README ya quedó documentado este esquema:

* **Piso 1**

  * ESTUDIANTES: `192.198.15.0/26`
  * ADMIN: `192.198.15.64/28`
* **Piso 2**

  * WLAN1: `192.198.25.0/25`
  * WLAN2: `192.198.25.128/25`
* **Piso 3**

  * WLAN1: `192.198.35.0/25`
  * WLAN2: `192.198.35.128/25`
* **Biblioteca central**

  * WEB_SERVERS: `192.198.100.0/25`
  * DHCP_SERVERS: `192.198.100.128/25`
* **Red de enrutamiento**

  * `10.2.5.0/24` subdividida en enlaces /30.  

## 3. LACP implementado

Se dejaron operativos los enlaces agregados exigidos por la práctica, usando **4 interfaces por bundle**:

* **Po1:** `MLS4 ↔ MLS2`
* **Po2:** `MLS4 ↔ MLS3`
* **Po3:** `MLS4 ↔ MLS0`  

### 3.1 Po2 — MLS4 ↔ MLS3

Red usada: `10.2.5.4/30`

* MLS4: `10.2.5.5`
* MLS3: `10.2.5.6`

Puertos:

* MLS4: `Fa0/6`, `Fa0/7`, `Fa0/8`, `Fa0/23`
* MLS3: `Fa0/6`, `Fa0/7`, `Fa0/8`, `Fa0/23`

### 3.2 Po3 — MLS4 ↔ MLS0

Red usada: `10.2.5.8/30`

* MLS4: `10.2.5.9`
* MLS0: `10.2.5.10`

Puertos:

* MLS4: `Fa0/9`, `Fa0/10`, `Fa0/11`, `Fa0/22`
* MLS0: `Fa0/9`, `Fa0/10`, `Fa0/11`, `Fa0/22`

### 3.3 Comandos base usados en MLS3 y MLS0

```bash
enable
configure terminal

ip routing

interface range fa0/x-y, fa0/z
 no switchport
 channel-group <n> mode active
 no shutdown
exit

interface port-channel <n>
 no switchport
 ip address <IP> <MASCARA>
 no shutdown
exit

router eigrp 100
 network <RED> <WILDCARD>
 no auto-summary
exit

end
write memory
```

### 3.4 Verificación usada

```bash
show etherchannel summary
show ip interface brief
show ip eigrp neighbors
show ip route
```

---

## 4. DHCP central para red cableada

El enunciado indica que los servicios DHCP y DNS para la red cableada permanecen en la biblioteca central, y que todos los dispositivos finales deben recibir IP dinámicamente. También exige que el servidor DHCP entregue direcciones correctas para cada VLAN.  

### 4.1 ServerDHCP

Se dejó configurado con IP fija en la red de **DHCP_SERVERS**:

* **IP:** `192.198.100.130`
* **Máscara:** `255.255.255.128`
* **Gateway:** `192.198.100.129`

### 4.2 Relay DHCP en piso 1

Se agregaron `ip helper-address 192.198.100.130` en:

* `Router1 G0/1.15`
* `Router1 G0/1.25`
* `Router0 G0/1.15`
* `Router0 G0/1.25`

Esto permite que las solicitudes DHCP de **ADMIN** y **ESTUDIANTES** lleguen al servidor central.

### 4.3 Pools creados para piso 1

**ESTUDIANTES**

* Red: `192.198.15.0/26`
* Gateway: `192.198.15.62`
* DNS: `192.198.100.10`
* Inicio: `192.198.15.4`
* Máscara: `255.255.255.192`

**ADMIN**

* Red: `192.198.15.64/28`
* Gateway: `192.198.15.78`
* DNS: `192.198.100.10`
* Inicio: `192.198.15.68`
* Máscara: `255.255.255.240`

Las IP virtuales de HSRP de piso 1 ya estaban definidas en el README como:

* ADMIN: `192.198.15.78`
* ESTUDIANTES: `192.198.15.62` 

### 4.4 Validación realizada

Se probó un host nuevo en cada VLAN cableada:

* ESTUDIANTES recibió IP del rango correcto
* ADMIN recibió IP del rango correcto
* ambos tomaron DHCP desde `192.198.100.130`

---

## 5. Biblioteca central y HSRP necesarios para que el DHCP funcionara

Aunque originalmente esto estaba en la sección de Persona 3, fue necesario dejarlo operativo para que el datacenter y el DHCP funcionaran.

El PDF exige **HSRP en piso 1 y biblioteca**. 

### 5.1 Switch3

Se dejó así:

* `Fa0/1` → access VLAN 35 (**WEB_SERVERS**)
* `Fa0/2` → access VLAN 45 (**DHCP_SERVERS**)
* `Fa0/3` → trunk hacia Router2, VLANs permitidas `35,45`
* `Fa0/4` → trunk hacia Router3, VLANs permitidas `35,45`

### 5.2 Enlaces MLS2 ↔ Router2 / Router3

Según el README:

* `MLS2 ↔ Router2` = `10.2.5.20/30`

  * MLS2: `10.2.5.21`
  * Router2: `10.2.5.22`
* `MLS2 ↔ Router3` = `10.2.5.24/30`

  * MLS2: `10.2.5.25`
  * Router3: `10.2.5.26` 

### 5.3 HSRP en biblioteca

Se dejó así:

**VLAN 35 — WEB_SERVERS**

* VIP: `192.198.100.1`
* Router2: `192.198.100.2`
* Router3: `192.198.100.3`

**VLAN 45 — DHCP_SERVERS**

* VIP: `192.198.100.129`
* Router2: `192.198.100.131`
* Router3: `192.198.100.132`

Resultado:

* Router2 quedó **Active**
* Router3 quedó **Standby**

Esto permitió que el `ServerDHCP` alcanzara su gateway y que el DHCP funcionara correctamente.

---

## 6. Configuración inalámbrica — Piso 2

El PDF define para piso 2:

* SSID: `PISO2_GX_R1`, `PISO2_GX_R2`
* Broadcast: **No**
* Seguridad: **WPA2 Personal**
* Contraseña red: `GX_PISO2`
* Contraseña router: `GrupoX_P2` 

Para **Grupo 41 (X = 5)** se dejó así:

* `PISO2_G5_R1`
* `PISO2_G5_R2`
* clave WiFi: `G5_PISO2`
* clave router prevista: `Grupo5_P2` 

### 6.1 Observación técnica importante

Los equipos usados fueron **WRT300N**, y en Packet Tracer ese modelo **no tiene CLI**, solo pestañas **Config/GUI**. Por eso la parte inalámbrica se dejó funcional usando la interfaz del equipo. A nivel técnico, quedaron operando como **AP**, no como routers con NAT, para respetar el subnetting del README.

### 6.2 Piso 2 — AP 1

Conectado a:

* `MLS0 Fa0/1 ↔ Ethernet 1` del WRT

Configuración:

* IP LAN: `192.198.25.2`
* Máscara: `255.255.255.128`
* DHCP: **Disabled**
* SSID: `PISO2_G5_R1`
* Broadcast: **Disabled**
* Seguridad: `WPA2 Personal`
* Clave: `G5_PISO2`

Gateway de la red:

* `MLS0 Fa0/1 = 192.198.25.1`

### 6.3 Piso 2 — AP 2

Conectado a:

* `MLS0 Fa0/2 ↔ Ethernet 1` del WRT

Configuración:

* IP LAN: `192.198.25.130`
* Máscara: `255.255.255.128`
* DHCP: **Disabled**
* SSID: `PISO2_G5_R2`
* Broadcast: **Disabled**
* Seguridad: `WPA2 Personal`
* Clave: `G5_PISO2`

Gateway de la red:

* `MLS0 Fa0/2 = 192.198.25.129`

### 6.4 Relay DHCP en MLS0

Se agregaron:

* `ip helper-address 192.198.100.130` en `Fa0/1`
* `ip helper-address 192.198.100.130` en `Fa0/2`

### 6.5 Pools DHCP creados para piso 2

**PISO2_WLAN1**

* Gateway: `192.198.25.1`
* DNS: `192.198.100.10`
* Inicio: `192.198.25.3`
* Máscara: `255.255.255.128`

**PISO2_WLAN2**

* Gateway: `192.198.25.129`
* DNS: `192.198.100.10`
* Inicio: `192.198.25.131`
* Máscara: `255.255.255.128`

### 6.6 Validación realizada

Se probó un cliente nuevo en cada SSID:

* `PISO2_G5_R1` → obtuvo IP `192.198.25.3`, gateway `192.198.25.1`
* `PISO2_G5_R2` → obtuvo IP `192.198.25.132`, gateway `192.198.25.129`

Ambos llegaron a:

* su gateway local
* `10.2.5.9`
* red de piso 1
* datacenter

---

## 7. Configuración inalámbrica — Piso 3

El PDF define para piso 3:

* SSID: `PISO3_GX_R1`, `PISO3_GX_R3`
* Broadcast: **Sí**
* Seguridad: **WPA2 Personal**
* Contraseña red: `GX_PISO3`
* Contraseña router: `GrupoX_P3` 

Para **Grupo 41 (X = 5)** se dejó así:

* `PISO3_G5_R1`
* `PISO3_G5_R3`
* clave WiFi: `G5_PISO3`
* clave router prevista: `Grupo5_P3` 

### 7.1 Piso 3 — AP 1

Conectado a:

* `MLS3 Fa0/1 ↔ puerto LAN del WRT`

Configuración:

* IP LAN: `192.198.35.2`
* Máscara: `255.255.255.128`
* DHCP: **Disabled**
* SSID: `PISO3_G5_R1`
* Broadcast: **Enabled**
* Seguridad: `WPA2 Personal`
* Clave: `G5_PISO3`

Gateway:

* `MLS3 Fa0/1 = 192.198.35.1`

### 7.2 Piso 3 — AP 2

Conectado a:

* `MLS3 Fa0/2 ↔ puerto LAN del WRT`

Configuración:

* IP LAN: `192.198.35.130`
* Máscara: `255.255.255.128`
* DHCP: **Disabled**
* SSID: `PISO3_G5_R3`
* Broadcast: **Enabled**
* Seguridad: `WPA2 Personal`
* Clave: `G5_PISO3`

Gateway:

* `MLS3 Fa0/2 = 192.198.35.129`

### 7.3 Relay DHCP en MLS3

Se agregaron:

* `ip helper-address 192.198.100.130` en `Fa0/1`
* `ip helper-address 192.198.100.130` en `Fa0/2`

### 7.4 Pools DHCP creados para piso 3

**PISO3_WLAN1**

* Gateway: `192.198.35.1`
* DNS: `192.198.100.10`
* Inicio: `192.198.35.3`
* Máscara: `255.255.255.128`

**PISO3_WLAN2**

* Gateway: `192.198.35.129`
* DNS: `192.198.100.10`
* Inicio: `192.198.35.131`
* Máscara: `255.255.255.128`

### 7.5 Validación realizada

Se probó un cliente nuevo en cada SSID:

* `PISO3_G5_R1` → obtuvo IP `192.198.35.3`, gateway `192.198.35.1`
* `PISO3_G5_R3` → obtuvo IP `192.198.35.131`, gateway `192.198.35.129`

Ambos llegaron a:

* su gateway local
* `10.2.5.5`
* datacenter
* piso 1

---

## 8. Pools DHCP que deben quedar en el servidor

Al final, `ServerDHCP` debe tener estos pools:

* `ESTUDIANTES`
* `ADMIN`
* `PISO2_WLAN1`
* `PISO2_WLAN2`
* `PISO3_WLAN1`
* `PISO3_WLAN2`

Esto también es consistente con la rúbrica, que evalúa por separado:

* DHCP cableado desde servidor
* DHCP correcto en routers/redes inalámbricas piso 2
* DHCP correcto en routers/redes inalámbricas piso 3 

---

## 9. Comandos de verificación usados

```bash
show etherchannel summary
show ip interface brief
show ip eigrp neighbors
show ip route
show standby brief
show interfaces trunk
show running-config interface <interfaz>
ping <ip>
ipconfig /all
write memory
```

Los mismos tipos de comandos también están contemplados en el README como comandos de verificación/documentación. 

---

## 10. Lo que la siguiente persona debe respetar

La siguiente persona no debería cambiar estos puntos:

1. **No mover el direccionamiento ya definido**

   * Piso 2: `192.198.25.0/25` y `192.198.25.128/25`
   * Piso 3: `192.198.35.0/25` y `192.198.35.128/25`
   * DHCP server: `192.198.100.130`

2. **No cambiar los gateways ya usados**

   * `192.198.25.1`
   * `192.198.25.129`
   * `192.198.35.1`
   * `192.198.35.129`
   * HSRP biblioteca: `192.198.100.1` y `192.198.100.129`

3. **No volver a conectar los WRT por WAN/Internet**
   Deben quedarse conectados por **LAN**, funcionando como AP.

4. **Mantener los SSID y claves exactamente así**

   * `PISO2_G5_R1`
   * `PISO2_G5_R2`
   * `PISO3_G5_R1`
   * `PISO3_G5_R3`
   * `G5_PISO2`
   * `G5_PISO3`

5. **DNS esperado por los clientes**

   * `192.198.100.10`
     Esa IP quedó reservada para que la siguiente persona configure ahí el **ServerWeb/DNS**.

---
>  ** Persona 3**

### 6.1 HSRP Biblioteca Central (Router2 y Router3)



### 6.2 Servidor Web — HTTP

- Dominio: `www.practica2_Grupo5.com`
- Página estática con datos 



### 6.3 Servidor DNS



---

## 7. Comandos de verificación utilizados

```
show vlan brief
show ip interface brief
show etherchannel summary
show ip eigrp neighbors
show ip route
show standby brief
show interfaces trunk
ping <IP>
write memory
```

---

## 8. Pruebas de comunicación

| Prueba | Resultado |
|--------|-----------|
| Ping PC1 (VLAN 25) ↔ Laptop1 (VLAN 15) |  Exitoso |
| Ping MLS2 ↔ Router1 Piso 1 |  Exitoso |
| HSRP Router1 Active / Router0 Standby |  Verificado |
| LACP MLS4 ↔ MLS2 (Po1) |  RU — 4 puertos activos |
| LACP MLS4 ↔ MLS3 (Po2) |  Pendiente Persona 2 |
| LACP MLS4 ↔ MLS0 (Po3) |  Pendiente Persona 2 |
| Ping Piso 2 ↔ Piso 1 |  Pendiente Persona 2 |
| Ping Piso 3 ↔ Piso 1 |  Pendiente Persona 2 |
| DNS www.practica2_Grupo5com |  Pendiente Persona 3 |
| Página web accesible | Pendiente Persona 3 |


