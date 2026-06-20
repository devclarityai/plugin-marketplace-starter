# Plugin Marketplace Starter

A ready-to-fork starting point for distributing your team's **Claude Code skills** through a private **plugin marketplace**.

Fork it, rename it, confirm it works with the included demo plugin, then drop in your own skills. That's the whole idea.

---

## First, the three concepts (skills → plugins → marketplace)

These three words get used interchangeably, but they nest inside each other. Getting this straight makes everything below obvious.

```
Marketplace  (a git repo with a catalog file)
└── Plugin   (a folder with a manifest)
    └── Skill (a folder with a SKILL.md)
```

### Skill
A **skill** is a single capability you give Claude — one `SKILL.md` file with instructions, optionally alongside scripts or reference files. "Write a commit message," "summarize a PR," "run our release checklist" are each a skill. On its own, a skill lives in a project's `.claude/skills/` folder and only exists in the one repo where you created it.

### Plugin
A **plugin** is a shareable bundle of **one or more skills** (and optionally agents, hooks, or MCP servers) with a small manifest (`plugin.json`) that gives it a name. Packaging skills as a plugin is what makes them *shareable and versioned* instead of copy-pasted between people.

- A plugin can hold **one** skill (see `plugins/simple-demo`) or **many** (see `plugins/dev-toolkit`, which bundles three).
- A plugin's skills are **namespaced** by the plugin name, so they never collide: the `pr-summary` skill inside the `dev-toolkit` plugin is invoked as `/dev-toolkit:pr-summary`.
- A skill installed from a plugin behaves *identically* to the same skill sitting in a repo's `.claude/skills/` folder — the only difference is the plugin-name prefix.

### Marketplace
A **marketplace** is a **catalog of one or more plugins** — and it's nothing more than a git repo with a `.claude-plugin/marketplace.json` file listing them. **This repo is a marketplace.** Once it's on GitHub, your whole team adds it once and installs whichever plugins they want.

> **In one line:** a *skill* is a capability, a *plugin* packages one or more skills, and a *marketplace* is a repo that lists one or more plugins for your team to install.

### How that maps to the files in this repo

```
plugin-marketplace-starter/
├── .claude-plugin/
│   └── marketplace.json          ← the catalog (this file makes the repo a marketplace)
└── plugins/
    ├── simple-demo/              ← a plugin with ONE skill (the operational test)
    │   ├── .claude-plugin/
    │   │   └── plugin.json
    │   └── skills/
    │       └── address-me/
    │           └── SKILL.md
    └── dev-toolkit/              ← a plugin with THREE skills (a copy-me template)
        ├── .claude-plugin/
        │   └── plugin.json
        └── skills/
            ├── commit-message/
            │   └── SKILL.md
            ├── pr-summary/
            │   └── SKILL.md
            └── changelog-entry/
                └── SKILL.md
```

> **One rule worth memorizing:** only the `*.json` manifests live inside a `.claude-plugin/` folder. The `plugins/` and `skills/` directories always sit at the repo or plugin **root** — never inside `.claude-plugin/`.

---

## Step-by-step: get this running in your company

Do these in order. **Don't add your own skills until Step 5 confirms the marketplace is operational** — that way, if something breaks later, you know it's your skill and not the plumbing.

### Step 1 — Fork / copy this repo into your org

Use the green **"Use this template"** button on GitHub, or clone and push to a new repo under your organization:

```bash
git clone https://github.com/devclarityai/plugin-marketplace-starter.git
cd plugin-marketplace-starter
rm -rf .git && git init
# create an EMPTY repo in your org first (e.g. your-org/claude-plugins), then:
git remote add origin git@github.com:your-org/claude-plugins.git
git add -A && git commit -m "Initial marketplace from starter template"
git push -u origin main
```

A **private** repo is fine — Claude Code can install from private repos using your existing GitHub credentials (`gh auth login`). "Private" vs "public" isn't a setting on the marketplace itself; it's just whether people can reach the repo. Use private if your skills reference anything internal.

### Step 2 — Rename the marketplace

Open `.claude-plugin/marketplace.json` and set it to your company:

```json
{
  "name": "your-org-plugins",
  "description": "Internal Claude Code plugins for Your Org.",
  "owner": { "name": "Your Org", "email": "devtools@your-org.com" },
  "plugins": [ /* leave simple-demo + dev-toolkit here for now */ ]
}
```

- `name` is **public-facing** — it's what teammates type when installing (`/plugin install simple-demo@your-org-plugins`). Use kebab-case, no spaces.
- Keep the two example plugins in the list for now. You'll remove the demo once you've confirmed everything works.

Commit and push.

### Step 3 — Add the marketplace in Claude Code

From any project, point Claude Code at your new repo (use *your* `owner/repo`):

