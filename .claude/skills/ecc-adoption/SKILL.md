---
name: ecc-adoption
description: Decide which parts of the affaan-m/ECC agent toolkit to borrow for this repo (system prompt archive), and how to adopt them safely. Use when the user asks about ECC, wants to pick skills from it, asks whether to install the ECC plugin, or wants to add docs-governance, prompt-optimizer, or AgentShield checks to this repo.
metadata:
  origin: local
  reviewed-commit: 4eb71d92a39cab44ad40ac9d8a6a5ccb4029d6c2
---

# ECC adoption for system_prompts_leaks

This skill records a review of `affaan-m/ECC` (v2.2.3) and the decisions taken from it. It is advisory. Re-check the upstream source before acting on it, because ECC changes weekly.

## Context

This repo is a verbatim archive of AI system prompts, organized by vendor (`Anthropic/`, `OpenAI/`, `Google/`, and others) with a hand-maintained "Most recent additions" table in `README.md`. It is a document corpus, not an application. ECC is built for coding-agent harnesses, so most of it does not apply.

## What to adopt

Rank these before adding anything:

1. **`docs-governance`** (ECC `skills/docs-governance/`). Read-only audit of documentation links and index consistency. Best fit for keeping the README table and vendor folders in sync. Run it as an audit, not as a rewriter.
2. **`prompt-optimizer`** (ECC `skills/prompt-optimizer/`). Advisory rewrite of draft prompts. Use only for prompts the user is drafting, never for archived content. It is community-origin, so read it before use.
3. **`regex-vs-llm-structured-text`** (ECC `skills/regex-vs-llm-structured-text/`). Decision framework for extracting structured sections from prompt text. Regex first, LLM only for edge cases.
4. **`security-scan`** (AgentShield, ECC `skills/security-scan/`). Only if a project in this workspace contains a `.claude/` directory, CLAUDE.md, MCP config, or hooks.
5. **`deep-research`, `market-research`, `data-scraper-agent`**. Optional, for finding new prompt releases. Check each source's terms before scraping, and do not republish content without a check.

## What not to do

- Do not install the full `ecc@ecc` plugin into this workspace. It adds about 300 skills and 71 agents, most of them for unrelated stacks.
- Do not stack install methods. Choose one: guided `npx ecc-universal@2.2.3 setup`, or native `/plugin install ecc@ecc`.
- Do not enable ECC hooks without reading them first. Hooks execute Node commands on every Bash and PowerShell tool call.
- A version pin is not an integrity check. Review the release source before running package code.
- Do not treat non-Claude harness adapters as feature-complete. Their support is capability-limited.

## How to adopt a skill

1. Read the upstream `SKILL.md` and any scripts it references, at the reviewed commit.
2. Copy only that skill folder into `.claude/skills/<name>/` in this repo.
3. Remove any hook, MCP, or network-touching parts you did not review.
4. Run it once on a small input and confirm it changes nothing outside the intended files.
5. Record the upstream commit in the copied skill's `metadata` block.

## Answering questions

- "Should we install ECC?" Answer no for the full plugin. Offer the individual skills above.
- "Which ECC skill fits X?" Match X against the list above. If nothing fits, say so rather than stretching one.
- "Is ECC safe?" Say that the hooks run arbitrary code on every tool call, that review is needed before enabling them, and that the install path is not an audit.
