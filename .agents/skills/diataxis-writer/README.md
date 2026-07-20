# Diataxis Writer Skill

**Language / Ngôn ngữ / 语言:** [English](./README.md) | [Tiếng Việt](./README.vi.md) | [中文](./README.zh.md)

This skill helps an agent write, restructure, and review documentation using the
Diataxis framework.

## What Is Diataxis?

Diataxis is a documentation framework organized around the reader's real need.
Instead of putting every related piece of information on one page, Diataxis
separates documentation into four types:

| Document type | When the reader wants to | Goal |
| --- | --- | --- |
| Tutorial | Learn through practice | Guide the reader along a safe path toward initial competence |
| How-to guide | Complete a specific task | Provide clear steps to reach an outcome |
| Reference | Look up accurate information | Provide complete, consistent, scannable facts |
| Explanation | Understand context or reasons | Explain concepts, decisions, tradeoffs, and mental models |

The core idea is that each document should make one clear promise to the reader.
A tutorial should not try to become an API reference. A how-to guide should not
be stretched by architecture explanations. A reference page should not force the
reader through a long story before they can find the field, option, or command
they need.

## Benefits

- **Reduces confusing, overgrown docs**: Diataxis separates learning, doing,
  lookup, and understanding into distinct reader goals.
- **Improves reader experience**: readers find the kind of document that matches
  what they are trying to do instead of filtering through unrelated material.
- **Makes review easier**: reviewers can ask "which reader job does this page
  serve?" before editing details.
- **Improves maintainability**: each page has a clearer scope and promise, so
  updates are less likely to sprawl.
- **Applies beyond technical docs**: it works well for onboarding, knowledge
  bases, process docs, manuals, runbooks, product docs, API docs, and operational
  guides.

## Why Use Diataxis?

Many docs are frustrating not because they lack information, but because they
mix several purposes in one place. A newcomer needs guided steps. Someone
handling a task needs concise instructions. Someone looking something up needs
tables, fields, defaults, and constraints. Someone trying to understand needs
context and reasons.

Diataxis forces the writer to choose the main reader goal before writing. That
makes documentation more natural to navigate, easier to read, and less likely to
contradict itself.

## When To Use This Skill

Use this skill when you need to:

- write new documentation with Diataxis;
- review a confusing or overgrown document;
- split a "getting started" page into tutorial, how-to, reference, and
  explanation pages;
- redesign a knowledge base or docs site;
- write onboarding docs, process docs, runbooks, manuals, product docs, or API
  docs.

Do not apply Diataxis mechanically to marketing copy, sales proposals, legal
contracts, press releases, fiction, or writing whose main job is emotional
persuasion. Those formats can borrow some Diataxis thinking, but they should not
be forced into the framework.

## Quick Usage

In an agent that supports skills, ask for tasks like:

```text
Review this document with Diataxis and suggest how to split it.
```

```text
Write a getting-started tutorial for this internal tool.
```

```text
Create reference docs for this CLI command and keep them easy to scan.
```

The skill also includes a heuristic script for classifying Diataxis signals in a
document:

Run it from the skill directory:

```bash
bash ./scripts/classify-doc.sh path/to/doc.md
```

The script is only a quick diagnostic aid. The final decision should still be
based on the reader job and documentation context.

## Installation

### 1. Using CLI (Recommended)

```bash
npx skills add tronghieu/agent-skills --skill diataxis-writer
```

### 2. Manual Installation (For Non-Technical Users)

1. **Download:** Go to the `skills/` folder in this repository and download `diataxis-writer.zip`.
2. **Extract & Copy:** Extract `diataxis-writer.zip` and copy the `diataxis-writer` folder into one of the following directories:

**For a Specific Project:**
Copy the `diataxis-writer` folder to `.agents/skills/` or `.claude/skills/` in your project's root directory.

**Globally (Available for all projects):**
* **Mac / Linux:** `~/.agents/skills/` or `~/.claude/skills/`
* **Windows:** `%USERPROFILE%\.agents\skills\` or `%USERPROFILE%\.claude\skills\` (usually `C:\Users\<YourUsername>`)
