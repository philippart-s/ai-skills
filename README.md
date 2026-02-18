# ✨ AI Skills for Claude code and compatible tools ✨

## 🧑‍💻 [methodical-dev](./methodical-dev/) 🧑‍💻

 - 👤: [@k33gorg](https://bsky.app/profile/k33gorg.bsky.social)
 - 🐙: https://codeberg.org/ai-skills/methodical-dev

The methodical-dev skill enforces a structured, step-by-step development workflow with five phases:

  1. Information Gathering — Asks about feature scope, technical constraints, documentation references, and coding conventions. Also corrects the user's English grammar.
  1. Git Verification — Checks repo status and recommends (but never auto-creates) a dedicated feature branch.
  1. Detailed Planning — Analyzes existing code, creates a 5–8 step plan via TodoWrite, and waits for user approval before proceeding.  
  1. Guided Implementation — Executes one step at a time with mandatory checkpoints after each. The user must validate before the next step begins. No shortcuts, no unauthorized deletions, no architecture changes without consent.
  1. Final Validation — Provides a full summary of changes, runs through a quality checklist, and proposes a conventional commit message (with emoji) — but never commits automatically.

**Core principle:** The user stays in control at all times. The AI must stop after every step, explain decisions transparently, and never skip validation or silently modify existing code.