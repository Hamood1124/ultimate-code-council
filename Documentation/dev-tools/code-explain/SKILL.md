---
name: code-explain
description: "Explains an unfamiliar codebase in plain English — what it does, how it's structured, what the key files are, and how data flows through it. Different from /zoom-out (architectural context) — this starts from zero for someone who has never seen the code. TRIGGER on: 'explain this code', 'what does this do', 'explain this codebase', 'I don't understand this code', 'walk me through this', '/code-explain'. Works on any language or stack."
---

# Code Explain

Explains unfamiliar code to someone starting from zero. Reads the codebase and produces a plain-English explanation — no assumed knowledge, no jargon without definition. Works on a single file, a folder, or an entire project.

---

## Phase 0 — Load context (silently)

```bash
# Get the full picture first
cat CONTEXT.md 2>/dev/null
cat README.md 2>/dev/null | head -50

# Understand the project structure
find . -not -path '*/node_modules/*' -not -path '*/.git/*' \
       -not -path '*/dist/*' -not -path '*/build/*' \
       -type f | sort

# Read entry points
cat package.json 2>/dev/null | grep -E '"main"|"scripts"'
find . -name "index.*" -not -path '*/node_modules/*' | head -5
```

---

## Phase 1 — Scope check

Ask one question only:

```
What do you want me to explain?

A) The whole project — give me the full picture
B) A specific file or folder — [tell me which one]
C) A specific concept — [e.g. "how does auth work here", "what is this service doing"]
D) Just the entry point — how does this thing start up

Also: what's your background? Are you a developer who's new to this codebase,
or a non-technical person who needs the general idea?
```

Adjust depth and vocabulary based on answer.

---

## Phase 2 — Explain

### For whole project:

```markdown
## What this project is

[One paragraph. What does this thing do? Who uses it? What problem does it solve?
Written like you're explaining to a smart person who's never seen it.]

## The big picture — how it's structured

[Folder structure with purpose of each key directory.
Not every folder — just the ones that matter.]

src/
├── components/    — React UI components (what the user sees)
├── services/      — Business logic and API calls (how it works)
├── hooks/         — Reusable React state logic
└── types/         — TypeScript type definitions

## How it starts

[Trace from the entry point. Step by step, plain English.
"When the app loads, it first does X, then Y, then Z."]

## The main things it does

[List the key features/functions. For each one:]

### [Feature name]
What it does: [one sentence]
Where it lives: `src/services/LeaveService.ts`
How it works: [2-3 sentences, plain English]

## How data flows

[Where does data come from → how is it transformed → where does it go]

## The parts that are confusing (and why)

[Flag anything non-obvious. "You might wonder why X — the reason is Y."]

## What to read first if you want to understand this codebase

1. Start with `[file]` — this is the entry point
2. Then read `[file]` — this is the core business logic
3. Then `[file]` — this is how it talks to external systems
```

### For a specific file:
Line-by-line walkthrough if short. Section-by-section if long. Plain English throughout.

### For a non-technical person:
Skip all code references. Analogies encouraged. Focus on what it does for the user, not how.

---

## Phase 3 — Offer follow-up

After explaining:
> *"Anything specific you want me to go deeper on? I can explain any specific file, function, or concept in more detail."*

Do not auto-generate more — wait for a specific request.
