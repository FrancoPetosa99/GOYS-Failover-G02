# F0 — Edge: defectos 1 y 5, servicios y firewall (R1)

## Defectos del diagrama

| # | Defecto detectado | Corrección aplicada | Justificación |
|:-:|-------------------|---------------------|---------------|
| 1 | Firewall sin par HA (SPOF) | En el lab el filtrado vive en EDGE (firewall de RouterOS). Se documenta que en producción van dos firewalls en failover activo/standby. | Un único firewall es punto único de falla: si cae, cae toda la salida a Internet aunque haya dos ISP. |
| 5 | iBGP Route Reflector mal ubicado (a través del firewall) | eBGP directo en el edge con cada ISP, sin RR. | Con un solo router de borde y dos ISP no hace falta RR; eBGP en el edge es lo correcto y más simple. |

### 4.2 Servicios que se deshabilitan

- Telnet, FTP, WWW (HTTP), API y API-SSL en los 7 routers.
- Winbox: solo si hace falta, restringido por IP; si no, deshabilitado.
- MAC-server / MAC-winbox y neighbor discovery (`/ip neighbor discovery-settings`) hacia enlaces externos.
- Bandwidth-test server.
- Se deja: SSH (restringido) y SNMPv3/v2c solo lectura para monitoreo.

### 4.4 Firewall edge (diseño)

- Entrada (input) en EDGE: aceptar established/related, ICMP limitado, BGP solo desde las IP de los ISP, SSH solo desde la red interna; descartar el resto.
- Forward: permitir tráfico desde redes internas hacia afuera y respuestas; descartar tráfico entrante no solicitado.
- Anti-spoofing: descartar en la interfaz hacia los ISP origen con IP privadas/propias.
