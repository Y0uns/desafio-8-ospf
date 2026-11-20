# Compartir el proyecto y las VMs con el compañero

## En GitHub (esta carpeta)
- `gns3-project/proyecto-compartido/`: proyecto GNS3 (`Desafio8-OSPF.gns3`), startups de VPCS (PC1-PC3) y startups de R1 y R2.
- No incluye el IOS c2691 ni archivos de runtime (logs, locks, ROM, NVRAM, ilt_*).

## Fuera de GitHub (~47 GB, no caben en GitHub)
Carpeta `Virtual Machines` de OneDrive:
- `Windows Server 2019\` (~19 GB, incluye Snapshot1.vmem de 4 GB; se puede omitir si no se usa el snapshot)
- `Windows 10 x64\` (~15 GB, incluye Snapshot1.vmem de 2 GB; omitible igual)
- `Ubuntu (64 bits)\` (~13 GB; `autoinst.iso` y `autoinst.flp` son opcionales)
Excluir en todas: `*.log`, `*.scoreboard`, `*.lck`.

## Excluir siempre
- `Desafio8-OSPF\project-files\dynamips\c2691-adventerprisek9_sna-mz.124-13b.image-192.ghost` (IOS de Cisco, con licencia; pasarlo por otro medio).
- Runtime de GNS3 (se regenera): `*_lock`, `*_log.txt`, `*_stdout.txt`, `ubridge.log`, `vpcs.log`, `ilt_*`, `*_rom`, `*_slot0`, `*_slot1`, `*_ssa`, `*_rommon_vars`.

## Para el compañero
1. Instalar VMware Workstation y crear VMnet2 (host-only, sin DHCP, 192.168.71.0/24).
2. Poner la imagen IOS c2691 en la plantilla de GNS3 (Dynamips, 192 MiB) y seleccionarla.
3. Copiar el proyecto a su carpeta de GNS3 y abrir `Desafio8-OSPF.gns3`. Revisar que Cloud1 use "VMware Network Adapter VMnet2".
