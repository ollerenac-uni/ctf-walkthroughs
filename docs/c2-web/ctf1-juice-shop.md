# CTF1 · OWASP Juice Shop

La app deliberadamente vulnerable de OWASP. En esta clase (Easy, ~45 min) resolvemos el nivel **1–2**
que toca las familias del día: **SQLi**, **XSS** e **IDOR** (y vemos por qué el JWT se deja para
[CTF2](index.md)). Todo lo que sigue está hecho sobre una **instancia real**.

## 1 · Montaje

```bash
docker run -d --name juiceshop -p 3000:3000 bkimminich/juice-shop
```

Abre **http://localhost:3000**, cierra el diálogo de bienvenida y el banner de cookies.

## 2 · Score Board

El **reto 0**: el tablero de retos existe pero **no está enlazado**. En una SPA las rutas ocultas
siguen ahí — la pruebas por nombre obvio:

```
http://localhost:3000/#/score-board
```

Encontrarlo ya cuenta como reto, y te da el **mapa**: categorías (Injection, XSS, Broken Access
Control…) y su dificultad.

<figure>
  <img src="../assets/js-score-board.jpg" alt="Score Board de Juice Shop con las categorías de retos">
</figure>

!!! tip "Lección"
    Lo que no está enlazado no está protegido. Mira el JavaScript de la SPA (o prueba rutas obvias):
    el enrutado del cliente delata páginas que el menú no muestra.

## 3 · SQLi: Login Admin

**Objetivo:** entrar como administrador sin su contraseña.

**Razonamiento:** el login mete tu email en una **consulta SQL**. Si la app no separa datos de código,
una comilla rompe la consulta. En el campo **email**:

```
' OR 1=1--
```

(contraseña: cualquiera).

**Por qué funciona** — la consulta queda, en esencia:

```sql
SELECT * FROM Users WHERE email = '' OR 1=1--' AND password = '...'
```

El `OR 1=1` vuelve el `WHERE` **siempre verdadero**; `--` **comenta** el resto (la comprobación de la
contraseña). Devuelve la **primera fila**: el admin.

<figure>
  <img src="../assets/js-sqli-login.jpg" alt="Payload SQLi en el campo email del login">
</figure>

**Resultado:** sesión como `admin@juice-sh.op` → reto **Login Admin** resuelto. El token que recibes
es un **JWT** (`header.payload.signature`); recuérdalo para más tarde.

## 4 · XSS: DOM XSS

**Objetivo:** ejecutar JavaScript en la página a través del buscador.

**Razonamiento:** el término de búsqueda se **inserta en el DOM**. Si no se escapa, tu texto se
interpreta como HTML/JS. Payload en el buscador:

```html
<iframe src="javascript:alert(`xss`)">
```

<figure>
  <img src="../assets/js-dom-xss.jpg" alt="El iframe inyectado se renderiza en los resultados de búsqueda">
</figure>

El `<iframe>` **se renderiza** (el recuadro vacío sobre los resultados): tu input se tomó como
**HTML**, no como texto. En un navegador normal salta el `alert`. Reto **DOM XSS** resuelto.

!!! note "¿Por qué *DOM* XSS?"
    Porque es el **JavaScript del cliente** quien escribe tu input en la página (sin sanear), no el
    servidor. El payload nunca "da la vuelta" por el backend.

## 5 · IDOR: View Basket

**Objetivo:** ver la cesta de **otro** usuario.

**Razonamiento:** la cesta se pide por **id** a la API. Si el servidor no comprueba que es *tuya*,
cambiando el id ves la de cualquiera. Prueba directa contra la API, **con tu propio token de admin**:

```text
GET /rest/basket/1  → 200   owner 1   (la tuya)
GET /rest/basket/2  → 200   owner 2   Raspberry Juice     ← ajena
GET /rest/basket/3  → 200   owner 3   Raspberry Juice     ← ajena
```

La API responde **200** a cestas que no te pertenecen → **IDOR / Broken Access Control**. Por la UI
basta cambiar tu identificador de cesta (`bid` en `localStorage`) a `2` y recargar la cesta.

<figure>
  <img src="../assets/js-solved.jpg" alt="Los cuatro retos resueltos: Score Board, Login Admin, DOM XSS, View Basket">
</figure>

Reto **View Basket** resuelto (arriba se ven las cuatro familias ya marcadas).

## 6 · JWT

Juice Shop **sí** tiene retos de JWT, pero son **forjar un token firmado con RSA** (nivel 4–5):
demasiado para la franja Easy. El token que recibiste al loguearte es un JWT; su explotación a fondo
(clave débil, `alg:none`, re-firmado) la trabajamos en **[CTF2 · WebGoat](ctf2-webgoat.md)**.

## 7 · Mapa familia → reto

| Familia | Reto Juice Shop | Nivel |
|---|---|:--:|
| *(orientación)* | Score Board | 1 |
| **SQLi** | Login Admin | 2 |
| **XSS** | DOM XSS | 1 |
| **IDOR** | View Basket | 2 |

## 8 · Limpieza

```bash
docker rm -f juiceshop
```

---

**Siguiente:** [CTF2 · WebGoat](ctf2-webgoat.md) — JWT e IDOR a fondo.
