---
name: add-note
description: 'Add a knowledge entry (rule of thumb, best practice, lesson learned) to a topic-specific notes file. Use when: add note, keep this knowledge, note this in {topic}, add learning tip, add-note.'
argument-hint: 'Required: the topic (e.g. JS, Java, PostgreSQL) and the knowledge to note. Optional: notes directory path'
model: gemini-3.5-flash
---

# Add Note
Create notes entries that are approachable for someone encountering the concept for the first time. Prioritize understanding over compactness: explain what the concept is, why it matters, how it works, and when to use it.

## When to Use
- The user has just learned something from an AI conversation (best practice, difference between approaches, gotcha, or rule of thumb) and wants to persist it.
- The user explicitly says "keep this knowledge in {topic}", "add-note", or "note this in {topic}".

## Teaching Approach

- Treat the learner as a complete beginner. Do not assume prior knowledge of the language, framework, terminology, or underlying concepts.
- Introduce unfamiliar terms in plain language before using them repeatedly. When a term has a precise technical meaning, give both the simple definition and the technical name.
- Explain the reason behind a rule, not only the rule itself. Include the problem it prevents and the situation in which it is useful.
- Prefer short paragraphs and bullet points for lists of characteristics, steps, comparisons, trade-offs, or common mistakes. Use bullets when they make the concept easier to scan; do not turn every paragraph into a list.
- Use small, self-contained examples with neutral names. Explain the expected result of the example when that result is not obvious.

### Selective Line-by-Line Explanation

When a code example contains syntax or control flow that a beginner may not understand, add a focused line-by-line walkthrough after the example. Explain only the lines that teach the concept or prevent a likely misunderstanding; do not mechanically repeat every trivial line.

For each selected line or tightly related fragment:

1. Show the exact source in a fenced code block.
2. Explain what happens at that point in execution.
3. Define any new syntax, API, keyword, operator, or punctuation.
4. Use a tiny before-and-after example when it clarifies the result.
5. Add a horizontal rule (`---`) between explanations so the walkthrough is easy to follow.

### Diagrams and Visual Explanations

Add a diagram or other visual aid when the concept involves a sequence, lifecycle, branching decision, data transformation, ownership relationship, or several components interacting. Prefer a concise Mermaid diagram when it communicates the relationship more clearly than prose; use a simple Markdown table for comparisons and an ASCII diagram when Mermaid would be excessive.

Every visual must be accompanied by a short explanation of how to read it. Keep diagrams small, label arrows and states clearly, and do not add decorative visuals that do not teach anything.

## Learning Directory Resolution
The notes live in a directory chosen by the user (usually a dedicated repository), separate from this skill. Resolve it in this order:

1. **Explicit path in the request** (e.g. "note this in JS in ~/work/learning"): use it for this call only. Do not save it unless the user asks.
2. **Saved config**: read `$XDG_CONFIG_HOME/add-note/config`, falling back to `~/.config/add-note/config` when `XDG_CONFIG_HOME` is not set. The file contains a single line: `learning_dir=<absolute path>`.
3. **Nothing found**: ask the user for the path of their learning directory, then:
   - Expand `~` and convert the path to an absolute path.
   - If the directory does not exist, ask before creating it.
   - Save it to the config file (create the `add-note/` config folder if needed) and tell the user where it was saved.

Rules:
- If the saved directory no longer exists, do not recreate it silently. Tell the user and ask for a new path.
- If the user says "change my learning directory" or "set learning dir to X", update the config file.
- Never write notes inside this skill's own directory or repository, or inside agent configuration folders (`~/.copilot/`, `~/.claude/`, etc.).

## Topic Resolution
The user provides a topic name (e.g. "JS", "Java", "PostgreSQL", "Lit", "CSS", "Docker"). Map it to a file inside the learning directory:

- **Convention**: `{topic-lowercase}/{topic-lowercase}_notes.md`. The folder name is lowercase, and the file name keeps the topic's usual spelling.
  - "JS" or "JavaScript" → `javascript/js_notes.md`
  - "Java" → `java/java_notes.md`
  - "PostgreSQL" or "Postgres" → `postgresql/postgresql_notes.md`
  - "Lit" → `lit/lit_notes.md`
  - etc.

