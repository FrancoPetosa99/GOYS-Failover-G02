# F0 — Operación: usuarios, change log y backup (R5)

### 4.1 Usuarios y privilegios

- `admin` por defecto: se le cambia la contraseña en los 7 routers (contraseña fuerte, distinta de las claves de protocolos).
- Usuario `monitor`: grupo `read`, usado solo para monitoreo/SNMP y chequeos.
- Se evita compartir la cuenta `admin` entre integrantes: cada cambio queda atribuido en el change log y en git.
- Acceso de gestión solo por SSH, limitado por IP de origen (red de gestión/loopbacks internas).

### 5.1 Formato del change log

Convención de commits (Conventional Commits):

```
tipo(alcance): descripción breve
```

Tipos: `feat`, `fix`, `docs`, `ops`, `chore`. Ejemplos: `feat(configs): agrego VRRP vrid10 en DIST-1`, `fix(ospf): corrijo auth-key en CORE-2`, `ops(backup): backup post-F2`.

Reglas:

- Un commit por cambio lógico; cada rol commitea su parte.
- `main` siempre funciona; lo roto se arregla antes de mergear.
- El change log de la memoria (sección 6.1) refleja los commits.

Registro en la memoria:

| Fecha | Responsable | Cambio | Motivo | Cómo se revierte |
|-------|-------------|--------|--------|------------------|

### 5.2 Política de backup

- **Cuándo:** antes de cada cambio riesgoso, al cerrar cada fase (F1 a F4) y antes de cada drill.
- **Cómo:** `/export file=<router>-<AAAAMMDD>-<fase>` (texto, versionado en git) y `/system backup save name=<router>-<AAAAMMDD>-<fase>` (binario, no se sube al repo).
- **Dónde:** exports en `backups/<fecha>/`; configs finales en `configs/`.
- **Restore probado:** se prueba en al menos un router en F1 y se documenta la evidencia.
- **Snapshot BASE:** snapshot de GNS3 tras desplegar la topología, antes de configurar.
- **Responsable:** R5 es dueño de backups, change log y backlog.
