# Delta: rule-based context-aware escalation engine

**Status:** proposed, 2026-11-09. Needs team and adviser approval before build.
**Source:** design session of 2026-11-09. The project title promises "Context-Aware Grievances Escalation"; the proposal (objective 1, lines 64, 76, 106, 156) and the 2026-09-30 consultation describe what it must do, but no document names a method. The team decided the method is a deterministic algorithm with **no LLM** and no external AI service. Proposal line 145 leaves NLP models to future researchers, which this keeps true.

Related: [ADR 0006](../../architecture/adr/0006-rule-based-context-aware-escalation.md) (why no LLM), [implementation plan](../../backend/docs/plans/2026-11-09-context-aware-escalation-plan.md), [paper text](../context-aware-escalation-paper.md).

## Existing behavior touched

- `backend/src/services.ts:702` decides a bypass when the student ticks "Is this concern about your adviser?", when `conflictedAdviser` or `involvedUserIds` name the adviser, or when the narrative contains the adviser's **full name** as an exact substring. Nicknames, surnames alone, typos, "sir J", "adviser namin" and subject references are missed.
- The bypass always goes to the Program Chair. Bypass above the chair is undefined ([sitemap §5](../sitemap.md), D-04).
- Similar-report groups are seeded (`CLUSTER-0101` in `backend/src/store.ts:216`). Nothing computes similarity. P23/P24 show a hard-coded threshold of 80 (`services.ts:372`) marked "not approved" (D-07).
- The post-to-report prompt on P06 (STU-08) has no agreement threshold (D-07).
- Category and severity are chosen only by the student (P19 step 2).

## Requested change

### 1. Meaning of "context-aware"

The engine reads the report text **together with the reporter's roster links**: their adviser, subject professors, Program Chair and Dean. The same sentence can route differently depending on who wrote it. The context is roster data, not a language model.

### 2. Pipeline (runs on submit, inside the API Lambda)

