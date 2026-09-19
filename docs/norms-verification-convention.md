# Norms-verification convention

This marketplace holds four plugins — `legal-ru`, `patent-ru`,
`gost-ed-mashiny`, `arbitrazh-ru` — each of which cites Russian legal/technical
norms (statutes, приказы, ГОСТы, техрегламенты, court procedure) in its own
drafting work. Each plugin implements its own citation registry
(`references/norms-registry*.md`) and its own lookup script, **deliberately
not shared code**: every plugin must install and work standalone, with no
dependency on a sibling plugin in this marketplace.

What they *do* share is a convention for how a registry entry should be
shaped, so the three don't drift into incompatible habits. This document
states that convention, derived from the three registries as actually built,
not invented in the abstract. Every point below names which plugin's registry
it comes from.

## 1. The hybrid citation model

**Decision rule:** for each norm, judge amendment frequency and source count
*for that norm* — don't default to one treatment for an entire registry.

- **Rarely-amended + double-sourced → a verbatim excerpt is acceptable.**
  `patent-ru/references/norms-registry.md` §2 inlines dozens of literal
  quotations from order № 107 (Требования Минэкономразвития) and §3 from
  Регламент № 163, because these acts "меняются редко" and were
  cross-checked against two independent sources on the same date (the
  official Роспатент catalogue and a local downloaded copy) — the file calls
  this "самый надёжный статус в этом реестре."
- **Frequently-amended or single-sourced → paraphrase only, dated, with a
  re-check instruction.** The same `patent-ru` registry explicitly refuses
  verbatim text for ГК РФ (§1) and for Постановление № 941 on пошлины (§4):
  *"Текст сюда не вшивается — по требованию заказчика... эти... источники
  меняются существенно чаще... вшитое число или цитата устареют быстрее, чем
  кто-то заметит."* `legal-ru`'s registries (e.g.
  `references/norms-registry-statutes.md`) apply the same rule to 98-ФЗ, ГК
  РФ and ТК РФ entries — every entry there is paraphrase, never a quoted
  clause. `gost-ed-mashiny/references/norms-registry.md` states the same
  rule even more bluntly for its own single-source situation: *"В этом
  реестре таких выдержек нет: содержание ниже дано как парафраз."*
- One registry can legitimately mix both treatments side by side — that is
  exactly what `patent-ru`'s single file does (verbatim for §2–3, paraphrase
  for §1 and §4). Don't pick a treatment for the whole file; pick it per
  entry.

## 2. Status-marker vocabulary — reuse this wording, don't invent new phrasing

Three tiers, in the words the existing registries actually use. A future
registry in this family should reuse these phrasings rather than rewording
them from scratch:

