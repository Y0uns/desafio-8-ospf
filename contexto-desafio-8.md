# Contexto del proyecto: Desafío 8, OSPF entre sedes
Actualizado: 18 de noviembre de 2026

## Instrucciones para Claude
Este archivo resume el proyecto, las decisiones tomadas y el estado actual.
Úsalo como contexto de trabajo y ayuda a implementar, probar y preparar la
demostración. Responde en español. No cambies decisiones cerradas sin avisar.
Lo marcado como "pendiente" todavía no se ha probado en GNS3.

## 1. Asignatura y evaluación

- Universidad: USM
- Ramo: Taller de Administración de Sistemas (EIN-090B)
- Equipo: 2 estudiantes
- Caso: TechnoMarket Ltda.
- Variante asignada: Equipo 8, OSPF entre sedes
- Evaluación: demostración en vivo de 15 minutos, más preguntas. No hay informe.
- 5 min: arquitectura y decisiones.
- 10 min: operación integrada, pruebas y variante.
- El profesor puede elegir quién ejecuta cada prueba.
- Ambos deben conocer toda la solución.
- Capturas, archivos o diapositivas no reemplazan las pruebas en vivo.

### Rúbrica grupal (60 %)
- Infraestructura y servicios comunes: 20 %
- Funcionamiento e integración de la variante: 15 %
- Seguridad y controles de acceso: 10 %
- Demostración y pruebas funcionales: 10 %
- Gestión de incidencia o cambio controlado: 5 %

### Rúbrica individual (40 %)
- Dominio técnico de la solución: 15 %
- Ejecución autónoma de pruebas o configuración: 15 %
- Fundamentación y respuestas al docente: 10 %

## 2. Caso y requisitos

TechnoMarket tiene una Casa Matriz en Santiago y una Sucursal en Valparaíso.
Hoy usa una red plana, sin segmentación ni control de accesos internos.
Las transferencias grandes desde Soporte Técnico causan congestión.
El diseño debe segmentar departamentos, aplicar controles por área y permitir
una tercera sucursal en los próximos 12 meses.

### Requisitos obligatorios
1. Representar ambas sedes y un enlace entre ellas.
2. Implementar al menos 3 VLAN, enrutamiento inter-VLAN y conectividad entre sedes.
3. Implementar DHCP con ámbitos diferenciados y DHCP Relay cuando corresponda.
4. Implementar DNS interno para el servicio HTTP corporativo.
5. Comprobar acceso HTTP por dominio desde redes autorizadas.
6. Aplicar al menos 2 restricciones ACL y probar tráfico permitido y denegado.
7. Demostrar la variante y ejecutar una modificación controlada del profesor.

### Variante 8
- Intercambiar las redes corporativas mediante OSPF entre los routers de ambas sedes.
- Controlar las redes anunciadas.
- Mostrar adyacencias, rutas y conectividad.
- Incorporar una red y verificar que la ruta se aprende.

## 3. Decisiones tomadas

- El profesor aprobó usar router Cisco en lugar de pfSense.
- Diseño: un router por sede, dos en total.
- Imagen usada: `c2691-adventerprisek9_sna-mz.124-13b.image` (plantilla c2691 en GNS3, RAM 192 MiB).
- Las ACL serán ACL extendidas de Cisco.
- pfSense queda descartado para este proyecto.
- Windows Server 2019, Windows 10 y Ubuntu 22.04 se ejecutan en VMware Workstation 25H2.
- La VM de Ubuntu usa VMnet2 (solo host, DHCP deshabilitado, red 192.168.71.0/24). Esta red no tiene internet, por eso la instalación no descarga actualizaciones.

### Software disponible en el curso
- GNS3 2.2.55
- VMware Workstation 25H2
- Windows Server 2019 y Windows 10
- Ubuntu 22.04.5 Desktop
- Rocky 8.6 y Rocky 10.1
- Switch de GNS3 y VPCS

## 4. Topología

### Santiago
- Router Cisco R1 (FastEthernet0/0 troncal hacia Switch1, FastEthernet0/1 hacia R2)
- Switch1 (GNS3): puerto 0 troncal dot1q; puerto 1 access VLAN 10; puerto 2 access VLAN 40; puerto 3 access VLAN 100
- VLAN 10, VLAN 40 y VLAN 100

### Valparaíso
- Router Cisco R2 (FastEthernet0/0 troncal hacia Switch2, FastEthernet0/1 hacia R1)
- Switch2 (GNS3): puerto 0 troncal dot1q; puerto 1 access VLAN 20; puerto 2 access VLAN 30; puerto 3 access VLAN 50
- VLAN 20, VLAN 30 y VLAN 50

