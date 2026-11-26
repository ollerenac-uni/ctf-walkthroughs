# C2 · Web — Mini-labs

Cuatro prácticas cortas (~5–7 min cada una), **una por familia**. Son labs **aislados y gratis**: no
montas nada. Haz el de **PortSwigger**; las alternativas son equivalentes por si prefieres picoCTF o
Juice Shop.

!!! note "Qué necesitas"
    Cuenta gratis en **PortSwigger Web Security Academy** y **Burp Suite Community** (para los fáciles
    basta el navegador). Cada lab abre una instancia propia con su URL temporal.

## SQLi · bypass de login

*(~5 min)*

- **Objetivo:** entrar como `administrator` sin su contraseña.
- **Lab:** [SQL injection vulnerability allowing login bypass](https://portswigger.net/web-security/sql-injection/lab-login-bypass) · *Apprentice*
- **Practicas:** inyectar en el campo de usuario para **neutralizar** la comprobación de contraseña.
- **Pista:** ¿qué le hacen una comilla `'` y un comentario `-- ` a la cláusula `WHERE`?
- **Éxito:** sesión iniciada como administrador.
- **Stretch:** [UNION attack, retrieving data from other tables](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables) — leer otra tabla.
- **Alt:** picoCTF *SQLiLite* / *More SQLi* · Juice Shop *Login Admin*.

## XSS · reflejado

*(~5 min)*

- **Objetivo:** ejecutar `alert()` en la página.
- **Lab:** [Reflected XSS into HTML context with nothing encoded](https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded) · *Apprentice*
- **Practicas:** inyectar un `<script>` en el buscador, que se refleja **sin escapar**.
- **Pista:** mira el **código fuente** de la respuesta: ¿dónde aparece tu texto, y qué lo rodea?
- **Éxito:** salta el `alert`.
- **Alt:** Juice Shop *DOM XSS* (buscador) · PortSwigger *Stored XSS into HTML context…*.

## JWT · clave débil

*(~7 min)*

- **Objetivo:** llegar al panel de admin **re-firmando** tu propio token.
- **Lab:** [JWT authentication bypass via weak signing key](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-weak-signing-key) · *Practitioner*
- **Practicas:** crackear la **clave secreta** y firmar un JWT con `sub: administrator`.
- **Pista:** la clave es débil → `hashcat`/`john` con una lista de secretos comunes; para editar y
  firmar el token, la extensión **JWT Editor** de Burp.
- **Éxito:** acceso al panel de administración.
- **Alt:** picoCTF *JaWT Scratchpad* · Juice Shop *Forged Signed JWT*.

## IDOR · referencia directa

*(~5 min)*

- **Objetivo:** leer un dato de **otro usuario** (la contraseña de `carlos`).
- **Lab:** [Insecure direct object references](https://portswigger.net/web-security/access-control/lab-insecure-direct-object-references) · *Apprentice*
- **Practicas:** cambiar un **identificador** en la petición para acceder a un objeto ajeno.
- **Pista:** observa cómo la app recupera tu *chat log*. ¿El nombre del recurso es **predecible**?
- **Éxito:** obtienes la contraseña de `carlos` y entras como él.
- **Alt:** Juice Shop *View Basket* (la cesta de otro) · HTB *Cap* (IDOR sobre una captura).

---

Con los cuatro labs hechos, ya has practicado una vez cada familia antes de los CTFs.
