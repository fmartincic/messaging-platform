# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Status

Pre-implementation. No application code exists yet. This repo is scaffolded with BMad for AI-assisted product and engineering workflows.

## BMad Setup

BMad modules installed: `bmm` (planning), `tea` (test architecture), `bmb` (builder).

Key paths from `_bmad/config.toml`:
- Planning/implementation artifacts: `_bmad-output/planning-artifacts/`, `_bmad-output/implementation-artifacts/`
- Test artifacts: `_bmad-output/test-artifacts/`
- Project knowledge docs: `docs/`
- Skills output: `skills/`

Config override layers (last wins):
1. `_bmad/config.toml` — installer-managed, do not edit
2. `_bmad/custom/config.toml` — team overrides, committed
3. `_bmad/custom/config.user.toml` — personal overrides, gitignored
4. `_bmad/config.user.toml` — installer-managed personal answers, do not edit

## BMad Agents (Team)

| Skill | Name | Role |
|---|---|---|
| `bmad-agent-analyst` | Mary | Business Analyst |
| `bmad-agent-pm` | John | Product Manager |
| `bmad-agent-ux-designer` | Sally | UX Designer |
| `bmad-agent-architect` | Winston | System Architect |
| `bmad-agent-dev` | Amelia | Senior Software Engineer |
| `bmad-agent-tech-writer` | Paige | Technical Writer |
| `bmad-tea` | Murat | Test Architect |

Use `/bmad-help` to see available workflows. Use `/bmad-party-mode` for multi-agent discussions.

## Workflow

Typical order: PRD → Architecture → UX Design → Epics & Stories → Story implementation → Test design.

Use `/bmad-create-prd` to start, `/bmad-create-architecture` for tech design, `/bmad-create-story` for individual stories, `/bmad-dev-story` for implementation.