### Enlace
- R1 FastEthernet0/1 10.255.12.1/30 <-> R2 FastEthernet0/1 10.255.12.2/30
- OSPF área 0

## 5. Direccionamiento

### Bloques
- Santiago: 10.1.0.0/16
- Valparaíso: 10.2.0.0/16
- Tercera sucursal: 10.3.0.0/16 reservado
- Enlace entre sedes: 10.255.12.0/30

| Sede | VLAN | Red | Subred | Gateway | Dirección |
| --- | --- | --- | --- | --- | --- |
| Santiago | 10 | Administración y Finanzas | 10.1.10.0/24 | 10.1.10.1 | DHCP con relay |
| Santiago | 40 | TI | 10.1.40.0/24 | 10.1.40.1 | DHCP con relay |
| Santiago | 100 | Servidores | 10.1.100.0/24 | 10.1.100.1 | Estático |
| Valparaíso | 20 | Ventas | 10.2.20.0/24 | 10.2.20.1 | DHCP con relay |
| Valparaíso | 30 | Soporte | 10.2.30.0/24 | 10.2.30.1 | DHCP con relay |
| Valparaíso | 50 | Bodega | 10.2.50.0/24 | 10.2.50.1 | DHCP con relay |

Gerencia General comparte la VLAN 10 en esta maqueta.

### Servidores
- Windows Server 2019: 10.1.100.10, DHCP y DNS (pendiente de configurar)
- Ubuntu 22.04: 10.1.100.20, HTTP con nginx (pendiente de instalar y configurar)
- Dominio: `intranet.technomarket.lan`

## 6. Configuración aplicada y verificada

### R1, Santiago (verificado con `show ip interface brief`, `show ip protocols` y `show running-config | section router ospf`)
```
hostname R1
no ip domain-lookup
interface FastEthernet0/0
 no shutdown
!
interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.1.10.1 255.255.255.0
 ip helper-address 10.1.100.10
!
interface FastEthernet0/0.40
 encapsulation dot1Q 40
 ip address 10.1.40.1 255.255.255.0
 ip helper-address 10.1.100.10
!
interface FastEthernet0/0.100
 encapsulation dot1Q 100
 ip address 10.1.100.1 255.255.255.0
!
interface FastEthernet0/1
 ip address 10.255.12.1 255.255.255.252
 no shutdown
!
router ospf 1
 router-id 1.1.1.1
 log-adjacency-changes
 passive-interface FastEthernet0/0.10
 passive-interface FastEthernet0/0.40
 passive-interface FastEthernet0/0.100
 network 10.1.10.0 0.0.0.255 area 0
 network 10.1.40.0 0.0.0.255 area 0
 network 10.1.100.0 0.0.0.255 area 0
 network 10.255.12.0 0.0.0.3 area 0
```

### R2, Valparaíso (verificado con `show ip interface brief`, `show ip protocols` y `show ip ospf neighbor`)
```
hostname R2
no ip domain-lookup
interface FastEthernet0/0
 no shutdown
!
interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.2.20.1 255.255.255.0
 ip helper-address 10.1.100.10
!
interface FastEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.2.30.1 255.255.255.0
 ip helper-address 10.1.100.10
!
interface FastEthernet0/0.50
 encapsulation dot1Q 50
 ip address 10.2.50.1 255.255.255.0
 ip helper-address 10.1.100.10
!
interface FastEthernet0/1
 ip address 10.255.12.2 255.255.255.252
 no shutdown
!
router ospf 1
 router-id 2.2.2.2
 passive-interface FastEthernet0/0.20
 passive-interface FastEthernet0/0.30
 passive-interface FastEthernet0/0.50
 network 10.255.12.0 0.0.0.3 area 0
 network 10.2.20.0 0.0.0.255 area 0
 network 10.2.30.0 0.0.0.255 area 0
 network 10.2.50.0 0.0.0.255 area 0
```
Nota: `network 10.2.50.0 0.0.0.255 area 0` se agregó en vivo durante la prueba de la variante. Si R2 se reinicia sin haber guardado, Bodega puede quedar anunciada o no según el estado guardado. Antes de la demostración hay que decidir si Bodega va anunciada desde el inicio o en vivo, y dejar la configuración de acuerdo a eso.

