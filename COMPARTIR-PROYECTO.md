# Compartir el proyecto y las VMs con el compañero

## En GitHub (esta carpeta)
- `gns3-project/proyecto-compartido/`: proyecto GNS3 oficial (`Desafio8-OSPF.gns3`, copiado del PC el 23-11-2026, con nombres finales y recuadros por VLAN), startups de VPCS (PC-TI, PC-Ventas, PC-Soporte, PC-Bodega) y startups de R1 y R2 (incluye las 3 ACL).
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
1. Instalar VMware Workstation y crear VMnet2 (host-only, sin DHCP, 192.168.71.0/24). Crearlo **antes** de abrir GNS3.
   - ⚠️ Revisar con `ipconfig` la IP del adaptador "VMware Network Adapter VMnet2" del PC. **No puede ser 10.1.100.1**, porque choca con el gateway de R1 y Windows Server deja de responder. Si lo es, cambiarla a 10.1.100.254 (ver `guia-vmware-gns3.md`, sección 3.2).
2. Poner la imagen IOS c2691 en la plantilla de GNS3 (Dynamips, 192 MiB) y seleccionarla.
3. Copiar el proyecto a su carpeta de GNS3 y abrir `Desafio8-OSPF.gns3`. Revisar que Cloud-VMnet2 use "VMware Network Adapter VMnet2".
4. El nodo Win10-Admin apunta a `C:\Users\megav\Desktop\Maquinas\Windows 10 x64\Windows 10 x64.vmx`. En otro PC, hay que volver a elegir la VM en la configuración del nodo (clic derecho → Configure).

## Ubicación en Google Drive
Carpeta `TAS` (contiene `Windows Server 2019`, `Windows 10 x64` y `Ubuntu (64 bits)`):
https://drive.google.com/drive/u/0/folders/1ug4Siy7FCB9hwsE6Q3upiuEVoh0WSfI8
