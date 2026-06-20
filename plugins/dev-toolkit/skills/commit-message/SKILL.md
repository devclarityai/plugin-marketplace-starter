---
name: commit-message
description: Write a clear git commit message from the staged changes. Use when the user asks for a commit message or to commit their work.
---

When asked for a commit message:

1. Look at the staged diff (`git diff --cached`) to see what actually changed.
2. Write a concise summary line in the imperative mood, under ~60 characters (e.g. "Add retry logic to upload client").
3. If the change needs context, add a short body explaining the *why*, not the *what*.
4. Follow the project's existing commit style if there is one (e.g. Conventional Commits).

Keep it tight. One good summary line beats a paragraph.
