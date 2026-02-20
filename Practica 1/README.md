# Resumen Técnico: Configuración de Red para Práctica 1 - Monte Alto y la Conectividad

## Información General
| Integrante | Enfoque Principal | Carnet |
|------------|-------------------|------------------|
| Harold Alejandro Sánchez Hernández | Infraestructura y VTP | 202200100 |
|  Engel Emilio Coc Raxjal| STP y Seguridad | 202200314 |
| Madeline Fabiola Prado Reyes | Enrutamiento | 202100039 |

## Resumen de Topología 
Edificio izquierdo: Switches 2960-24TT en pirámide conectados al router izquierdo. Hosts en VLANs 15, 25, 35. Enlaces trunk entre switches y router.  



## Pasos de Configuración y Comandos Clave

### 1. Configuración Básica del Hostname
**Propósito**: Identificación única de switches.  
**Comando (en cada switch izquierdo):**  
```
enable
conf t 
hostname SW(10)1_G41
```  
**Verificación**: `show running-config | include hostname`.

### 2. Contraseñas de Seguridad
**Propósito**: Protección del VTP Server.  
**Comandos (solo en SW0_G41):**  
```
enable secret redes2grupo41
line con 0
password redes2grupo41
login
```  
**Verificación**: Acceso con `enable`.

### 3. VTP Server y VLANs
**Propósito**: Propagación centralizada de VLANs.  
**Comandos (en SW0_G41):**  
```
enable
configure terminal
vtp mode server
vtp domain G41
vlan 15
name Primaria
vlan 25
name Básicos
vlan 35
name Bachillerato
```  
**Verificación**: `show vtp status` (Server, G41); `show vlan brief` (VLANs listadas).

### 4. VTP en Clientes
**Propósito**: Recepción de VLANs.  
**Comandos (en switches clientes):**  
```
enable
configure terminal
vtp mode client
vtp domain G41
end
copy running-config startup-config
```  
**Verificación**: `show vtp status` (Client, G41).

### 5. Puertos Trunk
**Propósito**: Paso de VLANs entre dispositivos.  
**Comandos (en todos switches izquierdos):**  
```
enable
configure terminal
interface range fa0/1 - 2
 switchport mode trunk
 switchport trunk allowed vlan 15,25,35
exit
exit
write memory
```  
**Verificación**: `show interfaces trunk` (VLANs allowed: 15,25,35).

### 6. Puertos Access para Hosts
**Propósito**: Asignación de hosts a VLANs.  
**Comandos (en switches relevantes):**  
```
configure terminal
interface range fa0/3
 switchport mode access
 switchport access vlan 15
exit
interface range fa0/2
 switchport mode access
 switchport access vlan 35
exit
interface range fa0/3
 switchport mode access
 switchport access vlan 25
exit
end
write memory
```  
**Verificación**: `show vlan brief` (puertos en VLANs correctas).

### 7. STP - PVST
**Propósito**: Prevención de loops por VLAN.  
**Comandos (en todos switches izquierdos):**  
```
enable
configure terminal
spanning-tree mode pvst
end
write memory
```  
**Justificación**: PVST elegido por simplicidad y compatibilidad; convergencia ~50 seg, comparativa con Rapid PVST por Integrante 2.  
**Verificación**: `show spanning-tree` (modo PVST).

#  Tabla de Tiempos STP

**Rapid Spanning Tree Protocol (RSTP)** en modo **Rapid-PVST**

RSTP es una mejora de:

Spanning Tree Protocol



##  Comparación de tiempos

| Parámetro              | STP Tradicional (802.1D)                  | RSTP (802.1W / Rapid-PVST)       |
| ---------------------- | ----------------------------------------- | -------------------------------- |
| Hello Time             | 2 segundos                                | 2 segundos                       |
| Max Age                | 20 segundos                               | 6 segundos aprox                 |
| Forward Delay          | 15 segundos                               | No usa como tal                  |
| Tiempo de convergencia | 30–50 segundos                            | 1–6 segundos                     |
| Estados de puerto      | Blocking, Listening, Learning, Forwarding | Discarding, Learning, Forwarding |



 Convergencia aproximada: 1 a 6 segundos
 Mucho más rápido que STP tradicional



## Configuración de Conectividad y Seguridad



## 1 Configuración de VTP

Se configuró el protocolo:

VLAN Trunking Protocol

Para permitir la propagación automática de VLANs

### En los switches cliente (lado derecho):

```
conf t
vtp domain G41
vtp mode client
```

Verificación:

```
show vtp status
show vlan brief
```

Resultado:
Las VLAN 65, 75 y 85 fueron propagadas correctamente desde el switch servidor.



