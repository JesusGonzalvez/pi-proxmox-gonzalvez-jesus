# Inventario investigado y viabilidad

## Contexto de la observacion

- Fecha y sistema utilizado: 13/10/26 LLiurex.
- Modalidad, equipo anfitrión y VM VirtualBox si procede: Modalidad fisica, Proxmox se instalaría directamente sobre este equipo.
- Version de Proxmox prevista: Proxmox VE 9.2.

## Inventario inicial

| Area | Dato observado y evidencia | Especificacion y fuente | Limite o diferencia | Justificacion |
|---|---|---|---|---|
| CPU | CPU-Z: Intel Pentium G4560, Kaby Lake, Socket 1151 LGA, 14 nm, Max TDP 54.0 W, nucleos 2, hilos 4.| Intel ARK: 2 núcleos, 4 hilos, 3,50 GHz. | Sin Turbo, solo 2 núcleos físicos. Reloj minimo de 800MHz | VT-x es el requisito de CPU para KVM. |
| RAM | CPU-Z: DDR4, 8 GBytes, Slot #1 y Slot #3, 4 GBytes cada uno, DDR4-2400 (1200 MHz), Crucial Technology, DRAM Micron Technology, CT4G4DFS824A.C8FBD2. | Intel ARK: máx. 64 GB, DDR4 2400, 2 canales. | 8 GiB instalados frente a 7,6 GiB visibles: unos 0,4 GiB no disponibles (estimación: reserva de firmware/gráfica integrada). | Ocupa solo 2 de 4 slots, así que puede ampliarse |
| Almacenamiento | CrystalDiskInfo: HDD TOSHIBA DT01ACA050, 500.1 GB, SATA/600, 7200 RPM. | Hoja de datos del TOSHIBA DT01ACA050 | HDD mecánico único, sin redundancia. | Basta para el proyecto en espacio (minimo 465 GiB). Es un HDD, no un SSD, y todo el servicio dependerá de él. |
| Red |	HWiNFO: RealTek RTL8168/8111 PCI-E Gigabit Ethernet NIC. | Realtek RTL8111/8168, 1000 Mbps máx. (HWiNFO). | No se midió la velocidad real, porque no había cable. | Una NIC Gigabit cumple para gestión y tráfico de 2 invitados con un único puente. |
| Plataforma | CPU-Z: ASUS PRIME B250M-A, chipset B250, BIOS AMI 0810 del 09/08/2017. | Pagina de soporte ASUS | BIOS de 2017, no consta si arranca en UEFI o Legacy. | La placa es compatible con el G4560. |

## 1. Puedo instalar Proxmox en este equipo?

Contrastar el inventario con requisitos oficiales de la version prevista.
Responder por separado y justificar con datos y fuentes:

- Puedo instalar y arrancar Proxmox:
    • La wiki oficial pide una CPU de 64 bits con Intel VT o AMD-V, 1 GB de RAM para la evaluación, al menos 2 GB para el sistema y los servicios más la RAM de los invitados, un disco y una NIC.
    • El equipo tiene CPU x86_64 con VT-x, 8 GiB de RAM, un HDD de 465,8 GiB y una NIC. Cumple con los requisitos mínimos.
- Puedo ejecutar el servidor ASO y el cliente del proyecto: Sí, con condiciones y sin un margen amplio. Con el reparto de abajo caben srv-aso y cli-linux a la vez, suponiendo que srv-aso ofrece servicios ligeros. No conozco sus servicios concretos, así que ese reparto es un supuesto mio.
- Condiciones o comprobaciones pendientes:
    • Hay que hacer una copia de los 235 GB del disco actual. El instalador de Proxmox usa el disco entero y lo borrara por defecto.
    • Comprobar que VT-x está activado en la BIOS y que /dev/kvm existe.
    • Conectar el cable de red y medir la velocidad real del enlace.
    • Copiar la ISO de Proxmox al pendrive Ventoy y probar el arranque. Anotar si es UEFI o Legacy.

