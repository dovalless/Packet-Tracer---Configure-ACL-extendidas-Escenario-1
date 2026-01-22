# 🛡️ Packet Tracer: Configuración de ACL Extendidas - Escenario 1

<div align="center">

**Laboratorio CISCO - ACL Extendidas Numeradas y con Nombre**

[![Cisco Packet Tracer](https://img.shields.io/badge/Cisco-Packet_Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)](https://www.netacad.com)
[![ACL Protocol](https://img.shields.io/badge/Protocol-ACL_Extendida-00A86B?style=for-the-badge)](https://www.cisco.com/)
[![CCNA](https://img.shields.io/badge/Certification-CCNA-blue?style=for-the-badge)](https://www.cisco.com/c/en/us/training-events/training-certifications/certifications/associate/ccna.html)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

[🎯 Objetivos](#-objetivos) • 
[📊 Tabla de Direcciones](#️-tabla-de-asignación-de-direcciones) • 
[📋 Escenario](#️-antecedentesescenario) • 
[⚙️ Configuración](#️-configuración-paso-a-paso) • 
[🔍 Verificación](#️-verificación) • 
[👨‍💻 Autor](#️-autor)

</div>

---

## 📋 Descripción del Proyecto
Este laboratorio de Cisco Packet Tracer implementa **ACL Extendidas** tanto **numeradas como con nombre** para controlar el acceso desde dos redes LAN diferentes a un servidor central. PC1 solo necesita acceso FTP e ICMP, mientras que PC2 solo necesita acceso HTTP e ICMP. El laboratorio demuestra cómo configurar, aplicar y verificar ACL específicas por protocolo y puerto.

### 🎯 Objetivos
**Parte 1:** Configurar, aplicar y verificar una ACL numerada extendida (ACL 100)  
**Parte 2:** Configurar, aplicar y verificar una ACL con nombre extendida (HTTP_ONLY)  

### 📋 Antecedentes/Escenario
Dos empleados necesitan acceso a servicios proporcionados por un servidor:
- **PC1** solo necesita acceso FTP e ICMP (ping)
- **PC2** solo necesita acceso web (HTTP) e ICMP (ping)

---

## 📊 Tabla de Asignación de Direcciones

| Dispositivo | Interfaz | Dirección IP | Máscara de Subred | Gateway |
|-------------|----------|--------------|-------------------|---------|
| **R1** | G0/0 | 172.22.34.65 | 255.255.255.224 | N/A |
| **R1** | G0/1 | 172.22.34.97 | 255.255.255.240 | N/A |
| **R1** | G0/2 | 172.22.34.1 | 255.255.255.192 | N/A |
| **Server** | NIC | 172.22.34.62 | 255.255.255.192 | 172.22.34.1 |
| **PC1** | NIC | 172.22.34.66 | 255.255.255.224 | 172.22.34.65 |
| **PC2** | NIC | 172.22.34.98 | 255.255.255.240 | 172.22.34.97 |

### 🌐 Topología de Red y Subredes
```
             [Server:172.22.34.62/26]
                     |
              G0/2:172.22.34.1/26
                  [R1]
              /           \
   G0/0:172.22.34.65/27   G0/1:172.22.34.97/28
            |                     |
   [PC1:172.22.34.66/27]  [PC2:172.22.34.98/28]
   
Subredes:
- Server Network: 172.22.34.0/26  (172.22.34.1 - 172.22.34.62)
- PC1 Network:   172.22.34.64/27 (172.22.34.65 - 172.22.34.94)
- PC2 Network:   172.22.34.96/28 (172.22.34.97 - 172.22.34.110)
```

---

## ⚙️ Configuración Paso a Paso

### Parte 1: Configurar, Aplicar y Verificar ACL Numerada Extendida (ACL 100)

#### Política para PC1:
- ✅ Permitir FTP (TCP puerto 21) desde red PC1 (172.22.34.64/27) al Server
- ✅ Permitir ICMP (ping) desde red PC1 al Server
- ❌ Denegar todo el resto (implícito)

#### Paso 1: Configurar ACL 100
```cisco
! Permitir FTP desde red PC1 (172.22.34.64/27) al Server
R1(config)# access-list 100 permit tcp 172.22.34.64 0.0.0.31 host 172.22.34.62 eq ftp

! Permitir ICMP desde red PC1 al Server
R1(config)# access-list 100 permit icmp 172.22.34.64 0.0.0.31 host 172.22.34.62

! Nota: No hay 'permit ip any any' - política implícita deny all
```

**Cálculo de Wildcard Mask:**
- Máscara de subred: /27 = 255.255.255.224
- Wildcard: 255.255.255.255 - 255.255.255.224 = 0.0.0.31
- O en binario: 11111111.11111111.11111111.11100000 → 00000000.00000000.00000000.00011111

#### Paso 2: Aplicar ACL 100
```cisco
! Aplicar ACL 100 en interfaz G0/0 (hacia PC1) dirección IN
R1(config)# interface gigabitEthernet 0/0
R1(config-if)# ip access-group 100 in
```

**Justificación:**
- El tráfico desde PC1 hacia el Server entra por G0/0
- Aplicar ACL en IN filtra el tráfico entrante desde la red PC1
- Posición eficiente: cerca del origen

#### Paso 3: Verificar Configuración
```cisco
! Ver ACL 100 con números de secuencia
R1# show access-lists 100

! Salida esperada:
Extended IP access list 100
    10 permit tcp 172.22.34.64 0.0.0.31 host 172.22.34.62 eq ftp
    20 permit icmp 172.22.34.64 0.0.0.31 host 172.22.34.62
```

### Parte 2: Configurar, Aplicar y Verificar ACL con Nombre Extendida (HTTP_ONLY)

#### Política para PC2:
- ✅ Permitir HTTP (TCP puerto 80) desde red PC2 (172.22.34.96/28) al Server
- ✅ Permitir ICMP (ping) desde red PC2 al Server
- ❌ Denegar todo el resto (implícito)

#### Paso 1: Configurar ACL HTTP_ONLY
```cisco
! Crear ACL extendida con nombre
R1(config)# ip access-list extended HTTP_ONLY

! Permitir HTTP desde red PC2 (172.22.34.96/28) al Server
R1(config-ext-nacl)# permit tcp 172.22.34.96 0.0.0.15 host 172.22.34.62 eq www

! Permitir ICMP desde red PC2 al Server
R1(config-ext-nacl)# permit icmp 172.22.34.96 0.0.0.15 host 172.22.34.62

! Salir del modo ACL
R1(config-ext-nacl)# exit
```

**Cálculo de Wildcard Mask:**
- Máscara de subred: /28 = 255.255.255.240
- Wildcard: 255.255.255.255 - 255.255.255.240 = 0.0.0.15

#### Paso 2: Aplicar ACL HTTP_ONLY
```cisco
! Aplicar ACL HTTP_ONLY en interfaz G0/1 (hacia PC2) dirección IN
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# ip access-group HTTP_ONLY in
```

#### Paso 3: Verificar Configuración Completa
```cisco
! Ver todas las ACL configuradas
R1# show access-lists

! Salida esperada:
Extended IP access list 100
    10 permit tcp 172.22.34.64 0.0.0.31 host 172.22.34.62 eq ftp
    20 permit icmp 172.22.34.64 0.0.0.31 host 172.22.34.62
Extended IP access list HTTP_ONLY
    10 permit tcp 172.22.34.96 0.0.0.15 host 172.22.34.62 eq www
    20 permit icmp 172.22.34.96 0.0.0.15 host 172.22.34.62
```

---

## 🔍 Verificación y Pruebas

### Pruebas desde PC1
```cisco
! Desde PC1:

1. Ping al Server (debe funcionar):
PC1> ping 172.22.34.62
Reply from 172.22.34.62: bytes=32 time=1ms TTL=127 ✓

2. FTP al Server (debe funcionar):
PC1> ftp 172.22.34.62
Username: cisco
Password: cisco
ftp> quit ✓

3. HTTP al Server (debe FALLAR):
PC1> browser http://172.22.34.62
Connection failed ✗

4. Ping a PC2 (debe FALLAR - ACL no lo permite):
PC1> ping 172.22.34.98
Request timed out ✗
```

### Pruebas desde PC2
```cisco
! Desde PC2:

1. Ping al Server (debe funcionar):
PC2> ping 172.22.34.62
Reply from 172.22.34.62: bytes=32 time=1ms TTL=127 ✓

2. HTTP al Server (debe funcionar):
PC2> browser http://172.22.34.62
Web page displayed ✓

3. FTP al Server (debe FALLAR):
PC2> ftp 172.22.34.62
Connection failed ✗

4. Ping a PC1 (debe FALLAR - ACL no lo permite):
PC2> ping 172.22.34.66
Request timed out ✗
```

### Verificación de Contadores ACL
```cisco
! Ver contadores de coincidencias
R1# show ip access-lists

! Limpiar contadores para nuevas pruebas
R1# clear access-list counters 100
R1# clear access-list counters HTTP_ONLY
```

---

## 💡 Conceptos Fundamentales Aprendidos

### 🎯 ACL Extendidas Numeradas vs Con Nombre
| Característica | ACL 100 (Numerada) | HTTP_ONLY (Con Nombre) |
|----------------|---------------------|------------------------|
| **Sintaxis** | `access-list 100 permit...` | `ip access-list extended HTTP_ONLY` |
| **Rango** | 100-199 | Cualquier nombre descriptivo |
| **Edición** | Solo añadir al final | Insertar/eliminar/modificar líneas |
| **Documentación** | Menos descriptiva | Más descriptiva (HTTP_ONLY) |
| **Aplicación** | `ip access-group 100 in` | `ip access-group HTTP_ONLY in` |

### 🔧 Cálculo de Wildcard Masks
**Método 1: Resta**
```
255.255.255.255
- 255.255.255.224  (/27)
= 0.0.0.31
```

**Método 2: Complemento Binario**
```
Máscara: 255.255.255.224 = 11111111.11111111.11111111.11100000
Wildcard:                    00000000.00000000.00000000.00011111 = 0.0.0.31
```

### 🌐 Protocolos y Puertos Utilizados
| Servicio | Protocolo | Puerto | Descripción |
|----------|-----------|--------|-------------|
| **FTP** | TCP | 21 | Control FTP (File Transfer Protocol) |
| **HTTP** | TCP | 80 | World Wide Web (Hypertext Transfer Protocol) |
| **ICMP** | ICMP | - | Internet Control Message Protocol (ping) |

### 📖 Comandos Clave de Configuración
```cisco
! ACL Numerada Extendida
access-list [100-199] [permit/deny] [protocolo] [origen] [destino] [eq puerto]

! ACL Extendida con Nombre
ip access-list extended [NOMBRE]
 [permit/deny] [protocolo] [origen] [destino] [eq puerto]

! Aplicar ACL a interfaz
interface [interfaz]
 ip access-group [NOMBRE/NÚMERO] [in/out]

! Verificación
show access-lists
show ip access-lists
clear access-list counters
```

---

## 🚀 Solución de Problemas Comunes

### ❌ Ping no funciona desde PC1/PC2 al Server
```cisco
! Verificar ACL aplicada correctamente
R1# show ip interface g0/0
R1# show ip interface g0/1

! Verificar reglas ICMP en ACL
R1# show access-lists | include icmp

! Probar conectividad básica (desactivar ACL temporalmente)
R1(config)# interface g0/0
R1(config-if)# no ip access-group 100 in
```

### ❌ FTP/HTTP no funciona pero ping sí
```cisco
! Verificar puertos correctos en ACL
R1# show access-lists

! FTP usa puerto 21, HTTP usa puerto 80
! Verificar que las reglas especifiquen eq ftp o eq www

! Probar servicio FTP/HTTP desde router
R1# telnet 172.22.34.62 21
R1# telnet 172.22.34.62 80
```

### ❌ ACL no filtra tráfico como se espera
```cisco
! Verificar dirección de aplicación (in/out)
R1# show running-config interface g0/0
R1# show running-config interface g0/1

! Verificar contadores para ver qué reglas coinciden
R1# show ip access-lists
R1# clear access-list counters
! Realizar prueba específica
! Verificar contadores nuevamente
```

### 📋 Checklist de Configuración Correcta
- [ ] Wildcard masks calculadas correctamente
- [ ] Protocolos y puertos especificados correctamente
- [ ] ACL aplicadas en interfaz correcta
- [ ] Dirección correcta (IN para tráfico entrante)
- [ ] Política implícita deny all (no se necesita regla final)
- [ ] Verificación con pruebas específicas

---

## 📚 Recursos Adicionales

### Documentación Oficial Cisco
- [Cisco Extended ACL Configuration Guide](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_data_acl/configuration/15-mt/sec-acl-15-mt-book.html)
- [Wildcard Mask Calculation](https://www.cisco.com/c/en/us/support/docs/ip/access-lists/26448-ACLs-wildcard-mask.html)
- [ACL Best Practices](https://www.cisco.com/c/en/us/support/docs/security/ios-firewall/23602-confaccesslists.html)

### Libros Recomendados
- "CCNA 200-301 Official Cert Guide" - ACL Chapter
- "Cisco IOS Access Lists" - O. Held
- "Network Security Fundamentals" - Cisco Press

### Laboratorios Relacionados
- **ACL Estándar:** Filtrado básico por dirección de origen
- **ACL Reflexivas:** Control de sesiones bidireccional
- **ACL Basadas en Tiempo:** Filtrado por horario específico
- **ACL para VPN:** Filtrado en túneles VPN

### 🎓 Preguntas de Práctica CCNA
1. ¿Cuál es la diferencia entre `eq ftp` y `eq 21` en una ACL?
2. ¿Por qué se aplica la ACL en dirección IN en lugar de OUT?
3. ¿Cómo afecta la wildcard mask 0.0.0.31 al filtrado?
4. ¿Qué ventajas ofrecen las ACL con nombre sobre las numeradas?

---

## 👨‍💻 Autor

<div align="center">

**Darwin Manuel Ovalles Cesar**

<p align="center">
<a href="https://www.linkedin.com/in/darwin-manuel-ovalles-cesar-dev" target="_blank">
<img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/linked-in-alt.svg" alt="LinkedIn - Darwin Ovalles" height="40" width="50" />
</a>
<a href="https://github.com/dovalless" target="_blank">
<img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/github.svg" alt="GitHub - Darwin Ovalles" height="40" width="50" />
</a>
</p>

💼 **LinkedIn**: [darwin-manuel-ovalles-cesar-dev](https://www.linkedin.com/in/darwin-manuel-ovalles-cesar-dev/)  
🌐 **GitHub**: [@dovalless](https://github.com/dovalless)  
🎓 **Certificaciones**: CCNA, Network+, A+  

*"Las ACL extendidas son herramientas de precisión en el arsenal del administrador de redes. Este laboratorio demuestra cómo implementar controles granulares que permiten solo el acceso necesario, siguiendo el principio de mínimo privilegio."*

**#Cisco #PacketTracer #ACL #ExtendedACL #NetworkSecurity #CCNA #AccessControl**

</div>

---

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo [LICENSE](LICENSE) para más detalles.

```
MIT License
Copyright (c) 2024 Darwin Manuel Ovalles Cesar
```

---

## 🙏 Agradecimientos

- **Cisco Networking Academy** - Por Packet Tracer y recursos educativos
- **Comunidad de Networking** - Por compartir conocimiento y experiencias
- **Profesionales de Seguridad** - Por destacar la importancia del acceso controlado

<div align="center">

### ⭐ Si este laboratorio te ayudó a entender ACL extendidas, compártelo ⭐

### 🔄 **Reflexión Final:**
*"Configurar ACL es como programar un guardia de seguridad inteligente: le dices exactamente quién puede entrar (dirección IP), a qué puerta (puerto), y para hacer qué (protocolo). Las ACL extendidas nos permiten ser tan específicos como 'solo los empleados del departamento A pueden usar la fotocopiadora B entre 9am y 5pm'."*

**Desarrollado con 💙 para futuros ingenieros de red**

---
*Laboratorio completado: Packet Tracer - Configure Extended IPv4 ACLs: Scenario 1*  
*Habilidades demostradas: ACL Numeradas, ACL con Nombre, Wildcard Masks, Filtrado por Puerto*

</div>
```
