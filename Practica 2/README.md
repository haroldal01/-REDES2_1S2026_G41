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

> ** Persona 2**

### 5.1 LACP MLS4 ↔ MLS3 (Piso 3) — Po2


### 5.2 LACP MLS4 ↔ MLS0 (Piso 2) — Po3


### 5.3 Configuración inalámbrica

| Piso | SSID | Broadcast | Seguridad | Contraseña RED | Contraseña Router |
|------|------|-----------|-----------|----------------|-------------------|
| 2 | PISO2_G5_R1, PISO2_G5_R2 | No | WPA2 Personal | G5_PISO2 | Grupo5_P2 |
| 3 | PISO3_G5_R1, PISO3_G5_R3 | Sí | WPA2 Personal | G5_PISO3 | Grupo5_P3 |



### 5.4 Configuración DHCP — ServerDHCP



### 5.5 Configuración DHCP — Routers inalámbricos



---

## 6. Configuración Persona 3 — Servidor Web, DNS y HSRP Biblioteca

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
