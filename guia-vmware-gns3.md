# Contexto del proyecto — TechnoMarket Ltda. (GNS3 + VMware)

**Curso:** EIN-090B — USM · **Equipo 8** · **Proyecto GNS3:** `Desafio8-OSPF`
**Última verificación completa:** 23-11-2026 (todas las pruebas OK)

Este archivo resume la configuración final y todo lo que hubo que corregir para que las máquinas virtuales de VMware funcionen dentro de la topología de GNS3.

---

## 1. Software utilizado

| Componente | Versión / detalle |
|---|---|
| GNS3 | 2.2.55 (Windows), servidor local "CraKen" |
| Routers | Cisco c2691 (Dynamips, IOS 12.4) |
| Switches | Ethernet switch simple de GNS3 (no Cisco IOS) |
| Hipervisor | VMware Workstation Pro |
| Captura de red | Npcap (lo necesita el nodo Cloud de GNS3) |
| Servidor | Windows Server 2019: AD DS, DNS (dominio `technomarket.lan`) |
| Intranet | Ubuntu 22.04 + nginx |
| Cliente | Windows 10 (nodo VMware dentro de GNS3: **Win10-Admin**) |
| PCs de prueba | VPCS: PC-TI, PC-Ventas, PC-Bodega, PC-Soporte |

---

## 2. Topología y direccionamiento

```
                 WAN 10.255.12.0/30 - OSPF área 0
 RTR-Santiago (1.1.1.1) .1 ───────────── .2 RTR-Valparaiso (2.2.2.2)
        │ Fa0/0 (trunk 802.1Q)                    │ Fa0/0 (trunk 802.1Q)
   SW-Santiago ─── Cloud-VMnet2              SW-Valparaiso
    │       │        (WS2019 + Ubuntu)        │      │       │
 Win10-Admin PC-TI                        PC-Ventas PC-Bodega PC-Soporte
```

| VLAN | Sector | Sede | Red | Gateway | Equipos |
|---|---|---|---|---|---|
| 10 | Administración | Santiago | 10.1.10.0/24 | 10.1.10.1 | Win10-Admin (10.1.10.100, DHCP) |
| 40 | TI | Santiago | 10.1.40.0/24 | 10.1.40.1 | PC-TI (10.1.40.100, DHCP) |
| 100 | Servidores | Santiago | 10.1.100.0/24 | 10.1.100.1 | Windows Server 2019 **10.1.100.10** (estática)<br>Ubuntu nginx **10.1.100.20** (estática) |
| 20 | Ventas | Valparaíso | 10.2.20.0/24 | 10.2.20.1 | PC-Ventas (10.2.20.100, DHCP relay) |
| 30 | Soporte | Valparaíso | 10.2.30.0/24 | 10.2.30.1 | PC-Soporte (10.2.30.100, DHCP relay) |
| 50 | Bodega | Valparaíso | 10.2.50.0/24 | 10.2.50.1 | PC-Bodega (10.2.50.100, DHCP relay). Red reservada para la demo de OSPF |
| — | WAN | Stgo ↔ Valpo | 10.255.12.0/30 | — | RTR-Santiago .1 / RTR-Valparaiso .2 (ambos Fa0/1) |

- DNS para todas las VLAN: **10.1.100.10**. Dominio: **technomarket.lan**.
- DHCP centralizado en RTR-Santiago. Los rangos .1 a .99 están excluidos, así que los clientes reciben desde .100.
- Registro DNS clave: `intranet.technomarket.lan` → A → **10.1.100.20**.

---

## 3. Configuración de VMware (lo que hizo funcionar las VMs)

### 3.1 Crear VMnet2 (host-only)
VMnet2 **no existía**. Por eso GNS3 mostraba:
`Could not find NPF id for interface VMware Network Adapter VMnet2`

**Solución:** VMware → Editar → **Editor de red virtual** → *Agregar red* → **VMnet2** → tipo **Solo host**.
- **Desmarcar** "Usar servicio DHCP local" (el DHCP del proyecto lo dan los routers).
- Comprobar en PowerShell que el adaptador existe: `Get-NetAdapter` debe mostrar *VMware Network Adapter VMnet2* en estado **Up**.

