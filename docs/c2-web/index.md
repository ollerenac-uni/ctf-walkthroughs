# C2 · Web

**Objetivo de la clase:** saltarse un login con **SQLi**, extraer una base de datos con **UNION**,
robar una cookie con **XSS** y abusar de una **API** mal protegida (**JWT / IDOR**).

## 1 · Estructura (2 h)

| Tramo | Tiempo | Contenido |
|---|---|---|
| **Conceptos (embudo)** | 30 min | [Conceptos](conceptos.md) + [mini-labs](mini-labs.md) de apoyo |
| **CTF1 · Easy** | 45 min | **OWASP Juice Shop** (nivel 1–2) · SQLi/XSS/JWT/IDOR |
| **CTF2 · intermedio** | 45 min | **OWASP WebGoat** · JWT/IDOR/SQLi/XSS |
| **Tarea** | casa | **picoCTF — Cookies** (Easy) · mini-writeup |

## 2 · Retos

<div class="grid cards" markdown>

-   **CTF1 · [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)** · Easy

    ---

    App OWASP deliberadamente vulnerable; cubre **las cuatro familias** de la clase. En clase se juega
    el nivel **1–2**: SQLi de login, XSS en el buscador, IDOR de la cesta y un primer JWT.

    **Cubre:** SQLi · XSS · JWT · IDOR
    **Plataforma:** self-host (Docker), con modo CTF.
    **Walkthrough:** [resolverlo paso a paso](ctf1-juice-shop.md)

-   **CTF2 · [OWASP WebGoat](https://owasp.org/www-project-webgoat/)** · intermedio

    ---

    App OWASP self-host con lecciones guiadas. Cubre **a fondo** lo que Juice Shop toca ligero:
    **JWT**, **control de acceso / IDOR**, **SQLi avanzada** y **XSS**. Gratis y documentable.

    **Cubre:** JWT · IDOR · SQLi · XSS
    **Plataforma:** self-host (Docker).
    **Walkthrough:** [resolverlo paso a paso](ctf2-webgoat.md)

</div>

!!! note "Tarea (CTF3) · picoCTF — Cookies (Easy)"
    Reto autocontenido de [picoGym](https://play.picoctf.org/) (manipular una **cookie** para volverte
    admin): refuerza *input del cliente = no confiable* y prepara el terreno de JWT/IDOR. Se entrega un
    **mini-writeup**. *(Cualquier web Easy de picoGym sirve si prefieres otra — p. ej. `SQLiLite`.)*

## 3 · Temas

**SQLi · XSS · APIs (JWT / IDOR).**

## 4 · Lectura previa (~2 h)

Los *learning paths* de PortSwigger Web Security Academy (teoría + labs guiados):

- [SQL injection](https://portswigger.net/web-security/sql-injection)
- [Cross-site scripting (XSS)](https://portswigger.net/web-security/cross-site-scripting)
- [Access control / IDOR](https://portswigger.net/web-security/access-control)
- [JWT attacks](https://portswigger.net/web-security/jwt)

## 5 · Herramientas

Burp Suite Community · `curl` · `sqlmap` · DevTools del navegador (F12). Para JWT, la extensión
**JWT Editor** de Burp; para crackear, `hashcat` o `john`.

---

**Empieza por:** [Conceptos (30 min)](conceptos.md) → [Mini-labs](mini-labs.md) → CTFs.