## 2. Que limitaciones tendre con este hardware?

Analizar RAM, CPU, disco y red para las cargas previstas. Identificar lo
medido y lo estimado. En VirtualBox, investigar host real y VM: version,
RAM reservada al anfitrion, vCPU, VDI, espacio fisico y red local.
No se utiliza el Proxmox del centro como recurso.

### Reparto de recursos

| Elemento | CPU/vCPU | RAM | Disco | Justificacion |
|---|---|---|---|---|
| Proxmox | Compartidas con los invitados | 2,0 GiB | 40 GiB | Proxmox recomienda 2 GiB como minimo. Sin ZFS, con ext4/LVM. |
| srv-aso | 2 vCPU | 2,0 GiB | 80 GiB | Supuesto de servidor Linux con servicios ligeros |
| cli-linux | 2 vCPU | 2,0 GiB | 50 GiB | Cliente con escritorio ligero. |
| Plantilla, ISOs y copias | No aplica | No aplica | 120 GiB (plantilla 25, ISOs 15, copias 80) | Plantilla base para clonar, ISOs y copias locales de las VM. Las copias locales no protegen de un fallo del disco. |
| Margen de operacion | 0 | 1,6 GiB | 175,8 GiB | Abajo en los calculos |

Mostrar calculos de RAM y disco, reparto de vCPU, unidades y supuestos.
Explicar las cargas que pueden funcionar simultaneamente.

Cálculos:
RAM: 7,6 − (2,0 + 2,0 + 2,0) = 1,6 GiB de margen (21 %).
Disco: 465,8 − (40 + 80 + 50 + 120) = 175,8 GiB. Se deja sin asignar a propósito porque el aprovisionamiento ligero (thin) no debe llenarse.
vCPU: 2 + 2 = 4 vCPU sobre 4 hilos lógicos (2 nucleos fisicos). Es un reparto 1:1 contra los hilos, pero en la práctica va cargado, porque los hilos de HT no equivalen a núcleos.
Cargas simultáneas: Proxmox + srv-aso + cli-linux pueden estar encendidos a la vez. srv-aso y cli-linux encendidos a la vez funcionan si no se hacen tareas pesadas a la vez, como actualizar, compilar o hacer copias en las dos VM.

| Limitacion | Evidencia o estimacion | Consecuencia | Medida y limite que permanece |
|---|---|---|---|
| RAM | 7,6 GB | Como máximo 3 maquinas de 2GB de RAM | Sin ZFS, 2GB de RAM por VM |
| CPU | 2 núcleos, 4 hilos | 2 VM con carga al mismo tiempo se frenan | Repartir 2 vCPU por VM |
| Disco en rendimiento | HDD de 7200 rpm | Arrancar o actualizar las 2 VM al mismo tiempo es un proceso lento | Hacer arranques escalonándolos y copias fuera de horas |
| Disco en integridad | 1 disco | Un solo fallo haria perderlo todo, incluidas las copias del disco | Copiar las copias a un medio externo al disco |
| Red | 1 NIC de 1 Gbps | Un punto unico de fallo | Una red interna vmbr1 entre srv-aso y cli-linux |

## 3. Que componentes son mas criticos en mi servidor y por que?

Ordenar al menos tres componentes para este equipo y su servicio.
Distinguir cuello de botella, disponibilidad e integridad de los datos.

| Prioridad y componente | Funcion | Efecto si es insuficiente | Efecto si falla | Evidencia y justificacion |
|---|---|---|---|---|
| 1. Disco (HDD Toshiba) | Guardar el Proxmox, la plantilla, las VM y las copias. | Cuello de botella con varias VM. | Pérdida de todo: host, servicios, datos y copias locales (disponibilidad e integridad). Es el componente cuyo fallo tiene mayor impacto | Un solo disco HDD de 7200 rpm, SMART. Es el único punto de fallo para los datos. |
| 2. RAM | Memoria de Proxmox y de las VM. | Cuello de botella: con poca RAM aparece swap en el HDD o el kernel mata los procesos. | Si falla un módulo, el equipo puede no arrancar o quedarse con 4 GiB y no cabrian las 2 VM | 7,6 GiB visibles. Es el límite de cuántas VM caben.
 | 
