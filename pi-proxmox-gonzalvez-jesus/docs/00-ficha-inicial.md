# Ficha inicial

| Campo | Contenido |
|---|---|
| Identificador del alumno | Jesus Gonzalvez |
| URL GitHub | https://github.com/JesusGonzalvez/pi-proxmox-gonzalvez-jesus |
| Acceso docente comprobado | Sí |
| Modalidad fisica o virtualizada | Virtualizada: Proxmox VE 9.2 en una VM de VirtualBox |
| Recursos conocidos | VM de 3 vCPU, 8 GB de RAM y VDI de 80 GB, sobre un anfitrión Linux del instituto (Intel Core i3-12100, 4 núcleos y 8 hilos, 15 GiB de RAM) |
| Nombre del nodo | pve-gonzalvez |
| Servicio ASO previsto | Samba (puerto 445) |
| Red de laboratorio | 192.168.56.0/24 (vmbr1, solo-anfitrión); salida a Internet por vmbr0 (NAT, 10.0.2.0/24) |
| Riesgos y dudas iniciales | Un único disco sin redundancia, recursos limitados de la VM y virtualización anidada |

[Volver al indice](../README.md)
