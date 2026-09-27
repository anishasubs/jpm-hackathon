# Atlas — knowledge copilot and cadence layer

JPM hackathon entry (submission **Oct 6, 2026**, category: Assistants & Interfaces).

Atlas makes a product team's operating knowledge (end-to-end flows, checkpoints, ownership and
reporting cadence) answerable in natural language and self-maintaining. ShiMR is the first deployment
and the demo case.

## Links

| What | Where |
|---|---|
| **Figma — screens** | https://www.figma.com/design/iXDUVq6xs7RFJiSkXe7Ovk (page **⭐ Atlas — Screens**) |
| Design notes | [`design/README.md`](design/README.md) |
| Mock data | [`data/cadence-mock.json`](data/cadence-mock.json) |

The Figma file is built on J.P. Morgan's open-source [Salt Design System](https://www.saltdesignsystem.com/).
Figma access is by invite. If the link says you don't have access, ask Anisha to share the file or the
JPM Hackathon folder.

## Screens

Four tabs in one persistent frame:

| Tab | Purpose | Design |
|---|---|---|
| Flow Explorer | See the end-to-end flow, its checkpoints and what is live per region | Next |
| Ask | Ask a question or be walked through a flow step by step | Planned |
| Cadence | Update the sections you own and clear flags against them | [Done](design/screens/cadence-my-sections.png) |
| Admin | Roles, node visibility, section ownership, lens defaults (admin only) | Planned |

![Cadence — My sections](design/screens/cadence-my-sections.png)

## Data

Everything in this repo is **fictional mock data**. No customer data, PII, transaction records or live
rule content belongs here. The real corpus stays in the approved environment.