3. CPU | Cálculo de hosts y VM. | Cuello de botella con cargas simultáneas. | El host cae y todo con él. | Tiene 2 núcleos y 4 hilos. | 
4. NIC | Acceso de los clientes al servidor. | Un enlace lento limita el uso del servicio desde el cliente, aunque 1 Gbit es suficiente para este laboratorio. | Sin red no hay acceso remoto. | Una unica NIC | 

Componente que mejoraria primero y motivo para priorizarlo frente a otro:
Un SSD frente a más RAM. Con este reparto la RAM alcanza para 2 VM y en cambio el HDD es a la vez el cuello de botella, riesgo de pérdida de datos y el factor que más se nota al arrancar varias VM. La RAM sería la segunda mejora, ya que hay slots libres.

## Fuentes tecnicas

Consultar al menos tres fuentes primarias, incluyendo Proxmox y
fabricante o plataforma virtual. Relacionarlas con las tres respuestas.

| Fuente y URL | Version/modelo y fecha de consulta | Afirmacion respaldada | Aplicacion al nodo |
|---|---|---|---|
| Proxmox VE, System Requirements: https://pve.proxmox.com/wiki/System_Requirements | Página wiki editada por última vez el 28/11/2024. Consultada el 13/10/2026 | Mínimos de evaluación (CPU de 64 bits con VT, 1 GB de RAM, un disco, una NIC). | Contexto, inventario y preguntas 1, 2 y 3 |
| Intel ARK, Pentium G4560: https://www.intel.com/content/www/us/en/products/sku/97143/intel-pentium-processor-g4560-3m-cache-3-50-ghz/specifications.html | Intel Pentium G4560 (ficha modificada el 05/08/2026). Consultada el 13/10/2026 | VT-x, núcleos/hilos, memoria soportada | Inventario y preguntas 1, 2 y 3. |
| ASUS, PRIME B250M-A: https://www.asus.com/motherboards-components/motherboards/prime/prime-b250m-a/ | Placa PRIME B250M-A. Consultadas el 13/10/2026. | Slots de RAM, máximo de RAM, M.2 | Inventario y preguntas 1, 2 y 3. |
| Toshiba, serie DT01 (DT01ACA) — https://toshiba.semicon-storage.com/eu/product/storage-products/client-hdd/dt01acaxxx.html | Modelo DT01ACA050 (500 GB). Consultada el 13/10/2026 | Rendimiento de un HDD de 7200 rpm | Inventario (almacenamiento) y preguntas 1, 2 y 3. |
| Proxmox VE, "Installing Proxmox VE" (guía de instalación) — https://pve.proxmox.com/pve-docs/chapter-pve-installation.html | Documentación de Proxmox VE 9.2, actualizada el 24/09/2026. Consultada el 13/10/2026 | Mismos mínimos y recomendaciones | Plataforma y arranque (pregunta 1) y reparto de disco y ZFS (pregunta 2) |

## Dictamen de viabilidad

Apto, apto con condiciones o no apto para la propuesta actual: 
Apto con condiciones para la propuesta actual (srv-aso y cli-linux).
    • Cumple los requisitos de Proxmox y admite 2 VM ligeras a la vez.
    • Las limitaciones son el HDD, la RAM y la ausencia de redundancia.
    • Las condiciones son las cinco del apartado 1: copia de los datos, VT-x/KVM, cable de red, arranque desde el USB y versión de Proxmox.
    

Sintetizar las tres respuestas y las condiciones pendientes de comprobar.

## Evidencias y revision docente

Enlazar salidas revisadas desde ../evidencias/. Un listado automatico no
sustituye la investigacion. Registrar la pregunta de defensa y respuesta.

[Volver al indice](../README.md)
