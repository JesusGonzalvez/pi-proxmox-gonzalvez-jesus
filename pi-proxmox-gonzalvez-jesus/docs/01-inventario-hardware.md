# Inventario investigado y viabilidad

## Contexto de la observacion

- Fecha y sistema utilizado: 13/10/26 LLiurex.
- Modalidad, equipo anfitrión y VM VirtualBox si procede: Modalidad fisica, Proxmox se instalaría directamente sobre este equipo.
- Version de Proxmox prevista: Proxmox VE 9.2.

## Inventario inicial

| Area | Dato observado y evidencia | Especificacion y fuente | Limite o diferencia | Justificacion |
|---|---|---|---|---|
| CPU | CPU-Z (CPU): Intel Pentium G4560, Kaby Lake, Socket 1151 LGA, 14 nm, Max TDP 54.0 W, Cores 2, Threads 4, Instructions: MMX, SSE, SSE2, SSE3, SSSE3, SSE4.1, SSE4.2, EM64T, VT-x, AES. Comando lscpu: x86_64, CPU(s) 4, 2 núcleos por «socket», 2 hilos por núcleo, 1 socket, CPU MHz máx. 3500 mín. 800 Virtualización VT-x, flags vmx, ept, vpid, caché L1d 64 KiB, L1i 64 KiB, L2 512 KiB, L3 3 MiB | Intel ARK: 2 núcleos, 4 hilos, 3,50 GHz, sin Turbo Boost, VT-x, VT-d y EPT: sí, extensiones SSE4.1/4.2 (sin AVX), TDP 54 W, producto descontinuado y sin actualizaciones desde el 31/03/2024 | Sin Turbo, solo 2 núcleos físicos (los hilos de SMT no equivalen a núcleos). Sin AVX/AVX2.El comando lscpu muestra "Itlb multihit: KVM: Mitigation: VMX disabled", que no demuestra si VT-x está activo en la BIOS | El hardware soporta virtualización, pero falta comprobar que está habilitada (kvm-ok, dmesg, ajustes en UEFI). La CPU es modesta y sin servicio de Intel, por lo que depende de las mitigaciones del kernel |
| RAM | Comando free -h: Mem total 7,6Gi, usado 994Mi, libre 5,6Gi, disponible 6,3Gi; Inter 511Mi, usado 0B. CPU-Z (Memory): DDR4, 8 GBytes, Channel Dual, DRAM Frequency 1196.5 MHz, CL 17, tRCD 17, tRP 17, tRAS 39, tRFC 312, CR 2T. CPU-Z (SPD): Slot #1 y Slot #3, 4 GBytes cada uno, DDR4-2400 (1200 MHz), Crucial Technology, DRAM Micron Technology, CT4G4DFS824A.C8FBD2, Ranks Single, 1.20 V, Week/Year 33/17 (slot 1) y 34/17 (slot 3) | Intel ARK: máx. 64 GB, DDR4-2133/2400, 2 canales. ASUS (ficha reproducida por distribuidores): 4 DIMM, máx. 64 GB, DDR4 2400/2133 no ECC, memoria compartida de vídeo hasta 1024 MB | 8 GiB instalados frente a 7,6 GiB visibles: unos 0,4 GiB no disponibles (estimación: reserva de firmware/gráfica integrada, por confirmar). 2 de 4 slots libres. Sin ECC | Cumple con el mínimo de evaluación (1 GB) y el recomendado para el host (2 GB). El límite está en repartir 7,6 GiB entre host e invitados. La swap no cuenta como RAM adicional |
| Almacenamiento | CrystalDiskInfo 9.2.2: TOSHIBA DT01ACA050, 500.1 GB, Serial ATA, SATA/600, 7200 RPM, Buffer Size 23652 KB, Health Status Good, 39 °C, Power On Count 1245, Power On Hours 3573; atributos 05, C4, C5, C6 y C7 con raw 0. Comando lsblk: sda 465,8G con sda1 465,8G en /; sr0 1024M. Comando df -h: /dev/sda1 458G, 235G usados, 201G disp., 54 %. disco.txt: sda1 ext4, FSAVAIL 200,1G, FSUSE% | Proxmox VE (requisitos): "Hard drive" en evaluación; en recomendados, almacenamiento rápido y redundante, mejor con SSD; ZFS necesita almenos 1 GB de RAM por 1 TB usado. ASUS: 6 puertos SATA 6 Gb/s y sockets M.2. | HDD mecánico único, sin redundancia ni SSD. Los 235 GB actuales se perderían al instalar Proxmox en todo el disco. | Basta para el proyecto en espacio (minimo 465 GiB). Es el punto débil en rendimiento y en integridad (un único disco para host, invitados y copias locales) |
| Red |	HWiNFO64 v7.68: RealTek Semiconductor RTL8168/8111 PCI-E Gigabit Ethernet NIC, Maximum Link Speed 1000 Mbps, Hardware ID PCI\VEN_10EC&DEV_8168&SUBSYS_86771043&REV_15, driver Realtek PCIe GbE Family Controller 1168.16.1123.2023 (23-Nov-2023). Comando ip a: eth0 <NO-CARRIER,BROADCAST,MULTICAST,UP> state DOWN, mtu 1500, altname enp2s0, sin dirección IP; virbr0 192.168.122.1/24, br-64d9270d060e 172.18.0.1/16 y docker0 172.17.0.1/16, los tres en state DOWN; lo 127.0.0.1/8. Rutas: 172.17.0.0/16, 172.18.0.0/16 y 192.168.122.0/24 | Proxmox VE: mínimo de evaluación, 1 NIC; recomendado, NIC Gbit redundantes. ASUS: 1 × Realtek RTL8111H Gigabit LAN | Solo una NIC, sin redundancia. El estado DOWN significa cable sin conectar o sin enlace en el momento del inventario | Una NIC Gigabit cumple para gestión y tráfico de 2 invitados con un único puente. Hay que verificar enlace y velocidad con un cable conectado |
| Plataforma | CPU-Z (Mainboard): ASUSTeK COMPUTER INC., PRIME B250M-A, Rev X.0x, Bus Specs PCI-Express 3.0 (8.0 GT/s), Chipset Intel Kaby Lake Rev. 05, Southbridge Intel B250 Rev. 00, LPCIO Nuvoton NCT6793/NCT5563, BIOS Brand American Megatrends Inc., Version 0810, Date 09/08/2017. Comando [ -d /sys/firmware/efi ] && echo UEFI || ECHO BIOS: solo contiene BIOS lo que significa que arranco sin UEFI. | [S5] ASUS: UEFI AMI BIOS. [S4] Intel ARK: socket FCLGA1151. [S2] Guía de instalación de Proxmox: solo exige arranque UEFI en arm64 (no lo impone en x86); el arranque desde el medio de instalación debe estar habilitado en el firmware; Secure Boot solo había que desactivarlo en instaladores anteriores a Proxmox VE 8.1 | El Linux de inspección arrancó en modo BIOS (heredado), no en UEFI (Comando [ -d /sys/firmware/efi ] && echo UEFI || ECHO BIOS). Secure Boot y el modo con el que se instalará Proxmox sin comprobar. BIOS de 2017 sin actualizar. | Plataforma compatible con virtualización y 64 bits [S4][S5]. Quedan pendientes el modo de arranque, los ajustes de virtualización y el arranque de la ISO desde Ventoy |

