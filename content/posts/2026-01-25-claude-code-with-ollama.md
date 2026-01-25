---
title:  "Claude Code with Ollama"
date:   2026-01-25 15:17:00 +0100
---

During 2025 the AI race has shifted from creating best models to creating best agents. By end of year, most developers argue that Claud Code has taken lead.

Standard Claude Code usage typically requires an expensive Anthropic subscription.

But beyond the cost savings, using Ollama with Claude Code offers significant advantages regarding data privacy, confidentiality, and service autonomy. By redirecting the Claude Code interface to a local instance, you effectively bypass several of Anthropic's cloud-based data collection and usage policies. These include model training opt-out, technical information collection, usage tracking, regional constraints, intellectual property ownership and more.

Using Ollama with Claude Code allows you to run a powerful coding agent locally on your own hardware using open-source models instead of relying on Claude's cloud-based models. This setup provides a way to use the Claude Code interface while leveraging local computation.

## Setup

- Download and install Ollama.
- Verify it's running `curl http://localhost:11434`.
- Pick a model. It should be a thinking model that supports tools and can run on local hardware.
- Download chosen model `ollama pull <model_name>`.
- Download and install Claude Code.
- Launch Claude Code `ollama launch claude --config`
