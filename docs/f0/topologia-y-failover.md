# F0 — Topología y failover por capa (R1)

## 1. Topología lógica (5 capas, 11 nodos, 15 enlaces)

Diagrama objetivo: [`docs/diagramas/topologia-objetivo.png`](../diagramas/topologia-objetivo.png). Además de los 11 nodos del lab (7 routers, 2 switches, 2 hosts), el diagrama muestra dos nodos *Cloud* (Cloud1 y Cloud2) que dan salida a Internet a ISP-1 e ISP-2 vía NAT de GNS3; no cuentan como nodos ni como enlaces del lab.

```
[INTERNET]       Cloud1 ─ ISP-1 (AS 65001)          ISP-2 (AS 65002) ─ Cloud2
                      \                      /          eBGP
[EDGE]                 EDGE (AS 65000)
                      /                \                OSPF área 0
[CORE]           CORE-1 ───────────── CORE-2            core-core
                  |  \                /  |
                  |   \______  ______/   |
                  |          \/          |
                  |          /\          |
                  |   ______/  \______   |
                  |  /                \  |
[DISTRIBUTION]   DIST-1               DIST-2            VRRP + OSPF
                  |  \                /  |
[ACCESS]          SW-ACC-USERS        SW-ACC-SERVERS            (cada switch dual-homed a DIST-1 y DIST-2)
                     |                 |
                  PC-USER             SRV
```

Los 15 enlaces:

1. ISP-1 – EDGE
2. ISP-2 – EDGE
3. EDGE – CORE-1
4. EDGE – CORE-2
5. CORE-1 – CORE-2 (core-core)
6. CORE-1 – DIST-1
7. CORE-1 – DIST-2
8. CORE-2 – DIST-1
9. CORE-2 – DIST-2
10. DIST-1 – SW-ACC-USERS
11. DIST-2 – SW-ACC-USERS
12. DIST-1 – SW-ACC-SERVERS
13. DIST-2 – SW-ACC-SERVERS
14. SW-ACC-USERS – PC-USER
15. SW-ACC-SERVERS – SRV

> Nota: el reparto de los 4 enlaces DIST↔acceso (un switch por LAN, cada uno conectado a DIST-1 y DIST-2) y los 2 enlaces de hosts coincide con el diagrama objetivo. Los nombres de puerto del diagrama (0/0, 1/0, …) se mapean a `etherN` de los CHR en F1.

**Diferencias con el diagrama original ("Enterprise Network Design (Cisco)", IT With Manas):** el original muestra 2 ISR 4331 (AS 65001/65002) con un iBGP entre ellos, un ASA 5506-X como edge, 2 Nexus 9300 en el core con HSRP (VLAN 10/20/30), 4 switches de distribución (D1–D4, Catalyst 2960-X) y 4 sitios de acceso con VLANs repetidas. El lab lo reduce a 1 EDGE (AS 65000), 2 CORE, 2 DIST y 2 LANs (USERS y SERVERS), conservando la lógica de 5 capas.

### 3.6 Diseño de failover por capa (mapa de la clase)

| Capa | Pregunta | Mecanismo en nuestro diseño | Drill |
|------|----------|-----------------------------|:-----:|
| 1. Primer salto | ¿Y si se cae mi gateway? | VRRP en DIST-1/DIST-2 (vrid 10 y 20) | 1 y 5 |
| 2. Enlace | ¿Y si se cae el link? | Rutas por defecto con `check-gateway=ping` y `distance` 1/2 | 4 |
| 3. IGP | ¿Y si se cae un camino interno? | OSPF área 0 con core–core (reconvergencia por CORE-2) | 2 |
| 4. Inter-AS | ¿Y si se cae mi proveedor? | eBGP ×2 en EDGE; ISP-1 primario, ISP-2 backup (local-pref) | 3 |

Decisiones a validar con el docente (el detalle fino está en la guía `laboratorio-failover-routing.md`):

- **BGP:** ISP-1 como salida primaria vía `local-pref` mayor en EDGE; ISP-2 como respaldo. Los ISP originan `default-originate`. Tiempo esperado de reconvergencia: hasta el hold timer (~90 s según la clase) si no se detecta por caída de interfaz; el valor real se mide y se documenta en el drill 3.
- **Check-gateway (drill 4):** propuesta de rutas estáticas por defecto en EDGE hacia ISP-1 (distance 1) e ISP-2 (distance 2) con `check-gateway=ping`, como respaldo estático independiente de BGP. A confirmar contra la guía.
- **Limitación conocida de VRRP:** VRRP solo conmuta cuando cae el enlace LAN del master (`laboratorio-vrrp.md`, sección 7). Si un DIST pierde sus uplinks hacia el core pero conserva su LAN, sigue siendo master y deja a los hosts sin salida. Lo dejamos documentado como riesgo; en el lab lo cubre el diseño con doble uplink por DIST (a CORE-1 y CORE-2), y en producción se resolvería con tracking del uplink.
- **Tiempos de referencia:** VRRP debería conmutar en pocos segundos (el warm-up lo mide en < 3 s); OSPF y BGP dependen de los timers configurados. Cada drill registra su tiempo medido.

### 3.7 Observaciones del análisis que quedan fuera del alcance del lab

Del `analisis.md` hay puntos que reconocemos pero no implementamos porque los switches de acceso del lab son simples: VLAN de gestión y native VLAN hardening, QoS para voz, STP root designado, y DHCP snooping/DAI/port-security. Los documentamos como mejoras de producción.