> VMnet2 debe existir **antes** de abrir GNS3. Si se crea con GNS3 abierto, hay que cerrar GNS3 y volver a abrirlo.

### 3.2 IP del adaptador VMnet2 en el PC anfitrión ⚠️ (problema crítico)
VMware le asignó a VMnet2 la IP **10.1.100.1** en el PC anfitrión, que es **la misma IP que el gateway del router** (Fa0/0.100).

**Síntomas:**
- Windows Server no respondía a ping desde GNS3, aunque el firewall estuviera apagado.
- Ubuntu (.20) sí respondía, porque su caché ARP tenía la MAC del router.
- Desde el PC anfitrión, el ping a .10 respondía en 0 ms con TTL 128 (respondía el propio PC, no el router).

**Solución** (PowerShell **como administrador** en el PC anfitrión):
```powershell
netsh interface ip set address "VMware Network Adapter VMnet2" static 10.1.100.254 255.255.255.0
```
Luego, en Windows Server: `arp -d *` y `ping 10.1.100.1`.
✅ Señal de que quedó bien: la respuesta trae **TTL=255** y entre 30 y 60 ms, que corresponde al router Cisco en GNS3.

> **Revisar cada vez que se reinicie el PC:** `ipconfig` debe mostrar VMnet2 = **10.1.100.254**. Si vuelve a aparecer .1, repetir el comando.

### 3.3 Adaptador de red de cada VM
| VM | Configuración en VMware → Adaptador de red |
|---|---|
| Windows Server 2019 | Personalizado → **VMnet2 (Solo host)**, Conectado + Conectar al encender |
| Ubuntu nginx | Personalizado → **VMnet2 (Solo host)** |
| Windows 10 (Win10-Admin) | La administra GNS3 como nodo VMware y la conecta a SW-Santiago e1 |

### 3.4 Disco de Windows 10 (error al encender)
**Error:** `No se ha encontrado el archivo: Windows 10 x64.vmdk` y luego `No se pudo abrir el elemento primario de este disco virtual`.
- La cadena de snapshots estaba rota: no existía el disco base, solo `Windows 10 x64-004.vmdk` (9,55 GB) y `Windows 10 x64-000001-005.vmdk` (2,40 GB).
- **Solución:** en Configuración de la VM → Disco duro, se **seleccionó directamente el disco existente**. Después de eso la VM arrancó.

### 3.5 Firewall de Windows Server
Se desactivó para las pruebas:
```powershell
netsh advfirewall set allprofiles state off
```
Alternativa más segura (firewall encendido, solo permite ping):
```powershell
netsh advfirewall set allprofiles state on
netsh advfirewall firewall add rule name="ICMP Allow" protocol=icmpv4:8,any dir=in action=allow
```
En Win10-Admin también se agregó la regla ICMP, para que responda ping en la prueba de ACL.

---

## 4. Configuración de GNS3

### 4.1 Nodo Cloud-VMnet2
- Interfaz: **VMware Network Adapter VMnet2**. Si muestra un triángulo de advertencia, VMnet2 no existe o Npcap no lo detecta (ver 3.1).
- Conectado a **SW-Santiago puerto e3**.

### 4.2 SW-Santiago (switch simple de GNS3)
| Puerto | VLAN | Tipo | Conectado a |
|---|---|---|---|
| e0 | 1 | **dot1q** (trunk) | RTR-Santiago Fa0/0 |
| e1 | 10 | access | Win10-Admin |
| e2 | 40 | access | PC-TI |
| e3 | 100 | access | Cloud-VMnet2 (servidores) |

### 4.3 Recomendaciones
- Abrir GNS3 **sin** "Ejecutar como administrador". Si se abre como administrador, Claude no puede controlarlo con computer use.
- Topología decorada: recuadros de color por VLAN y etiqueta del enlace WAN (guardado en el proyecto).

---

## 5. Configuración de los routers (resumen)

