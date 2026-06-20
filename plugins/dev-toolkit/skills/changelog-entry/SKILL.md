---
name: changelog-entry
description: Draft a changelog entry from recent changes. Use when the user asks to write a changelog entry, update CHANGELOG.md, or summarize what changed for release notes.
---

When asked to write a changelog entry:

1. Look at the recent commits or the staged diff to understand what actually changed.
2. Group the changes under **Added**, **Changed**, **Fixed**, and **Removed**.
3. Write concise, user-facing bullets — what changed and why it matters, not the implementation detail.
4. Follow the [Keep a Changelog](https://keepachangelog.com) format unless the project already uses another convention.

Keep entries short and skimmable. One line per change is ideal.
