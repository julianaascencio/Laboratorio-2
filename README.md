# Laboratorio de Redes - Segmentación y Monitoreo

## Descripción
Este laboratorio consiste en la implementación de una red segmentada en dos subredes utilizando máquinas virtuales, configurando un nodo central como router con NAT para permitir comunicación entre redes y acceso a internet.

## Topología

- Subred 1: 192.168.10.0/24
- Subred 2: 192.168.20.0/24
- Router: Debian-Admin

## Tecnologías utilizadas

- VirtualBox
- Debian 13
- Redes internas
- NAT
- iptables

## Objetivos

- Implementar segmentación de red
- Configurar routing entre subredes
- Habilitar acceso a internet
- Validar conectividad entre hosts

  # Instalación de Máquinas Virtuales

## Debian-Admin

Se instaló Debian 13 en modo mínimo sin entorno gráfico, con el objetivo de optimizar recursos.

### Configuración:
- RAM: 4096 MB
- CPU: 2
- Disco: 30 GB

### Instalación del sistema Debian-Admin

Se realizó la instalación del sistema operativo Debian 13 utilizando una imagen ISO oficial en una máquina virtual creada en VirtualBox.
Se seleccionó la opción de instalación gráfica, configurando parámetros básicos como idioma, red y particionado automático del disco.
El sistema se instaló en modo mínimo, sin entorno gráfico, con el objetivo de optimizar recursos para su uso como router dentro de la topología de red.
<img width="640" height="480" alt="image" src="https://github.com/user-attachments/assets/174ad471-e306-41af-960b-c838b7c0d7fd" />

<img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/b5817487-dce3-47bd-97f3-bb0884c3efe7" />

<img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/ac2b39c3-ba56-47af-b78a-3146862511c0" />

<img width="800" height="600" alt="image" src="https://github.com/user-attachments/assets/6942bde8-ab21-435e-bcce-30e4da14f404" />

---

## Host1

Se instaló Debian como cliente en la subred 1.

### Configuración:
- RAM: 1024 MB
- CPU: 1
- Disco: 10 GB
- 
## Host1 (Subred 1)

El Host1 fue configurado como un equipo cliente dentro de la subred 1, permitiendo validar la conectividad hacia el router y otras redes.

---

### Configuración de red

Se asignó una dirección IP estática correspondiente a la subred 1:

```bash
ip addr flush dev enp0s3
ip addr add 192.168.10.10/24 dev enp0s3
ip link set enp0s3 up
ip route add default via 192.168.10.1
```

<img width="784" height="371" alt="image" src="https://github.com/user-attachments/assets/0f8c680b-35a4-4afd-9063-b356bdc9a7a4" />
Login en consola

<img width="823" height="246" alt="image" src="https://github.com/user-attachments/assets/d20fe5fe-1a73-40f1-86cc-565c27551566" />


---

## Host2

Se creó a partir de clonación de Host1.

### Ajustes realizados:
- Cambio de hostname
- Cambio de red interna
- 
## Host2 (Subred 2)

El Host2 fue creado a partir de la clonación de Host1, con el objetivo de agilizar la implementación del laboratorio.

Posteriormente, se realizaron ajustes para adaptarlo a la subred 2.

---

### Configuración de red

Se asignó una dirección IP estática correspondiente a la subred 2:

```bash
ip addr flush dev enp0s3
ip addr add 192.168.20.10/24 dev enp0s3
ip link set enp0s3 up
ip route add default via 192.168.20.1
```

<img width="783" height="910" alt="image" src="https://github.com/user-attachments/assets/211aa70a-c7c0-478d-8c27-ce248e5e2fe2" />

<img width="820" height="332" alt="image" src="https://github.com/user-attachments/assets/00801b3e-4fd9-4935-83df-c6939b34afe4" />


  # Configuración de Red

## Debian-Admin (Router)

Se configuraron 3 interfaces:

- enp0s3 → NAT (internet)
- enp0s8 → Subred1
- enp0s9 → Subred2

### Configuración IP

```bash
ip addr add 192.168.10.1/24 dev enp0s8
ip addr add 192.168.20.1/24 dev enp0s9
```
## Verificación del Router (Debian-Admin)

Se realizaron pruebas de conectividad desde el nodo central hacia ambas subredes y hacia internet.

```bash
ping 192.168.10.10
ping 192.168.20.10
ping 8.8.8.8
ping google.com
```
<img width="1030" height="1000" alt="image" src="https://github.com/user-attachments/assets/796f9cb8-877a-4a9c-a6c9-2f981973b9d8" />
