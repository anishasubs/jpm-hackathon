# Atlas — knowledge copilot and cadence layer

JPM hackathon entry (submission **Oct 6, 2026**, category: Assistants & Interfaces).

Atlas makes a product team's operating knowledge (end-to-end flows, checkpoints, ownership and
reporting cadence) answerable in natural language and self-maintaining. ShiMR is the first deployment
and the demo case.

## Links

| What | Where |
|---|---|
| **Figma — screens** | https://www.figma.com/design/iXDUVq6xs7RFJiSkXe7Ovk (page **⭐ Atlas — Screens**) |
| PRD | [`docs/PRD - Atlas.docx`](<docs/PRD - Atlas.docx>) |
| Functional requirements (FRD) | [`docs/Functional requirements - Atlas.docx`](<docs/Functional requirements - Atlas.docx>) |
| Design notes | [`design/README.md`](design/README.md) |
| Mock data | [`data/cadence-mock.json`](data/cadence-mock.json) |

The Figma file is built on J.P. Morgan's open-source [Salt Design System](https://www.saltdesignsystem.com/).
Figma access is by invite. If the link says you don't have access, ask Anisha to share the file or the
JPM Hackathon folder.

The PRD and FRD carry the Sep 27 revision as **tracked changes** (chat panel on every frame, document
upload and assignment, email/Teams notifications, home to-dos, Outlook-synced calendar, Word export).
Open them in Word and use Review → Accept/Reject.

## Screens

Four tabs in one persistent frame, with a chat panel docked on every tab:

| Tab | Purpose | Design |
|---|---|---|
| Home | Your to-dos: assigned sections, what's due this week, recent activity | Planned |
| Flow Explorer | See the end-to-end flow, its checkpoints and what is live per region | Next |
| Cadence | Update the sections you own and clear flags; admins upload, assign, schedule and export to Word | [My sections done](design/screens/cadence-my-sections.png) |
| Admin | Roles, node visibility, lens defaults (admin only) | Planned |
| Chat panel (every tab) | Ask about whatever is on screen; walk through a flow; draft a section | Planned |

![Cadence — My sections](design/screens/cadence-my-sections.png)

## Data

The PRD and FRD in `docs/` describe the real use case. Everything in `data/` and the Figma screens is
**fictional mock data**. No customer data, PII, transaction records or live rule content belongs here.
The real corpus stays in the approved environment.
