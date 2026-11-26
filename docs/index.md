# CTF Walkthroughs

Colección de walkthroughs centrados en cómo se construye la solución de una CTF: el razonamiento
(razonar, deducir, probar, concluir, repetir), no solo la lista de comandos. Cada entrada documenta
una CTF por su cadena de deducciones.

!!! info "Cómo leer este sitio"
    Cada CTF vive en su propia sección (menú de arriba). Dentro, el **Setup** (cómo recibes tu
    plataforma de ataque) se separa del **juego deductivo**, organizado por **tácticas MITRE ATT&CK**.

## 1 · Walkthroughs publicados

<div class="grid cards" markdown>

-   :material-server-network: **[Layover](layover/index.md)** · C1 · Máquina HackTheBox

    ---

    Boot2root que cruza varias categorías: recon, **network sniffing**, RCE web, acceso a
    credenciales, movimiento lateral y privesc. Publicado hasta *Network Sniffing*; el resto es
    **[tu tarea](layover/tarea.md)**.

    **Estado:** ✅ Setup → Táctica 2 · 🧩 Tarea (montar tu propio blog + resolver 3–6)

    [:material-book-open-variant: Conceptos necesarios](layover/conceptos.md) ·
    [:material-play: Walkthrough](layover/index.md)

</div>

## 2 · Ejercicios CTF

| Clase | Categoría | Enfoque | Estado |
|---|---|---|---|
| C1 | **[Introducción](layover/index.md)** | CTF, VPN, nmap, recon, network sniffing · *Layover (HTB)* | ✅ Setup → Táctica 2 · 🧩 tarea |
| C2 | **[Web](c2-web/index.md)** | SQLi, XSS, APIs (JWT/IDOR) | 🛠 conceptos, mini-labs y CTFs fijados |
| C3 | <span class="coming-soon">Criptografía</span> | RSA con fallas, AES ECB, hashes | 🔜 próximamente |
| C4 | <span class="coming-soon">Forense</span> | PCAP, memoria (Volatility), estego | 🔜 próximamente |
| C5 | <span class="coming-soon">Reversing</span> | Ghidra, ASM x86-64, crackmes | 🔜 próximamente |
| C6 | <span class="coming-soon">Pwn</span> | buffer overflow, ret2win, ROP | 🔜 próximamente |
| C7 | <span class="coming-soon">Hardware</span> | firmware, UART, analizador lógico | 🔜 próximamente |
| C8 | <span class="coming-soon">Simulacro</span> | CTF interno por equipos, 6 categorías | 🔜 próximamente |

!!! tip "Al terminar cada táctica"
    Pregúntate si esa era la única forma de llegar a la conclusión, o solo la primera que encontraste.
    Buscar rutas alternativas suele enseñar más que la solución en sí.
