# Personal AI Skills

In this repository you'll find the skills I have developed and I'm using in my personal projects

## Index

  1. [/add-note](#add-note)
  2. [/line-by-line](#line-by-line)

## /add-note

### Purpose

Turns something you just learned while chatting with an AI (a best practice, a gotcha, a rule of thumb, the difference between two approaches) into a lasting entry in a personal notes file, organized by topic (JS, Java, PostgreSQL, Docker, ...).

Entries are written for a beginner: they explain what the concept is, why it matters, how it works and when to use it, instead of just stating the rule. The result is a set of personal reference files that are easy to reread months later.

### Setup

Symlink or copy `add-note/` into your agent's skills folder. The notes themselves are kept in a separate directory of your choice (typically their own repository), not in this repo.

### How it works

Invoke it with a topic and the knowledge to note, e.g. `/add-note JS the difference between == and ===`. If no topic is given, the skill asks for one.

1. **Find the learning directory.** On first use the skill asks where your notes are kept and saves the answer to `~/.config/add-note/config`. Later calls reuse it silently. You can target another directory for a single call by including its path in the request, or ask the skill to change the saved one.
2. **Find the topic file.** The topic maps to `{topic-lowercase}/{topic-lowercase}_notes.md` inside the notes directory (e.g. `javascript/js_notes.md`). If the file doesn't exist, it is created from a standard skeleton with an Index and an Entries section.
3. **Read the whole file** to know the current entry count, existing titles, and whether the notes are flat or already split into chapters.
4. **Check for duplicates.** If a similar entry already exists, the skill asks whether to expand it or add a new one.
5. **Generalize the content.** Project-specific names, business logic, internal URLs and other proprietary details are stripped out so the entry reads as generic guidance.
6. **Write the entry**, numbered sequentially, with:
   - a teacher-like explanation (short paragraphs, bullets, tables or Do/Don't pairs where useful)
   - a small, well-commented code example when it adds clarity
   - a selective line-by-line walkthrough for code a beginner might not follow
   - a Mermaid, table or ASCII diagram for flows, lifecycles or comparisons
   - a **Further reading** link to an authoritative source
7. **File it** at the bottom of the flat list (and the Index), or under the matching chapter if the notes have been reorganized.
8. **Offer to reorganize.** Once the notes reach about 15 entries, or 3+ entries cluster around the same subject, the skill proposes grouping them into chapters, and only does it after you confirm.
9. **Confirm** which entry was added, the full path of the file, and where in it.


## /line-by-line

### Purpose
### How it works

