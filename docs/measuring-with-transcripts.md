# Measuring a proposed mechanism against real transcripts

A method for answering "would this mechanism have caught anything?" *before*
building it, by counting what actually happened in past sessions rather than
by reasoning about what could happen.

Written down because it has already decided one question in this family:
a proposed verification pass over every citation-shaped claim was **measured
and rejected** — 331 transcripts, 31 sessions of real use, 488 claims of
citation shape, and nothing on that layer for the pass to catch. That is a
stronger result than a RED→GREEN run: RED→GREEN would have shown the
mechanism works; the measurement showed it has nothing to work on.

Method and all three traps below are `arbitrazh-ru`'s, handed over 2026-09-19.
The traps are not hypothetical — each one produced a wrong conclusion that was
published in a journal and corrected afterwards.

## Where the data is

Session transcripts: `~/.claude/projects/**/*.jsonl` (on Windows,
`%USERPROFILE%\.claude\projects\**\*.jsonl`). Token accounting lives in
`message.usage`, with `cache_creation_input_tokens` and
`cache_read_input_tokens`.

## Trap 1 — count invocations, not files

One `outline + full` pair from 14.09 appeared in **five** transcript files at
once. Counting files gave "six of nine" where the truth was two.

Deduplicate by `tool_use.id`:

```python
for b in message.content:
    if b.type == "tool_use":
        calls[b["id"]] = ...
```

## Trap 2 — a section read has no `mode`

`inp.get("mode") or "full"` silently counts section reads as full reads. The
one case where the mechanism worked as designed was invisible, and the
published conclusion said it had never worked at all.

```python
"outline" if mode == "outline" else ("section" if inp.get("section") else "FULL")
```

## Trap 3 — line order in the file lies

Resume appends older records after newer ones. Sort by `timestamp`, never by
position in the file.

## What to count

For a proposed verification pass: claims of citation shape (article number,
date, rate, deadline) in assistant text after the skill was loaded — and,
separately and decisively, **how many of them a check would have overturned**.
The first number tells you the mechanism has work; only the second tells you
the work is worth doing. Here the second was zero out of 488.

## Two traps when the thing being counted is your own compliance

Both found on 2026-09-19, while counting how many shipped fields already
violated a rule written the same hour.

**A rough pattern over-counts, and it does so every time.** A scan for
constants named `*NOTE|CAVEAT|WARNING` containing a digit returned 16 fields
"carrying a measurement"; reading them cut it to 11. The five removals were a
statute number read as a figure, two notes describing tool behaviour, and one
that substitutes its date at run time. Third instance in two days of a
non-zero count collapsing on inspection, **and all three collapsed downward** —
a rough pattern inflates, it does not deflate, so a count nobody has read is
an upper bound and should be reported as one.

**And the two directions together.** A truncated output errs the other way:
`head -5` over nineteen matches produced a false *zero*, not an inflated
count. So an unread rough count is an **upper** bound, a truncated one is a
**lower** bound, and **neither is a number.** Report whichever bound you
have, or read the output.

**A date inside a field is not necessarily the date of the measurement.**
`HEAD_NOTE` in the same repository reads «(замер: ИНН 9103086750, дата
прекращения 13.01.2022)» — a date, but the *measured content*, not when the
measuring happened. An automatic "does this field carry a date?" check passes
it and reports false compliance. Counting compliance mechanically fails in the
opposite direction from counting violations mechanically: the first
under-reports, the second over-reports.

## Why this belongs in the marketplace docs

The measurement is cheaper than the build in every case where it applies, and
its result is transferable in one direction only: a mechanism measured useless
on one plugin's traffic may still pay off on another's, because the traffic
differs. Measure per plugin; do not generalise a null result across the family.
