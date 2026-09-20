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
      missed look. Stated as a wrong way and a right way rather than as
      anyone's credit, because the rule has to work for a reader who knows
      none of the people involved: **`git grep` returns zero on the very
      tree where a filesystem walk finds three copies.**)*
- [ ] **State a repository's visibility only after `gh repo view <repo>
      --json isPrivate`, never from memory.** *(2026-09-19: two independent
      sessions each told the owner "public repository" about the same
      private one, in the same direction, within hours. That is not
      carelessness twice — it is a property: four of this family's six
      repositories are public, so "public" is the default assumption, and
      it is wrong exactly when it matters, which is while someone is
      sizing a leak. **Third instance the same day, and mine:** I relayed a
      consequence premised on a repository being installable by the public
      — 1.66 MB of development history reaching "whoever installs the
      plugin" — without running the command, hours after writing this very
      item. `arbitrazh-ru` is private. The consequence is not void, but it
      is a future risk if the repo is ever opened, not an active harm, and
      those are argued differently. Checked all four afterwards: three
      public, one private.)*
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
- [ ] **Before moving anything out of what ships, grep what ships for
      references to it** — `git grep -l "<dir>/" -- SKILL.md references/
      scripts/`. *(2026-09-19: the plan was to move a skill's `docs/` out of
      the agent-facing tree as development clutter. Eight shipped files
      reference it, one procedure file eight times, and a script names a
      spec there as its own. Moving them to a dot-path would have been
      **worse than leaving them**: today the file is present and readable in
      the imported copy, and after the move the reference resolves to
      nothing, because the importer drops leading-dot paths. The right split
      is by role, not by directory: what shipped prose tells an agent to
      follow **is** a resource of the skill, whatever the folder is called,
      and moves to `references/`; only the rest becomes a dot-path.)*
      **Count the citing files, not the cited ones** — you will be editing
      the citers. *(Same day: counting by target gave "three journals
      referenced" and hid a fourth citing file, which would have been left
      with a dangling reference after the move. Counting by source gave six
      citations in four files. Also use `git grep`, not `grep -r`: the
      latter counted a `.pyc` as a ninth file, matching binary content that
      is not even tracked.)*
      **Then read each hit and say whether it points inside this skill or
      at a sibling** — only the first kind breaks when you move the file.
      *(2026-09-20: the library's `--check-library` scan reports a dangling
      reference for `arbitrazh-ru/SKILL.md:10`, which cites `legal-ru`,
      `references/pre-litigation.md`. That file exists — in `legal-ru`. The
      detector reads every path as relative to the skill it is scanning,
      and `arbitrazh-ru` has no `references/pre-litigation.md` of its own:
      `git ls-files references/ | grep -c pre-litigation` → `0`, which
      reproduces the false positive without running the tool. The lock
      records the truth — zero dangling references for all four family
      skills. The surface grows with how well the family is cross-routed:
      a `git grep -lE` for a sibling's name over `SKILL.md references/`
      finds 9 shipped files, five of them in `arbitrazh-ru`. So do not
      build a move gate on that scan. **A gate that errs toward alarm
      trains you to ignore it as surely as a silent one, only more
      slowly.**)*
- [ ] **Shipped prose must not cite a private development journal.** If a
      `references/` file needs something from `PROJECT-MEMORY.md`, a
      dispatch history or a branch log, put the substance in the prose —
      do not drag the journal into the distribution to satisfy the
      reference. *(Found the same day, by the repository's own session,
      while sorting the previous item: three such citations. It is the
      standing README rule applied where it also belongs.)*
- [ ] **Do not filter by a guessed directory name — filter on what the
      author declares.** *(The importing library was about to exclude
      `docs/` by name, at the moment the one repository that has a large
      one was preparing to rename it to a dot-path; the filter would have
      been dead code on the day it shipped, and would have broken the first
      skill where `docs/` means documentation for the agent. A declared
      marker already exists — the leading dot, refused on every path
      component — so the fix may be documenting the contract rather than
      writing a heuristic.)*
- [ ] `git config user.name`/`user.email` set **locally in that repo**
      before the first commit — global config is not set on this machine.
      *(Hit twice this session: "Author identity unknown," both times on a
      repo's very first commit.)*

## After any **push** to a plugin repo — sync the marketplace

- [ ] **One writer for the pin.** A skill session reports «запушил `<sha>`»;
      the marketplace session verifies against `git ls-remote` and writes the
      `ref` and the `description` in `.claude-plugin/marketplace.json`. Nobody
      else commits to `ru-legal-skills`. *(2026-09-20: two conventions were
      running at once — `patent-ru` had been told the marketplace session
      would move its pin, while `legal-ru`, finishing its own review fixes,
      moved its pin itself and committed into the marketplace session's
      working directory, the same physical checkout. The content was correct
      — pin equal to `ls-remote`, descriptions byte-identical at 975/975 —
      and it was verified. It was safe only because that checkout happened to
      have nothing uncommitted at that minute; fifteen minutes earlier it had
      two unfinished sections of this convention in it. The session's own
      words, kept because they are better than the rule: **it was empty by
      accident, not by rule.** The reason for a single writer is not
      ownership: the pin is the one point where the state of four
      independent repositories is compressed into one claim, and from outside
      "the pin moved" and "the pin moved twice" look the same.)*
      **This does not extend to a skill's own repository.** Fixing your own
      files, ordering work inside an authorised scope, and setting aside a
      question of a different calibre are the lane's own judgment. A rule
      requiring confirmation for every edit would buy nothing and would
      teach asking permission where judgment is wanted — see the standing
      caution about forbidding something without naming what goes in its
      place. What is asked instead is one line **before**, not after:
      "taking these items, starting", so the coordinating session does not
      report as pending what is already done.
      *(Corollary, same day: `gost-ed-mashiny` keeps a **third** copy of its
      description in `.claude-plugin/plugin.json`, byte-identical to the
      other two at 522 chars. Three copies, and until now no sync step for
      the third anywhere in this checklist.)*

*(Trigger corrected 2026-09-19. This section said "after any commit" for
nine days and that is the wrong event: a pin is stale relative to what is
**published**, not to what exists in somebody's working copy. `patent-ru`
made the distinction while declining to push — its commit sat on a worktree
branch, `ls-remote` still showed the previous head, and the pin was
therefore **correct**, not stale. The reflex "I committed, so the pin is
stale" would have pointed the marketplace at a SHA nobody can fetch. That
did not happen today only because the sync script compares against
`ls-remote` rather than against a local ref — a mechanical check covering
for a wrong rule, which is the argument for having the check.)*

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

- [ ] **Turning a week's findings into instructions: count the journal lines
      you deleted, not the rules you moved.** A finding that has become an
      instruction is no longer needed in the store; leaving both makes two
      long documents and looks like work. *(The library session's measure,
      2026-09-20, after both of us counted where the day's writing had gone:
      85 % of mine and 92 % of theirs landed in documents read when
      searching. The reason is the same for both — the storing document is
      easier to write, because a claim there only has to be true, while an
      instruction has to name whose repeating action it changes. And the
      volume is not merely idle: it competes for attention with the few
      lines that must be findable, so a journal answers a future question
      worse the more of today is in it.)*
- [ ] **Handing anything to another session: name the composition, not the
      assessment.** "SKILL.md and four norms registries were touched" is read
      and judged by the receiver; "a documentation commit" forces them either
      to trust it or to check anyway. Importance is the receiver's job,
      performed out of their context. *(Case and the payoff on first use: see
      the convention, "Name the composition, not the assessment".)*
- [ ] **A neighbour's number looks wrong: ask what they ran and when, before
      diagnosing their tool.** A figure can be stale with nothing broken, and
      a paraphrase of a figure can be wrong while both figure and tool are
      right. Advice about their instrument may still be worth giving — as a
      suggestion to check, never as the diagnosis; the two fit in one
      sentence, so separating them has to be deliberate. *(Case: see the
      convention, "Building a cause under someone else's error".)*
- [ ] **A number about state is said together with the command that just
      produced it, or it is not said.** *(The library session's own rule,
      adopted 2026-09-20 after its "three of four are current" turned out to
      be a stale paraphrase of a run it had not repeated that evening — the
      tool was right, the sentence was old. It costs one command and saves
      one investigation: the receiver spent a round building a mechanism to
      explain the wrong number, and the mechanism was real but not the cause.
      The general form, which is what makes this worth a checklist line:
      **a paraphrase of a measurement ages separately from the
      measurement.** The figure keeps being true of the moment it was taken;
      the sentence goes on being repeated after that moment has passed, and
      nothing in the sentence records when it was.)*

- [ ] **A closed vocabulary holds because the seventh value fails a
      mechanical check, not because the reader knows the six.** *(2026-09-20,
      `arbitrazh-ru`, found by a full run rather than by review: a test stood
      on one registry entry being "without a weak marker", and it was without
      one only because the invented tier it carried was not in the linter's
      set. An invented status is therefore not a vocabulary problem — it is a
      status that escaped the check, and a passing test was quietly resting
      on the escape. The session set out to fix wording and was in fact
      fixing the linter's reach.)*
- [ ] **When your own change breaks your own test, re-run the repaired test
      against the pre-fix commit.** A repair that also stops catching the
      original defect looks exactly like a correct repair: both end in a
      green suite. Only the old commit tells them apart. *(2026-09-20,
      `arbitrazh-ru`: two rows added to the routing table turned its
      capability test red — "действует ли он" from a new row about checking
      a Пленум was being read as a claim about `by-title` eighteen lines
      above. The session narrowed the granularity and then re-ran the test
      against `ffaee0a` to confirm it still failed there. Verified
      independently by extracting that commit to a scratch tree with `git
      archive`, dropping the new test into it, and running both: `FAILED
      (failures=2)` on the old tree, `OK` on the live one. Blunting a test
      is the cheapest way to make a suite green, and it is invisible in the
      suite.)*
- [ ] **A regression test whose fixture is not in the repository is a claim,
      not a test.** *(2026-09-20: `patent-ru/references/audit-compliance.md`
      §4 presented three numbers — "exactly 5 independent claims: 1, 16, 27,
      32, 37" among them — as the skill's own regression test. They were
      measured on the live claim set of the author's real case, which is
      deliberately not in the repository. So the thing the file called a
      regression test **could not be run by any maintainer, including its
      author**, for as long as it stood there. The repair was to rebuild the
      fixture by its declared *shape* rather than its text — 41 claims, 5
      independent at those numbers — which made the documented figures
      checkable by anyone. Note the order that makes the result trustworthy:
      the fixture was built to the declared shape **before** the code was
      touched, and gave 35 on the old code. A baseline constructed after the
      fix proves only that the fix is self-consistent.)*

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
