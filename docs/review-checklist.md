# Review checklist for agents leading work on this family

For any agent picking up `legal-ru`, `patent-ru`, `gost-ed-mashiny`,
`arbitrazh-ru`, or a future plugin added to this marketplace. Every item
below is here because it was a real gap found in this family, not a
hypothetical — see the note after each one.

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
- [ ] Has a "Протокол пробела в базе" (or equivalent) covering questions
      outside every registry entry — see `docs/norms-verification-convention.md`
      §7. *(Added 2026-09-14 after the owner asked directly what the skill
      should do when a question falls outside the registry entirely; adopted
      same-day into all four plugins, each RED-GREEN tested separately —
      don't assume one plugin's wording covers another's edge cases.)*
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
      imported into). *(Precision added 2026-09-19, from the importing
      project's own report: its rule refuses any leading-dot path without
      exception, which is why `gost-ed-mashiny` can be **read** from that
      library but not **installed** from it — its `.claude-plugin/` is
      dropped on import. So the item is not "never have a dot path": a
      plugin needs `.claude-plugin/`. It is "know that a dot path does not
      survive import, and take the plugin from its own repository when you
      need to install it." `patent-ru/.secrets/` and
      `gost-ed-mashiny/.claude-plugin/` are both correct as they stand.)*
- [ ] **Anything moved into a repository from a scratch or temp directory
      has been read in full by whoever moved it** — saving someone else's
      work is a publication and obeys the rules of one. *(2026-09-19:
      review probes were moved wholesale out of a scratchpad to keep the
      harness from dying with the session, and were not read first. They
      carried the owner's login, and — a step worse than the leak class
      this family already rewrote two histories for — his account and
      organisation UUIDs. Caught the same day by the session that did it,
      after an unrelated warning about a path in a message. The cause is
      not "check paths": that is a consequence. The cause is that rescuing
      material did not feel like publishing, and the read-before-publish
      rule was never mentally applied to it. The artefact being someone
      else's is exactly why it went unread — your own text you read while
      writing it, so the rule fires precisely where you least expect to
      need it.)*
- [ ] **In a repository with more than one remote, "pushed" is not a state
      but a question of *where*.** Check `git for-each-ref refs/remotes`
      rather than remembering. *(2026-09-19: a leak fix was pushed to the
      GitHub mirror and not to the NAS `origin` this project treats as
      primary, leaving both offending files live in the tip of the primary
      remote while the author believed the matter closed. The cause was not
      the known key problem — `core.sshCommand` has been in `.git/config`
      since 8 September and plain `git push origin` worked first try — but
      habit: `git push github HEAD:master`, typed because that was the
      evening's pattern in a different repository, without looking at what
      remotes this one has.)*
- [ ] **A fix at the tip does not remove the content from other lanes'
      working copies.** After scrubbing anything sensitive, remember that a
      lane whose branch was merged *before* the fix still holds the old
      lines in its checkout — and in its caches. *(2026-09-19: after the
      leak above was fixed at both tips, both offending versions were still
      live in a neighbouring lane's worktree, with a third copy in its
      `.ruff_cache`. A merge will not bring them back, since master is
      ahead; a pass that copies or commits the file wholesale will. The
      importing project's answer, worth copying: scan the tree for the
      login and both UUIDs **before every merge** — and **not with `git
      grep`**, which sees tracked files at a tip while the copy that
      matters lives as a loose file or inside a cache; walk the filesystem,
      excluding `.venv` and `__pycache__`, on the grounds that the
      last one to look before publishing to two mirrors is cheaper than one
      missed look.)*
- [ ] **State a repository's visibility only after `gh repo view <repo>
      --json isPrivate`, never from memory.** *(2026-09-19: two independent
      sessions each told the owner "public repository" about the same
      private one, in the same direction, within hours. That is not
      carelessness twice — it is a property: four of this family's six
      repositories are public, so "public" is the default assumption, and
      it is wrong exactly when it matters, which is while someone is
      sizing a leak.)*
- [ ] **A watchdog script is not a watchdog until something invokes it.**
      Before writing one, name what will trigger it. *(2026-09-19: the
      importing project has had a working upstream-drift checker since
      14 September, routing for what to do with its findings, and a
      `schtasks` line in a document that has never been run. Over those
      five days drift was caught three times — every time by a letter from
      another session, never by the tool. The same repository has a
      scheduled task whose nightly run has never once been observed. The
      gap is not in the code and not in the intent: nothing calls it. A
      trigger tied to an action that already happens beats a schedule
      nobody installs.)*
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

## Public marketplace, private plugin repo — a real gotcha, not yet fixed

*(Found 2026-09-14.)* `marketplace.json`'s `source.url` can point at a
private GitHub repo without erroring — the marketplace file itself is public
and installs fine, but `/plugin install <name>@ru-legal-skills` fails for
anyone who isn't the repo owner, silently from the marketplace's point of
view (the failure surfaces only as a clone error on the installer's own
machine). `arbitrazh-ru` shipped this way: listed in the public README and
`marketplace.json` while `serjdrej/arbitrazh-ru` is private. The owner's
explicit call, asked directly: leave it as-is for now rather than make the
repo public or pull the listing — re-check this before telling anyone
outside the owner that this marketplace's fourth plugin is installable.
**Check `gh repo view <owner>/<repo> --json isPrivate` for every plugin
listed in `marketplace.json`** — don't infer visibility from the fact that a
plugin is listed at all.

## Testing status — track it, don't assume it

- [ ] `legal-ru`: full RED→GREEN→REFACTOR pressure-tested (4 branches, one
      scenario per highest-risk item, Sonnet 5 subagents) — done
      2026-09-10; gap protocol RED-GREEN pass added 2026-09-14.
- [ ] `patent-ru`: pre-dates this family's process; regression-tested
      against a live case per its own `references/audit-compliance.md` —
      not the same methodology, don't conflate the two when reporting
      status. Gap protocol RED-GREEN pass added 2026-09-14 (surfaced that
      WebFetch and raw `urllib` both fail against rospatent.gov.ru on a
      certificate-chain error — different mechanism from legal-ru's
      http/https bug, don't conflate the two).
- [ ] `gost-ed-mashiny`: **live citation data verified** (rst.gov.ru open
      data, real status pulled for both ГОСТы) — this is NOT the same as
      pressure-testing. Behavioral discipline under pressure (does it
      resist fabricating a нормируемое значение when pushed, does it
      actually call `gost_lookup.py` instead of answering from memory) —
      closed 2026-09-14 via the gap-protocol RED-GREEN pass.
- [ ] `arbitrazh-ru`: gap-protocol RED-GREEN pass done twice — the first was
      self-graded inside one dispatch and explicitly not trusted as
      sufficient; re-run as two independent subagents (one blind, one
      loaded with SKILL.md) on 2026-09-14. The RED baseline fabricated a
      named, dated Постановление Пленума with no verification marker — a
      live example of the exact failure this whole family's discipline
      exists to prevent, not a hypothetical.
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
