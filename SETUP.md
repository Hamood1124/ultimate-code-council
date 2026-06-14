# Setup Guide — Ultimate Code Council

Step-by-step: publish to GitHub, install in Claude Code, use it.

---

## Part 1 — Publish to GitHub

### Step 1 — Create the GitHub repo

1. Go to https://github.com/new
2. Repository name: `ultimate-code-council`
3. Description: `Self-driving code quality pipeline for Claude Code`
4. Set to **Public** (required for `npx skills` to install it)
5. Do NOT check "Add a README" — we have our own
6. Click **Create repository**

### Step 2 — Push the files

Open your terminal, navigate to where you downloaded the skill files, then run:

```bash
cd ultimate-code-council
git init
git add .
git commit -m "Initial commit — Ultimate Code Council"
git branch -M main
git remote add origin https://github.com/Hamood1124/ultimate-code-council.git
git push -u origin main
```


### Step 3 — Verify on GitHub

Go to `https://github.com/Hamood1124/ultimate-code-council` and confirm you see:
```
ultimate-code-council/
├── README.md
├── council-start/
│   └── SKILL.md
└── code-council/
    ├── SKILL.md
    └── references/
        ├── reviewers.md
        ├── depth-modes.md
        ├── file-classification.md
        ├── report-format.md
        └── delivery.md
```

---

## Part 2 — Install in Claude Code

### Step 1 — Install Matt Pocock's skills first (dependency)

```bash
npx skills@latest add mattpocock/skills -g
```

The `-g` flag installs globally — available in every project.

### Step 2 — Install your skills

```bash
npx skills@latest add Hamood1124/ultimate-code-council -g
```

### Step 3 — Verify installation

```bash
npx skills@latest list
```

You should see both `code-council` and `council-start` in the list alongside Matt's skills.

### Where skills are stored

Global skills live at:
- **Mac/Linux:** `~/.claude/skills/`
- **Windows:** `C:\Users\YourName\.claude\skills\`

You can also open that folder and drag skills in manually if you prefer.

---

## Part 3 — Per-repo setup (run once per project)

When you open a new project in Claude Code for the first time:

```
/setup-matt-pocock-skills
```

This creates:
- `CONTEXT.md` — your project's domain glossary (all skills read this)
- `docs/adr/` — architectural decision records
- `docs/agents/` — issue tracker config, triage labels

The orchestrator (`/council-start`) will also run this automatically if it detects it hasn't been done yet.

---

## Part 4 — Use it

### Start a new feature from scratch
```
/council-start let's build X
```
The pipeline takes over from there.

### Review code you've already written
```
/code-council
```
or just say:
```
ship check
```
```
review this
```
```
is this safe to ship
```

### Quick scan before a push
```
quick scan
```

### Full deep audit
```
deep review
```

### Debug a specific bug
```
/diagnose
```

### Weekly architecture review
```
/improve-codebase-architecture
```

---

## Part 5 — Update your skills

When you want to update the skills after making changes:

```bash
# Edit your files locally
# Then push to GitHub
git add .
git commit -m "Update: [what you changed]"
git push

# Reinstall in Claude Code to get the latest
npx skills@latest update YOUR_USERNAME/ultimate-code-council
```

---

## Part 6 — Install on a new machine

```bash
npx skills@latest add mattpocock/skills -g
npx skills@latest add YOUR_USERNAME/ultimate-code-council -g
```

Two commands and you have the full pipeline on any machine.

---

## Troubleshooting

**Skills not triggering?**
- Make sure you're in Claude Code (not claude.ai chat)
- Run `npx skills@latest list` to confirm they're installed
- Try calling the skill explicitly: `/council-start`

**CONTEXT.md missing errors?**
- Run `/setup-matt-pocock-skills` in your project root first

**Matt Pocock's skills not found?**
- Run `npx skills@latest add mattpocock/skills -g` again
