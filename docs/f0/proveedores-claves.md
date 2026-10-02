# F0 — Proveedores y claves de autenticación (R2)

Los enlaces y loopbacks de ISP-1/ISP-2 se validaron contra `ipam.md`. Las claves reales **no** se suben al repo: se comparten por canal privado del grupo.

### 4.3 Claves de autenticación

| Mecanismo | Dónde | Identificador | Clave |
|-----------|-------|---------------|-------|
| OSPF MD5 | Todas las interfaces OSPF (EDGE, CORE, DIST) | key-id 1 | `<KEY>` (a definir por el grupo) |
| BGP TCP-MD5 | EDGE↔ISP-1 | secret propio | `<KEY>` (a definir por el grupo) |
| BGP TCP-MD5 | EDGE↔ISP-2 | secret propio (distinto al de ISP-1) | `<KEY>` (a definir por el grupo) |
| VRRP auth | Grupos 10 y 20 en DIST-1/DIST-2 | VRRP v2, auth simple | `<KEY>` (a definir por el grupo) |

Reglas: una clave distinta por mecanismo, no usar claves triviales, no commitear las claves reales al repo (en `configs/` y la memoria se muestran como `<KEY>`; las reales se comparten por canal privado del grupo y se entregan al docente si las pide). Nota: VRRP en RouterOS v7 soporta autenticación solo con `version=2`; verificarlo en F2.