### Estado de OSPF (verificado)
- Adyacencia R1 <-> R2 en FULL sobre FastEthernet0/1 (R2 es DR, R1 es BDR).
- R1 aprende 10.2.20.0/24, 10.2.30.0/24 y 10.2.50.0/24 vía 10.255.12.2.
- R2 aprende 10.1.10.0/24, 10.1.40.0/24 y 10.1.100.0/24 vía 10.255.12.1.
- Ping de R1 a 10.2.50.1: 5/5, después de anunciar Bodega.
- Antes de anunciar Bodega, R1 no tenía ruta a 10.2.50.0/24 y el ping daba timeout (prueba 7).

## 7. Servicios (pendiente)

### Enrutamiento y relay
- Cada subinterfaz de clientes usa `ip helper-address 10.1.100.10`.
- VLAN 100 (servidores) no tiene helper-address.

### DHCP y DNS en Windows Server 2019 (10.1.100.10)
- Un ámbito DHCP por cada VLAN de clientes (10, 40, 20, 30 y 50).
- Gateway de cada ámbito: la IP de la subinterfaz correspondiente.
- DNS: zona `technomarket.lan` y registro `intranet` hacia 10.1.100.20.

### Ubuntu 22.04 (10.1.100.20)
- nginx publica el sitio corporativo en el nombre `intranet.technomarket.lan`.

### Controles de acceso (pendiente)
- Soporte hacia Ventas: bloqueado.
- Ventas y Soporte hacia Administración y Finanzas: bloqueado.
- HTTP permitido para Ventas, TI y Administración y Finanzas.
- HTTP bloqueado para Soporte.

Borrador de ACL para R2 (pendiente de aplicar y probar):
```
ip access-list extended VENTAS_IN
 deny ip 10.2.20.0 0.0.0.255 10.1.10.0 0.0.0.255
 permit ip any any
!
ip access-list extended SOPORTE_IN
 deny ip 10.2.30.0 0.0.0.255 10.2.20.0 0.0.0.255
 deny ip 10.2.30.0 0.0.0.255 10.1.10.0 0.0.0.255
 deny tcp 10.2.30.0 0.0.0.255 host 10.1.100.20 eq 80
 permit ip any any
!
interface FastEthernet0/0.20
 ip access-group VENTAS_IN in
interface FastEthernet0/0.30
 ip access-group SOPORTE_IN in
```

## 8. Estado de la infraestructura

- GNS3: proyecto `Desafio8-OSPF` abierto y con los cuatro nodos encendidos (R1, R2, Switch1 y Switch2).
- VMware:
  - Windows Server 2019: encendida, con sesión iniciada. Adaptador en NAT (pendiente de cambiar a la red que se use con GNS3).
  - Windows 10 x64: apagada. Adaptador en NAT.
  - Ubuntu 22.04: instalado. nginx activo. Sin IP estática todavía. Se usó temporalmente el adaptador NAT para descargar nginx; debe volver a VMnet2 o a la red de GNS3.
- Conexión VMware <-> GNS3: pendiente. Se planea un nodo Cloud en GNS3 enlazado a VMnet2, o usar otra VMnet.

## 9. Pruebas para la demostración

1. `show ip ospf neighbor` en R1 y R2.
2. `show ip route ospf` en R1 y R2.
3. Obtener IP en Ventas y comprobar gateway y DNS.
4. Resolver `intranet.technomarket.lan` y abrir el sitio desde Ventas.
5. Comprobar el bloqueo HTTP desde Soporte.
6. Comprobar el bloqueo de Soporte hacia Ventas.
7. Mostrar que R1 no tiene ruta hacia 10.2.50.0/24 antes de anunciar Bodega.
8. Anunciar Bodega y comprobar ruta, DHCP y conectividad.
9. Ejecutar la modificación solicitada por el profesor.

### Cambios para ensayar
- Crear otra VLAN.
- Modificar una ACL.
- Cambiar el costo OSPF.

### Conceptos que ambos deben explicar
- Adyacencia OSPF, DR/BDR, Hello y Dead
- Área 0, costo y router ID
- Passive-interface y sentencias network
- Diferencia entre OSPF y rutas estáticas
- Funcionamiento de DHCP Relay y giaddr

## 10. Pendientes
- Decidir si Bodega se anuncia desde el inicio o en vivo, y guardar la configuración de acuerdo a eso.
- Cambiar el adaptador de Windows Server 2019 y Windows 10 a la red que conecte con GNS3.
- Configurar IP estática 10.1.100.20 en Ubuntu y publicar el sitio `intranet.technomarket.lan` en nginx.
- Configurar DHCP y DNS en Windows Server 2019.
- Aplicar y probar las ACL VENTAS_IN y SOPORTE_IN.
- Ensayar la demostración completa de 15 minutos.

