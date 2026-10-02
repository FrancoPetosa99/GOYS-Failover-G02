# F0 — Plan de direccionamiento (IPAM) consolidado (R5)

Regla: cero solapamiento. Cada enlace un /30 distinto, cada LAN un /24 distinto.

### 3.1 Enlaces punto a punto

| Enlace / Red | Subred | Dispositivo A (IP/iface) | Dispositivo B (IP/iface) |
|--------------|:------:|--------------------------|--------------------------|
| ISP-1 ↔ EDGE | 192.0.2.0/30 | ISP-1: 192.0.2.1 | EDGE: 192.0.2.2 |
| ISP-2 ↔ EDGE | 192.0.2.4/30 | ISP-2: 192.0.2.5 | EDGE: 192.0.2.6 |
| EDGE ↔ CORE-1 | 10.0.0.0/30 | EDGE: 10.0.0.1 | CORE-1: 10.0.0.2 |
| EDGE ↔ CORE-2 | 10.0.0.4/30 | EDGE: 10.0.0.5 | CORE-2: 10.0.0.6 |
| CORE-1 ↔ CORE-2 (core-core) | 10.0.0.8/30 | CORE-1: 10.0.0.9 | CORE-2: 10.0.0.10 |
| CORE-1 ↔ DIST-1 | 10.0.0.12/30 | CORE-1: 10.0.0.13 | DIST-1: 10.0.0.14 |
| CORE-1 ↔ DIST-2 | 10.0.0.16/30 | CORE-1: 10.0.0.17 | DIST-2: 10.0.0.18 |
| CORE-2 ↔ DIST-1 | 10.0.0.20/30 | CORE-2: 10.0.0.21 | DIST-1: 10.0.0.22 |
| CORE-2 ↔ DIST-2 | 10.0.0.24/30 | CORE-2: 10.0.0.25 | DIST-2: 10.0.0.26 |

Los enlaces ISP↔Cloud (ISP-1↔Cloud1, ISP-2↔Cloud2) no forman parte del plan: reciben dirección por DHCP del NAT de GNS3 y el ISP hace masquerade hacia el cloud. Verificar que el rango del cloud no solape con 192.168.10.0/24 ni 192.168.20.0/24.

Los enlaces a ISP usan 192.0.2.0/24 (TEST-NET-1, RFC 5737) para simular direccionamiento público sin pisar el espacio privado interno. Los enlaces internos usan 10.0.0.0/24 troceado en /30 (quedan libres 10.0.0.28 en adelante para crecer).

### 3.2 LANs de acceso

| Enlace / Red | Subred | Dispositivo A (IP/iface) | Dispositivo B (IP/iface) |
|--------------|:------:|--------------------------|--------------------------|
| USERS (gateway VRRP) | 192.168.10.0/24 | DIST-1: 192.168.10.2 | DIST-2: 192.168.10.3 |
| SERVERS (gateway VRRP) | 192.168.20.0/24 | DIST-1: 192.168.20.2 | DIST-2: 192.168.20.3 |

Hosts: PC-USER 192.168.10.100/24 (gw 192.168.10.1) · SRV 192.168.20.100/24 (gw 192.168.20.1).

### 3.4 Router-IDs y loopbacks

| Nodo | Router-ID | Loopback | AS |
|------|-----------|----------|----|
| ISP-1 | 1.1.1.1 (*) | 198.51.100.1/32 (TEST-NET-2, destino de la prueba "Internet") | 65001 |
| ISP-2 | 2.2.2.2 (*) | 198.51.100.2/32 | 65002 |
| EDGE | 3.3.3.3 (*) | 10.255.0.1/32 | 65000 |
| CORE-1 | 4.4.4.4 | 10.255.0.11/32 | — |
| CORE-2 | 5.5.5.5 | 10.255.0.12/32 | — |
| DIST-1 | 6.6.6.6 (*) | 10.255.0.21/32 | — |
| DIST-2 | 7.7.7.7 (*) | 10.255.0.22/32 | — |

Los router-id de CORE-1 y CORE-2 (4.4.4.4 y 5.5.5.5) vienen del diagrama objetivo. (*) Los demás siguen el mismo patrón (un número por nodo) y son propuesta nuestra. El router-id es solo un identificador: no se asigna ni se anuncia como dirección; las direcciones alcanzables son las loopbacks.

### 3.5 Verificación anti-solapamiento

Rangos usados: 192.0.2.0/29 (ISP), 10.0.0.0/27 (enlaces), 10.255.0.0/24 (loopbacks), 192.168.10.0/24 y 192.168.20.0/24 (LANs), 198.51.100.0/24 (loopbacks ISP). Ninguno se pisa con otro.

VRRP: ver `distribucion-vrrp.md`.
