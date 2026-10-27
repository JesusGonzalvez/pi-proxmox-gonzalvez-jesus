# 03-instalacion-proxmox

## Objetivo

Instalar Proxmox VE dentro de una máquina virtual de VirtualBox y configurar su red según el diseño de `02-diagrama-red.md`.

## Entorno

| Elemento | Valor |
|---|---|
| Anfitrión | Windows (CPU AMD) |
| Versión de VirtualBox | VirtualBox 7.2.20 |
| Nombre de la VM | pve-gonzalvez |
| Tipo / versión | Linux / Debian |
| CPU / RAM / disco | 6 CPU / 10 GB (10240 MB) / 80 GB VDI |
| ISO | proxmox-ve_9.2-1.iso |

## Procedimiento

### 1. Registrar la versión de VirtualBox

En **Ayuda → Acerca de VirtualBox**

### 2. Crear la VM Linux de 64 bits

**Máquina → Nueva**: nombre pve-gonzalvez, tipo Linux, versión Debian (64-bit), 6 CPU, 10240 MB de RAM y un disco VDI de 80 GB.

### 3. Reservar recursos para el anfitrión

La VM usa 10 GB de RAM y 6 CPU, y deja el resto de recursos para Windows.

### 4. Adaptadores de red

Red solo-anfitrión (**Archivo → Herramientas → Red → Redes solo-anfitrión**): IPv4 192.168.56.1, máscara 255.255.255.0, con DHCP desactivado.

| Adaptador | Conectado a | Interfaz en Proxmox | Uso |
|---|---|---|---|
| Adaptador 1 | NAT | nic0 | Salida a Internet (vmbr0) |
| Adaptador 2 | Adaptador solo-anfitrión | nic1 | Red de laboratorio y gestión (vmbr1) |

En el Adaptador 2, *Modo promiscuo = Permitir todo*.

### 5. Virtualización anidada

Con la VM apagada se activó la virtualización anidada y se comprobó:


& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" modifyvm "pve-gonzalvez" --nested-hw-virt on

& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" showvminfo "pve-gonzalvez" | findstr /i nested


Resultado: Nested VT-x/AMD-V: enabled.

La comprobación real fue negativa: el instalador de Proxmox avisó de que no detecta virtualización por hardware (ver Incidencias).

### 6. Instalar Proxmox en el VDI

Se montó la ISO y se arrancó con *Install Proxmox VE (Graphical)*. El disco de destino es el VDI de 80 GB.

| Campo | Valor |
|---|---|
| País / zona horaria | España / Europe/Madrid |
| Interfaz de gestión | nic0 (NAT) |
| Hostname (FQDN) | pve-gonzalvez.lab.local |
| IP / CIDR | 10.0.2.30/24 |
| Gateway | 10.0.2.2 |
| DNS | 8.8.8.8 |

Al terminar se quito la ISO y se reinició.

### 7. Configurar IP de gestión, hostname y DNS, y abrir el panel

Se editó /etc/network/interfaces para añadir vmbr1 (red de laboratorio y gestión) junto a vmbr0 (NAT):


auto lo
iface lo inet loopback

iface nic0 inet manual

auto vmbr0
iface vmbr0 inet static
        address 10.0.2.30/24
        gateway 10.0.2.2
        bridge-ports nic0
        bridge-stp off
        bridge-fd 0
        
iface nic1 inet manual

auto vmbr1
iface vmbr1 inet static
        address 192.168.56.20/24
        bridge-ports nic1
        bridge-stp off
        bridge-fd 0

source /etc/network/interfaces.d/*


Se reinició y se comprobó con ip -br addr, ip route y hostname -i.

El panel se abre desde el navegador del anfitrión en https://192.168.56.20:8006, con usuario root y dominio **Linux PAM standard authentication**.

## Evidencias

### Panel de Proxmox

![Panel de Proxmox](img/s03-panel-proxmox.png)

### Red

![Red del nodo](img/s03-red-proxmox.png)

### Almacenamiento

![Almacenamiento](img/s03-almacenamiento-proxmox.png)

### Salida de comandos

Salida de hostnamectl, ip -br addr, pveversion y df -h:

![Salida de comandos](img/s03-salida-comandos.png)

## Incidencias

**1. Aviso de KVM durante la instalación.** El instalador mostró *"No support for hardware-accelerated KVM virtualization detected"*, aunque la BIOS tiene AMD-V activado y VirtualBox muestra Nested VT-x/AMD-V: enabled.
*Causa:* Windows mantiene activa la seguridad basada en virtualización (msinfo32 indica "Ejecutándose"), por lo que VirtualBox no puede ofrecer AMD-V a la VM.
*Decisión:* no se desactivaron las protecciones del núcleo de Windows. Proxmox se instala y funciona con normalidad, pero sin KVM anidado.

**2. Error 401 en el login del panel.** Se intentaba entrar con el dominio *Proxmox VE authentication server* (root@pve), donde el usuario no existe. El log de pvedaemon mostraba no such user ('root@pve').
*Solución:* usar el dominio **Linux PAM standard authentication**.


## Conclusión

Se instaló Proxmox VE en una VM de VirtualBox con 6 CPU, 10 GB de RAM y 80 GB de disco. La red de gestión está en vmbr1 (192.168.56.20/24) y la salida a Internet en vmbr0 (10.0.2.30/24), según el diseño de 02-diagrama-red.md. El panel es accesible desde el navegador del anfitrión en https://192.168.56.20:8006. El nesting no pudo usarse por la seguridad basada en virtualización de Windows, lo que queda documentado en Incidencias.

[Volver al índice](../README.md)
