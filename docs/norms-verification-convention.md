# Norms-verification convention

This marketplace holds three plugins — `legal-ru`, `patent-ru`,
`gost-ed-mashiny` — each of which cites Russian legal/technical norms
(statutes, приказы, ГОСТы, техрегламенты) in its own drafting work. Each
plugin implements its own citation registry (`references/norms-registry*.md`)
and its own lookup script, **deliberately not shared code**: every plugin
must install and work standalone, with no dependency on a sibling plugin in
this marketplace.

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

## 3. Verify via a script the user runs himself — never the agent's own network

House rule, stated identically in all three plugins' SKILL.md: an agent
dispatched to draft with these skills never treats its own live network
access as a source of truth for a norm's current text. Verification runs as
a script **the user executes on his own machine**, or a documented manual
navigation route.

The concrete, counterintuitive finding behind this rule, from this project's
own build history:

- `publication.pravo.gov.ru` (`legal-ru`'s primary source) was **unreachable
  from every agent sandbox tried** — Claude's WebFetch, Bash/curl, and
  Codex's own network-enabled `os-sandbox` all timed out on every path,
  reading as a geo-block on non-Russian egress IPs (`legal-ru/docs/BRIEF.md`,
  "The lookup mechanism"). It was confirmed reachable only from the project
  owner's own browser/network.
- `rst.gov.ru` and `eaeunion.org` (`gost-ed-mashiny`'s sources) were **not**
  blocked — reachable directly during development, to the point that the
  open-data CSV catalogues were actually downloaded and read while building
  `gost-ed-mashiny/references/norms-registry.md` (see its "Что не
  проверено" section).

**The lesson is not "agent sandboxes can't reach .gov domains" — that would
be wrong.** The two outcomes sit side by side in this same project. The
actual lesson: reachability must be tested per source, never assumed either
way in either direction, and a lookup script must still be written to run
standalone on the user's own machine regardless of what happened to work (or
not) during development — because a working-today sandbox path is not a
guarantee for the user's own later run, and an unreachable-today sandbox path
does not mean the user's machine will also fail.

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

## 7. What this document is for

This is **style guidance for whoever adds the next plugin to this
family** — not a schema to validate against, and not code to import. There
is no shared library and none is planned: read this document, then write
your new plugin's own `references/norms-registry*.md` and its own lookup
script (or documented navigation route) following the same shape, entirely
within that new plugin's own repository.
