# Context-aware grievance escalation: paper-ready text

Proposed for the methodology chapter. Status: draft, 2026-11-09, pending team and adviser review. Specification: [`deltas/2026-11-09-context-aware-escalation-engine.md`](deltas/2026-11-09-context-aware-escalation-engine.md).

## Meaning of the term

In this study, *context-aware* means the system interprets a grievance using both the text of the report and the institutional context of the student who submitted it: the student's assigned adviser, subject professors, Program Chair, and Dean. The same sentence may therefore be routed differently depending on who wrote it. The context is drawn from the section roster and faculty assignments already stored in the platform.

## Method

The escalation engine is a deterministic, rule-based pipeline. It does not use large language models or any external artificial intelligence service, so every routing decision can be explained and reproduced, and no report text leaves the institution's hosting environment.

1. **Text normalization.** The report is converted to lowercase, punctuation is removed, repeated letters are collapsed, and English and Filipino stopwords are removed. Honorifics such as *sir*, *ma'am*, and *prof* are mapped to a single token so that "Sir Dela Cruz" and "Prof. Dela Cruz" are treated alike.
2. **Mention detection.** For each faculty member linked to the student, the system builds a set of aliases (full name, surname, first name, initials, honorific forms, role phrases such as "adviser namin", and subject references such as "prof sa networking"). The normalized text is searched with exact multi-pattern matching, and with the Jaro-Winkler similarity measure (threshold 0.85) to tolerate misspellings.
3. **Conflict scoring.** A mentioned faculty member is treated as the subject of the complaint when a term from a complaint lexicon appears near the mention. The conflict score is the product of the match score, a proximity weight, and a role weight. A score of 0.60 or higher causes the report to bypass that person; a score between 0.30 and 0.60 sends the report to the Program Chair with a "possible conflict" flag for human review.
4. **Scope and severity.** Keyword lexicons estimate whether the concern affects one student, a section, a program, or the whole college, and flag critical concerns such as harassment or threats.
5. **Routing.** A decision table applied to the escalation state machine selects the first office. The default route is the adviser; conflicts move the report to the next office that is not involved.
6. **Similar-report consolidation.** Reports are represented as TF-IDF vectors and compared with cosine similarity. Reports in the same program with a similarity of 80% or higher are grouped, and the group is presented to the Program Chair together with supporting and dissenting statements. The system does not decide whether any claim is true.

## Safeguards

- The student may state directly that a concern is about their adviser; this always bypasses the adviser, regardless of the engine's score.
- The engine suggests routing only. Program Chairs confirm or dismiss similar-report groups, and every decision on a case is made by an authorized person.
- Each routing decision records the matched name or phrase, the matched complaint term, the scores, and the rule applied, and this record is visible to the handler.

## Evaluation

The engine will be tested with at least 150 labeled synthetic reports written in English, Filipino, and Taglish. Conflict detection will be measured by precision and recall (targets 0.85 and 0.95), routing by accuracy (target 0.90), and consolidation by F1 score. These measures support the functional suitability characteristic of ISO/IEC 25010.

## Limitations

The engine cannot detect a conflict when the report neither names the person nor uses a role phrase, for example "the professor who is always late". Such reports rely on the student's own answer or on Program Chair triage. Similarity based on shared words may miss reports that describe the same issue in entirely different words. Lexicons require periodic review to keep up with student language.
