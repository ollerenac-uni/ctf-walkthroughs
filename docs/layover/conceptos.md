# Conceptos necesarios · Layover

Antes (o mientras) resuelves Layover, conviene tener claros estos conceptos. **No es la solución**:
es el *fondo* que hace que cada paso tenga sentido. Cada fila enlaza a dónde aparece en el
walkthrough.

!!! info "Cómo usar esta página"
    Si un término te suena a chino al leer una táctica, vuelve aquí. Lo marcado con 🔒 pertenece a la
    parte que **no está publicada** (es tu [Tarea](tarea.md)): lo dejamos a nivel de *concepto*, sin
    revelar el camino.

## Prerrequisitos (nivelación)

Lo mínimo para no perderte. Si algo de esto es nuevo, repásalo primero.

- **Terminal de Linux:** moverte, leer archivos, encadenar comandos y redirigir salida (`2>/dev/null`).
- **Redes, lo básico:** IP, subred, gateway, rutas; qué es un puerto y un servicio.
- **HTTP:** petición/respuesta, métodos (`GET`/`POST`), formularios, cabeceras, cookies.

## Mapa de conceptos del reto

| Concepto | Qué es (en una línea) | Dónde aparece |
|---|---|---|
| **VPN / `tun`** | túnel que enruta tu tráfico hacia la red del lab | [Etapa 0 · Setup](etapa0-setup.md) |
| **SSH vs RDP** | dos formas de acceso remoto; una autentica por clave/PAM, otra da escritorio | [Etapa 0 · Setup](etapa0-setup.md) |
| **Enumeración del host** | *¿quién soy, qué permisos tengo, dónde estoy?* (`id`, `sudo -l`, `ip`) | [1 · Discovery](01-discovery.md) |
| **Interfaces WiFi simuladas** | tarjetas de red creadas por software (`mac80211_hwsim`) | [1 · Discovery](01-discovery.md) |
| **Managed vs Monitor** | el modo *monitor* deja oír **todo** el canal, no solo lo tuyo | [2 · Network Sniffing](02-monitoreo.md) |
| **WiFi abierta** | sin cifrado → cualquiera en el canal ve el tráfico de todos | [2 · Network Sniffing](02-monitoreo.md) |
| **Sniffing pasivo** | *escuchar* el aire sin inyectar ni suplantar nada | [2 · Network Sniffing](02-monitoreo.md) |
| **HTTP en claro** | tráfico sin TLS → credenciales legibles si alguien se loguea | [2 · Network Sniffing](02-monitoreo.md) |
| **`tshark` / campos de Wireshark** | la CLI de Wireshark; disecar un paquete por campos (`http.file_data`) | [2 · Network Sniffing](02-monitoreo.md) |

## Conceptos de la parte no publicada 🔒

Lo que **explorarás en la [Tarea](tarea.md)**. A nivel de idea, para que sepas qué terreno pisarás
—sin el *cómo*—:

- **Fingerprinting de aplicaciones web** 🔒 — identificar qué software y **qué versión** corre, y de
  ahí buscar su fallo conocido (CVE) en la fuente correcta.
- **Ejecución de código** 🔒 — pasar de "tengo sesión en una app" a "ejecuto comandos en el servidor".
- **Criptografía aplicada** 🔒 — distinguir un valor **cifrado** (reversible con su llave) de un
  **hash** (no); recuperar secretos de la configuración de una app.
- **Movimiento lateral** 🔒 — reutilizar una credencial recuperada para volverte un **usuario real**.
- **Escalada de privilegios** 🔒 — abusar algo que **ya corre con privilegios** para llegar a `root`.

!!! tip "Dónde reforzar cada tema"
    Estos conceptos son justo las **categorías del curso** (Web, Cripto, Forense, Pwn…). El plan de
    clases los trata uno por uno; esta máquina los mezcla todos en una sola cadena.

---

¿Listo? Entra al walkthrough: **[Etapa 0 · Setup](index.md)**.
