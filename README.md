# Laboratorio de Redes - Segmentación y Monitoreo

## Integrantes
Lesly Juliana Ascencio Peréz

## 1. Introducción
En este laboratorio se implementó una arquitectura de red segmentada basada en máquinas virtuales, con el objetivo de simular un entorno real donde múltiples subredes se comunican a través de un router central.
Adicionalmente, se integraron herramientas de monitoreo y análisis de tráfico que permiten observar el comportamiento de la red en tiempo real, así como aplicar políticas de control (QoS) para gestionar el uso del ancho de banda.



## 2. Diseño de la arquitectura de red
Se diseñó una topología compuesta por tres nodos:

- Debian-Admin: Router central con NAT
- Host1: Cliente en Subred 1
- Host2: Servidor en Subred 2

### Segmentación de red
- Subred 1: 192.168.10.0/24
- Subred 2: 192.168.20.0/24

La segmentación permite aislar dominios de broadcast, mejorar la organización de la red y simular entornos empresariales donde diferentes departamentos operan en redes separadas.

## Tecnologías utilizadas

## Tecnologías utilizadas

Durante el desarrollo del laboratorio se utilizaron las siguientes tecnologías:

- **VirtualBox**: Plataforma de virtualización para la creación de las máquinas virtuales.
- **Debian 13**: Sistema operativo base utilizado en todos los nodos.
- **Redes internas y NAT**: Configuración de red en VirtualBox para segmentación y acceso a internet.
- **IP forwarding (Linux)**: Habilitación del reenvío de paquetes para funcionamiento como router.
- **iptables**: Implementación de NAT para permitir salida a internet.
- **iPerf3**: Generación y medición de tráfico de red (throughput).
- **tcpdump**: Captura y análisis de paquetes en la red.
- **Wireshark**: Análisis detallado de capturas de tráfico.
- **Prometheus**: Recolección y almacenamiento de métricas del sistema.
- **Node Exporter**: Exportación de métricas del sistema para Prometheus.
- **Grafana**: Visualización de métricas mediante dashboards.
- **Zabbix Agent**: Monitoreo basado en agente del sistema.
- **softflowd**: Simulación de exportación de flujos tipo NetFlow.
- **tc (Traffic Control)**: Implementación de políticas de QoS para control de tráfico.

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

## 3. Configuración de direccionamiento IP

Se asignaron direcciones IP estáticas para garantizar control total sobre el entorno:

- Host1
IP: 192.168.10.10/24
Gateway: 192.168.10.1

- Host2
IP: 192.168.20.10/24
Gateway: 192.168.20.1

El uso de gateways permite enrutar tráfico fuera de la subred local hacia el router.
---

## Host1

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

El Host2 fue creado a partir de la clonación de Host1, con el objetivo de agilizar la implementación del laboratorio. Posteriormente, se realizaron ajustes para adaptarlo a la subred 2.

Se asignó una dirección IP estática correspondiente a la subred 2:

```bash
ip addr flush dev enp0s3
ip addr add 192.168.20.10/24 dev enp0s3
ip link set enp0s3 up
ip route add default via 192.168.20.1
```
<img width="951" height="1004" alt="image" src="https://github.com/user-attachments/assets/167b7c8a-e3dd-42da-90ba-7bdadd50931f" />


## Configuración del router (Debian-Admin)
El router se configuró con tres interfaces de red:

