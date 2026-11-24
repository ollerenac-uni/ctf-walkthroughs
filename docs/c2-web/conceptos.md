# C2 · Web — Conceptos (30 min)

!!! abstract "Embudo"
    De la **idea madre** (arriba) a las **tres familias** concretas (abajo). Cada nivel acota más;
    al final, cada familia tiene su [mini-lab](mini-labs.md) de práctica.

| Nivel | Min | Qué |
|---|---|---|
| 1–3 | 0–15 | lo general: confianza, HTTP, el patrón común |
| 4–6 | 15–28 | lo específico: SQLi, XSS, APIs (JWT/IDOR) |
| 7 | 28–30 | puente a los CTFs |

## 1 · Modelo de confianza web

*(min 0–5)*

- Cliente ↔ servidor: el navegador manda **peticiones**, el servidor **responde**.
- Regla de oro: **todo lo que llega del cliente es input no confiable** — URL, formularios, headers,
  cookies. El usuario puede cambiarlo **todo**.
- Una vulnerabilidad web = **el servidor confía en algo que no debería**. De aquí cuelgan las tres
  familias de hoy.

## 2 · El canal: HTTP

*(min 5–12)*

- Petición/respuesta; métodos **GET** (datos en la URL) y **POST** (datos en el cuerpo).
- Dónde viajan los datos del usuario: **query string**, **cuerpo**, **headers**, **cookies**.
- **Sesión:** tras loguearte, el servidor te da una cookie/token que te identifica en cada petición.
- Herramientas para *ver y tocar* el canal:
    - **DevTools (F12)** → pestañas *Network* y *Storage*.
    - **`curl`** → repetir y modificar una petición desde la terminal.
    - **Burp Suite** → proxy que intercepta y edita el tráfico (el bisturí).

??? note "Anatomía de una petición"
    ```http
    POST /login HTTP/1.1
    Host: tienda.htb
    Cookie: session=eyJ...              ← quién eres
    Content-Type: application/x-www-form-urlencoded

    username=jenny&password=1234        ← tu input (no confiable)
    ```
    Todo lo editable por ti —ruta, cookie, cuerpo— es un punto de entrada potencial.

## 3 · El patrón común

*(min 12–15)*

Una sola idea, tres caras. Un dato del usuario **cruza una frontera** y el otro lado lo **interpreta**
como algo con poder:

- como **consulta a la base de datos** → **SQLi**
- como **HTML / JavaScript en el navegador** → **XSS**
- como **identidad o permiso** → **APIs: JWT / IDOR**

Si entiendes esto, las tres familias son variaciones del **mismo error**: mezclar **datos** con
**código** o con **identidad**.

## 4 · SQLi

*(min 15–19)*

- El input entra en una **consulta SQL** sin separar datos de código.
- **Bypass de login:** `' OR 1=1 -- ` vuelve el `WHERE` siempre-verdadero.
- **UNION:** pegar un `UNION SELECT` para leer **otras tablas** (usuarios, hashes).
- **Blind:** sin ver el resultado directo; se infiere por **booleano** o por **tiempo**.
- **Señal:** una comilla `'` rompe la página o cambia el comportamiento.

## 5 · XSS

*(min 19–23)*

- El input se **refleja en el HTML/JS** y se ejecuta en el navegador de **otra víctima**.
- Tipos: **reflejado** (en la respuesta inmediata), **almacenado** (guardado y servido a otros),
  **DOM** (en el JavaScript del cliente).
- Impacto típico: **robar la cookie de sesión** → suplantar al usuario.
- **Señal:** tu input aparece *tal cual* en la página, sin escapar.

## 6 · APIs: JWT e IDOR

*(min 23–28)*

**JWT** — el token *es* tu identidad firmada:

- `header.payload.signature`, cada parte en base64url; el payload dice quién eres (`"sub":"user"`).
- Fallos: **clave débil** (crackearla y re-firmar como `admin`), **`alg:none`** (sin firma).
- **Señal:** una cookie/token con **tres partes** separadas por puntos.

**IDOR** — referencia directa sin control de acceso:

- `/api/user/123` → cambias a `124` y ves datos de otro, porque **nadie comprueba que te pertenece**.
- **Señal:** identificadores **predecibles** en la URL o el cuerpo (secuenciales, tu nombre…).

## 7 · Puente a los CTFs

*(min 28–30)*

- Checklist mental para cualquier web: **¿dónde entra mi input?** y **¿como qué lo interpreta el
  otro lado?** (consulta, marcado, identidad).
- CTF1 (Easy) y CTF2 (intermedio) son estas familias **encadenadas** en una app real.
- Antes de los retos, pasa por los **[mini-labs](mini-labs.md)**: una práctica corta por familia.
