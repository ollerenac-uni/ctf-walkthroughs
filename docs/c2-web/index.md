# C2 · Web

**Objetivo de la clase:** saltarse un login con **SQLi**, extraer una base de datos con **UNION**,
robar una cookie con **XSS** y abusar de una **API** mal protegida (**JWT / IDOR**).

## Estructura (2 h)

| Tramo | Tiempo | Contenido |
|---|---|---|
| **Conceptos (embudo)** | 30 min | [Conceptos](conceptos.md) + [mini-labs](mini-labs.md) de apoyo |
| **CTF1 · Easy** | 45 min | **OWASP Juice Shop** (1–2⭐) · SQLi/XSS/JWT/IDOR |
| **CTF2 · intermedio** | 45 min | **HTB GoodGames** (retirada) · SQLi→IDOR→SSTI |
| **Tarea** | casa | CTF3 Easy · *por fijar* |

## Retos

<div class="grid cards" markdown>

-   :material-fruit-citrus: **CTF1 · [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)** · Easy

    ---

    App OWASP deliberadamente vulnerable; cubre **las cuatro familias** de la clase. En clase se juega
    el tier **1–2⭐**: SQLi de login, XSS en el buscador, IDOR de la cesta y un primer JWT.

    **Cubre:** SQLi · XSS · JWT · IDOR
    **Plataforma:** self-host (Docker), con modo CTF.

-   :material-server-network: **CTF2 · [HTB GoodGames](https://www.hackthebox.com/machines/goodgames)** · intermedio

    ---

    Máquina **retirada** de HackTheBox. Cadena web real: **SQLi** (bypass + extraer y crackear hashes)
    → **IDOR** → **SSTI** → fuga de Docker. Sube el listón al *encadenar* fallos.

    **Cubre:** SQLi · IDOR · SSTI + escape
    **Plataforma:** HackTheBox (retirada, documentable).

</div>

!!! note "Tarea (CTF3) — por fijar"
    El reto de casa (Easy) se fija aparte. Idea: otra Easy que refuerce lo menos tocado en clase
    (p. ej. **XSS almacenado** o una **API JWT**).

## Temas

**SQLi · XSS · APIs (JWT / IDOR).**

## Lectura previa (~2 h)

Los *learning paths* de PortSwigger Web Security Academy (teoría + labs guiados):

- [SQL injection](https://portswigger.net/web-security/sql-injection)
- [Cross-site scripting (XSS)](https://portswigger.net/web-security/cross-site-scripting)
- [Access control / IDOR](https://portswigger.net/web-security/access-control)
- [JWT attacks](https://portswigger.net/web-security/jwt)

## Herramientas

Burp Suite Community · `curl` · `sqlmap` · DevTools del navegador (F12). Para JWT, la extensión
**JWT Editor** de Burp; para crackear, `hashcat` o `john`.

---

**Empieza por:** [Conceptos (30 min)](conceptos.md) → [Mini-labs](mini-labs.md) → CTFs.