## 2️ Creación y Administración de VLANs

Se trabajó con las siguientes VLANs:

| VLAN | Nombre       |
| ---- | ------------ |
| 65   | Primaria     |
| 75   | Basicos      |
| 85   | Bachillerato |

Asignación a puertos:

```
interface fa0/x
switchport mode access
switchport access vlan XX
```

Verificación:

```
show vlan brief
```



## 3️ Configuración de Enlaces Troncales

Para permitir el transporte de múltiples VLANs entre switches:

```
interface fa0/x
switchport mode trunk
```

Verificación:

```
show interfaces trunk
```



## 4️ Configuración de Rapid-PVST

Se habilitó el protocolo:

Rapid Spanning Tree Protocol

Comando:

```
spanning-tree mode rapid-pvst
```

Verificación:

```
show spanning-tree summary
```

Resultado:
Switch operando en modo rapid-pvst



## 5️ Configuración de Port Security (Solo VLAN 75 Y LA VLAN 25)

Se aplicó seguridad  laS VLAN 75 Y VLAN 25 (Básicos)

Configuración:

```
interface fa0/x
switchport mode access
switchport access vlan 75
switchport port-security
switchport port-security maximum 1
switchport port-security violation shutdown
switchport port-security mac-address sticky
```



###  Parámetros implementados

* Máximo 1 dirección MAC por puerto
* Aprendizaje automático (Sticky)
* Violación: Apagado automático del puerto



###  Prueba de funcionamiento

1. Se conectó PC original → puerto aprendió MAC
2. Se realizó ping exitoso
3. Se conectó PC diferente
4. Se detectó violación
5. Se bloqueó comunicación

Verificación:

```
show port-security
show port-security address
```

Resultado:
Registro de MAC y funcionamiento correcto de seguridad


## 6️ Prueba de Convergencia

Se realizó prueba desconectando un enlace trunk

Resultado:

* Pérdida temporal de conectividad
* Recuperación en aproximadamente 1–6 segundos
* Confirmación de funcionamiento de Rapid-PVST



#  Conclusion para esta parte 

Se implementó correctamente:

* Segmentación por VLAN
* Propagación automática con VTP
* Prevención de loops con Rapid-PVST
* Seguridad de acceso mediante Port Security

La red quedó funcional, segmentada y protegida contra accesos no autorizados


---

# PARTE 3 - Enrutamiento e Interconectividad
### Integrante 3: Madeline Fabiola Prado Reyes (202100039)

## Descripción
Configuración de protocolos de enrutamiento dinámico (EIGRP, RIP y OSPF), 
asignación de IPs a hosts y verificación de comunicación entre ambos edificios.

## Direccionamiento IP de Hosts

| Dispositivo | VLAN | IP | Gateway |
|---|---|---|---|
| PC0 | Básicos 25 | 192.178.25.2 | 192.178.25.1 |
| PC1 | Bachillerato 35 | 192.178.35.2 | 192.178.35.1 |
| Laptop0 | Bachillerato 35 | 192.178.35.3 | 192.178.35.1 |
| PC2 | Primaria 15 | 192.178.15.2 | 192.178.15.1 |
| PC3 | Primaria 15 | 192.178.15.3 | 192.178.15.1 |
| PC4 | Bachillerato 85 | 192.178.85.2 | 192.178.85.1 |
| PC5 | Básicos 75 | 192.178.75.2 | 192.178.75.1 |
| Laptop1 | Básicos 75 | 192.178.75.3 | 192.178.75.1 |
| PC6 | Primaria 65 | 192.178.65.2 | 192.178.65.1 |
| PC7 | Primaria 65 | 192.178.65.3 | 192.178.65.1 |

## Redes de Enrutamiento Dinámico

| Protocolo | Red | Cálculo |
|---|---|---|
| EIGRP | 10.10.8.0/24 | X = 3 + 5 = 8 |
| RIP | 10.10.7.0/24 | X = 2 + 5 = 7 |
| OSPF | 10.10.6.0/24 | X = 1 + 5 = 6 |

## Configuración Router0 - EIGRP
```
enable
configure terminal
hostname Router0
interface GigabitEthernet0/1
no shutdown
exit
interface GigabitEthernet0/1.15
encapsulation dot1Q 15
ip address 192.178.15.1 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/1.25
encapsulation dot1Q 25
ip address 192.178.25.1 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/1.35
encapsulation dot1Q 35
ip address 192.178.35.1 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/0
ip address 10.10.8.1 255.255.255.0
no shutdown
exit
router eigrp 1
network 192.178.15.0 0.0.0.255
network 192.178.25.0 0.0.0.255
network 192.178.35.0 0.0.0.255
network 10.10.8.0 0.0.0.255
no auto-summary
exit
ip route 192.178.65.0 255.255.255.0 10.10.8.2
ip route 192.178.75.0 255.255.255.0 10.10.8.2
ip route 192.178.85.0 255.255.255.0 10.10.8.2
ip route 10.10.7.0 255.255.255.0 10.10.8.2
ip route 10.10.6.0 255.255.255.0 10.10.8.2
end
write memory
```

