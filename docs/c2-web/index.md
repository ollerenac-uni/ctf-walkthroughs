# C2 · Web

**Objetivo de la clase:** saltarse un login con **SQLi**, extraer una base de datos con **UNION**,
robar una cookie con **XSS** y abusar de una **API** mal protegida (**JWT / IDOR**).

## Estructura (2 h)

| Tramo | Tiempo | Contenido |
|---|---|---|
| **Conceptos (embudo)** | 30 min | [Conceptos](conceptos.md) + [mini-labs](mini-labs.md) de apoyo |
| **CTF1 · Easy** | 45 min | *por fijar — candidato: OWASP Juice Shop (1–2⭐)* |
| **CTF2 · intermedio** | 45 min | *por fijar — candidato: HTB GoodGames (retirada)* |
| **Tarea** | casa | CTF3 Easy *por fijar* |

!!! note "CTFs por confirmar"
    Los retos (CTF1/2/3) se fijan aparte. Los conceptos y mini-labs de esta sección **no dependen**
    de cuáles sean: cubren las familias (SQLi, XSS, JWT, IDOR) que luego aparecen en los retos.

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
