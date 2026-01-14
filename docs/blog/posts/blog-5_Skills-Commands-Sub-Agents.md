---
date: 2026-01-12
authors:
  - miguelmirandadias
categories:
  - Tools
  - Development
  - Agents
description: "How Claude Code's recent updates collapsed Skills, Commands, and Sub-agents into one unified abstraction."
---

# Skills, Commands, Sub-agents... It's All the Same Thing Now!

Claude Code ***used to*** give us a neat little taxonomy:

- **Commands**: quick, explicit “saved prompt” shortcuts (`/test` → runs a prompt).
- **Skills**: richer **procedural** knowledge packs that the model could ***auto-discover*** based on intent.
- **Sub-agents**: “personas” that worked in a **forked context** so your main conversation stayed clean.

It was tidy. It was elegant. It was… doomed.

<!-- more -->

---
> *Condensed mini-blog from my piece on how Claude Code’s recent updates collapsed 3 abstractions into 1 (and why that’s fine)*

> [Read the full article here](https://miguelmirandadias.substack.com/p/skills-commands-sub-agents-its-all)


## The old world: three lanes, no overlap

Back then, the decision was pretty straightforward:

- Want a shortcut you *manually* trigger? **Command**.
- Want a reusable procedure the model can *find* and apply? **Skill**.
- Want an “expert worker” persona that runs separately? **Sub-agent**.

You could actually draw a decision tree and not hate yourself.

## The update that broke everyone’s mental model

Then the recent Claude Code updates arrived and casually knocked down the fences:

### 1) Skills can be invoked like Commands
Skills aren’t only “auto-discovered” anymore. You can now **explicitly run a Skill with a slash**, just like a Command.

That erases one of the main functional differences.

### 2) Skills can fork context like Sub-agents
Skills also gained a YAML front matter option called:

```yaml
context_fork: true
```

Meaning a Skill can run in its own independent context window — the signature move of sub-agents.

So now:

- A **Skill** can act like a **Command** (explicit invocation)
- A **Skill** can act like a **Sub-agent** (forked context)
- A **Sub-agent** is basically a prompt that can load Skills

At some point you stop asking “which abstraction is correct?” and start asking “are we just naming the same thing three times?”

## A mental model that still works

When implementation details blur, the only way to stay sane is to categorize by **intent**, not mechanics.

### Skills = Knowledge Packs (SOPs)
Treat Skills like **Standard Operating Procedures** — reusable *capabilities* and *step libraries*.

Use a Skill when you have something like:

- a specific script/workflow (e.g., “parse a PDF and extract tables”),
- a checklist (e.g., “code review steps”),
- a repeatable procedure that should exist independently of any persona.

**Practical tip:** model your Skill as a folder:
- `SKILL.md`
- any resources it needs (snippets, templates, helper files)

### Sub-agents = Personas (Workers)
Sub-agents are the **identity layer** — conceptual knowledge, role, worldview.

Use a Sub-agent when you want:

- “Security Engineer” energy for a review,
- “QA Tester” behavior to drive a test run,
- an agent that should work in a contained context (especially useful when it goes off exploring).

### The composition: tools + workers
The punchline is simple:

> **Skills are the tools. Sub-agents are the workers who wield them.**

Build a “Code Reviewer” sub-agent that loads:
- a “Security Review” Skill
- a “Style Guide” Skill

Then let that persona execute in a forked context while your main workspace stays uncluttered.

## Wrap-up: the vocabulary is messy, your architecture doesn’t have to be

Claude Code’s feature set is in a transition period: powerful, flexible, and a little linguistically chaotic. The old “choose Command vs Skill vs Sub-agent” decision trees don’t apply anymore because **Skills absorbed everyone’s superpowers**.

So instead:

- **Organize by purpose**: *procedures* go in Skills, *identities* go in Sub-agents.
- **Compose**: load Skills into Sub-agents like tools into a toolkit.
- **Use context forking intentionally**: keep exploratory or noisy work in a separate window.

Try it: take one workflow you already use, refactor it into **a Skill (procedure)** plus **a Sub-agent (persona)** that loads it. Your future self will thank you — quietly, from a cleaner context window.

---

## 📖 Read the Full Article

<div class="medium-card" style="border: 1px solid var(--md-default-fg-color--lightest, #e1e5e9); border-radius: 8px; padding: 16px; margin: 20px 0; background: var(--md-code-bg-color, #f8f9fa); transition: all 0.2s ease;">
  <div style="display: flex; align-items: center; gap: 12px;">
    <div style="flex-shrink: 0;">
      <!-- Replace with your article image -->
      <img src="https://substackcdn.com/image/fetch/$s_!LKa2!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fb012e3ff-3d17-4218-a4a5-17ae776fbd6b_1200x896.jpeg" alt="Article Preview" style="width: 80px; height: 80px; border-radius: 8px; object-fit: cover; border: 1px solid var(--md-default-fg-color--lightest, #e1e5e9);">
    </div>
    <div style="flex: 1;">
      <h4 style="margin: 0 0 8px 0; font-size: 16px; color: var(--md-default-fg-color, #1a1a1a);">Skills, Commands, Sub-Agents... it's all the same thing now!</h4>
      <!-- Replace with your custom description -->
      <p style="margin: 0 0 12px 0; font-size: 14px; color: var(--md-default-fg-color--light, #6b7280); line-height: 1.4;">How Claude Code's recent updates collapsed three abstractions into one (and why that's fine).</p>
      <a href="https://miguelmirandadias.substack.com/p/skills-commands-sub-agents-its-all" target="_blank" style="display: inline-flex; align-items: center; gap: 4px; color: var(--md-accent-fg-color, #059669); text-decoration: none; font-weight: 500; font-size: 14px; transition: color 0.2s ease;">
        📖 Full article available on Substack
        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M7 17L17 7M17 7H7M17 7V17"/>
        </svg>
      </a>
    </div>
  </div>
</div>