## 1. Puedo instalar Proxmox en este equipo?

Contrastar el inventario con requisitos oficiales de la version prevista.
Responder por separado y justificar con datos y fuentes:

- Puedo instalar y arrancar Proxmox: El equipo supera los mínimos de evaluación y también los 2 GB recomendados para el host, pero no cumple las recomendaciones de producción.
Componente por componente:
CPU: lscpu y CPU-Z muestran x86_64 con 2 núcleos y 4 hilos y soporte de Intel 64 (EM64T). Proxmox pide una CPU de 64 bits Intel 64 o AMD64 [S1][S2], y la ficha de Intel lo confirma [S4].
Virtualización: lscpu indica VT-x y los flags vmx, ept y vpid; Intel confirma VT-x, VT-d y EPT para este procesador [S4]. Proxmox exige CPU y placa con Intel VT [S1][S2]. Sin embargo, lscpu también muestra Itlb multihit: KVM: Mitigation: VMX disabled, que no demuestra que VT-x esté activado en la UEFI.
RAM: 7,6 GiB visibles (free -h), con 2 módulos DDR4-2400 de 4 GB en dual channel (CPU-Z). Proxmox pide 1 GB para evaluación y 2 GB para el SO y sus servicios, más la RAM de los invitados [S1][S2]. Cumple, y la placa admite hasta 64 GB [S4][S5].
Almacenamiento: un único HDD Toshiba DT01ACA050 de 465,8 GiB, 7200 rpm, SATA/600, con estado "Good" (CrystalDiskInfo) [S6]. Cumple el requisito de evaluación (un disco duro), pero no la recomendación de almacenamiento rápido y redundante, mejor con SSD [S1].
Red: una NIC Realtek RTL8168/8111 Gigabit (HWiNFO64), cuyo eth0 figuraba como NO-CARRIER y DOWN (ip). Cumple el mínimo de una NIC [S1][S2], pero no las NIC redundantes recomendadas; el enlace queda pendiente de probar.
Plataforma y arranque: placa ASUS PRIME B250M-A con BIOS American Megatrends 0810 del 09/08/2017 (CPU-Z) [S5]. bios.txt indica que este Linux arrancó en modo BIOS. La guía de instalación solo impone UEFI en arm64 y pide que el arranque desde el medio de instalación esté habilitado en el firmware [S2]. Queda pendiente decidir el modo con el que se instalará.
- Puedo ejecutar el servidor ASO y el cliente del proyecto: Sí, con condiciones y sin un margen amplio. Instalar Proxmox solo exige los mínimos, pero ejecutar el proyecto exige repartir los recursos. El reparto cabe en 7,6 GiB de RAM (host 2,0 + srv-aso 2,0 + cli-linux 2,0 + margen 1,6) y en 465,8 GiB de disco, con 4 vCPU sobre 4 hilos lógicos que son solo 2 núcleos físicos [S1][S4]. Mientras srv-aso y cli-linux sean Linux estándar y sin carga pesada, deberían funcionar a la vez. No es seguro que aguanten servicios pesados ni varias tareas intensivas simultáneas (clonar, hacer una copia de seguridad y usar un cliente gráfico) sobre un HDD [S1][S6].
- Condiciones o comprobaciones pendientes: VT-x activo en la UEFI y módulo KVM cargado (kvm-ok, lsmod | grep kvm, ajuste "Intel Virtualization Technology") [S1][S2][S4].
Modo de arranque: Comando [ -d /sys/firmware/efi ] && echo UEFI || ECHO BIOS indica que este Linux arrancó en modo BIOS. Falta decidir el modo con el que se instalará Proxmox y revisar el Secure Boot [S2].
Enlace de red y driver (ethtool eth0, lspci -nnk) con el cable conectado, comprobando los 1000 Mbps [S5].
Medio de instalación: la guía recomienda grabar la ISO híbrida en un USB con dd (Linux), Etcher o Rufus en modo DD y no menciona Ventoy, así que si no arranca desde Ventoy hay que grabarla directamente [S2].
Rendimiento real del disco (sudo hdparm -tT /dev/sda, smartctl -a; pveperf tras instalar), porque la ficha del fabricante no da tasa sostenida ni latencia [S6].
Contrastar las especificaciones de la placa con el manual oficial de ASUS [S5].
Confirmar con el profesor qué ISO de Proxmox VE usar [S3] y decidir qué hacer con los 235 GB actuales, porque instalar sobre todo el disco los borra.

