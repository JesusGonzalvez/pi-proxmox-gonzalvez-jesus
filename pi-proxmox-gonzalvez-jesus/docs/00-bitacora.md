# Bitacora de sesiones

La bitacora es el registro cronologico del trabajo. Anade una entrada al final
de cada sesion y conserva las anteriores. Escribe datos comprobables y enlaza
las evidencias utiles; evita frases generales como "he trabajado en Proxmox".

## Sesion 01 - 13/10/2026

- Objetivo: iniciar GitHub y la documentacion Markdown.
- Acciones realizadas: Inventario de un equipo fisico con Lliurex ( lscpu, free, lsblk, ip a, df ) y con Windows desde Hiren's Boot (CPU-Z, HWiNFO64 y CrystalDiskInfo). Rellenado el informe 01-inventario-hardware.md
- Decisiones y motivos: Versión prevista Proxmox VE 9.2. Instalación directa en el equipo físico, sin VirtualBox. Uso de ext4/LVM en lugar de ZFS para no gastar RAM en cache. Reparto previsto: 2 GB y 2 vCPU para srv-aso y para cli-linux , con 1,6 GB de RAM de margen. Se prioriza mejorar el disco (SSD) antes que la RAM, porque el HDD es el cuello de botella y el único punto de fallo.
- Pruebas y resultados: Se esperaba confirmar que el equipo cumpliese con los requisitos de Proxmox. Resultado: CPU x86_64 de 64 bits con VT-x (Pentium G4560, 2 nucleos , 4 hilos), 7,6 GiB de RAM, HDD Toshiba DT01ACA050 de 465,8 GB.
- Documentos o evidencias: 00-ficha-inicial.md y 01-inventario-hardware.md.
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

## Sesion 04 - 17/11/2026

- Objetivo: configuración base del nodo (Fase 4, apartados 1 y 2: almacenamiento y red).
- Acciones realizadas: Reinstalada la VM pve-gonzalvez en el equipo del instituto (anfitrión Linux, Intel Core i3-12100, 15 GiB de RAM; VM de 3 CPU, 8 GB de RAM y VDI de 80 GB), sustituyendo la instalación del portátil del 27/10. Revisado el almacenamiento en Datacenter → Storage (local y local-lvm) y creado backups en /srv/backups. Revisadas las interfaces vmbr0 y vmbr1 (ip -br addr, ip route) y ejecutado apt update.
- Decisiones y motivos: Copias en un almacenamiento propio (backups), aunque está en el mismo disco físico que el laboratorio (riesgo documentado). Instalación en el equipo del instituto por ⟨motivo de la reinstalación⟩.
- Pruebas y resultados: pvesm status muestra backups activo (23,6 GiB libres; local-lvm 29,9 GiB). ip -br addr muestra vmbr0 10.0.2.30/24 y vmbr1 192.168.56.20/24, e ip route la ruta por defecto vía 10.0.2.2. egrep -c '(vmx|svm)' /proc/cpuinfo devuelve 6 y existe /dev/kvm: KVM disponible. apt update descarga las listas de Debian, pero los repositorios enterprise de Proxmox dan error 401.
- Documentos o evidencias: 03-instalacion-proxmox.md y capturas s03-KVM, s04-Almacenamiento, s04-backup, s04-red y s04-comandos.
- Incidencias y solucion o estado pendiente: Error 401 en los repositorios enterprise (pendiente). Faltan los apartados 3 y 4 de la fase 4.
- Siguiente paso: completar los accesos de administración y el reparto de recursos.

## Sesion 05 - 24/11/2026

- Objetivo: completar la fase 4 (apartados 3 y 4: accesos y reparto de recursos) y ampliar el apartado 1 según la página actualizada (identificación de discos y relación de cada almacenamiento con su contenido).
- Acciones realizadas: Identificados los discos del nodo (lsblk, findmnt /, pvesm status, df -h, pvs, vgs y lvs) y localizados los archivos del VDI en el anfitrión (pve-gonzalvez.vdi y el disco diferencial de la instantánea). Ampliada la tabla de almacenamientos con backend, ruta o pool y disco que lo sostiene. Documentadas las cuentas de administración (pveum user list, pveum acl list y pve-firewall status) y medidos los recursos del nodo (nproc y free -h). Rellenada la tabla de reparto de recursos y actualizada la tabla Entorno del 03 con los datos del anfitrión.
- Decisiones y motivos: Disco de 10 GB por invitado (antes 80 y 50 GiB), porque local-lvm solo tiene 29,9 GiB. Reparto de 2 vCPU y 2 GB por invitado, con 2,2 GiB de RAM libres en la VM y 7 GiB y 5 hilos reservados al anfitrión. Única cuenta root@pam y firewall sin activar, al no ser obligatorio en la práctica.
- Pruebas y resultados: lsblk muestra un disco de 80 GiB con sda3 como volumen LVM del grupo pve (pve-root 29,9 GiB, pve-swap 7,8 GiB y thin pool pve-data con 0,00 % de datos y 1,58 % de metadatos y 9,88 GiB sin asignar). nproc devuelve 3 y free -h muestra 7,8 GiB de RAM con 1,6 GiB en uso, lo que deja 2,2 GiB libres con los dos invitados encendidos.
- Documentos o evidencias: 03-instalacion-proxmox.md y capturas s04-nodo, s04-discos, s04-discos-lvm, s04-pveum, s04-usuarios, s04-nproc y s04-resumen.
- Incidencias y solucion o estado pendiente: Error 401 en los repositorios enterprise (pendiente). Pendiente también en el 03: sección Incidencias, Conclusión de la fase 3 y tabla de cuentas del apartado 3.
- Siguiente paso: cerrar los pendientes del 03 y continuar con la fase 5, plantilla Linux.

Anadir una entrada con esta estructura en cada sesion. El historial de
GitHub registra los commits; no hace falta incluir el hash del propio
commit dentro de su entrada de bitacora.

[Volver al indice](../README.md)
