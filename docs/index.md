# CTF Walkthroughs — la manufactura

Colección de walkthroughs centrados en **cómo se construye** la solución de una CTF: el
*razonamiento* —razonar → deducir → testear → concluir → re-iterar—, no solo la lista de comandos.
Cada entrada documenta una CTF por su **cadena de deducciones**.

!!! info "Cómo leer este sitio"
    Cada CTF vive en su propia sección (menú de arriba). Dentro, el **Setup** (cómo recibes tu
    plataforma de ataque) se separa del **juego deductivo**, organizado por **tácticas MITRE ATT&CK**.

## Walkthroughs publicados

<div class="grid cards" markdown>

-   :material-server-network: **[Layover](layover/index.md)** · Máquina HackTheBox

    ---

    Boot2root que cruza varias categorías: recon, **network sniffing**, RCE web, acceso a
    credenciales, movimiento lateral y privesc. Publicado hasta *Network Sniffing*; el resto es
    **[tu tarea](layover/tarea.md)**.

    **Estado:** ✅ Setup → Táctica 2 · 🧩 Tarea (montar tu propio blog + resolver 3–6)

    [:material-book-open-variant: Conceptos necesarios](layover/conceptos.md) ·
    [:material-play: Walkthrough](layover/index.md)

</div>

## Ejercicios CTF

| Clase | Categoría | Enfoque | Estado |
|---|---|---|---|
| C2 | **[Web](c2-web/index.md)** | SQLi, XSS, APIs (JWT/IDOR) | 🛠 conceptos + mini-labs listos |
| C3 | **Criptografía** | RSA con fallas, AES ECB, hashes | 🔜 próximamente |
| C4 | **Forense** | PCAP, memoria (Volatility), estego | 🔜 próximamente |
| C5 | **Reversing** | Ghidra, ASM x86-64, crackmes | 🔜 próximamente |
| C6 | **Pwn** | buffer overflow, ret2win, ROP | 🔜 próximamente |
| C7 | **Hardware** | firmware, UART, analizador lógico | 🔜 próximamente |
| C8 | **Simulacro** | CTF interno por equipos, 6 categorías | 🔜 próximamente |

!!! tip "La pregunta de oro (vale para toda CTF)"
    En cada táctica pregúntate: *¿era esta la única forma de llegar a la misma conclusión, o solo la
    primera que encontré?* Ahí está el aprendizaje.
