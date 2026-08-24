# Chicago-Simulation

A lightweight, pilot-scale reimplementation of **CitySim**'s methodology, adapted to Chicago.

CitySim is an LLM-driven urban agent simulation framework from Bougie & Watanabe (Woven by Toyota),
arXiv:[2506.21805](https://arxiv.org/abs/2506.21805). Agents plan realistic daily schedules through
recursive value-driven planning, form beliefs about places via a Kalman-filtered belief model, choose
destinations through a belief-weighted gravity model, and interact socially through an evolving
belief-weighted contact network — validated against real Japanese time-use, mobility, and POI data.

This project is an independent adaptation of that methodology to Chicago, using only public data
(ACS PUMS, LEHD LODES, OpenStreetMap, CTA GTFS, CMAP travel surveys, Divvy trip data, ATUS) in place
of the original paper's proprietary Japan-specific datasets, at a much smaller pilot scale
(~200–500 agents, 1–3 Chicago community areas, 1–2 simulated weeks).

It is not a fork of CitySim's code — CitySim itself is not open-sourced. This is a from-scratch
re-derivation of its documented methodology.

## Where to start

Everything — the methodology digest, architecture plan, per-module adaptation decisions, Chicago data
sourcing, parameters, evaluation plan, and open questions — lives in [`brain/`](brain/INDEX.md).
Start at [`brain/INDEX.md`](brain/INDEX.md).

The source paper (`CitySim.pdf`) is kept locally but is git-ignored — it's a copyrighted preprint, not
ours to redistribute. Read it at the arXiv link above.

## Status

Planning phase. No simulation code yet — see [`brain/roadmap.md`](brain/roadmap.md) for the phased
build plan and current status.
