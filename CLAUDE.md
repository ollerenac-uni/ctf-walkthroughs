# CLAUDE.md — guía para trabajar en este repositorio

Sitio de walkthroughs de CTF en español, hecho con MkDocs Material. El contenido
fuente está en `docs/`; la salida (`site/`) la genera MkDocs, está ignorada por git
y no se edita a mano.

## Rama de trabajo

Desarrolla en `claude/adoring-babbage-0xtzqi`. No empujes a `main` sin permiso explícito.

## Verificación antes de commitear

Para cambios de contenido, comprueba que el sitio construye sin enlaces rotos:

```bash
NO_MKDOCS_2_WARNING=1 mkdocs build --strict
```

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
