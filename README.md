<div align="center">

# 📯 herald

**Tu consultor de ideación pre-spec.**
Convierte ideas sueltas, documentación y código real en una propuesta de implementación validada — y la entrega lista a tu flujo de specs.

`lee código, nunca lo modifica` · `separa el hecho de la propuesta` · `un sistema o varios` · `funciona sola`

</div>

---

## ⚡ Instalación

```bash
# agregarla al proyecto actual
npx skills add 3zequiel3/herald

# o de forma global, para todos tus proyectos
npx skills add 3zequiel3/herald -g
```

Compatible con Claude Code, Cursor, Codex, Copilot, Gemini y cualquier agente que siga la spec de [Agent Skills](https://skills.sh).

---

## 🤔 ¿Qué es herald?

herald es el **paso de ideación que va *antes* de escribir specs**. Le das material crudo — documentación existente, imágenes, prompts, archivos `.md` y código fuente (en modo solo lectura) — y te devuelve una **idea consolidada + propuesta de implementación**: análisis de factibilidad, gap analysis, diseño de integración, riesgos y supuestos.

Es la hermana — y el opuesto — de `chronicle`:

| | **chronicle** | **herald** |
|---|---|---|
| Rol | 📚 Notario | 💡 Consultor |
| Trabajo | Documenta lo que **existe** | Propone lo que **todavía no existe** |
| Inventa | Nada | Propuestas — pero **siempre marcadas como propuestas** |

Esa última fila es el punto central. herald *tiene permitido* proponer, así que lo único que lo mantiene honesto es una **línea dura entre el hecho citado y la propuesta marcada**. Nunca te vende una suposición como si fuera un hecho.

---

## 🎯 ¿Qué hace realmente?

Apuntás herald a uno o más sistemas y le hacés una pregunta como:

> *"El Sistema A maneja acreditaciones y usuarios; el Sistema B tiene su propio módulo de usuarios. Cuando alguien se registra en A, no se refleja en B. ¿Se puede implementar el registro cruzado entre los dos?"*

herald lee ambos (solo lectura), te entrevista sobre lo que una máquina no puede saber, y produce:

- ✅ **Una base factual** — qué hace cada sistema *hoy*, con cada afirmación citada al código/documentación real.
- 💡 **Una propuesta** — el diseño de integración, marcado como especulativo, con los supuestos explícitos.
- ⚠️ **Riesgos y preguntas abiertas** — los modos de falla (¿y si B está caído?, ¿los eventos son idempotentes?) puestos sobre la mesa, no escondidos.
- 🌱 **Un seed** — un prompt base listo para correr en tu flujo de Spec-Driven Development.

**No escribe código.** Produce el razonamiento que hace que valga la pena escribirlo.

---

## 🔧 ¿Cómo funciona?

```
  detecta sistemas ──▶ elige modo (Ideate / Bridge)
        │
  groundea en código real  (usa una KB fresca si existe; si no, lee la
        │                    rebanada relevante — delegado, solo lectura, acotado)
  te entrevista sobre los huecos  (el PORQUÉ, el límite, las restricciones)
        │
  consolida  →  idea + factibilidad + gaps + diseño + riesgos
        │
   ★ APPROVAL GATE ★  ves la propuesta completa, hecho vs propuesta separados
        │             ├─ ajustás → vuelve a consolidar
        │             └─ aprobás → continúa
        ▼
   entrega el seed  →  /sdd-new   (o te lo da a vos, si no hay flujo SDD)
```

**El approval gate no es negociable.** herald nunca entrega nada aguas abajo hasta que *vos* viste la propuesta y la aprobaste. Sin cajas negras.

---

## 🧭 Dos modos

### 🟢 Ideate — un sistema
*"Proponé un sistema de referidos para mi marketplace."* herald groundea la rebanada relevante y hace las preguntas que importan: qué problema resuelve, qué toca, si escala, cuál es la versión mínima que aporta valor.

### 🔵 Bridge — dos o más sistemas
*"Integrá A con B."* Todo lo de Ideate, **más** las preguntas de integración que hunden proyectos en silencio: **fuente de verdad**, dirección del sync, tiempo real vs batch, comportamiento ante fallo parcial, idempotencia, consistencia eventual, el contrato entre sistemas.

herald detecta solo cuántos sistemas hay en juego y propone el modo — vos lo confirmás.

---

<details>
<summary><b>🛡️ La disciplina: hecho vs propuesta</b></summary>

<br>

Cada línea que herald escribe está etiquetada, así nunca tenés que adivinar sobre qué está parada:

```
El Sistema A registra usuarios vía POST /api/register.   [code · src/auth/register.ts#register]
El Sistema B guarda usuarios en una tabla `users`.       [kb · 04_modelo]
La doc de integración describe un batch nocturno.         [doc · PLAN.md §Sync]  ⚠ unverified
[proposal] Al registrarse en A, emitir un evento `user.created` que B consume.
[assumption] B acepta IDs de usuario creados externamente sin colisión.
[risk] Si A está caído durante la escritura de B, el registro cruzado se pierde salvo que se encole.
[open-q] ¿Tiempo real, o es aceptable consistencia eventual?
```

Los hechos se citan. Las propuestas se marcan y se escriben en voz de propuesta. Las fuentes desactualizadas o sin verificar se señalan. Ese etiquetado viaja dentro del seed, así tu flujo de specs hereda las salvedades en vez de redescubrirlas.

</details>

<details>
<summary><b>🧊 Grounding con conciencia de frescura</b></summary>

<br>

herald es **agnóstico a la fuente** — usa una knowledge base de `chronicle`, una hecha a mano, o directamente el código. Lo que le importa es la *confianza*:

- **¿KB fresca?** → la usa, el camino más barato.
- **¿Stale, o vos decís que está desactualizada?** → **el código pasa a ser la fuente de verdad**; herald re-groundea la rebanada relevante desde el código real (delegando a un explorer como `sdd-explore` cuando está disponible).
- **¿Sin ledger de frescura?** → no asume; te pregunta o hace un spot-check.

La ausencia de evidencia de staleness **no** es evidencia de frescura. Un "hecho" sacado de una doc vieja es ficción con cita — y herald lo trata como tal.

</details>

<details>
<summary><b>🔌 Standalone u orquestada</b></summary>

<br>

**Standalone:** ¿no hay flujo SDD ni orquestador? herald igual corre todo el proceso y te entrega el seed en el chat.

**Orquestada:** pegá esto en el archivo de instrucciones de tu orquestador y herald se inserta en la cadena `ideación → herald → seed → /sdd-new → sdd-explore → …`:

```markdown
## herald — pre-spec ideation (front of funnel)

Route to herald when the user asks to ideate or propose something not yet spec'd
(single-system feature → Ideate; cross-system integration → Bridge). Do NOT route to
herald when the idea is already consolidated and code-grounded (go to /sdd-new), when
the user wants existing code documented (use chronicle), or when code must be written.

herald grounds in real code (read-only), separates fact from proposal, and STOPS at a
mandatory user approval gate. On approval it returns { status, mode, seed,
grounding_notes, next_action }:

- status: seed-ready  → fire /sdd-new with `seed` as the base prompt, then run the
                        normal SDD protocol; carry grounding_notes forward.
- status: inline-only → no SDD flow detected; present the proposal, suggest installing SDD.
- status: aborted     → user declined at the gate; do nothing downstream.

Chain: ideation → herald → seed → /sdd-new → sdd-explore → sdd-propose → …
```

</details>

<details>
<summary><b>🚫 Lo que herald nunca hace</b></summary>

<br>

- ✋ Modificar, refactorizar o escribir código fuente — **solo lectura, siempre.**
- ✋ Presentar una propuesta, un supuesto o un riesgo como si fuera un hecho verificado.
- ✋ Entregar algo sin tu aprobación en el gate.
- ✋ Lanzar subagentes de spec por su cuenta — devuelve un seed; el orquestador corre el protocolo.

</details>

<details>
<summary><b>🗂️ Estructura</b></summary>

<br>

```
herald/
├── SKILL.md                          # router liviano
└── assets/
    ├── detection-funnel.md           # detecta sistemas + elige el modo (barato primero)
    ├── grounding.md                  # cuatro estados de frescura, código-como-verdad, lectura delegada
    ├── ideate-interview.md           # batería de preguntas single-system
    ├── bridge-interview.md           # batería de integración cross-system
    ├── consolidation.md              # estructura de la propuesta + approval gate
    ├── seed-contract.md              # estructura del seed-prompt + contrato de retorno
    ├── orchestrator-integration.md   # routing + handshake + bloque pegable
    ├── provenance.md                 # taxonomía hecho vs propuesta + flags de frescura
    ├── edge-cases.md                 # conflictos, inputs faltantes, self-check final
    └── examples.md                   # un ejemplo trabajado por modo
```

</details>

---

## 📄 Licencia

Apache-2.0.

<div align="center">
<sub>herald propone. chronicle documenta. Vos decidís.</sub>
</div>
