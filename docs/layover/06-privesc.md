# 6 · Privilege Escalation: Exploitation  (T1068)

**Objetivo:** de `aporter` (usuario normal, sin sudo) a **root**.

## 6.1 · Enfoque

Primer reflejo como `aporter`: **¿puedo usar `sudo`?**

```text
aporter@portal:~$ sudo -l
[sudo] password for aporter:
Sorry, user aporter may not run sudo on portal.
```

**`aporter` no tiene sudo** → la vía fácil está cerrada. Sin sudo, la escalada tiene que venir de
**abusar algo que YA corre con privilegios**. Los sospechosos habituales: binarios **SUID**, tareas
**cron** de root, **capabilities**, exploits de **kernel**, archivos sensibles **escribibles**… y
**servicios corriendo como root** con los que puedas hablar.

**1) ¿Qué corre, y como quién?** No saltes directo a CUPS: mira **todo** con `ps aux` y vigila la
**1.ª columna (usuario)**. Buscas procesos de **root** que además puedas **tocar**:

```text
aporter@portal:~$ ps aux          (columnas de CPU/MEM omitidas)
USER       PID  COMMAND
root         1  /sbin/init                   ┐
root        53  systemd-journald             │ núcleo del sistema: root, pero
root       114  systemd-udevd                │ NO reciben tu input → descártalos
root       305  systemd-logind               ┘
root       280  /usr/sbin/cron -f            ← root; valdría SI hubiera un cron escribible
root       281  /usr/sbin/cupsd -f           ← root; escucha en :631 → le hablas   ★
root       330  udisksd                      ← root; servicio D-Bus (otra superficie)
root       343  sshd -D                      ← root, pero endurecido
root       291  php-fpm: master              ┐ root, pero solo lanzan workers www-data;
root       360  nginx: master                ┘ el master no toma tu input directo
www-data   515  php-fpm: pool www            ┐ NO root → inútiles para subir a root
mysql      507  mariadbd                     ┘
```

Del montón, el que cumple **root + te escucha + superficie gorda (con historial de CVEs)** es
**`cupsd`** (★). Los demás o son núcleo mudo, o están endurecidos, o no son root.

**2) ¿Está activo?**

```text
aporter@portal:~$ lpstat -r
scheduler is running                                     ← CUPS activo (escucha local en :631)
```

**3) ¿Qué versión?** `cupsd -v` da `Permission denied`, así que se la pides a su propio servicio:

```text
aporter@portal:~$ curl -sI http://localhost:631/ | grep -i server
Server: CUPS/2.4 IPP/2.1              ← la cabecera solo da la RAMA: 2.4
```
Para el **parche exacto** (lo que decide si un CVE aplica), al paquete:
```bash
dpkg -s cups-daemon | grep Version   # → Version: 2.4.16-...
```

> **Hallazgo (ya justificado):** **CUPS 2.4.16**, demonio de impresión **corriendo como root**,
> activo en local **y al que puedes hablarle**. → ahora, igual que con Craft: **versión → CVE**.

## 6.2 · Procedimiento · CVE-2026-34990

1. Capturar/forjar un token **`Authorization: Local`** de CUPS (confía en peticiones locales).
2. Con él, crear una **impresora persistente** cuyo destino es un **`file://`**:
   `file:///etc/sudoers.d/aporter`.
3. "Imprimir" un trabajo cuyo **contenido** es:
   ```
   aporter ALL=(ALL) NOPASSWD: ALL
   ```
   → el demonio **root** de CUPS **escribe ese archivo como root**.
4. `sudo` lee la nueva regla → `aporter` ya hace `sudo` sin contraseña:
   ```bash
   sudo -i        # root
   cat /root/root.txt   # ROOT FLAG
   ```

## 6.3 · Patrón: *confused deputy*

!!! info "Por qué funciona"
    Un proceso privilegiado (cupsd root) hace una acción (escribir un archivo) **en tu nombre**,
    **sin verificar** si deberías poder hacerla. Tú "engañas" al diputado para que use SUS
    privilegios. (Detalle a fondo en `vulnerabilidades-explicadas.md`.)

    `/etc/sudoers.d/` es el blanco perfecto: sudo lee **todos** los archivos de ese directorio como
    reglas, y `NOPASSWD: ALL` = root instantáneo. Otros blancos clásicos de escritura-como-root:
    `/root/.ssh/authorized_keys`, un cron en `/etc/cron.d/`, `/etc/passwd`.

## 6.4 · Conclusión

**ROOT FLAG** ✅

## 6.5 · Rutas alternativas

Correr `linpeas` para ver qué otros vectores marca (SUID, cron, capabilities, kernel) y comprobar si
CUPS era la única vía o solo la primera.

!!! note "📸 Captura"
    *(Espacio para `id` mostrando `uid=0(root)` y la root flag.)*

---

**Fin de la cadena.** Vuelve al [inicio](index.md) para ver el diagrama completo.
