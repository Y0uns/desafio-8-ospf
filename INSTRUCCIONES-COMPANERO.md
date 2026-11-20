# Instrucciones para el compañero: Desafío 8 (OSPF entre sedes)

Este documento explica cómo abrir el proyecto, en qué estado está y qué falta. Lee primero `contexto-desafio-8.md`.

## 1. Requisitos
- GNS3 2.2.55.
- Imagen Cisco `c2691-adventerprisek9_sna-mz.124-13b.image` (no está en el repositorio; la consigues con el profesor o en el aula virtual).
- VMware Workstation 25H2, con Windows Server 2019, Windows 10 y Ubuntu 22.04.5 Desktop.

## 2. Abrir el proyecto en GNS3
1. Instala la imagen Cisco como plantilla Dynamips c2691 con 192 MiB de RAM.
2. Copia la carpeta `gns3-project/` a tu equipo.
3. En GNS3: File > Open project > `Desafio8-OSPF.gns3`.
4. Si la plantilla te pide otra ruta de imagen, corrígela en la plantilla del router.
5. Enciende R1 y R2 (Start all).

## 3. Verificar que OSPF funciona
Abre la consola de cada router (clic derecho > Console) y ejecuta:

**En R1:**
- `show ip ospf neighbor`: debe aparecer 2.2.2.2 en FULL por FastEthernet0/1.
- `show ip route ospf`: debe mostrar 10.2.20.0/24, 10.2.30.0/24 y 10.2.50.0/24 vía 10.255.12.2.

**En R2:**
- `show ip ospf neighbor`: debe aparecer 1.1.1.1 en FULL.
- `show ip route ospf`: debe mostrar 10.1.10.0/24, 10.1.40.0/24 y 10.1.100.0/24 vía 10.255.12.1.

Si no ves vecinos, revisa `show ip interface brief` y que FastEthernet0/1 esté `up/up` en ambos.

## 4. Consideraciones importantes
- **Bodega (10.2.50.0/24):** en la configuración guardada de R2 ya está anunciada. La prueba 7 de la demostración pide que R1 NO tenga ruta a esa red antes de anunciarla. Antes de probar, quita el anuncio en R2:
  ```
  configure terminal
  router ospf 1
  no network 10.2.50.0 0.0.0.255 area 0
  end
  ```
  Luego vuelve a anunciarla durante la demostración con `network 10.2.50.0 0.0.0.255 area 0`.
- **No guardes Bodega sin decidirlo:** si haces `write memory` con Bodega anunciada, la prueba 7 no se puede mostrar después de reiniciar.
- Las interfaces de las VLAN de clientes (10, 40, 20, 30 y 50) son pasivas. Eso es correcto: no deben formar adyacencias.

## 5. Estado de la infraestructura
- **Switches de GNS3:** Switch1 y Switch2 ya tienen las VLAN de acceso y el troncal configurados (ver `contexto-desafio-8.md`, sección 4).
- **VMware:**
  - Windows Server 2019: instalada, con su sesión iniciada. Hoy usa NAT; hay que cambiarla a la red que conecte con GNS3.
  - Windows 10 x64: instalada, apagada. Usa NAT.
  - Ubuntu 22.04: instalado, con nginx activo (`systemctl status nginx` muestra `active (running)`). Todavía no tiene IP estática 10.1.100.20. Para descargar nginx se usó temporalmente el adaptador NAT; antes de conectar a GNS3 debe volver a VMnet2 o a la red del nodo Cloud.
- **Conexión VMware <-> GNS3:** pendiente.

## 6. Qué falta (en orden)
1. Asignar IP estática 10.1.100.20 a Ubuntu (Netplan) cuando esté conectada a GNS3 en VLAN 100.
2. Publicar el sitio en nginx para `intranet.technomarket.lan` (nginx ya está instalado y activo). Anota usuario y contraseña en un lugar privado, no en el repositorio.
3. Conectar Windows Server (10.1.100.10) a Switch1 en VLAN 100, y Windows 10 a una VLAN de cliente.
4. En Windows Server: DNS con la zona `technomarket.lan` (creada al promover el DC) y el registro `intranet` → 10.1.100.20. **DHCP ya no va en Windows Server**: R1 y R2 sirven DHCP (ver CAMBIO en contexto-desafio-8.md).
5. Probar el relay: un cliente de Valparaíso debe obtener IP desde Windows Server a través de R2 (`ip helper-address 10.1.100.10`).
6. Aplicar las ACL `VENTAS_IN` y `SOPORTE_IN` (borrador en `contexto-desafio-8.md`, sección 6) y probar tráfico permitido y denegado.
7. Ensayar la demostración de 15 minutos. El profesor puede cambiar algo en vivo: practica al menos una VLAN nueva y un cambio de ACL.

## 7. Notas
- No subas contraseñas, ISO, imágenes de Cisco ni discos de VM al repositorio.
- Cualquier cambio a la configuración de R1 o R2, guárdalo con `write memory` y actualiza `gns3-project/` para que quede sincronizado.

## Cambio 19-nov-2026
- DHCP movido a R1 (pools por VLAN) porque Windows Server no pudo autorizar DHCP en AD. R2 hace relay a R1 (10.255.12.1). Detalles en contexto-desafio-8.md.

## Estado al 20-nov-2026 (retomar desde aquí)
- Ya funciona: GNS3 con Cloud1 → VMnet2, Switch1 puerto 3 VLAN 100, Windows Server en 10.1.100.10 (DC), DHCP en R1 para VLAN 10/40/20/30/50 (R2 hace relay a 10.255.12.1). Cliente Windows 10 en VLAN 10 recibe IP y sale por 10.1.10.1.
- Problema abierto: R1 no puede hacer ping a 10.1.100.10 (el servidor no responde ICMP entrante). Revisar firewall/perfil de red de Windows Server antes de seguir.
- Falta: IP fija Ubuntu 10.1.100.20, registro DNS intranet, ACL, prueba de relay desde Valparaíso y decisión sobre Bodega.
- Detalles completos en contexto-desafio-8.md.

## Reparto de tareas (20-nov-2026)

**Tareas del compañero (requieren las VMs Windows y Ubuntu):**
- Resolver el ping de R1 a 10.1.100.10 (firewall o perfil de red de Windows Server). Ya se creó la regla ICMPv4 entrante, pero R1 aún no llega.
- Crear el registro DNS `intranet` → 10.1.100.20 en la zona `technomarket.lan` del servidor.
- IP fija de Ubuntu: 10.1.100.20/24, puerta 10.1.100.1, DNS 10.1.100.10. Verificar que nginx responda en intranet.technomarket.lan.
- Probar Windows 10 (VLAN 10) accediendo a la intranet.

**Tareas del líder (solo routers y VPCS, poca RAM):**
- Probar el relay desde Valparaíso con un VPCS en el puerto 1 de Switch2 (VLAN 20): `ip dhcp` y `show ip` deben dar 10.2.20.100–200 con puerta 10.2.20.1.
- Aplicar las ACL VENTAS_IN y SOPORTE_IN en R2 (borrador en contexto-desafio-8.md) y probar con VPCS: VLAN 20 → 10.1.10.1 denegado; VLAN 30 → 10.2.20.1 denegado; VLAN 20 → 10.2.30.1 permitido.
- Decidir si Bodega se anuncia desde el inicio o en vivo, y documentarlo.
- Preparar la demo de 15 minutos: comandos a mostrar (`show ip ospf neighbor`, `show ip route ospf`, `show ip dhcp binding`, pruebas de ping) y la explicación del cambio de DHCP a los routers.