If the subfolder or file doesn't exist yet, create them with this skeleton:

```markdown
# {Topic} Notes

A running personal collection of {Topic} rules of thumb, good practices, and lessons learned, gathered day after day while working on real projects (but written generically so they're useful anywhere).

Entries are added chronologically (oldest first). These notes will be reorganized into logical chapters once they grow large enough to need them.

## Index

## Entries
```

The user **must** specify the topic explicitly — if they don't, ask which topic file to use.

## Procedure

### 1. Read the whole document first
Never append blindly — read the full file to know the current entry count, existing titles/topics, and whether it's still in flat/chronological mode or has already been split into chapters.

### 2. Check for duplicates or overlap
Compare the new tip against existing entry titles and content. If a very similar rule already exists:
- **Ask the user** whether to merge/expand the existing entry or add a distinct new one. Do not silently duplicate.

### 3. Generalize the content
The source may come from a specific project/codebase. Strip out project-specific names, business logic, and file paths — rewrite the explanation and code example so they read as generic guidance that would make sense in any project using that technology.

Code examples must be client- and organization-agnostic. Never copy proprietary identifiers, domain names, internal URLs, credentials, configuration values, or business-specific models; replace them with neutral names and self-contained placeholders that illustrate the same principle.

### 4. Compose the entry
Each entry follows this exact shape (no dates, per user preference):

```markdown
### N. <Short, descriptive title>

<Explanation written in a clear, teacher-like tone — as if you're explaining the concept to a colleague who is smart but unfamiliar with this specific topic. Use complete sentences and short paragraphs (2-4 sentences each). Explain *why* something works a certain way, not just *what* it does. When comparing options, explain each one in its own sentence so the reader can follow the reasoning.>

\`\`\`<language>
// Small, self-contained, generic example illustrating the rule (when relevant)
\`\`\`

**Further reading:** [Descriptive resource title](https://example.com)
```

- `N` is the next sequential number (continue numbering across the whole document, even across chapters).
- **Write like a teacher, not a telegram.** Use complete sentences — but mix formats freely to make the content scannable and memorable: short paragraphs for context, bullet points for listing options, tables for comparisons, "Do / Don't" pairs for common mistakes, selective line-by-line explanations for difficult code, and well-commented code examples to tie it all together.
- The **commented code example is often the most valuable part** — invest effort in making comments clear and illustrative. A reader should be able to understand the rule just by reading the code + comments.
- Keep it concise but readable — aim for clarity over brevity or verbosity. Use whatever structure makes the concept easiest to grasp at a glance.
- Include a code/config example **only when it adds clarity** — purely conceptual tips can omit it.
- Use the appropriate language tag for the fenced code block (`js`, `java`, `sql`, `html`, `css`, `yaml`, etc.).
- Add a Mermaid, table, or ASCII visual when it materially improves understanding, especially for flow, lifecycle, relationships, or comparisons. Explain the visual immediately after it.
- Every entry must include a **Further reading** link to a web page or document that explains the learning in greater depth. Prefer official documentation, specifications, or other authoritative sources, and use a descriptive link title.

### 5. Decide where it goes: flat list vs. chapters

- **Flat/chronological mode** (default until the notes file grows large): append the new entry at the bottom of `## Entries`, and add a matching link at the bottom of `## Index`.
- **Chapter mode**: if the document has already been split into chapters (`## <Chapter Name>` headings under `## Entries`), file the new entry under the chapter that matches its topic, creating a new chapter heading if none fits. Update the `## Index` accordingly (grouped by chapter).

### 6. When to propose reorganizing into chapters
Once the flat list reaches roughly **15 entries**, or once **3+ entries** clearly cluster around the same topic, **ask the user** before restructuring:

> "These notes have grown — want me to reorganize them into chapters? I'll group existing entries by topic and keep the numbering intact."

Only restructure after explicit confirmation.

### 7. Confirm
After editing, briefly tell the user which entry number/title was added, the full path of the file, and where in it (flat list or which chapter).
