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

<img width="1280" height="925" alt="image" src="https://github.com/user-attachments/assets/b4ba94b1-14ca-4b06-a996-48d6d2dcf218" />

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
<img width="951" height="1004" alt="image" src="https://github.com/user-attachments/assets/167b7c8a-e3dd-42da-90ba-7bdadd50931f" />

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
<img width="983" height="920" alt="image" src="https://github.com/user-attachments/assets/eeb7155a-32f3-404d-9472-2a27668980c3" />

## Verificación del Router (Debian-Admin)

Se realizaron pruebas de conectividad desde el nodo central hacia ambas subredes y hacia internet.

```bash
ping 192.168.10.10
ping 192.168.20.10
ping 8.8.8.8
ping google.com
```
<img width="1030" height="1000" alt="image" src="https://github.com/user-attachments/assets/796f9cb8-877a-4a9c-a6c9-2f981973b9d8" />

## Generación de tráfico con iPerf3

Se realizó la generación de tráfico entre los hosts con el fin de validar la comunicación entre subredes y medir el rendimiento de la red configurada.

---

### Instalación de la herramienta

En ambos hosts (Host1 y Host2) se ejecutaron los siguientes comandos:

```bash
apt update
apt install -y iperf3
```

Durante la instalación se seleccionó la opción de no ejecutar iperf3 como demonio, ya que las pruebas se realizarán manualmente

### Configuración de la prueba
Se definieron los siguientes roles:
- Host1: Cliente (genera tráfico)
- Host2: Servidor (recibe tráfico)

### Ejecución en Host2 (Servidor)
En host2 se ejecuto:

``` bash
iperf3 -s
```
Esto permiete que el host quede escuchando conexiones en el puerto 5201

### Ejecución en Host1 (Cliente)
Desde Host1 se generó tráfico hacia Host2 con:

```bash
iperf3 -c 192.168.20.10 -t 20
```
Opcionalmente se utilizó:
```bash
iperf3 -c 192.168.20.10 -t 20 -i 1
```
para observar resultados por intervalos de tiempo.

### Resultados obtenidos
Durante la ejeción de la prueba se observo:

-Transferencia continua de datos entre los hosts
- Ancho de banda promedio entre 300 y 600 Mbits/sec
- Comunicación estable entre subredes

Esto confirma que:

✔ El enrutamiento configurado en Debian-Admin es funcional
✔ La conectividad entre Host1 y Host2 es correcta
✔ La red permite transmisión de datos sin pérdida significativa

Se adjunta evidencia de:
- Ejecución de iperf3 en Host2 (modo servidor)
- Ejecución de iperf3 en Host1 (modo cliente)
- Resultados de transferencia y ancho de banda

  <img width="1911" height="1005" alt="image" src="https://github.com/user-attachments/assets/65ce377c-03da-4009-973d-bd8776bd494d" />

La prueba con iPerf3 permitió validar el correcto funcionamiento de la red implementada, evidenciando la comunicación entre subredes y el rendimiento del enlace a través del router Debian-Admin.

### Configuración de carpeta compartida

Se configuró una carpeta compartida en VirtualBox para facilitar la transferencia de archivos entre la máquina virtual y el sistema anfitrión.

Se habilitó la opción de automontaje y se asignó el nombre "Shared", permitiendo exportar la captura de tráfico generada en el router hacia el equipo local para su análisis en Wireshark.

<img width="508" height="334" alt="image" src="https://github.com/user-attachments/assets/4ebcf77d-6cb3-4108-b08f-c3b043a57c00" />

### Captura de tráfico en el router

Se utilizó la herramienta `tcpdump` en la interfaz enp0s8 del router Debian-Admin para capturar el tráfico de la red interna.

```bash
tcpdump -i enp0s8 -w captura_final.pcap
```

<img width="847" height="330" alt="image" src="https://github.com/user-attachments/assets/3ce4ba33-6072-4035-9729-a52c4d5bc4b9" />

La captura permitió registrar paquetes ICMP, DNS, NTP y TCP generados durante las pruebas de conectividad y rendimiento.

### Exportación de la captura

El archivo de captura fue copiado desde la máquina virtual hacia el sistema anfitrión utilizando la carpeta compartida configurada previamente.

```bash
mount -t vboxsf Shared /mnt/Shared
cp /root/captura_final.pcap /mnt/Shared/
```

<img width="1017" height="958" alt="image" src="https://github.com/user-attachments/assets/a27b044f-06cb-4aa1-bee7-5032d0a30919" />

Esto permitió abrir el archivo en Wireshark para su análisis.

### Comunicación entre subredes

Se observa tráfico ICMP entre las direcciones:

- 192.168.10.10
- 192.168.20.10

Esto evidencia que los hosts en diferentes subredes pueden comunicarse correctamente a través del router Debian-Admin.

<img width="1009" height="836" alt="image" src="https://github.com/user-attachments/assets/e5c20535-7338-4e69-bc23-fee077cbd256" />

**Análisis:**

En la captura se observa tráfico ICMP (Echo Request y Echo Reply) entre las direcciones:

- 192.168.10.10 (Host1)
- 192.168.20.10 (Host2)

Esto confirma que el router Debian-Admin está realizando correctamente el enrutamiento entre subredes.

### Comunicación hacia internet

Se observa tráfico ICMP hacia la dirección 8.8.8.8, lo que confirma que los hosts tienen acceso a internet a través del NAT configurado en el router.
<img width="1028" height="1001" alt="image" src="https://github.com/user-attachments/assets/4bad2523-b047-4244-b278-1e10be81b4f0" />

### Tráfico de rendimiento (iperf3)

Se identifica tráfico TCP en el puerto 5201, correspondiente a la prueba de rendimiento realizada con iperf3 entre los hosts.
<img width="1028" height="1007" alt="image" src="https://github.com/user-attachments/assets/d1361d0e-1e62-4bbc-bf7e-12c8dcf2be19" />

## Conclusión

En este laboratorio se logró implementar correctamente una red segmentada en dos subredes interconectadas mediante un router basado en Debian.

Se configuró el enrutamiento entre subredes, permitiendo la comunicación entre los hosts, y se habilitó el acceso a internet mediante la implementación de NAT. Las pruebas de conectividad confirmaron el correcto funcionamiento de la red, evidenciando la comunicación entre los equipos y la salida hacia internet.

Adicionalmente, mediante el uso de iperf3 se validó el rendimiento de la red, observando una transmisión estable de datos entre los hosts. Finalmente, el análisis de tráfico con Wireshark permitió identificar distintos protocolos como ICMP, DNS y TCP, comprobando el flujo de paquetes a través del router.

En conjunto, el laboratorio permitió comprender de manera práctica el funcionamiento del enrutamiento, la traducción de direcciones (NAT) y las herramientas de monitoreo de red.
