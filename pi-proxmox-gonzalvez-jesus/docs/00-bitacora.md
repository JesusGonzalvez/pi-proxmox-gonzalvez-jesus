# Bitacora de sesiones

La bitacora es el registro cronologico del trabajo. Anade una entrada al final
de cada sesion y conserva las anteriores. Escribe datos comprobables y enlaza
las evidencias utiles; evita frases generales como "he trabajado en Proxmox".

## Sesion 01 - 13/10/2026

- Objetivo: iniciar GitHub y la documentacion Markdown.
- Acciones realizadas: Inventario de un equipo fisico con Lliurex ( lscpu, free, lsblk, ip a, df ) y con Windows desde Hiren's Boot (CPU-Z, HWiNFO64 y CrystalDiskInfo). Rellenado el informe 01-inventario-hardware.md
- Decisiones y motivos: Versión prevista Proxmox VE 9.2. Instalación directa en el equipo físico, sin VirtualBox. Uso de ext4/LVM en lugar de ZFS para no gastar RAM en cache. Reparto previsto: 2 GB y 2 vCPU para srv-aso y para cli-linux , con 1,6 GB de RAM de margen. Se prioriza mejorar el disco (SSD) antes que la RAM, porque el HDD es el cuello de botella y el único punto de fallo.
- Pruebas y resultados: Se esperaba confirmar que el equipo cumpliese con los requisitos de Proxmox. Resultado: CPU x86_64 de 64 bits con VT-x (Pentium G4560, 2 nucleos , 4 hilos), 7,6 GiB de RAM, HDD Toshiba DT01ACA050 de 465,8 GB.
- Documentos o evidencias: 01-ficha-inicial.md.
- Incidencias y solucion o estado pendiente: Ninguna.
- Siguiente paso: Continuar con la fase 2.

## Sesion 02 - 20/10/2026

- Objetivo: diseñar la red de Proxmox antes de instalarlo (Fase 2).
- Acciones realizadas: Rellenar el documento 02-diagrama-red.md.
- Decisiones y motivos: Nodo: pve-gonzalvez con vmbr0 (salida a Internet) y vmbr1 (laboratorio 192.168.56.0/24). IP: nodo 192.168.56.20, srv-aso 192.168.56.30, cli-linux 192.168.56.40. Puertos: 8006, 22 y 445.
- Pruebas y resultados: Ninguna todavía; se validarán al instalar Proxmox.
- Documentos o evidencias: 02-diagrama-red.md.
- Incidencias y solucion o estado pendiente: Ninguna.
- Siguiente paso: Fase 3, instalación de Proxmox.

## Sesion 03 - 27/10/2026

- Objetivo: instalar Proxmox VE (Fase 3) en una VM de VirtualBox y configurar su red según 02-diagrama-red.md.
- Acciones realizadas: Crear la VM pve-gonzalvez (6 CPU, 10 GB de RAM, VDI de 80 GB) con adaptador 1 NAT y adaptador 2 solo-anfitrión. Instalar Proxmox VE 9.2 con IP 10.0.2.30/24 (vmbr0) y añadir vmbr1 con 192.168.56.20/24. Acceso al panel de Proxmox en https://192.168.56.20:8006.
- Decisiones y motivos: Instalación virtualizada en VirtualBox en lugar de en el equipo físico (no hay equipos fisicos donde realizarlo). No se mantiene desactivada la seguridad de virtualización de Windows, por lo que no hay KVM anidado.
- Pruebas y resultados: Panel accesible y ip -br addr con vmbr0 10.0.2.30/24 y vmbr1 192.168.56.20/24 en UP. El instalador avisó de que no detecta KVM aunque AMD-V está activo en la BIOS.
- Documentos o evidencias: 03-instalacion-proxmox.md y capturas s03 en evidencias/capturas-o-salidas.
- Incidencias y solucion o estado pendiente: Error 401 en el login por usar el dominio pve en vez de Linux PAM (resuelto). Sin KVM anidado por la seguridad basada en virtualización de Windows (documentado).
- Siguiente paso: Continuar con la fase 4.

Anadir una entrada con esta estructura en cada sesion. El historial de
GitHub registra los commits; no hace falta incluir el hash del propio
commit dentro de su entrada de bitacora.

[Volver al indice](../README.md)
