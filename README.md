# Resumen Técnico: Configuración de Red para Práctica 1 - Monte Alto y la Conectividad

## Información General
| Integrante | Enfoque Principal | Carnet |
|------------|-------------------|------------------|
| Harold Alejandro Sánchez Hernández | Infraestructura y VTP | 202200100 |
|  2| STP y Seguridad | 2 |
| 3 | Enrutamiento | 2 |

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

