# CLAUDE.md — guía para trabajar en este repositorio

Sitio de walkthroughs de CTF en español, hecho con MkDocs Material. El contenido
fuente está en `docs/`; la salida (`site/`) la genera MkDocs, está ignorada por git
y no se edita a mano.

## Rama de trabajo y publicación

Desarrolla en `claude/adoring-babbage-0xtzqi`. Al hacer push a esa rama, un workflow
de GitHub Actions (`.github/workflows/deploy.yml`) construye el sitio y lo publica en
`gh-pages`. El sitio público refleja esta rama en uno o dos minutos.

`main` es una instantánea estable opcional. No empujes a `main` sin permiso explícito.

## Verificación antes de commitear

Para cambios de contenido, comprueba que el sitio construye sin enlaces rotos:

```bash
NO_MKDOCS_2_WARNING=1 mkdocs build --strict
```

## Estructura de las páginas (obligatorio)

En las páginas de `docs/`, las subsecciones van numeradas. Sigue el formato que ya
usan las páginas de Layover: `## 0.1 · La VPN`, `## 1.2 · Enfoque`. Si una
subsección tiene apartados propios, se numeran un nivel más: `### 0.1.1 · ...`.
El título de la página (`#`) no lleva número de subsección.

Las páginas que no son una etapa (índices, conceptos, tarea, páginas de C2) numeran
desde 1: `## 1 · ...`, `### 1.1 · ...`. Si el título ya tenía un separador `·`, se
cambia por dos puntos: `## 3 · SQLi: Login Admin`.

Al renumerar cambian las anclas de los títulos: revisa los enlaces `pagina.md#ancla`.

## Estilo de redacción (obligatorio)

Español, registro neutro y sobrio, tono técnico. El objetivo es que el texto no
suene a IA. Reglas:

- Escribe directo y plano. Di cada cosa una vez; no la reformules entre paréntesis.
- Nada de relleno motivacional ni de "coach" ("ahí está el aprendizaje", "la
  pregunta de oro", "tú puedes", "suerte", "a por ello"). Informa, no animes.
- Sin anglicismos si existe la palabra en español (onboarding → introducción,
  highlight → resaltar). Los términos técnicos establecidos se quedan: flag,
  payload, hash, sniffing, shell.
- Sin metáforas, analogías ni frases hechas ("te suena a chino", "el bisturí",
  "vivir de la tierra", "la aguja en el pajar", "un guiño del diseñador") salvo
  que sean un término técnico real.
- Enfatiza con moderación: negrita solo para el término clave la primera vez que
  aparece. Sin cursiva decorativa ni mayúsculas para dar énfasis.
- Evita el guion largo (—) como pausa dramática; usa punto, coma o dos puntos.
- Evita la cadencia de "regla de tres" (tres adjetivos o cláusulas seguidas).
- Frases cortas y declarativas. Si una oración pasa de unas 25 palabras, pártela.
- No cierres secciones con frases-resumen inspiradoras ni con emoji decorativo. El
  emoji funcional (p. ej. 🔒 para marcar contenido no publicado) sí se permite.

Ninguna regla elimina el tono por completo: revisa el resultado antes de dar algo
por terminado.
