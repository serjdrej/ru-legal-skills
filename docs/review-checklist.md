# Review checklist for agents leading work on this family

For any agent picking up `legal-ru`, `patent-ru`, `gost-ed-mashiny`, or a
future plugin added to this marketplace. Every item below is here because it
was a real gap found in this family, not a hypothetical — see the note after
each one.

## SKILL.md

- [ ] Every trigger phrase in `description` is backed by real content in
      the repo — grep for the term, confirm a file actually addresses it.
      *(Found clean on audit for gost-ed-mashiny — ЗИП/КДС/формуляр all
      real — but that was luck, not a given; check every time.)*
- [ ] `description` states any real scope limit a reader would otherwise
      assume away (e.g. "structural audit, not an engineering
      recalculation"). *(gost-ed-mashiny's audit step didn't say this out
      loud anywhere outside one reference file until fixed.)*
- [ ] Every path the routing table names actually exists.
- [ ] "Жёсткие правила" (or equivalent) ends with a standing disclaimer —
      draft for a named kind of professional's review, not advice/a
      ready-to-file document. *(gost-ed-mashiny had none until this was
      caught by auditing it against the other two.)*
- [ ] If the skill cites external norms/standards, the citation rule points
      at a `references/norms-registry*.md` and a lookup script/route — see
      `docs/norms-verification-convention.md` for the shape.
- [ ] Frontmatter under 1024 chars (hard limit); flag if pushing past ~700
      without a specific findability reason.

## README.md

- [ ] States the problem in plain language *before* any acronym/jargon —
      a reader with zero context should understand why this exists in the
      first paragraph. *(gost-ed-mashiny's original README opened straight
      into ГОСТ Р 2.610-2019 with no framing at all.)*
- [ ] An explicit "what it does NOT do" section, not just "what it does."
- [ ] Installation instructions, both paths: standalone
      (`git clone ... ~/.claude/skills/<name>`) and via this marketplace
      (`/plugin marketplace add serjdrej/ru-legal-skills` +
      `/plugin install <name>@ru-legal-skills`). *(Missing from all three
      plugins as of 2026-09-11 — fixed for gost-ed-mashiny, still open for
      legal-ru and patent-ru.)*
- [ ] Any file/module count or list in prose matches what's actually in the
      repo right now. *(gost-ed-mashiny's README said "5 модулей" for two
      commits after a 6th file was added.)*
- [ ] Any "status" line (imported / standalone / etc.) is still true —
      re-read it after every structural change, don't trust it because it
      was true when written.

## Repo hygiene

- [ ] `git status` clean before ending a session.
- [ ] `git log --all --oneline -- '*.log'` (and any other debug-artifact
      glob) is empty — dispatch run-logs must never enter history.
      *(legal-ru had four committed before this was caught; fixed via
      `git filter-repo --path-glob 'docs/*.log' --invert-paths --force`,
      which rewrites every commit sha — see the next section.)* `.gitignore`
      should cover `*.log` and `__pycache__/` going forward.
- [ ] No dot-prefixed path, no `hooks/`/`commands/`/`agents/` directory
      anywhere (breaks import into `lazy-skill-library` and similar
      importers).
- [ ] `git config user.name`/`user.email` set **locally in that repo**
      before the first commit — global config is not set on this machine.
      *(Hit twice this session: "Author identity unknown," both times on a
      repo's very first commit.)*

## After ANY commit to a plugin repo — sync the marketplace

`ru-legal-skills/.claude-plugin/marketplace.json` pins each plugin to an
exact commit sha. **A commit to `legal-ru`, `patent-ru`, or
`gost-ed-mashiny` immediately makes that plugin's marketplace entry stale**
— this happened four separate times in one session (three ordinary commits
plus one `git filter-repo` history rewrite, which changes every sha in the
rewritten repo, not just the tip).

- [ ] `git -C <plugin-repo> rev-parse HEAD` → update that plugin's
      `source.commit` **and** `source.sha` (both fields, kept identical —
      see below) in `marketplace.json`.
- [ ] If the plugin's `SKILL.md` `description` changed, copy the new one
      into the marketplace entry's `description` verbatim — don't leave the
      marketplace advertising stale wording (or a stale designation, as
      with the ГОСТ Р 2.601-2019 correction).
- [ ] `python -m json.tool .claude-plugin/marketplace.json` before
      committing — must parse clean.
- [ ] Commit and push `ru-legal-skills` itself after this sync — a stale
      pin left uncommitted is worse than an old commit, because it looks
      current.

*(Correction, 2026-09-11, found during an Opus 5 review of this family:
the earlier version of this note claimed `claude-plugins-official`'s own
`"source": "github"` entries use the same value for `commit` and `sha`.
**That is false — checked directly.** Its two real `"github"`-source
entries (`fullstory`, `jfrog`) each carry two genuinely different 40-char
hex values for `commit` and `sha`. What each field actually means is still
undocumented — `commit` is presumably a git commit sha; `sha` might be a
tree hash, a content digest, or something else entirely, and this has not
been confirmed. This family nonetheless sets both fields to the plugin's
same real commit sha, **as a deliberate simplification given the unknown
semantics, not because it mirrors the reference** — it previously did, and
does not. If this ever breaks an install, that field pair is the first
thing to question.)*

## Testing status — track it, don't assume it

- [ ] `legal-ru`: full RED→GREEN→REFACTOR pressure-tested (4 branches, one
      scenario per highest-risk item, Sonnet 5 subagents) — done
      2026-09-10.
- [ ] `patent-ru`: pre-dates this family's process; regression-tested
      against a live case per its own `references/audit-compliance.md` —
      not the same methodology, don't conflate the two when reporting
      status.
- [ ] `gost-ed-mashiny`: **live citation data verified** (rst.gov.ru open
      data, real status pulled for both ГОСТы) — this is NOT the same as
      pressure-testing. Behavioral discipline under pressure (does it
      resist fabricating a нормируемое значение when pushed, does it
      actually call `gost_lookup.py` instead of answering from memory) is
      still open as of 2026-09-11.
- [ ] A future plugin: run both — live-data verification is necessary but
      not sufficient; see `docs/norms-verification-convention.md` for the
      citation side and the parent `lazy-skill-library` project's
      `writing-skills` skill for the RED→GREEN→REFACTOR side.

## What this document is not

Not a gate that blocks work, and not exhaustive — it's the concrete list of
what this family's own history has actually gotten wrong, kept current.
When you find a new class of gap during a review, add it here with the same
"what happened, not just what to check" framing, so the next agent doesn't
re-discover it from scratch.