| Tier | Wording used | Source |
|---|---|---|
| Paraphrase, unverified | *"парафраз; drafted from training data on \<date\>, not yet verified against a live source — re-check before reliance"* | `legal-ru/references/norms-registry-statutes.md`, entries R-98-3, R-98-5, R-98-TK14, R-GK-431, R-GK-401-406 |
| Paraphrase, unverified (equivalent, plugin's own words) | *"содержание — парафраз из рабочих материалов skill, составлено \<date\>; не сверено с действующим текстом стандарта. Не использовать как цитату и перепроверить перед выпуском документа."* | `gost-ed-mashiny/references/norms-registry.md`, all three entries (ГОСТ Р 2.610-2019, ГОСТ Р 2.601-2019, ТР ТС 010/2011) |
| **PROCEDURAL DETAIL — extra caution** (most preclusive/fast-moving details) | *"Статус: PROCEDURAL DETAIL — extra caution. Paraphrase only, drafted from training data on 2026-09-10, not verified against a live source. Exact deadline, application form and filing route must be re-checked before any filing."* | `legal-ru/references/norms-registry-corporate.md`, entry R-CORP-EGRUL-129-5-17 (ЕГРЮЛ filing deadline/form) |
| ✅ Verified verbatim (double-sourced) | *"✅ сверено дословно по полному тексту акта"*, with the strongest variant *"Двойное подтверждение — самый надёжный статус в этом реестре"* | `patent-ru/references/norms-registry.md` §2 header and every row of its table |

`legal-ru`'s SKILL.md names the PROCEDURAL DETAIL tier explicitly as a
category, not a one-off: it lists the breach-notification timetable
(152-ФЗ ст.21 ч.3.1), the Р13014/ЕГРЮЛ filing deadline (129-ФЗ), and
pre-litigation procedural terms as the class of entry that gets it —
"Относиться к ним строже, чем к остальным записям."

## 3. Verify via the lookup script — run it yourself when you can, never guess

House rule, stated in all three plugins' SKILL.md: an agent never presents a
citation as current without a verification instruction attached. But **"the
user runs it himself" is not the same as "the agent never runs it"** — an
agent running Claude Code *locally*, as an installed skill on the user's own
machine, should actually run the lookup script itself before relying on a
registry entry, not only tell the user to. The distinction that matters is
local-vs-cloud execution, not agent-vs-human.

**A corrected finding, from this project's own build history — read this one
carefully, it cost several days:** `publication.pravo.gov.ru`
(`legal-ru`'s primary source) was documented for days as "unreachable from
every agent sandbox — a geo-block." That was wrong. The real cause: every
URL used `https://`, and this host's HTTPS hangs on connect for any client
tried, while plain `http://` answers in well under a second — same host,
same machine, same network. Two things hid this: command-line tools tested
first failed for an unrelated reason that looked similar (Windows/Schannel
TLS issues with some HTTP clients), and Claude's own `WebFetch` tool
**silently upgrades `http://` to `https://`** — so testing through that one
tool kept "reconfirming" the wrong diagnosis no matter how many times it was
retried. The actual script (plain Python `urllib`) was never tried against
`http://` until the scheme itself was questioned — full account in
`legal-ru/docs/BRIEF.md`, "The lookup mechanism."

- `rst.gov.ru` and `eaeunion.org` (`gost-ed-mashiny`'s sources) were reachable
  directly throughout, `https://` included — the open-data CSV catalogues
  were actually downloaded and read while building
  `gost-ed-mashiny/references/norms-registry.md`.

**The lesson is not "agent sandboxes can't reach .gov domains" — that was
always going to be wrong, and turned out to be wrong for a more specific
reason than "test reachability per source" alone would have caught.** Two
sharper corollaries, learned the hard way:

- **A single failing tool call is not a confirmed block.** Test with the
  actual client the script will use (here: plain `urllib`, not a browser
  automation tool, not a fetch service with its own silent rewriting rules),
  and test both `http://` and `https://` explicitly before concluding
  either scheme is broken — a hung `https://` connection looks identical to
  a geo-block from the outside.
- **Local-vs-cloud execution is the axis that actually matters**, not
  agent-vs-human. An agent running Claude Code locally, on the user's own
  machine, has the user's own network — it should run the lookup script
  itself. A cloud-hosted dispatch (a bridge sandbox, a fetch service running
  off-machine) may still fail for reasons specific to that infrastructure;
  that is a property of the specific tool, not evidence that "no agent" can
  reach the host.

### Known properties of the shared sources — measured, not inferred

Measured by `arbitrazh-ru` on the dates given and **not re-measured here**;
recorded because each one is a property of a source all four plugins query,
not of one plugin's code.

- **Case form in `by-title` loses amendments silently.**
  `publication.pravo.gov.ru` matches on substring without declension, so the
  nominative and genitive forms of the same title return different histories:
  for АПК the nominative stops at 2023-12-25 while the genitive reaches
  2024-06-20; for НК the gap exceeds a year; for ГК it is twelve years.
  Abbreviations («АПК РФ», «ГПК РФ») return zero matches. **Run both forms.**
  One form is not a wrong answer — it is a quiet loss, which is worse.
- **Redaction order in ИПС is not signing order.** Measured 2026-09-18 on
  АПК: the current redaction is `rdk=89` of 28.11.2025, while `rdk=88` was
  signed *later*, on 15.12.2025. The ordering reflects entry into force, not
  signature. "Latest by date" and "in force" are different questions and
  neither follows from the other.
- **For large codes the consolidated text is not prepared for every
  redaction.** НК: 31 of 870. АПК: 65 of 90. `text --rdk` on an unprepared
  redaction returns an empty result with a note, not text — a property of the
  source, and a script's output must distinguish the two.
- **The HTTP/HTTPS trap on `pravo.gov.ru`** is the same one documented at the
  top of this section; it is listed again in the plugins' own registries
  because it cost days.

## 4. A script is not mandatory — an honest navigation route is a legitimate outcome

Build a lookup script only when a real, testable, query-able source exists.
When it doesn't, document the navigation route instead — the way
`patent-ru/references/norms-registry.md` §5 does (a stable entry point plus
step-by-step instructions, explicitly not a brittle final URL) — and say
plainly that no script was built, rather than forcing one against an
unsuitable API.

`gost-ed-mashiny/references/norms-registry.md` shows both legitimate shapes
in the same registry file:

| Norm | Shape | Why |
|---|---|---|
| ГОСТ Р 2.610-2019, ГОСТ Р 2.601-2019 | Real script (`python scripts/gost_lookup.py status "<обозначение>"`) against `rst.gov.ru`'s open-data catalogue | A queryable, documented CSV export exists and was confirmed to answer the actual question ("Статус" field by "Обозначение") |
| ТР ТС 010/2011 | No script — a documented stable entry point (`eec.eaeunion.org/.../TR_general.php`, "официальный навигационный перечень ЕЭК") plus manual navigation instructions | No queryable API was found that answers this question (see the rejected candidate below) |

**Concrete cautionary example** (worth citing when this question comes up
again): `docs.eaeunion.org/api/documents.php` was investigated as a possible
source for ТР ТС 010/2011 and rejected — it returns a general XML document
stream but has **no free-text or document-number search parameter**; the
plausible-looking parameters `search`, `q`, `query`, `title`, `number`,
`document_number` were all tried and ignored, leaving only pagination, date
ranges, and an undocumented `type` code. `gost-ed-mashiny`'s registry records
this outcome explicitly so it isn't re-investigated from scratch: *"Не
строить lookup для ТР ТС 010/2011 на этом API и не продолжать подбор
значений `type`."* The general lesson: **an API existing is not the same as
an API answering this question** — confirm it has the specific query
capability the registry needs before building a script against it.

## 5. One registry file per skill with one branch; split per branch otherwise

`patent-ru` and `gost-ed-mashiny` are each a single-skill, single-branch shape
(patent-ru's two "branches" — search vs. drafting — share one norms
vocabulary), so each uses exactly one `references/norms-registry.md`.

`legal-ru` has four independent branches (commercial secrecy, personal data,
corporate, pre-litigation), each loaded separately, and splits into four
files instead: `references/norms-registry-statutes.md`,
`-personal-data.md`, `-corporate.md`, `-procedure.md`. This is a
progressive-disclosure decision, not an arbitrary one — `legal-ru`'s
`docs/BRIEF.md` states it directly: loading one branch's reference file
should not force-load another branch's citations along with it. A registry
that mixed all four branches' norms into one file would defeat the point of
having separate branch files in the first place.

**Rule:** match the split to the skill's own branch structure. One branch →
one registry file. Several independent branches → one registry file per
branch, named `norms-registry-<branch>.md`.

## 6. The standing disclaimer

Every plugin states, in its own words, that its output is a draft for a
qualified professional's review, never advice on its own:

- `patent-ru/SKILL.md`: *"Правовая оговорка. Результат — поисковый сигнал
  или черновик текста, а не правовая позиция. Решение по стратегии, редакции
  формулы и индексу МПК — за патентным поверенным."*
- `legal-ru/SKILL.md`: *"Каждый содержательный результат — черновик для
  проверки квалифицированным юристом, а не юридическая консультация."*
  (repeated verbatim at the end of `references/corporate.md`: *"Это черновик
  для проверки квалифицированным юристом, а не юридическая консультация."*)

*(Updated 2026-09-10, fourth review: `gost-ed-mashiny` added its own standing
line since this document was first written —* "Черновик, не готовый к подаче
документ" *— now near the top of `SKILL.md`'s rules section, matching the
other two in substance. All three plugins now carry one.)*

**Position matters as much as presence.** `legal-ru` and `gost-ed-mashiny`
place theirs early (paragraph three of the README, and near the top of the
rules list in `SKILL.md`). `patent-ru`'s — *"Правовая оговорка..."* — is
currently the **last** bullet of a seven-item list in `SKILL.md`, and the
last bullet of "Чего НЕ делает" in its README. A disclaimer a reader has to
scroll past everything else to reach does less work than one stated early —
if you're touching `patent-ru`, moving it isn't a rewrite, just a reorder.

## 7. The gap protocol — what to do when the registry has no entry at all

*(Added 2026-09-14. Started in `legal-ru`, adapted — not copied, each plugin
wrote its own — into `patent-ru`, `gost-ed-mashiny` and `arbitrazh-ru` the
same day.)*

Everything above governs a norm that already has a registry entry. None of it
says what to do when a question falls **outside every entry, and outside the
skill's own scope**: an adjacent area of RF law the plugin never built a
branch for, a different jurisdiction (an EAEU member state, any foreign law),
agency guidance, or court practice. Left unaddressed, the default failure
mode is exactly what this whole family has fought elsewhere — answer fluently
from training data as if it were a checked citation.

`SKILL.md` in each plugin states a "Протокол пробела в базе"
(`legal-ru`)/equivalent section, triggered *before* any registry is
consulted, splitting the gap into two cases:

1. **Still within scope, just not registered yet** (e.g. still RF federal
   legislation for `legal-ru`/`patent-ru`, still a ГОСТ/ТР ТС for
   `gost-ed-mashiny`, still a registered АПК/ГПК article for `arbitrazh-ru`).
   Search live first with the plugin's own lookup script — never answer from
   memory — then hand back the finding under a status tier deliberately
   weaker than a normal registry entry: **AD HOC** — found live on a date,
   not in any registry, not reviewed. **AD HOC has two sub-shapes and they
   must not be conflated. Since the ips_lookup wave of 2026-09-15..18 the
   line between them runs between *scripts*, not between plugins** — name
   the script that produced the finding, never "a live search":

   | Call | Establishes | Never establishes |
   |---|---|---|
   | `pravo_lookup.py by-title` (publication records) | that an act exists, is in force, and which amendments were published | the text of any article — its own docstring says so, and the word `article` does not occur anywhere in it |
   | `ips_lookup.py redactions <nd>` | **which redaction the source itself marks as current** (`current_rdk`), and what redactions exist at all | which redaction was in force on the date the disputed relations arose — ИПС stores no commencement dates, so that is a legal question, not a reference one |
   | `ips_lookup.py text <nd>` | the consolidated text of an article for one redaction | which redaction it just returned, unless `--rdk` was passed explicitly |
   | `gost_lookup.py status` (rst.gov.ru) | a standard's bibliographic status | anything about the standard's content |

   **The rule behind the table matters more than the table: a capability
   claim attaches to the call, not to the name above it.** This section got
   the same error three times in one day, each time one level finer —
   "a live search returns text" (false: two scripts, one of them a
   publication register), then "`ips_lookup` cannot say which redaction is
   current" (false: its `redactions` subcommand does, its `text` subcommand
   does not). Scripts get renamed and subcommands get added; a sentence
   pinned to what is actually invoked survives both. Note also that
   `current_rdk` is computed as `"selected" in attrs` — it reports **what the
   source marked**, not what the tool concluded, which is this same section's
   own principle applied to itself.

   A finding from a publication-record or status lookup still leaves every
   specific figure — a percentage, a fee, a day count — as unverified
   training-data recall. Only a finding whose **text was actually read** may
   label that figure live-confirmed, and then only if it records *which
   redaction was read* (`nd` and `rdk`): a ✅ without them is worse than an
   honest paraphrase, because it reads as verified and goes stale silently
   the next time the redaction changes.

   **An honest "not verified" marker does not close a question when the
   tool at hand answers it.** The wording is `arbitrazh-ru`'s, from its
   `docs/PROJECT-MEMORY.md`, and it generalises: in a plugin that ships a
   script reaching this source, "не проверял" about a federal act is a
   defect, not a careful answer. Its sharp edge — named by the session drafting the growth
   layer, not by the rule's author — which the rule alone does not cover: reading the text and establishing which redaction is current
   are **two separate questions, and answering the first does not close the
   second** — a figure read out of a named redaction is confirmed only as
   the text of that redaction until its currency is established separately.

   `ips_lookup.py` is carried by `legal-ru`, `patent-ru` and `arbitrazh-ru`;
   `gost-ed-mashiny` deliberately does not carry it and records why — ГОСТы
   and ТР ТС are not in that system at all, confirmed by a zero-match check.

   *(Corrected 2026-09-19. The previous wording cited `legal-ru`'s `by-title`
   as the canonical "confirms existence, never returns text" case and was
   falsified by the wave above. It erred in the dangerous direction: it told
   a reader that a figure obtained through that plugin is always recall, when
   the plugin can now read the article. The first attempt at this correction
   erred in the mirror direction — "a live search now returns text" — and was
   caught by `arbitrazh-ru` before it was written: `pravo_lookup` and
   `ips_lookup` are both "a live search" and only one of them returns text.
   Hence the rule above is stated per script.)*
2. **Out of scope entirely** — another jurisdiction, court practice/agency
   letters where the plugin's script cannot read practice (only acts), or a
   subject area the plugin explicitly excludes. The script cannot help here.
   Say so plainly and ask the user how to proceed (open web search marked
   unverified, the user supplies the exact citation to verify live, or refer
   to a qualified professional) rather than let general training-data
   knowledge stand in unlabeled.

**A statute/article number appearing in the question does not by itself make
it case 1** if the actual ask is about practice interpreting that
article rather than its text — `legal-ru`'s own RED-GREEN pass got this
right but flagged it as needing judgment, not a mechanical rule.

**RED→GREEN pressure-tested in all four plugins**, each against its own
three questions chosen to hit both cases plus this exact trap. Every pass
found and fixed a real wording gap rather than just confirming the design —
worth reading before assuming another plugin's version is identical:
`legal-ru/docs/BRIEF.md`, `patent-ru/README.md` ("Тестирование"),
`gost-ed-mashiny/README.md` ("Тестирование"), `arbitrazh-ru/docs/BRIEF.md`
and `docs/dispatch-history.md`. `arbitrazh-ru`'s second pass is worth noting
specifically: its first RED-GREEN test was self-graded inside one dispatch,
and was re-run as two genuinely independent subagents once that was
recognized as weaker evidence than the other plugins' tests — a real example
of not trusting a skill's own first self-report, the same discipline this
family's reviews apply to sessions.

## 8. The field reports the tool, not the world

*(Written by `arbitrazh-ru`, 2026-09-19, and rendered into this document's
language; the Russian field values are quoted as they actually read. Enters
the convention with the portability caveat stated at the end.)*

**The principle.** A value an instrument puts in its output asserts exactly
what the instrument did. «Отметки об отмене нет» means *the pattern found no
marker*, not *the act is in force*. «Текста нет» means *the source returned
no text*, not *no text exists*. «Статья не найдена» means *it is not in this
act*, not *no such norm exists*.

**What it rests on.** Three findings in three different scripts within the
24 hours of 17–18.09: a `repeal_absent_note` on records from a section where
the banner is never placed at all; `entity_status: terminated`, a name that
asserts about a legal person what honestly reads as "the register carries a
termination date"; an empty `--article` on an order that is divided into
clauses, not articles. **Each was found by someone other than the code's
author** — which is the point: the author reads the name as the intent.

**What breaks without it.** A negative result becomes indistinguishable from
the absence of a norm, and gets quoted as a conclusion. This is **one of two**
defect classes known in this family that neither a test run nor a file
comparison catches: the right answer and the wrong answer look identical.
The other — recorded twice, in `arbitrazh-ru/docs/BRANCH-proof-elements.md`
for the property and in that repository's `docs/dispatch-history.md` for the
three cases, the narrowing, and why the debt is deliberately left open — is a
file's statement *about nearby code* outliving a change to that code —
file-against-file comparison is blind to it by construction, because each
file is internally consistent and what lies is the link to a third. Stated
as two rather than one deliberately: a convention that closes the count at
one invites the reader to stop looking.

**Two naming rules.** A value names the source or the action («дата
прекращения не сообщена», «отметка не найдена»), never a state of the world
(«ликвидировано», «действует»). And a pair of values that are not each
other's complement must not pose as one: "not reported" is not "none".

**Mechanising it — linters.** A linter script checks what a human otherwise
checks from memory. `arbitrazh-ru` runs four: route reachability, filled-list
shape, field-map drift, and cited phrases against the act's text. The
principle above dictates two requirements on a linter's own output. **Skips
must appear in the output as a count and a list** — otherwise "skipped for
reason" becomes a stamp and the check goes green on what it never examined.
**An unverifiable measurement must be named in the output, not only in the
documentation** — if the linter compares the phrase but not the part number,
the output says so, or the reader completes the picture himself, in the
dangerous direction.

**A zero is a property of the instrument.** The sharpest special case, and
the reason this principle is worth a section rather than a sentence: a
negative result is never stronger than the query that produced it, so
"not found" is recorded together with **the form of the query and its
reach**, and a zero after a single query form is not a finding. Measured by
the `vsrf-practice` lane and relayed by `growth-research`: `vsrf.ru` search
is lexical and case form halves or doubles the hit count — «неустойка» 20,
«неустойки» 37, «неустойку» 8. `growth-research`'s own conclusion that
«обзоры» never carry full text came from one query and was refuted by the
second; and a neighbouring lane got a clean zero from a
scan whose regex class `[^.]` excluded full stops, i.e. excluded the very
dates it was scanning for. A zero looks identical whether it is true or the
instrument is broken — which is what makes it the worst of the negative
results.

**Two further ways to manufacture a false zero, both committed within the
hour by the two people writing this very section.** *Truncated output:*
`grep -rn … | head -5` over nineteen matches, reported as "the only record
in the whole directory" — a truncation is indistinguishable from a real
zero and cheaper to cause than a wrong query, so `head`, `-m` and a tool's
own output limit belong in the same warning as the wrong query form.
*Language:* in a bilingual repository a pattern written in one language
returns a confident zero over a passage written in the other; this
document's editor did it twice in one day, searching Russian for entries
that turned out to be in English. Neither failure feels like a failure:
both return promptly, and one of them returns exactly the line that
confirms what you expected.

**A field also reports what another lane measured, and that is invisible in
it.** A measurement travelling through an output field sheds its author, its
date and its corpus on the way. `arbitrazh-ru`'s `MORPHOLOGY_NOTE` reads
«…падеж существительного может менять число совпадений вдвое (измерено:
«неустойка»=20, «неустойки»=37, «неустойку»=8)» — the word *measured* and the
figures, but not by whom, when, or against what. At the point of use it reads
as a property of the source; the lane that cited it in good faith reported the
figures as its own finding, and the mis-attribution reached this document.
**The cost is not credit.** It is that an unattributed, undated measurement
**cannot be re-checked and cannot be allowed to expire** — the same defect
shape as a ✅ without its redaction identifiers in §7, arrived at from the
opposite direction. A measured value in a field carries the measuring lane and
the date, or it carries the command that reproduces it — **and the second
branch is the primary one, not the fallback.** A date cannot always be
recovered: `MORPHOLOGY_NOTE`'s figures sit in a measurements file with no
date, and the constant's first appearance in history, 2026-09-16, is when it
was committed rather than when it was measured. Back-filling that date would
pass the approximate off as measured, which is the defect the rule exists to
stop, so the lane refused to and put the command there instead. **A date is
sometimes unrecoverable; a command can always be constructed**, and it is the
form that survives the loss of the journal that recorded the measurement.

*Worked example of the requirement, and better than a signature would be.*
`arbitrazh-ru`'s `STATUS_CAVEAT` (`scripts/egrul_lookup.py`) states that an
empty termination-date field does not mean the entity is trading, and then
carries «замер 2026-09-18» together with **two ИНН** — one entity in
liquidation, one in bankruptcy — by which anyone can reproduce the finding in
a command. Date plus reproducer beats a lane's name: a name tells you whom to
ask, a reproducer survives the lane. Measured compliance across that
repository on 2026-09-19: of eleven fields carrying a measurement, eight
carry a date, three do not, and **none carries its measurer** — the rule was
written from a real and general gap, not an imagined one.

*The same loss runs in both directions, and the second is unnoticeable from
inside.* In one day the same lane read another lane's figures out of a field
and reported them as its own, and had its own measurement travel into a third
lane's shipped field with neither date nor author attached. Taking someone
else's finding for yours you can eventually catch; your own finding going
anonymous downstream you cannot, because nothing anywhere is now wrong — it is
merely unattributable.

**A number that arrives by retelling is documented by a reference to a
record, not to whoever said it.** "The lane measured it" is not provenance
even when the lane is right and the measurement was real. This is a distinct
failure from the substitutions elsewhere in this document, and the more
dangerous one: a substituted figure is caught by recomputing it, while a
transported figure is *correct* and caught by nothing except the question
"where is this written down?" — which nobody asks, because each retelling
makes the number look sturdier for having a new mouth behind it. Three
retellings put two percentages into §11 of this document; see the correction
there. Sharper, after the sequel: **a retold number without a reference to a
record is worse than no number at all.** An absent figure is visible as
absent. A retold one looks like knowledge and gains solidity with each new
mouth behind it — and the basis here was not even missing. Thirty-four logs
carrying `cmd`, `args` and `bytes` per call sat committed in the repository
while three sessions passed the figure between them by name, each step
sufficient-looking because whoever said it was present and could vouch. The
record existed; **the reference to it did not**, and that is the whole
distance between knowledge and hearsay.

**The second kind of transport loss is worse, and the first made it
visible.** What travels away is sometimes not the number but **the
qualifier attached to it**. §11's projection was downgraded to "a projection
under an assumption" within an hour of being stated; the downgrade went one
way, the figure went another, and the figure arrived bare. A missing
reference is detectable — someone eventually asks where it is written. **A
missing qualifier is detectable by nothing**, because a number stripped of
its caveat is byte-identical to a number that never needed one. So §10's
carrier rule applies to qualifiers moving between people, not only between
files: send the qualifier in the same message as the figure, or expect it to
arrive alone.

*A corollary about presenting figures, from the same exchange.* A quantity
chosen to suit a conclusion **does not thereby become false**, so no check
for correctness will catch it: 6 927 bytes and 39 % of the body are both true
of the same section, and the first makes it look negligible next to a
neighbour while the second makes it look structural. Show both **where they
diverge in meaning** — not always, or every figure acquires a companion and
the reader stops noticing the cases where the divergence is the point.

**Portability, measured rather than assumed (updated 2026-09-19).** One of
the four linters is now confirmed portable by live run: `router_lint.py`
against `legal-ru` returns `unreachable_references: []` and
`missing_scripts: []` — run by that plugin's own session and re-run here
independently. It also reads its siblings for comparison and reports
`gost-ed-mashiny` as having no routing-table line, which is a true
description of a different document shape rather than a finding against it.
The other three were **not applicable rather than failing**, and the
distinction is the point: `proof_list_lint.py`, `stage_map_verify.py` and
`claims_scan.py` expect artefacts of one plugin's domain — a proof list, a
stage map, a case journal with a source table — which `legal-ru` does not
have in that shape. "Nothing to apply it to" is not evidence either way, and
reporting it as a pass would have been the false positive this whole section
exists to prevent.

## 9. Mechanical or judgmental — the class decides what a skill may assert

*(From the `growth-research` lane of `arbitrazh-ru`, 2026-09-19. The
priority candidate, because the costliest measured defect in this project is
a wrong strategic recommendation carrying impeccable sources, and nothing
else in this document catches it.)*

**A claim is mechanical only if both hold:** (a) a source is named next to it
that settles it **without knowing the facts of the case**; and (b) it is
phrased **descriptively, not predictively** — "the norm requires X, X is
absent from the text", never "the court will refuse" or "the chances are
poor". **Checkability by itself says nothing about the class.**

**Why (a) is not "checkable ⇒ mechanical".** A counterexample, measured: the
check "the ходатайство under ч.4 ст.66 АПК does not name the evidence's
identifying details" is refuted by opening the article — a minute's work,
cheap and certain — and *simultaneously contradicts* п.38 ПП ВС РФ от
23.12.2021 № 46, which the lane reports as saying that naming those details
«не требуется» (verified there against the 26-page PDF; **not re-verified in
this repository**). One counterexample is logically enough to kill a
universal rule, which is why n=1 carries here although n=1 would not
establish a positive claim elsewhere in this document.

**Condition (b) rests on reasoning, not measurement** — the lane says so
itself. No run separates the harm of the predictive mood from the harm of
the advice it carries.

**What breaks without it.** A skill ships a check that is cheap, confident,
contrary to binding guidance — and **reports success**.

**Why it survives a moving boundary.** The criterion is pinned to *whether a
source exists*, not to the type of claim. When «отменён в части» acquired a
machine source (`scope: partial`), the claim changed class by itself and the
rule needed no edit. Test any reformulation against that: if it needs
rewriting when a new source appears, it is the wrong formulation.

## 10. One claim, one carrier — and a caveat that names its own deletion

**One claim lives in one place.** When a claim becomes machine-readable — an
output field, a marker, a test — the prose copy is **deleted**, not left
alongside. Copies drift; the check goes green on the machine copy while the
reader believes the prose one; and **a file-against-file comparison never
finds this**, because the contradiction is inside one file, which is
consistent with itself by definition.

*Evidence, and an instructive complication.* The lane's flagship example was
a registry file that told the reader to verify the text "here and now" on one
line and stated that no entry had been verified on another. Checked here on
2026-09-19: **it no longer reproduces** — `arbitrazh-ru` repaired it in
`503d775`, and every entry now carries ✅ with its `nd`. The principle stands
on the other cases (the lead reports three in one day), and the staleness of
the example is not a refutation but an illustration: a report about drifting
copies drifted while in flight.

**Where a new caveat goes — three tiers, and the third owes an
explanation.** (1) A field in a script's output: it travels with the data and
a regression test covers it. (2) A file the route forces open. (3) The
skill's root file. Choosing (3) requires **naming why (1) and (2) would not
do**. Measured in one day: a caveat living in a field was caught by a test,
while a caveat living in prose became false and was caught by nothing.

**The rule worth more than the tiers: a caveat should name the condition
under which it is deleted.** The `--article` caveat is the first in this
family ever *removed* rather than accumulated — it stated what would make it
unnecessary, that thing happened, and it went (ten lines to four in
`arbitrazh-ru`, nine to five in `patent-ru`). Without this, caveats only ever
accumulate, because no carrier has a deletion event, and the root file turns
into a graveyard of sentences that were once true.

## 11. Open questions — proposed, not adopted

Recorded so they are not re-proposed from scratch, and explicitly **not
rules**. Each failed the same filter every section above had to pass: name
what breaks for a skill that ignores it.

- **A prohibition with no executable alternative breeds lies.** Before
  forbidding an action, name what to do instead. Proposed by the
  `growth-research` lane on one case — a flat "never install packages
  yourself" that the owner corrected, since explicit human consent in the
  current dialogue does cover one install; the flat ban would have made
  honest behaviour a violation. The lane proposed it as a question rather
  than a rule, and that is how it stands. Holding the filter against a rule
  that sounds right is the reason the filter exists.
- **A ceiling on `SKILL.md` size.** `arbitrazh-ru` holds one near 100 lines
  on the reasoning that its file grew past 498 lines and carried three
  self-contradictions in a day.

  **Correction, 2026-09-19, and it is this document's own §8 defect.** The
  entry used to say a neighbouring lane had measured the opposite quantity —
  "at 43 KB its routes cost 70% less than a full read, and the planned cut
  makes the same route 83% dearer" — and set that against the reasoning as
  though both sides were measured. Those two percentages are **not
  reproducible**: a search across all six repositories finds them in this
  file and in a draft quoting this file, nowhere else. I wrote them here from
  a report, without the command, the date or the corpus — exactly what §8
  forbids, in the section that demands it, about a figure I did not measure.
  The "43 KB" traces to a body size in `lazy-skill-library`'s own eval; the
  percentages do not trace anywhere.

  **Second correction, same day, to the correction above.** It closed by
  calling both sides "honestly unmeasured". That is also wrong, and the
  difference is practical. The measurement *was* made — a shim over 21 runs
  and seven tasks with the model named, per the lane that ran it. Its harness
  is on disk and was checked here: `lazy-skill-library/docs/reviews/`
  `2026-09-18-outline/eval/` holds `lazy.py`, `grade.py`, `evals.json` and 34
  shim logs. What never happened is the **recording of the result**: the
  figure travelled from the lane's context into a message, into a letter, into
  this section, and did not land in a file at any step. (`results.json` there
  holds the eval's 33 answers, not the route costs.)

  **Unmeasured must be measured; unrecorded must be retrieved** — and this is
  the second. Retrieved the same day, and recomputed here independently
  rather than taken back on report: summing the `bytes` field over the shim
  logs gives **13 182 B in each of three byte-identical T3 runs against a
  43 629 B body — 30,2 %, a saving of 69,8 %**, and across all 34 logs a
  median of 7 576 B, a saving of 82,6 %. So the figure that travelled as
  "70 %" was real, and was computable by any of the three sessions retelling
  it, in one command, from a directory one of them had committed the day
  before.

  **Only one of the two numbers came back, and the asymmetry matters.** The
  second — "the planned cut makes the same route 83 % dearer" — concerns a
  cut nobody has made, so it cannot be in these logs. Its author has since
  named its origin: a calculation of his own, 13 741 + 10 433 = 24 174 B
  against 13 182, arithmetically sound and resting on an assumption the
  cutting lane disputed the same day and he accepted — that after extraction
  an agent reads the remainder whole. They agreed to write it as "a
  projection under an assumption". **That downgrade travelled a different
  route and never arrived here**, so the figure reached this document naked.

  Its status, in the words its author proposed and which are the right ones:
  **a projection about an operation that was never performed; the operation
  has since changed, and the number no longer refers to it.** Not refuted,
  not confirmed. The first cut has now been made and is a different
  operation — headings inside the file rather than extraction to
  `references/`, no word of content altered — taking the case-1 question
  from 19 455 to 4 925 B at a cost of 259 B for reading the outline
  (measured by `arbitrazh-ru`, not re-measured here).

  So the retracted pair was: one real measurement that lost its address, and
  one projection that lost its qualifier. The sentence presented both as
  measured.

  What is independently reproducible meanwhile: the other three plugins'
  bodies run 19.2–19.7 KB against a measured 20 063 B point where
  section-wise delivery won all five runs — which cuts against the
  comfortable reading that the neighbours are too small for this to matter.
- **Whether a caveat comes out good because it could not be fixed.** The
  exemplar in §8 was written where the defect was unfixable — the source
  does not report whether an entity is in liquidation, so the lane could not
  repair the code and could only state the gap honestly, with two ИНН that
  prove the emptiness means nothing. Its author's conjecture: fields turn
  out well precisely when the author cannot hide the defect behind a fix.
  Recorded here rather than as a rule because it fails the filter — there is
  no answer to what breaks for a skill that ignores it, and the evidence is
  one field and an inference. Its author agreed with the refusal before
  agreeing with the reasoning behind it, which is the useful part: **a filter
  applied only to other people's ideas is not a filter.**

  **It is falsifiable, and it is being measured.** The prediction: a caveat
  written where a fix *was* possible will be the weaker one — a general word
  where the other has an ИНН, «иногда» where the other has a number. Strong
  caveats from repairable situations refute it, and they refute it visibly.
  `arbitrazh-ru` sees every new caveat at merge review — roughly ten over two
  days — and is recording one question against each: could this have been
  repaired instead of described? That turns the conjecture into a count
  rather than an impression, at no extra cost to anyone.

  The mechanism it proposes, worth keeping even if the conjecture falls:
  **while a repair is still possible, the honest statement feels optional.**
  That explains more than fields — it is also why three of this document's
  own corrections came from prior substituted for a check by people who
  could have run the check, and did not precisely because they could.
- **Granularity of extraction.** `read_skill_resource` cannot address a
  section — a resource is read whole — so moving a section out as one large
  file can cost more than the section did inline. One measurement on one
  skill; possibly more important than the ceiling above.

## 12. An article number without the code of the act is not a reference

*(From `arbitrazh-ru` — occasioned by its lead, measured and brought by the
`growth-research` lane, 2026-09-19. Answers the "what breaks" filter more
sharply than anything else in this document: three years against one month.)*

**A reference to an article always carries the code it belongs to**, in prose
and in machine markup alike. «Ст. 321» is not a reference; it is a defect
waiting for a reader.

Three collisions in one corpus, all inside three days:

- **Ст. 321.** In АПК it is the three-year window for presenting a writ of
  execution — measured live, `ips_lookup.py text 102079219 --article 321`,
  2026-09-19. In ГПК it is the one-month window for an appeal — taken from
  `arbitrazh-ru`'s own ГПК registry, which marks that entry as an unverified
  paraphrase, so half of this collision rests on a registry entry and not on
  the code. Recorded that way rather than smoothed, per §8.
- **Ст. 131.** Form and content of a claim in ГПК; response to a claim in
  АПК. A search for the bare number returned three hits that were about to be
  counted as coverage of the response topic. All three were ГПК.
- **Ст. 303 and 306.** They sit inside a file about АПК evidence among two
  dozen АПК articles and belong to **УК РФ** — confirmed here,
  `references/evidence-gathering.md` lines 252–253. A verifier holding one
  act constant would resolve them to АПК and **report success on the wrong
  text**.

**What breaks without it.** The skill ships a deadline wrong by three years
against one month, and the error *looks like a citation*, so it survives
every check of form. In a corpus where АПК, ГПК, ГК, НК and УК coexist this
is the expected case, not the rare one: a bare number resolves to the code
the reader was already thinking about, not the code the norm came from.

**And it corrupts self-measurement, not only citation.** Twice in three days
a non-zero count on a bare number was read as "this topic is covered" while
the hits belonged to another code. A skill measuring its own coverage by bare
numbers measures someone else's.

**The operative test is not "repeat the code every time" but "nothing nearer
competes at the point of citation".** `legal-ru` calibrated it on its own
repository the day this section landed, and the three cases are worth more
than the rule alone:

- *A code named one sentence earlier is not enough.* Its
  `references/commercial-secrecy.md` read «…ст.10 нужно проверить…» on the
  strength of a 98-ФЗ named in the previous sentence. Fixed to «ст.10 98-ФЗ».
  This is what the skill hands a user, which is where the rule bites hardest.
- *A code opening the same sentence and governing a list is enough.* Its
  `docs/BRIEF.md` reads «ГК РФ Главы 27–29 …, ст.431 …, ст.333 …» — left as
  it stands, correctly: nothing competes for the antecedent.
- *And the configuration between them:* «two real amendments to 98-ФЗ … 86-ФЗ
  (amending ст.5) and 311-ФЗ (amending ст.6)». The governing code opens the
  sentence, so the test is met — but two **different** act numbers stand
  between it and the citation, and they are nearer. It resolves only because
  a reader knows an amending law amends the base law's articles. Sound as
  prose in a design document; **it would not be sound in anything a verifier
  parses or a user is handed**, which is the line to draw.

Write the rule to cover both carriers. The same requirement was already
mandatory in one plugin's verifier markup — act code, no defaulting to АПК —
on the strength of the УК case alone; that it now also holds for prose is the
evidence that it is one rule with two carriers rather than a detail of one
mechanism.

## 13. What this document is for

This is **style guidance for whoever adds the next plugin to this
family** — not a schema to validate against, and not code to import. There
is no shared library and none is planned: read this document, then write
your new plugin's own `references/norms-registry*.md` and its own lookup
script (or documented navigation route) following the same shape, entirely
within that new plugin's own repository.
