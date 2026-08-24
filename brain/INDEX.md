# ChicagoSim Brain — Index

This directory is the persistent knowledge base ("brain") for **ChicagoSim**: a lightweight, pilot-scale
reimplementation of [CitySim](../CitySim.pdf) (Bougie & Watanabe, Woven by Toyota, arXiv:2506.21805, Jun 2025)
adapted to Chicago. Anyone (human or Claude) picking up this project should start here.

CitySim itself is not open-sourced — this project is an independent re-derivation from the paper's
methodology, not a fork of their code.

## How to use this brain

- Start with `01-vision-and-scope.md` to understand what "lightweight pilot" means here and what's explicitly cut.
- `02-citysim-methodology-digest.md` is the granular distillation of the source paper — every module,
  equation, and parameter, so nobody needs to re-read the PDF to remember how the original works.
- `03-architecture.md` translates that methodology into a system design for the pilot.
- `modules/` has one file per cognitive/behavioral module, each with: what the paper does, what we're
  building instead, and why.
- `data/` maps every data need to a real, freely-available Chicago/US source.
- `parameters.md` is the single source of truth for every tunable constant.
- `evaluation-plan.md` defines what "working" means and how we'll check alignment with reality.
- `decisions-log.md` is an append-only ADR-style log — record every non-obvious choice here the moment it's made.
- `open-questions.md` is the parking lot for unresolved decisions. Check it before assuming something is settled.
- `roadmap.md` is the phased build plan and current status.

## Update discipline

- When a decision changes, edit `decisions-log.md` (append, don't rewrite history) and update whichever
  spec file it affects. Don't let the digest/architecture drift from what's actually decided.
- When a question gets resolved, move it from `open-questions.md` into `decisions-log.md` and delete it
  from the open list.
- `roadmap.md`'s status section should reflect reality after every work session, not aspiration.

## Project identity

- **Working title:** ChicagoSim
- **Source method:** CitySim (arXiv:2506.21805)
- **Target city:** Chicago, IL — pilot scope TBD in scope doc (likely 1–3 community areas, not city-wide)
- **Status:** Planning / brain-building phase. No code written yet.
