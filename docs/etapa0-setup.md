# Etapa 0 · Setup (la línea de salida, no el reto)

Aquí **no hay deducción**: la máquina te lo entrega. Es cómo llegas a tener un shell en el
workstation —VPN, los servicios expuestos y el comando de entrada—. El verdadero juego empieza
después, en [Discovery](01-discovery.md).

---

## 0.1 · La VPN (tender el cable)

OpenVPN levanta la interfaz `tun0` y **enruta** tu tráfico hacia la red del laboratorio. Sin
esto, tus paquetes nunca alcanzan `10.129.x.x`. No es atacar nada: es conectar tu máquina a la red.

```bash
sudo openvpn machines_us-4.ovpn
```

???+ success "Marcador de éxito"
    La VPN está arriba **solo** cuando aparece la línea:
    ```
    Initialization Sequence Completed
    ```
    Mientras no esté, NO estás conectado aunque el proceso siga vivo. Verifica siempre:
    ```bash
    ip a            # interfaz tun0 con IP 10.10.x.x
    ping -c3 10.129.247.82
    ```

??? note "Incidencia real de esta sesión"
    El primer `.ovpn` se quedaba en `TLS handshake failed` por el camino **UDP 1337**. Se
    resolvió cambiando a otro servidor / a **TCP 443** (443 es HTTPS: ninguna red lo bloquea).
    Detalle en `logs/vpn_connectivity.md`.

---

## 0.2 · Los dos servicios expuestos

Lo único que encuentra un escaneo de puertos:

```
22/tcp   open  ssh           OpenSSH 9.6p1 (Ubuntu)
3389/tcp open  ms-wbt-server xrdp
```

- **22 (SSH):** acceso por línea de comandos.
- **3389 (RDP / xrdp):** escritorio remoto gráfico.

Con credenciales en mano, el reflejo es SSH. Pero el intento es **rechazado**:

```bash
ssh contractor@10.129.247.82
# contractor@10.129.247.82: Permission denied (publickey,password).
```

Ese es el **mismo** mensaje que verías con una contraseña incorrecta: no distingue "credencial
mala" de "acceso denegado por política".

Intentemos con otro servicio: RDP.

---

## 0.3 · Construcción del comando xfreerdp

El comando con el que entramos:

```bash
xfreerdp /u:contractor /p:'Contractor2026!' /v:10.129.247.82 \
         /cert:ignore +clipboard /dynamic-resolution
```

Pieza por pieza (se construye, no se memoriza):

- `xfreerdp` — el cliente RDP de FreeRDP (abre el escritorio remoto).
- `/u:contractor` — **usuario** con el que te autenticas.
- `/p:'Contractor2026!'` — **contraseña**; comillas simples porque el `!` es especial para el shell.
- `/v:10.129.247.82` — **víctima / destino** (`IP[:puerto]`; `:3389` es el default, por eso se omite).
- `/cert:ignore` — xrdp usa un **certificado autofirmado**; sin esto el cliente se detiene a preguntar.
- `+clipboard` — portapapeles compartido host⇄remoto (pegar comandos/creds cómodamente).
- `/dynamic-resolution` — la ventana se adapta al redimensionar (opcional).

??? tip "Convención `/` vs `+` (clic para ver)"
    En FreeRDP, `/opcion:valor` **configura** algo y `+opcion` / `-opcion` **activa / desactiva**
    una función. Por eso es `+clipboard` (activar) y `/cert:ignore` (configurar).

??? question "Opcional: ¿por qué SSH rechazaba la conexión? (curiosidad, no es clave)"
    Ya dentro por RDP, inspeccionamos la config de SSH desde la terminal:

    ```bash
    grep -ri passwordauthentication /etc/ssh/
    # /etc/ssh/sshd_config.d/60-cloudimg-settings.conf:PasswordAuthentication no
    ```

    Un **drop-in** (`/etc/ssh/sshd_config.d/60-cloudimg-settings.conf`) con `PasswordAuthentication
    no` desactiva el login por contraseña vía SSH. En cambio, `xrdp` autentica vía **PAM**, que *no*
    se ve afectado por esa directiva → por eso RDP sí abría.

    <figure>
      <img src="../assets/puertos.svg" alt="SSH bloqueado por el drop-in, RDP abierto vía PAM">
    </figure>

    **Lección:** cuando SSH rechaza una contraseña que *sabes* correcta, revisa `sshd_config` **y**
    `sshd_config.d/`. Un override en un drop-in es fácil de pasar por alto.

<figure>
  <img src="../assets/airside-desktop.png" alt="Escritorio XFCE de airside-ws01 vía FreeRDP">
  <figcaption>Sesión RDP establecida: el escritorio XFCE de <code>airside-ws01</code> como
  <code>contractor</code>. Ya estamos dentro — aquí empieza el trabajo real.</figcaption>
</figure>

---

**Siguiente:** ya conectados, empieza el verdadero juego: descubrir la red donde vive el objetivo
→ [Táctica 1 · Discovery](01-discovery.md)