### RTR-Santiago (R1, router-id 1.1.1.1)
```
interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.1.10.1 255.255.255.0
interface FastEthernet0/0.40
 encapsulation dot1Q 40
 ip address 10.1.40.1 255.255.255.0
 ip access-group SOPORTE_NO_VENTAS in
interface FastEthernet0/0.100
 encapsulation dot1Q 100
 ip address 10.1.100.1 255.255.255.0
interface FastEthernet0/1
 ip address 10.255.12.1 255.255.255.252

router ospf 1
 router-id 1.1.1.1
 passive-interface Fa0/0.10 / Fa0/0.40 / Fa0/0.100
 network 10.1.10.0 0.0.0.255 area 0
 network 10.1.40.0 0.0.0.255 area 0
 network 10.1.100.0 0.0.0.255 area 0
 network 10.255.12.0 0.0.0.3 area 0

ip dhcp excluded-address 10.1.10.1 10.1.10.99   (igual para 10.1.40, 10.2.20, 10.2.30, 10.2.50)
ip dhcp pool VLAN10-Santiago-Ventas   -> 10.1.10.0/24, gw .1, dns 10.1.100.10, domain technomarket.lan
ip dhcp pool VLAN40-Santiago-Soporte  -> 10.1.40.0/24
ip dhcp pool VLAN20-Valparaiso        -> 10.2.20.0/24
ip dhcp pool VLAN30-Valparaiso        -> 10.2.30.0/24
ip dhcp pool VLAN50-Valparaiso        -> 10.2.50.0/24

ip access-list extended SOPORTE_NO_VENTAS
 10 deny ip 10.1.40.0 0.0.0.255 10.1.10.0 0.0.0.255
 20 permit ip any any
```
> Los nombres internos vienen del primer plan y no coinciden con los sectores finales: `VLAN10-Santiago-Ventas` = VLAN 10 Administración, `VLAN40-Santiago-Soporte` = VLAN 40 TI, `VLAN50-Valparaiso` = VLAN 50 Bodega, y `SOPORTE_NO_VENTAS` bloquea TI → Administración. Se dejaron así para no tocar el router antes de la demo. La tabla completa está en `INSTRUCCIONES-COMPANERO.md`.

### RTR-Valparaiso (R2, router-id 2.2.2.2)
```
interface FastEthernet0/0.20  -> 10.2.20.1/24, ip helper-address 10.255.12.1, ip access-group VENTAS_IN in
interface FastEthernet0/0.30  -> 10.2.30.1/24, ip helper-address 10.255.12.1, ip access-group SOPORTE_IN in
interface FastEthernet0/0.50  -> 10.2.50.1/24, ip helper-address 10.255.12.1
interface FastEthernet0/1     -> 10.255.12.2/30

router ospf 1
 router-id 2.2.2.2
 passive-interface Fa0/0.20 / Fa0/0.30 / Fa0/0.50
 network 10.2.20.0 0.0.0.255 area 0
 network 10.2.30.0 0.0.0.255 area 0
 network 10.255.12.0 0.0.0.3 area 0
 ! 10.2.50.0 se agrega EN VIVO en la demo (no guardar con write memory)

ip access-list extended VENTAS_IN
 10 deny ip 10.2.20.0 0.0.0.255 10.1.10.0 0.0.0.255
 20 permit ip any any
ip access-list extended SOPORTE_IN
 10 deny ip 10.2.30.0 0.0.0.255 10.2.20.0 0.0.0.255
 20 deny ip 10.2.30.0 0.0.0.255 10.1.10.0 0.0.0.255
 30 deny tcp 10.2.30.0 0.0.0.255 host 10.1.100.20 eq www
 40 permit ip any any
```

---

## 6. Orden de encendido (antes de cada demo)

1. En el PC anfitrión, `ipconfig`: VMnet2 debe ser **10.1.100.254** (si no, ver 3.2).
2. Abrir **VMware** y encender **Windows Server 2019** y **Ubuntu**. Esperar a que inicien.
3. Abrir **GNS3** (sin administrador) y el proyecto `Desafio8-OSPF`.
4. Iniciar todos los nodos (▶). Esperar a que estén en verde.
5. En PC-TI: `ip dhcp`. Si el DNS no queda configurado: `ip dns 10.1.100.10`.
6. Comprobación rápida desde RTR-Santiago: `ping 10.1.100.10` y `ping 10.1.100.20` (5/5).

---

## 7. Pruebas verificadas (23-11-2026)

