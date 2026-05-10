---
type: concept
tags:
  - context-engineering
  - ai-tools
  - mental-models
  - session-management
  - evergreen
---

# Context Engineering

Context engineering is the discipline of managing what an AI assistant has access to during a session: treating the context window not as a technical constraint to work around, but as a workspace to be actively organized.

## The Workbench Analogy

The context window behaves like a physical workbench, a finite shared surface where all active work must fit.

| Workbench | Context window |
|---|---|
| Desk surface | Context window (finite, shared space) |
| Power tools mounted on the wall | Connected capabilities: available but not consuming space until used |
| Post-it reminders | Skills and instructions: lightweight, persistent, behavioral |
| Papers spread across the desk | Loaded documents: each takes up surface area |
| Piling unrelated projects | Context pollution: degrades quality, not just efficiency |
| Clearing the desk between jobs | Session management: starting fresh for a new task |
| Professional organizer | Context engineer |

The analogy makes an invisible technical constraint tangible: you would not put materials for three different projects on the same desk and expect to work cleanly on any of them.

## Context Pollution

A professional organizer does not organize because they are tidy. They organize because clutter increases error rate.

Context pollution is not merely wasteful. It degrades output quality in ways that are hard to attribute. When an AI assistant has irrelevant information competing with relevant information, responses become less precise, earlier context can contaminate later reasoning, and the correct answer becomes harder to reach, not just slower.

This is the case for context engineering as a **discipline**, not a preference.

## Practical Implications

What takes up context space:
- Loaded documents and file contents
- Long conversation histories
- Verbose tool outputs
- Redundant instructions

What does not (when managed correctly):
- External capabilities, which only consume space when invoked
- Stored skills and preferences, which are referenced rather than embedded

Session management principles:
- Start new sessions for genuinely new tasks; do not carry over irrelevant context
- Load documents on demand, not preemptively
- Summarize rather than include raw outputs when possible
- Treat the end of a session as intentional closure, not abandonment

## Context Engineering as a Skill

Most users treat the context window as something that happens to them. Context engineers treat it as something they manage.

The skill involves knowing what to include (what the task actually requires), what to exclude (what will create noise), when to clear (when accumulated context is becoming a liability), and how to structure (so the most relevant information appears where it will be most useful).

See also: [[spec-driven-development]] for how specifications reduce context overhead in AI-assisted work, and [[ai-human-value-and-opportunity]] for the broader shift in human roles as AI capabilities expand.
