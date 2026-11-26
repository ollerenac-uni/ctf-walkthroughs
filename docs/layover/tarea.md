# Tarea · Tu propio blog del walkthrough

Ya seguiste el camino guiado de **Setup** a **Network Sniffing**. Ahora te toca **documentarlo tú**,
como este mismo sitio: vas a montar **tu blog personal** en GitHub (estilo MkDocs, publicado en
GitHub Pages) y a contar en él **tu** avance, etapa por etapa, con **capturas de pantalla** reales.

!!! info "Qué se evalúa"
    Dos cosas: **(1)** que tu blog **esté publicado y se vea** en GitHub Pages, y **(2)** que
    documente tu avance de la **Etapa 0** a la **Táctica 2** con **capturas tuyas** + una explicación
    corta de *qué hiciste y por qué*. No es copiar este sitio: es contar **tu** recorrido.

---

## Parte 1 · Crea tu blog (MkDocs + GitHub Pages)

Son pocos pasos. Necesitas una **cuenta de GitHub**, y en tu máquina **git** y **Python 3 (pip)**.

**1. Instala MkDocs con el tema Material** (el paquete trae MkDocs dentro):

```bash
pip install mkdocs-material
```

**2. Crea el proyecto** y entra en la carpeta:

```bash
mkdocs new mi-blog-htb
cd mi-blog-htb
```

Esto te deja un `mkdocs.yml` y una carpeta `docs/` con un `index.md`.

**3. Activa el tema Material** — edita `mkdocs.yml` y pon al menos:

```yaml
site_name: Mi blog HTB — Layover
theme:
  name: material
  language: es
```

**4. Previsualiza en local** mientras escribes (recarga sola al guardar):

```bash
mkdocs serve
```

Abre **http://127.0.0.1:8000** en el navegador. Mientras esto corra, cada cambio en `docs/` se ve al
instante.

**5. Crea el repositorio en GitHub.** Con la CLI `gh` (desde la carpeta del proyecto):

```bash
gh repo create mi-blog-htb --public --source=. --push
```

??? note "¿Sin la CLI `gh`? Hazlo desde la web"
    Crea el repo vacío en **github.com → New repository** (público, sin README), y conéctalo:
    ```bash
    git init && git add -A && git commit -m "Mi blog HTB"
    git branch -M main
    git remote add origin https://github.com/<tu-usuario>/mi-blog-htb.git
    git push -u origin main
    ```

**6. Publica en GitHub Pages** — un solo comando construye el sitio y lo empuja a la rama `gh-pages`:

```bash
mkdocs gh-deploy --force
```

**7. Tu blog ya está online** en:

```
https://<tu-usuario>.github.io/mi-blog-htb/
```

??? tip "Si la página no aparece en 1–2 min"
    Ve a **Settings → Pages** del repo y confirma que *Source* = **Deploy from a branch**, rama
    **`gh-pages`** / carpeta `/ (root)`. GitHub tarda un momento en construir la primera vez.

??? question "¿Cómo agrego páginas y menú?"
    Cada página es un `.md` dentro de `docs/`. El menú lateral lo controlas con la clave `nav:` en
    `mkdocs.yml`:
    ```yaml
    nav:
      - Inicio: index.md
      - "0 · Setup": setup.md
      - "1 · Discovery": discovery.md
      - "2 · Network Sniffing": sniffing.md
    ```
    Cada vez que cambies algo, repites el paso 6 (`mkdocs gh-deploy --force`) para actualizar lo
    publicado. Documentación oficial: **MkDocs** (mkdocs.org) y **Material** (squidfunk.github.io/mkdocs-material).

---

## Parte 2 · Documenta tu avance con capturas (Etapa 0 → Táctica 2)

Una página (o sección) por etapa. En cada una, **capturas de tu propia sesión** + un párrafo breve
de *qué hiciste y cómo lo supiste* (el razonamiento, no solo el comando).

**Etapa 0 · Setup**

- [ ] VPN arriba: la terminal con **`Initialization Sequence Completed`**.
- [ ] Los **dos servicios** expuestos (tu escaneo de puertos: `22` y `3389`).
- [ ] El **escritorio RDP** de `airside-ws01` ya abierto (`xfreerdp`).

**Táctica 1 · Discovery**

- [ ] Quién eres y qué puedes: salida de **`id`** y **`sudo -l`**.
- [ ] Las **interfaces de red** (`ip a`): la WiFi que aparece y se conecta.
- [ ] El **portal cautivo** que salta al coger conexión.

**Táctica 2 · Network Sniffing**

- [ ] La interfaz en **modo monitor** (`iw` / `ip link`).
- [ ] La **captura con `tshark`** corriendo (el contador de paquetes subiendo).
- [ ] El momento clave: las **credenciales de `jenny`** extraídas del tráfico.

!!! tip "Cómo tomar las capturas"
    - Pantalla completa o la ventana concreta — que se lea el texto.
    - Guárdalas en tu repo (p. ej. `docs/assets/`) e insértalas en Markdown:
      `![VPN conectada](assets/vpn-ok.png)`.
    - **Tapa lo que no quieras mostrar** (tokens, otras IPs); las flags de un lab de práctica no son
      sensibles, pero adquiere el hábito.

---

## Entregable

| Entregas | Qué |
|---|---|
| **URL del blog** | el enlace `https://<usuario>.github.io/...` funcionando |
| **Repo** | público, con tu `mkdocs.yml` y las páginas |
| **3 etapas** | Setup, Discovery y Network Sniffing, cada una con sus capturas |
| **Explicación** | por captura: qué muestra y por qué fue el paso correcto |

??? success "Rúbrica rápida (cómo me fijo yo)"
    - **El blog publica** y se ve en GitHub Pages. *(imprescindible)*
    - Cubre **las 3 etapas** de la 0 a la 2.
    - Las capturas son **de tu propia sesión**, no de este sitio.
    - Cada paso está **justificado** (razonamiento), no solo pegado.

---

Cuando lo tengas, pásame el enlace de tu GitHub Pages y lo revisamos.