| Prueba | Comando | Resultado |
|---|---|---|
| Vecino OSPF | `show ip ospf neighbor` | 2.2.2.2 en FULL ✅ |
| Rutas OSPF | `show ip route ospf` | Stgo aprende 10.2.20/30; Valpo aprende 10.1.10/40/100 ✅ |
| DHCP Santiago | PC-TI `ip dhcp` | 10.1.40.100, DNS 10.1.100.10 ✅ |
| DHCP relay Valpo | PC-Ventas / PC-Soporte `ip dhcp` | 10.2.20.100 / 10.2.30.100, servidor 10.255.12.1 ✅ |
| Conectividad entre sedes | PC-Ventas → 10.1.100.10 / .20 | 5/5 (TTL 126 / 62) ✅ |
| DNS | `ping intranet.technomarket.lan` | Resuelve a 10.1.100.20 desde ambas sedes ✅ |
| AD | `Get-ADDomain` | technomarket.lan, Windows2016Domain ✅ |
| nginx | `systemctl status nginx` | active (running) ✅ |
| Intranet web | Win10 → `http://intranet.technomarket.lan` | "Bienvenido a la intranet de technoMarket Ltda." ✅ |
| ACL Stgo | PC-TI → 10.1.10.100 | Bloqueado: `administratively prohibited` (10.1.40.1) ✅ |
| ACL Valpo | PC-Ventas → 10.1.10.100 | Bloqueado por VENTAS_IN (10.2.20.1) ✅ |
| Control ACL | RTR-Santiago → 10.1.10.100 | 5/5 (Win10 responde) ✅ |

---

## 8. Guion de la demo: incorporar la red de Bodega (variante del Equipo 8)

1. Encender PC-Bodega y ejecutar `ip dhcp`. **Falla**, porque Santiago no tiene ruta a 10.2.50.0 y no puede devolver la oferta DHCP.
2. RTR-Santiago: `show ip route ospf`. No aparece 10.2.50.0.
3. RTR-Valparaiso:
   ```
   conf t
   router ospf 1
   network 10.2.50.0 0.0.0.255 area 0
   end
   ```
4. RTR-Santiago: `show ip route ospf`. Aparece `O 10.2.50.0/24 via 10.255.12.2` ✅
5. PC-Bodega: `ip dhcp` recibe 10.2.50.100, y `ping intranet.technomarket.lan` responde ✅

**Volver al estado inicial después de ensayar:** `no network 10.2.50.0 0.0.0.255 area 0`. En PC-Bodega, `clear ip` y apagarlo. **No** hacer `write memory` en RTR-Valparaiso.

---

## 9. Solución rápida de problemas

| Síntoma | Causa | Solución |
|---|---|---|
| Cloud-VMnet2 con triángulo / "Could not find NPF id" | VMnet2 no existe o se creó después de abrir GNS3 | Crearlo en el Editor de red virtual y reiniciar GNS3 (3.1) |
| WS2019 no responde desde GNS3 y Ubuntu sí | El PC anfitrión tiene 10.1.100.1 en VMnet2 | Cambiar a 10.1.100.254 (3.2) |
| Ping a 10.1.100.x falla en todas las VMs | VM no está en VMnet2 o SW e3 no está en VLAN 100 | Revisar 3.3 y 4.2 |
| `Cannot resolve intranet.technomarket.lan` en VPCS | Falta DNS o el servidor .10 no responde | `ip dns 10.1.100.10` y verificar ping a .10 |
| `nslookup` muestra "Servidor: UnKnown" | Falta la zona inversa (solo estético) | `Add-DnsServerPrimaryZone -NetworkId "10.1.100.0/24" -ReplicationScope Forest` |
| Win10 no enciende: falta .vmdk | Cadena de snapshots rota | Seleccionar el disco existente en Configuración (3.4) |
| PCs de Valparaíso no reciben IP | Falta `ip helper-address` o ruta OSPF | Revisar Fa0/0.x en R2 y `show ip route ospf` en R1 |
| Claude no puede hacer clic en GNS3 | GNS3 abierto como administrador | Abrirlo normal (4.3) |
| Router no resuelve nombres | IOS no usa DNS por defecto | Normal: hacer ping a la IP |

---

## 10. Guardar cambios
- Routers: `write memory` (en RTR-Valparaiso, **solo** si no se quiere repetir la demo de la VLAN 50).
- GNS3: **Ctrl+S**.
- VMs: apagarlas desde el sistema operativo antes de cerrar VMware.