## CAMBIO DE DECISIÓN (19-nov-2026): DHCP en R1 y R2 en vez de Windows Server

- **Motivo:** Windows Server 2019 (10.1.100.10) no pudo autorizar DHCP en Active Directory. Errores: "El servicio DHCP no puede establecer contacto con Active Directory" en la consola, y `Add-DhcpServerInDC` devuelve `PermissionDenied` ("No se pudieron inicializar los recursos de servicio de directorio para el dominio technomarket.lan"). `nltest /dsgetdc:technomarket.lan` sí encuentra el DC, así que AD y DNS funcionan; el bloqueo está en los permisos de inicialización de DHCP. Se intentó: reinicio del servicio, reinicio del servidor, `netsh dhcp add securitygroups`, `netsh dhcp add server`.
- **Decisión:** DHCP lo sirve R1 con `ip dhcp pool`. Windows Server queda solo como DNS y controlador de dominio (`intranet.technomarket.lan` → 10.1.100.20).
- **Santiago (VLAN 10 y 40):** conectados a R1, pools locales en R1, sin relay.
- **Valparaíso (VLAN 20, 30 y 50):** R2 hace relay a R1 con `ip helper-address 10.255.12.1` (antes apuntaba a 10.1.100.10).
- **Exclusiones:** 10.x.x.1 a 10.x.x.99 en cada subred. Pools de .100 a .200 por defecto de IOS.
- **Pendiente para la presentación:** explicar que el DHCP está en el router por limitación de autorización de AD, y que el Windows Server cumple como DNS y controlador de dominio.
- Los comentarios "CAMBIO 19-nov-2026" en los archivos de configuración indican exactamente qué cambió.

## Estado al 20-nov-2026 (para retomar)

**Listo y verificado:**
- Cloud1 conectado a VMnet2 y al puerto 3 de Switch1 (VLAN 100, access). Ping de Windows Server a R1 (10.1.100.1) responde 4/4.
- R1: OSPF FULL con R2. DHCP local con pools VLAN 10, 40 (Santiago), 20, 30, 50 (Valparaíso). Exclusiones correctas (.1 a .99 de cada subred).
- Windows 10 en puerto 1 de Switch1 (VLAN 10): recibe 10.1.10.100, puerta 10.1.10.1, sufijo technomarket.lan. R1 le hace ping 5/5.
- Windows Server 2019 (WIN-N7GHMKRHJAN): IP 10.1.100.10/24, puerta 10.1.100.1, promovido a DC del dominio technomarket.lan. nltest encuentra el DC.
- R2: relay DHCP a 10.255.12.1 en VLAN 20, 30 y 50.

**Pendiente / problema abierto:**
- R1 no alcanza 10.1.100.10 (ping 0/5). El Windows Server sí responde a R1 por su propia salida, pero no al ping entrante. Se creó la regla de firewall ICMPv4 entrada (netsh advfirewall firewall add rule ... protocol=icmpv4 dir=in action=allow), pero el ping sigue sin responder. Revisar perfil de red del servidor y estado del firewall.
- DHCP en Windows Server no se pudo autorizar en AD (PermissionDenied al inicializar recursos). Se decidió DHCP en R1 y R2. Ver sección CAMBIO más arriba.
- Pendiente: IP fija de Ubuntu (10.1.100.20), registro DNS intranet → 10.1.100.20, ACL VENTAS_IN y SOPORTE_IN, prueba de relay desde Valparaíso, decisión sobre Bodega (anunciar desde inicio o en vivo) y write memory en R1 y R2.

## Avance 20-nov-2026 (tarde)
- Relay DHCP de Valparaíso probado con VPCS: PC1 (VLAN 20) → 10.2.20.100; PC2 (VLAN 30) → 10.2.30.100. Servidor DHCP 10.255.12.1 (R1).
- ACL aplicadas en R2 y probadas: VENTAS_IN bloquea 10.2.20.0/24 → 10.1.10.0/24; SOPORTE_IN bloquea 10.2.30.0/24 → 10.2.20.0/24 y → 10.1.10.0/24 y HTTP a 10.1.100.20. Permitido 10.2.20 → 10.2.30 y 10.2.30 → 10.2.30.1.
- Pendiente de ACL: prueba HTTP desde VLAN 30 a intranet (requiere nginx en Ubuntu, paso del compañero).
- Pendiente: Bodega (anunciar desde inicio o en vivo), write memory en R1 y R2 cuando se confirme, ping a 10.1.100.10 desde R1.