enp0s3 → NAT (salida a internet)
enp0s8 → Subred 1
enp0s9 → Subred 2
Habilitación del reenvío de paquetes
```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Esto activa el kernel de Linux para funcionar como router de capa 3.

## NAT (traducción de direcciones)

Se utilizó NAT para permitir que las subredes privadas accedan a internet mediante la interfaz NAT.

Concepto clave:
El NAT traduce direcciones privadas (RFC1918) a una dirección pública.

### Configuración IP

```bash
ip addr add 192.168.10.1/24 dev enp0s8
ip addr add 192.168.20.1/24 dev enp0s9
```
<img width="983" height="920" alt="image" src="https://github.com/user-attachments/assets/eeb7155a-32f3-404d-9472-2a27668980c3" />

## 5. Validación de conectividad

Se realizaron pruebas de conectividad desde el nodo central hacia ambas subredes y hacia internet.

```bash
ping 192.168.10.10
ping 192.168.20.10
ping 8.8.8.8
ping google.com
```
<img width="1030" height="1000" alt="image" src="https://github.com/user-attachments/assets/796f9cb8-877a-4a9c-a6c9-2f981973b9d8" />

## 6. Generación y análisis de tráfico (iPerf3)

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
- El enrutamiento configurado en Debian-Admin es funcional
- La conectividad entre Host1 y Host2 es correcta
- La red permite transmisión de datos sin pérdida significativa

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

### 7. Captura y análisis de tráfico

Se utilizó la herramienta `tcpdump` en la interfaz enp0s8 del router Debian-Admin para capturar el tráfico de la red interna.

```bash
tcpdump -i enp0s8 -w captura_final.pcap
```

### Análisis técnico

Se identificaron:

ICMP → pruebas de conectividad
TCP → tráfico de iPerf3
DNS → resolución de nombres
Esto permite observar tráfico a nivel de capa 3 y 4.

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

## 8. Monitoreo con Prometheus y Grafana
### Monitoreo con Prometheus y Grafana

Se implementó un sistema de monitoreo utilizando Prometheus y Grafana para visualizar métricas en tiempo real del sistema.

### Configuración

Se implementó un sistema de monitoreo basado en métricas.

### Arquitectura
- Node Exporter → expone métricas del sistema
- Prometheus → recolecta y almacena métricas
- Grafana → visualiza los datos
- Métricas monitoreadas
- CPU (uso por núcleo)
- Memoria disponible
- Estado de servicios

Prometheus utiliza un modelo pull, consultando periódicamente los endpoints.

### Evidencias

<img width="476" height="503" alt="login_grafana" src="https://github.com/user-attachments/assets/3ec16389-3e17-420d-bbd3-88f6374f8dcc" />

<img width="472" height="187" alt="configuracion_prometheus_grafana" src="https://github.com/user-attachments/assets/40be8084-167b-4414-b33d-897122c3b667" />

<img width="338" height="93" alt="protmetheus_API" src="https://github.com/user-attachments/assets/d9d7ed34-4d66-4fc0-ada9-23edfecbaaf7" />

<img width="475" height="371" alt="dashboard_grafana" src="https://github.com/user-attachments/assets/30e03e97-16cd-43d8-a250-6b40e6ca1507" />

## 9. Monitoreo con Zabbix
### Monitoreo con Zabbix
Se configuró Zabbix Agent en el nodo Debian-Admin.

### Función
Permite monitoreo basado en agentes, complementando Prometheus.

### Configuración
- Server=127.0.0.1
- ServerActive=127.0.0.1
- Hostname=Debian-Admin

Zabbix permite:
- monitoreo activo y pasivo
- recolección estructurada de métricas
  
### Evidencia

<img width="193" height="112" alt="configuracion_zabbix" src="https://github.com/user-attachments/assets/0d474562-6854-4748-b970-4b0a073ebf1f" />

<img width="478" height="289" alt="status_zabbix" src="https://github.com/user-attachments/assets/1145d221-afe7-4ad5-9be7-18a5648fe861" />

## 10. Análisis de flujo de red (NetFlow)
### Análisis de tráfico de red (NetFlow)
Se utilizó softflowd para simular exportación de flujos.

### Concepto de flujo
Un flujo se define por:
- IP origen/destino
- Puerto origen/destino
- Protocolo
- Cantidad de datos
Se implementó el análisis de flujo de red utilizando la herramienta softflowd.
Se generó tráfico entre Host1 y Host2 mediante iPerf3, permitiendo identificar los flujos de comunicación.
Posteriormente, se capturó el tráfico utilizando tcpdump en el router.
Se observaron paquetes TCP en el puerto 5201 correspondientes a iPerf3.

### Evidencia
<img width="1902" height="929" alt="netflow_tcpdump" src="https://github.com/user-attachments/assets/698077a0-bf97-4c4b-854c-a5fbe6ea05c4" />

<img width="1902" height="929" alt="netflow_tcpdump" src="https://github.com/user-attachments/assets/dc773b8c-e5a3-437f-a9e3-d11d2bfc9585" />

# Punto 2 - QoS
## Control de tráfico (QoS)

Se simuló tráfico de red utilizando iPerf3, generando un flujo constante de datos entre los hosts. Posteriormente, se aplicó una política de control de tráfico utilizando la herramienta tc en el router Debian-Admin.

### Funcionamiento
- Se limita el ancho de banda
- Se controla la tasa de transmisión
  
```bash
tc qdisc add dev enp0s8 root tbf rate 20mbit
```

Se observó una reducción en la velocidad de transferencia, evidenciando el impacto del control de tráfico.

### Aplicación real:
QoS se usa en redes para priorizar tráfico crítico (video, VoIP, etc.)

### Evidencia
<img width="833" height="494" alt="trafico_normal_sin_prioridad" src="https://github.com/user-attachments/assets/745a8216-8b20-4b20-a104-c6537a7cbf05" />

<img width="1915" height="939" alt="trafico_red_con_qos_estable" src="https://github.com/user-attachments/assets/5394ca16-b5bf-4380-9dda-a00cf93ab279" />

## Conclusión

En este laboratorio se implementó una red segmentada con enrutamiento y acceso a internet mediante NAT, validando la conectividad entre subredes. 
Se generó y analizó tráfico de red con herramientas como iPerf3 y tcpdump, y se integraron soluciones de monitoreo como Prometheus, Grafana y Zabbix para visualizar el estado del sistema. 
Finalmente, se aplicaron políticas de control de tráfico (QoS), evidenciando su impacto en el rendimiento de la red. Esto demuestra la importancia de combinar monitoreo y gestión para optimizar el funcionamiento de infraestructuras de red.
