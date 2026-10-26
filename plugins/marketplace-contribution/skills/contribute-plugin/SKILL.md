---
name: contribute-plugin
description: Walks someone from an idea, or a skill, command, or hook that already works on their machine, to a reviewed plugin and an open pull request in a plugin marketplace. Checks whether something like it already exists, builds or ports the files, gets them reviewed, and ships them. Use when the user asks to "add a plugin to the marketplace", "turn my local skill into a plugin", "share this skill with other teams", "update a marketplace plugin", or "help me build a plugin for the marketplace".
---

# Contribute plugin

Takes a contribution from "I have something" to an open pull request. All work happens on a branch in a checkout of the marketplace repo.

## 1. Learn the marketplace's rules

Find the marketplace checkout (ask where it is, or clone it). Read its CLAUDE.md, README, and any CONTRIBUTING or best-practices doc. They set the naming, folder layout, categories, versioning, and validation for everything below. Where they differ from this skill, they win.

## 2. Intake

Ask what they want to share and who else would use it. If it already works somewhere, ask where it lives (a repo's `.claude/skills/`, `~/.claude/skills/`, a command file, a hook in `settings.json`). The answer tells you whether this is a new idea, something working locally to port, or a change to a plugin already in the marketplace.

## 3. Should this be shared?

A skill that only makes sense in one repo is better as a project skill in that repo's `.claude/skills/`, where everyone who clones the repo gets it. It belongs in the marketplace when people in more than one repo or team would use it and it works without one repo's files.

Example: a Terraform validator that runs `terraform validate` on any `.tf` files belongs in the marketplace. A release runbook for one service, built on that repo's version file and build scripts, stays in that repo.

If part of it is general and part is one repo's facts, propose splitting it. The workflow goes to the marketplace, and the facts stay in the repo as team context (step 4). If it belongs in one repo, say so kindly, explain why, and stop unless they want to go ahead.

## 4. Does it already exist?

Read the marketplace's plugin list and the SKILL.md of anything that does a similar job. Compare by what it does, not by name. If something overlaps, recommend the lightest path and let them choose:

| Situation | Path |
|---|---|
| The existing plugin does the job | Use it. If it falls short, propose the change to its owner |
| Same job, but their team needs different steps in the middle | Add a variant flow to the existing skill |
| Same steps, but their team's facts differ (repos, project keys, conventions) | Have the existing skill read a team context file |
| Nothing does this job | Build a new plugin |

Two near-copies of one plugin are worse than one. People install the wrong one, and fixes land in only one of them. A variant or team context keeps one plugin, so build it as an update in step 5.

**Variant flow.** SKILL.md keeps the shared start and end. The middle moves to one file per variant, and SKILL.md picks one.

```
skills/write-stories/
  SKILL.md            # read the epic, pick a flow, write stories in the shared format
  flows/default.md
  flows/payments.md   # the payments team splits stories by environment first
```

**Team context.** The skill tells Claude to read a file the team keeps in its own repo, for example "If `team-context.md` exists at the repo root, read it first. Where it conflicts with a default below, it wins." The team owns that file, so marketplace updates never overwrite it.

If the marketplace's docs define their own way to do variants or team context, follow those.

When there is an overlap, stop and let them choose. Show the existing plugin, the path you recommend, and why. If they use the existing plugin as it is, the contribution ends here, and that is a good outcome. If they extend it with a change, a variant, or team context, continue to step 5 as an update.

## 5. Build it

Create a branch, then:

- **New:** ask for one or two real examples of when they'd use it. These become the trigger phrases in the description and the task they use in step 6.
- **Port:** read the original fully. Fix hardcoded personal paths (`/Users/...`, `C:\...`), references to its old unprefixed name (`/deploy` becomes `/<plugin-name>:deploy`), and calls to local scripts or skills other people won't have. Ask before dropping any of those. Confirm they want every file to come along, and stop if anything looks like a secret.
- **Update:** change the existing plugin as little as gets them what they need, and bump its version so installed copies update.

Settle the shape in plain words. If Claude should use it on its own when relevant, it's a skill. If someone runs it by name, it's a command. If it runs automatically at a moment like after every edit, it's a hook. For file formats, follow https://code.claude.com/docs/en/plugins.

Stop before writing files. Recap the name, shape, location, and description (for a port, also what changes from the original) and get a yes.

## 6. Try it

Have them open a new terminal and load the plugin. In Claude Code, run `claude --plugin-dir <plugin-folder>`. In Codex, run `codex plugin marketplace add <marketplace-checkout>`, then `codex plugin add <plugin-name>@<marketplace-name>`. Then they use it on one real task, calling it by its plugin name at least once. For a port the original still loads too, so the explicit call is the only way to know the plugin version ran. Fix what they find.

## 7. Review and fix

Run the `/marketplace-contribution:review-plugin` skill in a subagent, so the review comes from fresh eyes rather than the context that wrote the files. Then go through its report with the contributor one finding at a time, most serious first. Explain each one, offer the fix, and make it if they agree. A "should fix" they decline needs a one-sentence reason, which goes in the pull request description. Re-run the review after fixing, and move on when no "must fix" is left.

## 8. Ship

Run any validation the marketplace docs name, and register the plugin wherever they say (the marketplace's plugin list, the README). `git status` should list only this plugin and those files.

Stop. Tell them which files, branch, and pull request title are about to become visible to other people, and get a yes.

Commit, push, and open a pull request (or merge request) with the CLI for the repo's host. If they lack access or the CLI isn't set up, stop and say what's missing; the branch keeps the work. The description holds their own words on what it does, the review summary, and any declined findings with reasons.

After it merges, have them install it from the marketplace. For a port, offer to remove their original so the two don't both trigger.
