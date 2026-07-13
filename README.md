# Agent Skills 🧠

This repository contains portable AI Agent workflows and skills designed to be consumed by agentic frameworks (like Antigravity, Claude Code, Cursor, and Windsurf).

## Available Skills

- **[telegram-agent-builder](./telegram-agent-builder/SKILL.md)**: Automates the setup of a native, zero-prompt Telegram AI Agent that runs entirely in the background of your IDE using your local LLM session without requiring paid APIs.
- **[instagram-analyzer](./instagram-analyzer/SKILL.md)**: Performs deep data analysis and demographics reporting on Instagram metadata JSON exports.

## How to Install

These skills are natively compatible with the Vercel Labs `skills` CLI. To install a skill directly into your AI agent's harness, run:

```bash
npx skills add BaishyaDh/skills --skill <skill-name>
```

For example:
```bash
npx skills add BaishyaDh/skills --skill telegram-agent-builder
```
