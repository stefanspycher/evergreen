---
paths:
  - "incubator/**/*.md"
  - "notes/**/*.md"
---

# Note Writing Style

Rules for writing and editing `.md` note files in this project. Apply to all content in `incubator/` and `notes/`.

## Em-Dashes: Forbidden

Never use em-dashes ( — ) in any note. No exceptions.

Replace them as follows:
- **List item "**Term** — description"**: use a colon: "**Term**: description"
- **"X — because Y"**: remove the dash: "X because Y"
- **"not X — it is Y"**: split into two sentences: "not X. It is Y."
- **"X — A, B, C"**: use a colon: "X: A, B, C"
- **Parenthetical aside "X — extra thought — Y"**: use parentheses: "X (extra thought) Y"
- **Link annotation "[text](url) — description"**: use a colon: "[text](url): description"

The one permitted exception: verbatim quotation of another author's words inside a blockquote.

## AI Writing Patterns: Banned

These patterns signal AI-generated prose and must be rewritten:

### Section headers
Do not use these as section titles:
- "The Connection"
- "The Common Thread"
- "The Underlying Principle"
- "The Actual Problem"
- "The Shift"
- "Key Takeaways"
- "Practical Implications"
- "The Solution"

Replace with specific, descriptive titles that say what the section actually covers.

### Inline bold pseudo-headers
Do not use bold phrases as mini-headings mid-paragraph:
- "**The key insight:**"
- "**The shift:**"
- "**The value proposition:**"
- "**The result:**"
- "**The implication:**"

Dissolve these into normal prose or promote them to actual section headings.

### Filler phrases
Remove on sight:
- "In this context,"
- "It is worth noting that"
- "It is important to note that"
- "At its core,"
- "Moving forward,"
- "Delve into"
- "Navigate the [X] landscape"
- "The [X] landscape" (when used as scene-setting filler)
- "Robust" / "Seamless" / "Comprehensive" (as generic adjectives)

### Mechanical list structure
Avoid numbered or bulleted lists where every item uses the same bold label pattern ("**Term** — description"). This signals generated text. Either vary the structure or convert to prose.

## Prose Style

- Prefer short declarative sentences over long compound ones.
- Split "This is not X — it is Y" into two sentences: "This is not X. It is Y."
- Do not announce what a section will cover ("This section explores..."). Just write it.
- Do not close notes with a summary of what was just said.
- Translate all non-English content to English before saving.

## Sources / Further Reading

Notes that synthesize external sources must include a `## Further Reading` section at the bottom with the original URLs:

```markdown
## Further Reading

- [Article title](https://url): one-line description
- *Book Title* by Author
```

Do not use em-dashes in link annotations (use colons instead, as shown above).