## 2. Que limitaciones tendre con este hardware?

Analizar RAM, CPU, disco y red para las cargas previstas. Identificar lo
medido y lo estimado. En VirtualBox, investigar host real y VM: version,
RAM reservada al anfitrion, vCPU, VDI, espacio fisico y red local.
No se utiliza el Proxmox del centro como recurso.

### Reparto de recursos

| Elemento | CPU/vCPU | RAM | Disco | Justificacion |
|---|---|---|---|---|
| Proxmox | Comparte los 4 hilos con los invitados (sin vCPU asignadas) | 2,0 GiB | 40 GiB | Proxmox recomienda mínimo 2 GB para el SO y sus servicios [S1][S2]. El instalador por defecto crea una swap igual a la RAM (entre 4 y 8 GB) y un volumen raíz de hasta 96 GiB [S2]; para dejarlo en 40 GiB habría que fijar en las opciones avanzadas, por ejemplo, swapsize 4 y maxroot 36 (propuesta mía). El espacio libre que reserva minfree (16 GB por defecto) [S2] cae dentro del margen |
| srv-aso | 2 vCPU | 2,0 GiB | 40 GiB | Servidor Linux sin escritorio con los servicios de ASO. El tamaño real dependerá de los servicios definitivos. |
| cli-linux | 2 vCPU | 2,0 GiB | 30 GiB | Cliente con escritorio ligero. Un entorno gráfico consume más que un servidor sin interfaz |
| Plantilla, ISOs y copias | No aplica | No aplica | 82 GiB (plantilla 10 + ISOs 12 + copias locales 60) | Plantilla base para clonar, ISOs (la de Proxmox ocupa 1,71 GB [S3] y las de los invitados) y copias locales de las VM. |
| Margen de operacion | Reserva para picos | 1,6 GiB | 93 GiB (aproximadamente un 20 % del disco) | Caché del sistema, picos, clonados y crecimiento de los discos ligeros. Incluye el minfree del instalador [S2] |

Mostrar calculos de RAM y disco, reparto de vCPU, unidades y supuestos.
Explicar las cargas que pueden funcionar simultaneamente.

