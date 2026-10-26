---
name: review-plugin
description: Reviews a plugin for a plugin marketplace and reports each finding as must fix, should fix, or nice to have. Holds it to the marketplace's own rules, then checks that it loads, works on other machines, is safe to share, isn't a duplicate, and has a clear description. Reports only and never edits files. Use when the user asks "review my plugin", "will this pass review", "is this plugin ready", or "check this plugin", or when contribute-plugin or a CI pipeline needs a plugin reviewed.
---

# Review plugin

Reviews one plugin and reports what to fix. It never changes files. Fixing is up to whoever asked for the review (`/marketplace-contribution:contribute-plugin` offers the fixes after reading the report).

## 1. Find the plugin and the rules

Find the plugin folder (the one with `.claude-plugin/plugin.json`). If several are in view, ask which. Then read the marketplace's CLAUDE.md, README, and any CONTRIBUTING or best-practices doc. Hold the plugin to them. Run any validation command they name.

## 2. Check it

Check every rule in the marketplace's docs, then these, which hold for any marketplace:

- **Loads.** Every skill is a folder with a SKILL.md under `skills/`, since loose `.md` files there never load. Every file and skill it references exists, and it refers to its own skills and commands by their installed name (`/<plugin-name>:<skill>`). Bad: `skills/doc-framework/` holds six loose files and no SKILL.md, so none of them load.
- **Works on another machine.** No hardcoded user paths (`/Users/...`, `/home/...`, `C:\...`). It points to its own files with `${CLAUDE_PLUGIN_ROOT}`, and its SKILL.md names any tools it needs. Bad: `sed -i 's/a/b/' file` works on Linux and fails on every Mac.
- **Safe to share.** No secrets, tokens, private keys, `.env` files, internal hostnames, or customer data. Its description states anything powerful it does (hooks that run automatically, editing settings, installing software). Bad: an API key "that's expired anyway" still has to be removed and rotated.
- **Not a duplicate.** Compare it with the marketplace's other plugins by what they do, reading the SKILL.md of anything close. If one overlaps, name it and recommend extending it with a variant flow or team context over shipping a second copy.
- **Simple.** One job per skill. SKILL.md holds what every run needs, and the rest goes in references. It ships only files the plugin uses, with no plans or design notes. Bad: 3,400 lines of planning docs shipped to every installer.
- **Clear description.** The SKILL.md description says what it does and when to use it, in phrases people would type. Bad: `description: House style rules.` Claude can't tell when to use it.

## 3. Write the report

Rate each finding:

- **Must fix:** it won't load, won't work for someone else, or isn't safe to share.
- **Should fix:** it works, but it duplicates something, breaks a marketplace rule, or will be hard to find or maintain. It can merge with a reason. If the marketplace's docs rate a rule higher, use their rating.
- **Nice to have:** a suggestion.

When unsure between two ratings, pick the lower one. A review that blocks on wording keeps people from contributing.

Start with what's good. Then list findings grouped by rating, each with what's wrong, why it matters to the next person who installs it, the fix, and the file and line. If the caller names a path (a CI pipeline names a comment file), write the report there. Otherwise show it in the conversation.
