# F0 — Distribución: defecto 2 y VRRP (R4)

## Defecto del diagrama

| # | Defecto detectado | Corrección aplicada | Justificación |
|:-:|-------------------|---------------------|---------------|
| 2 | Subredes solapadas entre sitios (192.168.10.0/24 repetida) | IPAM con un /30 distinto por enlace y un /24 distinto por LAN, sin overlap (sección 3). | El solapamiento rompe el ruteo ante cualquier interconexión futura y hace ambigua la tabla de rutas. |

### 3.3 VRRP (load-sharing)

| Grupo | VRID | Master | Priority (master / backup) | IP virtual | MAC virtual |
|-------|:----:|:------:|:--------------------------:|:----------:|:-----------:|
| USERS | 10 | DIST-1 | 150 / 100 | 192.168.10.1/32 en `vrrp10` | 00:00:5E:00:01:0A |
| SERVERS | 20 | DIST-2 | 150 / 100 | 192.168.20.1/32 en `vrrp20` | 00:00:5E:00:01:14 |

Cada DIST es master de un grupo y backup del otro, así ambos están activos (drill 5). Cada DIST lleva dos interfaces VRRP (`vrrp10` sobre la interfaz de USERS y `vrrp20` sobre la de SERVERS). Siguiendo el warm-up (`laboratorio-vrrp.md`), la IP virtual se asigna como /32 a la interfaz `vrrp` y la IP real de cada router queda en la interfaz LAN (.2 en DIST-1, .3 en DIST-2). La MAC virtual sigue el patrón `00:00:5E:00:01:<VRID en hexa>` (RFC 5798): VRID 10 = 0x0A, VRID 20 = 0x14. Los hosts apuntan siempre al .1, nunca al .2 ni al .3. Autenticación: ver sección 4 (VRRP v2 con auth simple).
