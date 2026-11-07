# Traspaso: de la sesión en la nube a la máquina .191

Fecha: 2026-11-07. La sesión anterior corría en la nube y **no podía llegar al .131** (sin SSH, red
local). Desde la .191 sí se puede. Esta es la lista de lo hecho y lo pendiente.

## 1. Cómo arrancar en la .191
```bash
# una sola vez
curl -fsSL https://claude.ai/install.sh | bash        # instala Claude Code
git clone https://github.com/PabloThurler/pablothurler ~/correo
cd ~/correo
ssh dc131 true && echo "SSH al .131 OK"                # comprobar acceso
claude                                                # abre la sesión; lee CLAUDE.md solo
```
Para manejarla desde el celular o la app: dentro de `~/correo` correr `claude remote-control`.
Elegir modo **Auto** para que no pida permiso por cada acción.

## 2. Hecho (en Gmail, por conector)
- Creada la estructura de etiquetas (ver CLAUDE.md).
- Dos agentes clasificaron Gmail: ~390 hilos de proveedores chinos (2014–2026) y ~690 del resto del
  Inbox. Quedaron **a medio terminar** por reinicios del contenedor en la nube. Para retomar: buscar
  en Gmail `in:inbox has:nouserlabels` y lo que no tenga etiqueta de la estructura.
- Búsqueda "Ivy Zhong / goeateroof.com" (chapas de techo): **no está en Gmail**. Proveedores de chapa
  encontrados en Gmail: Henan Tiga Engineering (GI/PPGI, 2020), CUS Steel (2017), HengJi (2013).

## 3. Pendiente (hacer desde la .191)
1. **Buscar Ivy Zhong / goeateroof en todo el servidor** (todas las casillas y carpetas, solo lectura):
   ```bash
   ssh -t dc131 'sudo doveadm fetch -A "user mailbox date.received hdr.from hdr.to hdr.subject" mailbox "*" OR TEXT goeateroof TEXT "Ivy Zhong"'
   ```
   Armar ficha del proveedor (qué ofrece, precios, estado del hilo).
2. **Spam del .131** (`Spam/*`): leer, rescatar lo legítimo y moverlo a la carpeta que corresponda.
   Nada se borra.
3. **Reporte de todos los INBOX del .131** (Linkearsrl, Linkear, CNCA, Gmail, Dongcheng, Hotmail,
   KM37): qué es cada cosa, sospechosos (virus/phishing) y newsletters en resumen; el resto,
   clasificado con la misma estructura que Gmail.
4. **Reportes de abuso de IP** fuera de la carpeta abuse → moverlos a abuse, sin leer. La carpeta
   abuse (~1.800) no se toca.
5. **Migración .131 → .191** (ver sección 4). No arrancar sin OK explícito de Pablo.

## 4. Plan de migración del correo a la .191 (propuesta, a confirmar)
Objetivo: que toda la información de correo exista en la .191 (copia completa, luego decidir si la
.191 pasa a ser el servidor principal).

Fase A — Inventario en .131 (solo lectura):
- `doveadm user '*'` (casillas), `doveadm mailbox status -A "messages vsize" '*'` (tamaños),
  versiones de Dovecot/Exim/Roundcube/MariaDB, `/etc/fetchmailrc`, `/etc/exim4/`, `/etc/dovecot/`,
  `/etc/cron.d/mail-*`, `/usr/local/bin/mail-*`, `/var/lib/<tareas>/state.json`, base `roundcube`.
- Espacio libre en .191 (`df -h`) ≥ 1,5 × tamaño total del correo.

Fase B — Copia sin cortar el servicio:
- Instalar en .191 Dovecot (misma versión mayor) y crear las casillas.
- Copiar el correo con `doveadm backup` / `doveadm sync` vía SSH (o `rsync -aHAX` del Maildir con
  Dovecot detenido para el pase final). Conserva carpetas, flags (leído/no leído) y UIDs.
- Copiar configs y scripts `mail-*`, la base `roundcube` (`mysqldump roundcube`) y los `state.json`.

Fase C — Verificación:
- Comparar por casilla y carpeta la cantidad de mensajes y el tamaño (.131 vs .191).
- Probar lectura por IMAP y Roundcube en la .191.

Fase D — Corte (solo con OK de Pablo):
- Detener fetchmail y los crons en .131, último `doveadm sync`, mover fetchmail y crons a .191,
  actualizar DNS/MX o NAT si la .191 pasa a recibir correo. El .131 queda de respaldo, sin borrar
  nada.
