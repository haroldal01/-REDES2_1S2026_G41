# Resumen Técnico: Configuración de Red para Práctica 1 - Monte Alto y la Conectividad

## Información General
| Integrante | Enfoque Principal | Carnet |
|------------|-------------------|------------------|
| Harold Alejandro Sánchez Hernández | Infraestructura y VTP | 202200100 |
|  Engel Emilio Coc Raxjal| STP y Seguridad | 202200314 |
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