Cálculos:
RAM: 7,6 − (2,0 + 2,0 + 2,0) = 1,6 GiB de margen (21 %). RAM después del margen: 0.
Disco: 465,8 − (40 + 40 + 30 + 82) = 273,8 GiB; menos 93 GiB de margen = 180,8 GiB sin asignar. Se deja sin asignar a propósito para ampliar discos o copias.
vCPU: 2 + 2 = 4 vCPU sobre 4 hilos lógicos, que son solo 2 núcleos físicos [S4]. Las vCPU no equivalen a núcleos exclusivos: si ambos invitados están ocupados a la vez, compiten entre sí y con el host. La carga permanente debería ser baja, con ráfagas puntuales.
Cargas simultáneas: con 6,0 GiB reservados, Proxmox + srv-aso + cli-linux pueden estar encendidos a la vez. Un tercer invitado solo cabría si es un LXC pequeño (mas o menos 0,5–1,0 GiB) y con el margen reducido. Clonar o hacer copias mientras ambos invitados trabajan es lo que más probablemente dará problemas.

| Limitacion | Evidencia o estimacion | Consecuencia | Medida y limite que permanece |
|---|---|---|---|
| RAM limitada (7,6 GiB para host e invitados) | Medido: free -h y CPU-Z (2 × 4 GB). Estimado: reparto de 6,0 GiB reservados. Fuentes: [S1] 2 GB mínimos para el host; [S4][S5] máx. 64 GB y 4 slots | Con más de 3 cargas o con invitados que superen su RAM, el sistema recurre a swap en el HDD o el kernel termina los procesos | Limitar a 2 invitados + host y usar LXC cuando proceda. Se puede ampliar usando los 2 slots libres, hasta 64 GB [S4][S5]. No resuelve la falta de RAM si no se amplía. |
| CPU de 2 núcleos / 4 hilos sin Turbo | Medido: lscpu, CPU-Z. [S4]: 2 núcleos, 4 hilos, sin Turbo Boost. Mitigaciones activas observadas en lscpu | Poca capacidad si coinciden en el clonado, copia (compresión) y uso del cliente gráfico. Las mitigaciones de seguridad añaden sobrecoste (estimado, sin medir) | No superar 4 vCPU en total, evitar tareas pesadas en paralelo y programar copias fuera del uso. No elimina el límite de 2 núcleos físicos. |
| Sin AVX/AVX2 | Medido: ausentes en los flags de lscpu; [S4] solo lista SSE4.1/4.2 | Software o sistemas operativos invitados que exijan instrucciones más modernas podrían no arrancar. A comprobar en la distro elegida (previsión) | Usar Debian/Ubuntu estándar y el tipo de CPU por defecto de las VM. No se resuelve cambiando de invitado si el software exige AVX |
| HDD único de 7200 RPM |	Medido: CrystalDiskInfo (HDD, SATA/600). [S6]: 7200 rpm, SATA 6 Gbit/s, sin tasa sostenida ni latencia publicadas. Estimado: del orden de 100 IOPS y mas o menos 100–200 MB/s secuenciales, típico de un HDD (sin medir). [S1] recomienda SSD | Arranques, clonados y copias lentos. Un clon completo de 40 GiB a ≈ 100 MB/s tardaría unos 7 minutos (estimación) y peor con otras VM activas | Usar plantilla con clones enlazados, discos ligeros y hacer copias de una en una. No resuelve la latencia propia de un disco mecánico |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |

## 3. Que componentes son mas criticos en mi servidor y por que?

Ordenar al menos tres componentes para este equipo y su servicio.
Distinguir cuello de botella, disponibilidad e integridad de los datos.

| Prioridad y componente | Funcion | Efecto si es insuficiente | Efecto si falla | Evidencia y justificacion |
|---|---|---|---|---|
| 1. | | | | |
| 2. | | | | |
| 3. | | | | |

Componente que mejoraria primero y motivo para priorizarlo frente a otro:
PENDIENTE.

## Fuentes tecnicas

Consultar al menos tres fuentes primarias, incluyendo Proxmox y
fabricante o plataforma virtual. Relacionarlas con las tres respuestas.

| Fuente y URL | Version/modelo y fecha de consulta | Afirmacion respaldada | Aplicacion al nodo |
|---|---|---|---|
| | | | |

## Dictamen de viabilidad

Apto, apto con condiciones o no apto para la propuesta actual: PENDIENTE.

Sintetizar las tres respuestas y las condiciones pendientes de comprobar.

## Evidencias y revision docente

Enlazar salidas revisadas desde ../evidencias/. Un listado automatico no
sustituye la investigacion. Registrar la pregunta de defensa y respuesta.

[Volver al indice](../README.md)
