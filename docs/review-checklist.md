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
      (`git clone ... ~/.claude/skills/<name>`) and via this marketplace,
      **for both Claude Code and Codex CLI** (`/plugin marketplace add` +
      `/plugin install <name>@ru-legal-skills`, and
      `codex plugin marketplace add` + `codex plugin add <name>@ru-legal-skills`).
      *(All three plugins were missing install instructions entirely on
      2026-09-10; fixed the same day. Then, separately, all three needed a
      second pass to add the Codex half once the marketplace itself was made
      Codex-compatible — re-check this box after any change to
      `marketplace.json`'s `source` shape, not just once.)*
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
      anywhere (breaks import into skill-library tooling that scans a
      plugin's tree, such as the one this family's own skills have been
      imported into).
- [ ] `git config user.name`/`user.email` set **locally in that repo**
      before the first commit — global config is not set on this machine.
      *(Hit twice this session: "Author identity unknown," both times on a
      repo's very first commit.)*

## After ANY commit to a plugin repo — sync the marketplace

`ru-legal-skills/.claude-plugin/marketplace.json` pins each plugin to an
exact commit sha via `source.ref`. **A commit to `legal-ru`, `patent-ru`, or
`gost-ed-mashiny` immediately makes that plugin's marketplace entry stale**
— this has happened many times across this family's history (ordinary
commits, plus at least one `git filter-repo` history rewrite, which changes
every sha in the rewritten repo, not just the tip).

**Schema note:** entries use `"source": {"source": "url", "url": "https://github.com/<owner>/<repo>.git", "ref": "<sha>"}`
— not `"source": "github"` with `repo`/`commit`/`sha` fields, which is a
Claude Code-only shorthand that Codex's own plugin schema does not
recognize (confirmed live: Codex silently listed zero plugins from a
`github`-sourced marketplace, `url`-sourced entries installed correctly on
both runtimes). If you find a `github`-shaped entry anywhere in this file,
that's a regression — convert it to `url`.

- [ ] `git -C <plugin-repo> rev-parse HEAD` → update that plugin's
      `source.ref` in `marketplace.json`.
- [ ] If the plugin's `SKILL.md` `description` changed, copy the new one
      into the marketplace entry's `description` verbatim — don't leave the
      marketplace advertising stale wording (or a stale designation, as
      with the ГОСТ Р 2.601-2019 correction).
- [ ] `python -m json.tool .claude-plugin/marketplace.json` before
      committing — must parse clean.
- [ ] Commit and push `ru-legal-skills` itself after this sync — a stale
      pin left uncommitted is worse than an old commit, because it looks
      current.

*(History: this family briefly used the `"github"` source shorthand with
duplicated `commit`/`sha` fields, on a mistaken belief that
`claude-plugins-official`'s own `github`-source entries do the same — they
don't, checked directly, its two real examples carry two different values
for those fields with undocumented distinct meaning. Moot now: switched to
`url`/`ref` the same day, for the unrelated and more important reason that
Codex doesn't parse `github`-sourced entries at all. See the schema note
above.)*

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
      still open as of 2026-09-10.
- [ ] A future plugin: run both — live-data verification is necessary but
      not sufficient. See `docs/norms-verification-convention.md` for the
      citation side. For the RED→GREEN→REFACTOR side: run the scenario
      against a subagent without the skill loaded (RED, record the baseline
      failure verbatim), write/adjust the skill to address exactly that
      failure (GREEN, re-run and confirm it now complies), then hunt for a
      new rationalization under repeated pressure and close it (REFACTOR) —
      see `legal-ru/docs/dispatch-history.md` for a worked example of this
      cycle run against this family's own content.

## What this document is not

Not a gate that blocks work, and not exhaustive — it's the concrete list of
what this family's own history has actually gotten wrong, kept current.
When you find a new class of gap during a review, add it here with the same
"what happened, not just what to check" framing, so the next agent doesn't
re-discover it from scratch.
