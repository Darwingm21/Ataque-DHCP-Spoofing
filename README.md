# Ataque DHCP Spoofing

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Kali%20Linux-red)
![Environment](https://img.shields.io/badge/Environment-GNS3%20%7C%20IOSvL2-orange)
![Use](https://img.shields.io/badge/Use-Controlled%20Lab-yellow)
![Topic](https://img.shields.io/badge/Topic-Network%20Security-purple)

## Información del proyecto

Este repositorio documenta y demuestra un ataque de **DHCP Spoofing** dentro de un laboratorio controlado de seguridad de redes. El objetivo es mostrar cómo un atacante puede levantar un servidor DHCP falso en la misma red local y entregar a una víctima una configuración de red manipulada, incluyendo dirección IP, gateway y DNS controlados por el atacante.

La práctica también incluye una contramedida basada en **DHCP Snooping**, aplicada sobre el switch para bloquear respuestas DHCP provenientes de puertos no confiables.

| Dato                  | Información                                 |
| --------------------- | ------------------------------------------- |
| Autor                 | Darwing                                     |
| Matrícula             | 2024-2690                                   |
| Repositorio           | https://github.com/TuUsuario/DHCP-Spoofing  |
| Video demostrativo    | https://www.youtube.com/watch?v=UJvEugfFb7I |
| Documentación técnica | docs/documentacion-tecnica-profesional.pdf  |

---

## Aviso de uso responsable

Este proyecto fue desarrollado únicamente con fines educativos, académicos y de laboratorio controlado.

El script debe ejecutarse solamente en redes propias, laboratorios autorizados o entornos virtuales como GNS3, EVE-NG o PNETLab. No debe utilizarse en redes públicas, empresariales o de terceros sin autorización explícita.

---

## Objetivo del laboratorio

Demostrar cómo un servidor DHCP no autorizado puede entregar configuración de red falsa a una víctima, haciendo que esta utilice al atacante como gateway y servidor DNS. Esto permite al atacante posicionarse como intermediario (MitM) en el tráfico de red de la víctima, alterando la resolución de nombres y pudiendo redirigir consultas hacia servicios controlados.

---

## Objetivo del script

El script `dhcp_spoofing.py` permite levantar un servidor DHCP falso desde Kali Linux. Su función principal es escuchar solicitudes DHCP en la red y responder con una configuración de red manipulada antes que el servidor legítimo.

El ataque se basa en el proceso DHCP DORA:

```
Discover → Offer → Request → Acknowledge
```

En este laboratorio, Kali responde con un **DHCP Offer** y un **DHCP ACK** maliciosos, asignando como gateway y DNS la IP del atacante:

```
20.24.26.90
```

### Parámetros del script

| Parámetro        | Descripción                                          | Ejemplo              |
| ---------------- | ---------------------------------------------------- | -------------------- |
| `-i` / `--iface` | Interfaz de red desde donde se escucha y responde    | `-i eth0`            |
| `-g` / `--gateway` | IP del gateway falso entregado a la víctima        | `-g 20.24.26.90`     |
| `-d` / `--dns`   | IP del DNS falso entregado a la víctima              | `-d 8.8.8.8`         |
| `-b` / `--base`  | Red base para asignar IPs a las víctimas             | `-b 20.24.26.0`      |

### Requisitos para utilizar la herramienta

- GNS3 o entorno equivalente (EVE-NG, PNETLab).
- Router Cisco con DHCP configurado como servidor legítimo.
- Switch Cisco IOSvL2 o switch compatible.
- Kali Linux con Python 3 instalado.
- Librería Scapy instalada (`pip install scapy`).
- Permisos de superusuario (`sudo`).
- Conectividad de capa 2 entre Kali, PC1 y R1.

---

## Archivos del repositorio

| Archivo                                        | Descripción                                                        |
| ---------------------------------------------- | ------------------------------------------------------------------ |
| `dhcp_spoofing.py`                             | Script principal para ejecutar el servidor DHCP falso.             |
| `mitigacion-dhcp-spoofing.md`                  | Documento con contramedidas contra DHCP Spoofing.                  |
| `README.md`                                    | Guía principal del laboratorio.                                    |
| `docs/documentacion-tecnica-profesional.pdf`   | Documentación técnica profesional detallada del ataque.            |
| `images/`                                      | Capturas de pantalla de evidencia del laboratorio.                 |

---

## Topología del laboratorio

La topología utilizada está compuesta por un router R1 que actúa como servidor DHCP legítimo, un switch SW-1, una máquina Kali Linux atacante y una víctima PC1/VPCS.

```
         R1 (20.24.26.91)
         f0/0
          |
         Gi0/2
        [SW-1]
       /       \
   Gi0/0      Gi0/1
   Kali        PC1
(20.24.26.90)  (víctima)
```

| Dispositivo | Rol                      | Interfaz | Dirección IP   | Descripción                               |
| ----------- | ------------------------ | -------- | -------------- | ----------------------------------------- |
| R1          | Gateway / DHCP legítimo  | F0/0     | 20.24.26.91/24 | Router principal y servidor DHCP legítimo |
| SW-1        | Switch de capa 2         | Gi0/0~2  | N/A            | Switch que conecta todos los equipos      |
| Kali Linux  | Atacante                 | eth0     | 20.24.26.90/24 | Equipo que ejecuta el DHCP falso          |
| PC1 / VPCS  | Víctima                  | e0       | DHCP           | Cliente que recibe configuración falsa    |

---

## Configuración del DHCP legítimo en R1

Antes del ataque, R1 funciona como servidor DHCP legítimo:

```
service dhcp

interface fastEthernet0/0
 ip address 20.24.26.91 255.255.255.0
 no shutdown

ip dhcp excluded-address 20.24.26.1 20.24.26.45
ip dhcp excluded-address 20.24.26.48 20.24.26.254

ip dhcp pool LAN_20242690
 network 20.24.26.0 255.255.255.0
 default-router 20.24.26.91
 dns-server 20.24.26.91
 domain-name lab20242690.local
 lease 0 1
```

---

## Instalación y preparación

Clonar el repositorio:

```bash
git clone https://github.com/TuUsuario/DHCP-Spoofing-Attack.git
cd DHCP-Spoofing-Attack
```

Instalar dependencias:

```bash
pip install scapy
```

Ejecutar el script con permisos administrativos:

```bash
sudo python3 dhcp_spoofing.py -i eth0 -g 20.24.26.90 -d 8.8.8.8 -b 20.24.26.0
```

---

## Ejecución del ataque

Desde Kali Linux se ejecuta el script principal:

```bash
sudo python3 dhcp_spoofing.py -i eth0 -g 20.24.26.90 -d 8.8.8.8
```

La configuración maliciosa entregada por el servidor DHCP falso:

| Parámetro           | Valor           |
| ------------------- | --------------- |
| Servidor DHCP falso | 20.24.26.90     |
| Gateway entregado   | 20.24.26.90     |
| DNS entregado       | 8.8.8.8         |
| Rango falso         | 20.24.26.100+   |
| Máscara             | 255.255.255.0   |

---

## Verificación del ataque en PC1

Desde PC1 se solicita IP por DHCP:

```
PC1> clear ip
PC1> ip dhcp
```

El resultado esperado es que PC1 reciba IP desde Kali:

```
IP: 20.24.26.100/24
GW: 20.24.26.90   ← IP del atacante
DNS: 8.8.8.8
DHCP SERVER: 20.24.26.90
```

---

## Impacto del ataque

- La víctima recibe una IP entregada por el atacante.
- El gateway de la víctima apunta a Kali.
- El DNS de la víctima apunta al atacante.
- El atacante puede interceptar y redirigir el tráfico de la víctima (MitM).
- El atacante puede realizar DNS Spoofing sobre la víctima.

---

## Mitigación con DHCP Snooping

La contramedida principal es **DHCP Snooping** en el switch. Esta protección define qué puertos son confiables para enviar respuestas DHCP.

| Puerto | Dispositivo conectado | Estado DHCP Snooping |
| ------ | --------------------- | -------------------- |
| Gi0/0  | Kali / DHCP falso     | Untrusted            |
| Gi0/1  | PC1 / Cliente         | Untrusted            |
| Gi0/2  | R1 / DHCP legítimo    | Trusted              |

Configuración aplicada en SW-1:

```
enable
configure terminal
ip dhcp snooping
ip dhcp snooping vlan 1
no ip dhcp snooping information option

interface gigabitEthernet0/2
 description Puerto_confiable_hacia_R1_DHCP_legitimo
 ip dhcp snooping trust
 exit

interface gigabitEthernet0/0
 description Puerto_no_confiable_Kali_Atacante
 no ip dhcp snooping trust
 ip dhcp snooping limit rate 5
 exit

interface gigabitEthernet0/1
 description Puerto_no_confiable_PC1_Cliente
 no ip dhcp snooping trust
 ip dhcp snooping limit rate 5
 exit

end
write memory
```

---

## Verificación posterior a la mitigación

Después de aplicar DHCP Snooping, se limpia la configuración IP de PC1 y se solicita DHCP nuevamente:

```
PC1> clear ip
PC1> ip dhcp
```

El resultado esperado es que PC1 ya no acepte respuestas DHCP desde Kali, y solo reciba IP del servidor legítimo R1.

---

## Video demostrativo

La demostración práctica del ataque y su mitigación está disponible en YouTube:

[Ver video del laboratorio en YouTube](https://www.youtube.com/watch?v=PENDIENTE)

---

## Conclusión

Este laboratorio demuestra cómo un atacante puede levantar un servidor DHCP falso para entregar configuración de red maliciosa a una víctima. Al recibir como gateway la IP de Kali, la víctima queda expuesta a ataques MitM, DNS Spoofing y redirección de tráfico.

La mitigación aplicada, **DHCP Snooping**, bloquea respuestas DHCP provenientes de puertos no confiables, protegiendo la red contra este tipo de ataques.

---

## Autor

**Darwing**  
Matrícula: **2024-2690**  
Repositorio: https://github.com/TuUsuario/DHCP-Spoofing-Attack
