# 5 · Lateral Movement: Remote Services  (T1021.004 + T1078)

!!! question "¿Loguearse en el CP de Craft era movimiento lateral?"
    **No realmente.** Loguear a `jenny` en el panel fue *usar credenciales válidas para autenticarte
    a la app* (precondición del exploit) — eso vive en la [Táctica 3](03-exploitation.md). El
    **movimiento lateral de verdad** —saltar a una **sesión en otro host con otra identidad**— es
    **este** paso: SSH como `aporter`.

**Objetivo:** un shell interactivo como usuario real → user flag.

## 5.1 · Procedimiento

**¿Por qué SSH, y a esa máquina?** El razonamiento encadenado:

- Tu RCE es como **`www-data`**, una cuenta de *servicio*: sin shell útil, sin home, no lee el user
  flag. Necesitas ser un **usuario humano**.
- Recuperaste un **usuario + contraseña** (`aporter : Skyp0rt_Relay!26`). Una contraseña sirve para
  una cosa: **loguearte como ese usuario**.
- **¿Es `aporter` una cuenta de login?** Compruébalo con el RCE antes de intentar nada:
  ```bash
  RCE 'id aporter; grep aporter /etc/passwd'
  # … /home/aporter … /bin/bash   → tiene home y shell = cuenta real de login
  ```
- **¿A qué máquina?** Al **mismo host del portal** (`10.13.37.10`, donde ya tienes el RCE): ahí vive
  `aporter` y ahí está el user flag. Lo alcanzas porque estás en la WiFi `10.13.37.0/24`.
- **¿SSH y no `su`?** Podrías `su aporter` desde el RCE, pero el RCE es un *ejecuta-un-comando* sin
  TTY. **SSH te da un shell interactivo limpio** como `aporter`, directo desde tu box.

```bash
ssh aporter@10.13.37.10        # Skyp0rt_Relay!26
cat ~/user.txt                 # USER FLAG
```

!!! note "La lección: reutilización de contraseñas"
    Ese password estaba guardado como secreto de **relay de correo**, pero resultó ser también la
    contraseña **de sistema/SSH** de `aporter`. Un secreto recuperado **siempre** vale la pena
    probarlo como credencial de login. (Y a diferencia del workstation de la
    [Etapa 0](etapa0-setup.md), este host **sí** acepta login por contraseña.)

## 5.2 · Conclusión

```text
contractor@airside-ws01:~$ ssh aporter@10.13.37.10
The authenticity of host '10.13.37.10' can't be established.
ED25519 key fingerprint is SHA256:3zC8PGwDfCgcR1KjYS3E5WmhpZlk+tdqTI47GLwTS1k.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
aporter@10.13.37.10's password:
aporter@portal:~$ cat ~/user.txt
b1356da67fd527c82e43fcc6b2a2b519
```

- **Host key (primera vez):** SSH no conoce la máquina → te pide confirmar su huella (*TOFU*,
  *trust-on-first-use*). Dices `yes` y la guarda en `~/.ssh/known_hosts`.
- **El prompt cambia a `aporter@portal`** → ya eres usuario real, y el hostname **`portal`** confirma
  que es el servidor web.
- **USER FLAG:** `b1356da67fd527c82e43fcc6b2a2b519`.

**Siguiente:** [Táctica 6 · Privilege Escalation](06-privesc.md)
