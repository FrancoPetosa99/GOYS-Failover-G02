# F0 — Core: defectos 3 y 4 (R3)

| # | Defecto detectado | Corrección aplicada | Justificación |
|:-:|-------------------|---------------------|---------------|
| 3 | Sin enlace core–core | Se agrega CORE-1 ↔ CORE-2 dentro de OSPF área 0. | Sin él, si cae un enlace CORE↔DIST el tráfico puede quedar sin camino óptimo o blackholeado; el enlace da un camino alterno entre cores. |
| 4 | HSRP en el core (diseño "collapsed") | El FHRP se mueve a distribución con VRRP (RFC 5798, estándar abierto). El core queda como tránsito puro. | Separa responsabilidades: el core solo transporta, el primer salto vive donde están los usuarios. VRRP además funciona en MikroTik. |

Los enlaces EDGE↔CORE y CORE↔DIST se validaron contra el IPAM (`ipam.md`, sección 3.1).