```
/plugin marketplace add your-org/claude-plugins
```

For a non-GitHub host, use the full git URL instead:

```
/plugin marketplace add https://gitlab.your-org.com/team/claude-plugins.git
```

Adding the marketplace just registers the catalog so you can browse it — **nothing is installed yet.** (It's like adding an app store, not downloading an app.)

### Step 4 — Install the demo plugin

```
/plugin install simple-demo@your-org-plugins
/reload-plugins
```

`/reload-plugins` activates it without restarting Claude Code.

### Step 5 — Confirm it's operational ✅

Run the demo skill:

```
/simple-demo:address-me
```

If Claude asks what to call you and confirms the marketplace is working, **your distribution pipeline is proven end to end** — your org can add the marketplace, install a plugin, and run its skills. This is the checkpoint to reach before investing in real skills.

> Not working? See [Troubleshooting](#troubleshooting) below.

### Step 6 — Roll it out to your team

Tell teammates to run the commands from Steps 3–4, **or** make it automatic for a project by adding this to that project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "your-org-plugins": {
      "source": { "source": "github", "repo": "your-org/claude-plugins" }
    }
  },
  "enabledPlugins": {
    "dev-toolkit@your-org-plugins": true
  }
}
```

When a teammate trusts that project folder, Claude Code prompts them to add the marketplace and auto-enables the plugins you listed. This is the cleanest way to standardize tools across a team.

### Step 7 — Now make it yours

Once Step 5 passed:

1. **Delete the demo plugin:** remove `plugins/simple-demo/` and drop its entry from `marketplace.json`.
2. **Copy `dev-toolkit` as your template** (or create a new plugin folder) and add your skills under its `skills/` directory — one folder per skill, each with a `SKILL.md`. See [Adding a skill](#adding-a-skill) below.
3. Commit and push. Teammates pick up changes with `/plugin marketplace update your-org-plugins` (or automatically, if auto-update is on).

---

## Adding a skill

A skill is a folder containing a `SKILL.md`. The folder name becomes the skill name.

To add a `release-checklist` skill to the `dev-toolkit` plugin:

```
plugins/dev-toolkit/skills/release-checklist/SKILL.md
```

```markdown
---
description: Run our release checklist. Use when the user is preparing a release or asks to cut a version.
---

Walk through our release steps:
1. ...
2. ...
```

The `description` is how Claude decides when to use the skill automatically, so make it specific. Invoke it explicitly as `/dev-toolkit:release-checklist`.

To add a whole **new plugin** instead, copy an existing plugin folder, edit its `.claude-plugin/plugin.json` (`name` + `description`), and add a matching entry to `marketplace.json`.

---

## Validate before you push

Catch JSON and frontmatter mistakes before your team sees them:

```bash
# validates marketplace.json (schema, duplicate names, bad sources)
claude plugin validate .

# validates a single plugin's manifest + its skill frontmatter
claude plugin validate ./plugins/dev-toolkit
```

---

## A note on versioning

This starter intentionally **omits** the `version` field from every `plugin.json`. When version is omitted and the marketplace is hosted in git, **every commit is treated as a new version** — so teammates always get your latest push on update. That's the simplest setup and the right default while your plugins are actively changing.

If you later want controlled releases (so updates only land when you decide), add a `"version": "1.0.0"` to a plugin's `plugin.json` and **bump it on every release**. A pinned version that you forget to bump means teammates never get your changes.

---

## Security & trust

Plugins run with the same privileges as the person who installs them — they can execute arbitrary code. The good news: plugins are easy to build, so the safest source is one **you and your org control**. Trust the plugins built into Claude Code, trust ones published by the vendor themselves (e.g. Notion's own plugin), and trust your own internal marketplace. Be cautious with random third-party marketplaces.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `Marketplace not found` / can't add | Confirm `.claude-plugin/marketplace.json` exists at the repo root and the `owner/repo` is correct. For private repos, check `gh auth status`. |
| Plugin installs but skill doesn't appear | Run `/reload-plugins`. If it persists, clear the cache: `rm -rf ~/.claude/plugins/cache`, restart Claude Code, reinstall. |
| `Path contains ".."` on validate | Plugin `source` paths must start with `./` and stay inside the repo. |
| Teammates don't get your updates | If you set a `version`, bump it. Otherwise have them run `/plugin marketplace update your-org-plugins`. |
| Skill names collide | They can't — skills are namespaced by plugin (`/plugin-name:skill-name`). |

---

## Reference

- [Discover & install plugins](https://code.claude.com/docs/en/discover-plugins)
- [Create plugins](https://code.claude.com/docs/en/plugins)
- [Create & distribute a marketplace](https://code.claude.com/docs/en/plugin-marketplaces)
