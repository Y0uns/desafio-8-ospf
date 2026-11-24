# Desafío 8: OSPF entre sedes (TechnoMarket Ltda.)

Proyecto del ramo Taller de Administración de Sistemas (USM, EIN-090B). Equipo 8, variante OSPF entre sedes.

## Contenido
- `contexto-desafio-8.md`: contexto completo del proyecto, decisiones, topología, direccionamiento, configuración verificada en R1 y R2, estado de la infraestructura y pendientes.
- `guia-vmware-gns3.md`: cómo dejar funcionando las VMs de VMware dentro de GNS3 (VMnet2, conflicto de IP 10.1.100.1, disco de Win10, puertos del switch), orden de encendido, pruebas verificadas el 23-11-2026, guion de la demo de Bodega y solución de problemas.

## Simulación
- GNS3 2.2.55 con routers Cisco c2691 (imagen c2691-adventerprisek9_sna-mz.124-13b.image, no incluida).
- SW-Santiago (antes Switch1) y SW-Valparaiso (antes Switch2) de GNS3. La tabla de equivalencias de nombres está en `INSTRUCCIONES-COMPANERO.md`.
- Servidores en VMware Workstation: Windows Server 2019 (Active Directory y DNS) y Ubuntu 22.04 (nginx). El DHCP lo entrega R1 (RTR-Santiago); R2 hace relay.

## Nota
Este repositorio no incluye imágenes de Cisco, ISO de sistemas operativos, discos de máquinas virtuales ni contraseñas.

## Proyecto GNS3 (carpeta `gns3-project/`)
- **`proyecto-compartido/Desafio8-OSPF.gns3`: proyecto oficial y actualizado (23-11-2026), con nombres finales, PCs por VLAN y recuadros de color. Usar este.**
- `Desafio8-OSPF.gns3` (raíz de la carpeta): versión inicial solo con R1, R2, Switch1 y Switch2. Se mantiene como referencia.
- `R1-startup-config.cfg` y `R2-startup-config.cfg`: configuraciones de inicio comentadas de los routers (VLAN, OSPF, DHCP, relay y ACL). Coinciden con las del proyecto compartido.

### Para abrirlo
1. Instala la imagen `c2691-adventerprisek9_sna-mz.124-13b.image` en GNS3 como plantilla Dynamips c2691 (RAM 192 MiB). No está en el repositorio.
2. En GNS3: File > Open project > `Desafio8-OSPF.gns3`. Si la ruta de la imagen es distinta en tu equipo, corrígela en la plantilla.
3. Enciende los routers. Si quieres partir desde cero, revisa los archivos `.cfg` y aplícalos en la consola de cada router.