## Configuración Router1 - RIP
```
enable
configure terminal
hostname Router1
interface GigabitEthernet0/0
ip address 10.10.8.2 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/1
ip address 10.10.7.1 255.255.255.0
no shutdown
exit
router rip
version 2
network 10.10.8.0
network 10.10.7.0
no auto-summary
exit
ip route 192.178.15.0 255.255.255.0 10.10.8.1
ip route 192.178.25.0 255.255.255.0 10.10.8.1
ip route 192.178.35.0 255.255.255.0 10.10.8.1
ip route 192.178.65.0 255.255.255.0 10.10.7.2
ip route 192.178.75.0 255.255.255.0 10.10.7.2
ip route 192.178.85.0 255.255.255.0 10.10.7.2
end
write memory
```

## Configuración Router2 - RIP
```
enable
configure terminal
hostname Router2
interface GigabitEthernet0/0
ip address 10.10.7.2 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/1
ip address 10.10.6.1 255.255.255.0
no shutdown
exit
router rip
version 2
network 10.10.7.0
network 10.10.6.0
no auto-summary
exit
ip route 192.178.15.0 255.255.255.0 10.10.7.1
ip route 192.178.25.0 255.255.255.0 10.10.7.1
ip route 192.178.35.0 255.255.255.0 10.10.7.1
ip route 192.178.65.0 255.255.255.0 10.10.6.2
ip route 192.178.75.0 255.255.255.0 10.10.6.2
ip route 192.178.85.0 255.255.255.0 10.10.6.2
end
write memory
```

## Configuración Router3 - OSPF
```
enable
configure terminal
hostname Router3
interface GigabitEthernet0/0
ip address 10.10.6.2 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/1
no shutdown
exit
interface GigabitEthernet0/1.65
encapsulation dot1Q 65
ip address 192.178.65.1 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/1.75
encapsulation dot1Q 75
ip address 192.178.75.1 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/1.85
encapsulation dot1Q 85
ip address 192.178.85.1 255.255.255.0
no shutdown
exit
router ospf 1
network 192.178.65.0 0.0.0.255 area 0
network 192.178.75.0 0.0.0.255 area 0
network 192.178.85.0 0.0.0.255 area 0
network 10.10.6.0 0.0.0.255 area 0
exit
ip route 192.178.15.0 255.255.255.0 10.10.6.1
ip route 192.178.25.0 255.255.255.0 10.10.6.1
ip route 192.178.35.0 255.255.255.0 10.10.6.1
ip route 10.10.7.0 255.255.255.0 10.10.6.1
ip route 10.10.8.0 255.255.255.0 10.10.6.1
end
write memory
```

## Pruebas de Comunicación

Todas realizadas desde PC0 (192.178.25.2).

| Prueba | Descripción | Resultado |
|---|---|---|
| 4.1 | Básicos mismo lado - ping 192.178.25.1 |  Exitoso |
| 4.2 | Básicos ambos lados - ping 192.178.75.2 |  Exitoso |
| 4.3 | Inter-VLAN Básicos-Primaria - ping 192.178.15.2 |  Exitoso |
| 4.4 | Bachillerato mismo lado - ping 192.178.35.2 |  Exitoso |
| 4.5 | Bachillerato ambos lados - ping 192.178.85.2 |  Exitoso |
| 4.6 | Inter-VLAN Básicos-Bachillerato - ping 192.178.35.2 |  Exitoso |
| 4.7 | Primaria mismo lado - ping 192.178.15.3 |  Exitoso |
| 4.8 | Primaria ambos lados - ping 192.178.65.2 |  Exitoso |
| 4.9 | Inter-VLAN Básicos-Primaria derecho - ping 192.178.75.3 |  Exitoso |

## Justificación Protocolo STP Elegido: Rapid PVST

- **PVST** (edificio izquierdo): convergencia ~30-50 segundos
- **Rapid PVST** (edificio derecho): convergencia ~1-6 segundos

Se elige **Rapid PVST** porque converge hasta 10 veces más rápido, usa negociación 
proposal/agreement en lugar de temporizadores fijos, y minimiza el tiempo de 
inactividad en la red del Colegio Monte Alto.


