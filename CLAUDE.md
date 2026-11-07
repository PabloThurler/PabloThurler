# Correo de Pablo Thurler — instrucciones para Claude

Este repo es la memoria de trabajo del agente de correo. Se usa desde la máquina **.191**
(red local de Pablo), que tiene acceso SSH al servidor de correo **.131** (`ssh dc131`).
Leé también `docs/HANDOFF.md` (estado actual y tareas pendientes) antes de hacer nada.

## Reglas de Pablo (no negociables)
- REGLA DE ORO: simplificarle la lectura. Solo clasificar por origen/tema y mover a carpetas.
  Nada de resúmenes recurrentes, avisos, estrellas ni colores en sus bandejas.
- Ningún mail a clientes o prospectos sale sin que Pablo apruebe el TEXTO.
- Nunca borrar correo: mover a otra carpeta o a Trash, jamás eliminar.
- Datos personales (teléfono, CUIT, montos) solo si Pablo lo autoriza.
- La carpeta **abuse** de Linkear (~1.800 mails) NO se mueve ni se toca. Reportes de abuso de IP
  que aparezcan en otras carpetas se pasan a esa carpeta y quedan **sin leer**.
- Cambios en servidores: backup `.bak-<motivo>-<fecha>` antes de editar, cron siempre como archivo
  en `/etc/cron.d/`, y anotar en `/root/CHANGELOG-131.md` (o `CHANGELOG-191.md`) qué se cambió y
  cómo volver atrás.
- Claves: la de sudo está en `/home/pablo/CLAUDE.md` de cada servidor. Nunca escribirla en el chat,
  en un log, en este repo ni en un archivo nuevo. **Este repo es público**: no subir mails, listas de
  contactos, claves ni datos de clientes.

## Servidor de correo .131
- Una casilla Dovecot concentra todo: `pablo@thurler.ar`,
  Maildir `/home/linkearadmin/mail/thurler.ar/pablo/`.
- fetchmail (`/etc/fetchmailrc`) trae: Linkearsrl/INBOX (pablothurler@linkearsrl.com.ar),
  Linkear/*, CNCA/INBOX, Gmail/INBOX, Dongcheng/INBOX, Hotmail/INBOX, KM37/INBOX; además Sistemas/*,
  Publicidad/*, Spam/*, Sent.
- Leer sin marcar: `doveadm search` / `doveadm fetch` (`-u pablo@thurler.ar`, o `-A` para todas las
  casillas). Parsear en Python con `email.message_from_bytes(raw, policy=email.policy.default)`.
- Enviar: `sendmail -t -oi -f <identidad>`; Exim elige el smarthost por remitente
  (`/etc/exim4/identity_smarthosts`). Copia en `.Sent/cur/` (modo 600, mismo dueño) y verificar
  `R=identity_smarthost ... C="250` en `/var/log/exim4/mainlog`.
- Ya existen en `/etc/cron.d/`: mail-file-inbox, mail-triage, mail-subcarpetas,
  mail-clasifica-dipardo, mail-rag, mail-pendientes, mail-digest (scripts en
  `/usr/local/bin/mail-*`, logs en `/var/log/mail-*.log`). Extenderlos, no duplicarlos.

## Gmail (pablothurler@gmail.com)
También accesible por el conector Gmail (MCP). Estructura de etiquetas creada el 2026-11-06/07:
LINKEAR/{Proveedores-AR, NOC-Operacion, Administracion, CABASE, Abuse}, CNCA/Logistica-Embarques,
DONGCHENG/Ventas-Mayoristas, CHINA-PROVEEDORES/{Maquinaria/{Construccion-Vial,
Autoelevadores-Izaje, Agricola-Alimentos, Industrial}, Herramientas, Energia-Bombas-Motores,
Telecom-FTTH, Movilidad-Electrica, Autopartes, Construccion-Materiales, Iluminacion-Electronica,
Jardin, Consumo-Varios, Agentes-Trading, Ferias-Servicios}, CHINA-LOGISTICA,
FINANZAS/{Bancos-Impuestos, Inversiones}, CUENTAS-SEGURIDAD, TECH-SaaS, LECTURA/{Noticias, Promos},
REVISAR-Sospechosos, PERSONAL-Otros.
La misma estructura se debe replicar como carpetas IMAP en el servidor.
