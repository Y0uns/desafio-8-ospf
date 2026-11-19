# Desafío 8: OSPF entre sedes (TechnoMarket Ltda.)

Proyecto del ramo Taller de Administración de Sistemas (USM, EIN-090B). Equipo 8, variante OSPF entre sedes.

## Contenido
- `contexto-desafio-8.md`: contexto completo del proyecto, decisiones, topología, direccionamiento, configuración verificada en R1 y R2, estado de la infraestructura y pendientes.

## Simulación
- GNS3 2.2.55 con routers Cisco c2691 (imagen c2691-adventerprisek9_sna-mz.124-13b.image, no incluida).
- Switch1 (Santiago) y Switch2 (Valparaíso) de GNS3.
- Servidores en VMware Workstation: Windows Server 2019 (DHCP y DNS) y Ubuntu 22.04 (nginx).

## Nota
Este repositorio no incluye imágenes de Cisco, ISO de sistemas operativos, discos de máquinas virtuales ni contraseñas.

## Proyecto GNS3 (carpeta `gns3-project/`)
- `Desafio8-OSPF.gns3`: topología con R1, R2, Switch1 y Switch2 y sus enlaces.
- `R1-startup-config.cfg` y `R2-startup-config.cfg`: configuraciones de inicio de los routers (VLAN, OSPF y relay).

### Para abrirlo
1. Instala la imagen `c2691-adventerprisek9_sna-mz.124-13b.image` en GNS3 como plantilla Dynamips c2691 (RAM 192 MiB). No está en el repositorio.
2. En GNS3: File > Open project > `Desafio8-OSPF.gns3`. Si la ruta de la imagen es distinta en tu equipo, corrígela en la plantilla.
3. Enciende los routers. Si quieres partir desde cero, revisa los archivos `.cfg` y aplícalos en la consola de cada router.