| Step | Job | Algorithm |
|---|---|---|
| 1 Normalize | Lowercase, strip punctuation, collapse repeated letters, Taglish stopword list, honorifics (sir, ma'am, maam, mam, prof, miss) mapped to one token | String rules |
| 2 Mention detection | Find which of the reporter's own faculty are mentioned | Roster gazetteer: exact multi-pattern match (Aho-Corasick or a token-trie) plus fuzzy match (Jaro-Winkler ≥ 0.85) for typos |
| 3 Target detection | Decide whether a mentioned person is the subject of the complaint | Proximity rule: mention within N tokens of a complaint-lexicon term |
| 4 Scope | Individual, section, program or college | Scope keyword lexicon |
| 5 Severity hint | Flag critical concerns | Weighted keyword lexicon; critical terms force `critical` |
| 6 Similarity | Group reports about the same issue | TF-IDF vectors, cosine similarity |
| 7 Agreement | Count support and dissent on a post or a group | Agreement lexicon on comments plus Agree reactions |
| 8 Routing | Pick the first office | Decision table on the existing stage machine |

### 3. Alias set per faculty member

Built from the roster at submit time, only for the reporter's linked faculty:

```
Prof. Juan Dela Cruz, adviser of BSIT 4K, teaches IT302 Networking
  "juan dela cruz", "dela cruz", "delacruz", "juan", "jdc",
  "<HON> dela cruz", "<HON> juan", "<HON> jdc",
  "adviser namin", "adviser ko", "aming adviser", "our adviser",
  "prof sa it302", "prof sa networking", "networking prof"
```

Faculty may add nicknames to their own alias list (People, P37). Role phrases ("adviser namin") resolve to the adviser without a name.

### 4. Conflict score

```
conflict(F) = match(F) × proximity × role
  match      1.0 exact alias, 0.85–0.99 fuzzy score
  proximity  1.0 if a complaint term is within 5 tokens, 0.5 within 12, 0.2 otherwise
  role       1.0 adviser or subject professor, 1.0 chair, 1.0 dean

conflict ≥ 0.60       → bypass F
0.30 ≤ conflict < 0.60 → route to Program Chair triage, flagged "possible conflict"
conflict < 0.30        → no conflict
```

The student's own "about my adviser" answer still forces a bypass (score treated as 1.0). The engine can add a bypass; it never removes one the student asked for.

### 5. Routing table

| Condition (first match wins) | First office |
|---|---|
| Severity hint `critical` (harassment, threat, self-harm, violence) | Guidance, as a Case File (**needs D-04**) |
| Student marked "about my adviser", or adviser/subject professor conflict ≥ 0.60 | Program Chair |
| Program Chair conflict ≥ 0.60 | Dean (**needs D-04**) |
| Dean conflict ≥ 0.60 | Office of the President (**needs D-04**) |
| Possible conflict 0.30–0.60 | Program Chair, with a "possible conflict" flag |
| Scope `program` or `college`, no conflict | Program Chair |
| Otherwise | Adviser |

Rows marked D-04 stay off (fall through to Program Chair) until D-04 is approved. The consent rule stays: a concern goes to the Dean only when the student allowed it or the routing row requires it after approval.

### 6. Similar-report groups

- Vectors: TF-IDF over normalized tokens of open reports in the same program.
- `similarity(a, b) = cosine(vec(a), vec(b))`, shown as a percentage on P23.
- A report joins a group when its similarity to the group centroid is ≥ the threshold (draft 0.80, D-07). Otherwise it seeds a new candidate group of one, which is not shown until a second report joins.
- Shared phrases on P24 are the highest-weighted overlapping terms.
- Supporting and dissenting lists come from step 7. The group stays a suggestion; the Program Chair confirms or dismisses it (CHR-07).

### 7. Post-to-report prompt (STU-08)

A post with the `rant` or `complaint` flair is scored with steps 1–5. When the complaint score passes the threshold and agreement (Agree reactions plus agreeing comments) reaches the D-07 value (draft: 10 users or 30% of the section, whichever is lower), the author sees the existing P06 prompt. Nothing becomes a report without the author starting it.

### 8. Explainability and audit

Every routing decision stores: engine version, lexicon version, matched alias, matched complaint term, scores, rule fired, final office. The handler sees this on P22 under the existing `ConflictNotice`. The student sees only the destination ("With the Program Chair"); the bypass reason stays hidden (P19).

## Success criteria

1. A labeled test set of at least 150 synthetic Taglish reports exists in `backend/tests/fixtures/escalation/`, with expected conflict, scope and route.
2. Conflict detection on that set: recall ≥ 0.95, precision ≥ 0.85.
3. Routing accuracy ≥ 0.90 on the same set.
4. The reports `"si sir jdc laging absent, buong section namin hindi naturuan"` (adviser is Juan Dela Cruz) and `"si sir delacrus late lagi"` (typo) both bypass the adviser.
5. A report that mentions the adviser only as a helper ("tinulungan ako ni sir jdc, pero yung registrar...") does **not** bypass.
6. Two reports with cosine ≥ 0.80 form a P23 group with the computed percentage. Reports below the threshold do not.
7. P23/P24 read the threshold from settings, not a constant.
8. No request leaves AWS for any of this. No new paid service.
9. Engine runtime per report under 200 ms in the Lambda for a program of 2,000 open reports.

## Out of scope

- Any LLM, embedding service or external AI API.
- Machine-learned classifiers. They can come later if the lexicon is not enough.
- Detecting indirect references with no name and no role word ("yung prof na laging late"). These fall to Program Chair triage or the student's own answer. The paper states this limitation.
- Deciding truth, offenses or outcomes (sitemap rule 6).

## Open decisions this delta needs

| Decision | Draft in this delta |
|---|---|
| D-04 Escalation and bypass authority | Chair conflict goes to Dean; Dean conflict to President; critical to Guidance |
| D-07 Similarity and agreement thresholds | Cosine 0.80; agreement 10 users or 30% of section |
| New: conflict thresholds | Bypass ≥ 0.60, triage 0.30–0.60 |
| New: lexicon ownership | Program Chairs and Guidance review the complaint and critical word lists each semester |
