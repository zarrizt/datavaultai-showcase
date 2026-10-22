Voici l'intégralité du rapport de développement public n° 1 de DataVaultAI, dans sa version anglaise originale, directement dans le chat.

Je conserve son contenu et sa structure, sans résumé ni réécriture. Il s'agit toujours d'une version prépublication, en attente de ta validation.

# DataVaultAI: Development Report No. 1

## Architecture, Engineering Methodology, and the Limits of AI-Assisted Development

Document status: Public-report draft, awaiting owner approval Reporting date: October 21, 2026 Project: DataVaultAI (provisional product name: Scrollpendium) Author: Zarrizt Development model: Independent, self-taught, human-directed and AI-assisted Source availability: Development repository currently private; no public software release yet

> A note on evidence: This report is based on the project's internal architecture, status, implementation guidance, and development records. The source code and underlying test artifacts are not yet public, so external readers cannot independently reproduce the reported results today. Documented implementation, laboratory experiments, planned features, and research ambitions are distinguished throughout. This is a development report, not a security certification or release announcement.

## 1. Why I am building this

I started DataVaultAI because the problem interested me long before I knew how to solve it.

A research collection can contain hundreds of documents, incompatible file formats, imperfect scans, conflicting editions, personal notes, and fragments of information that are difficult to trace back to an original source. Finding a passage is one challenge. Understanding where it came from, whether it was extracted correctly, and what a later interpretation added to it is another.

I wanted a local tool that could preserve documents, reconstruct useful text, search it, and connect my notes to evidence without requiring me to surrender either my data or my judgment to a remote service.

But DataVaultAI is also a second experiment: what happens when a self-taught amateur uses increasingly capable AI tools to undertake software development beyond their original technical expertise? Can that process produce something maintainable and verifiable without turning the human participant into a passive approver of machine-generated answers?

I do not think the answer is simply let the model code.

My working hypothesis is that greater implementation capacity makes methodology, explicit human authority, and independent verification more important, not less.

## 2. Status at this checkpoint

The private development branch has completed and merged Milestones 0 through 5. M6.1 and M6.2 have also been integrated, establishing persistence structures and source-revision/invalidation safeguards for future semantic functionality.

The current documented database schema is SQLite schema version 8.

The latest full application regression recorded in the internal project status reports 675 passed, 0 failed, and 3 skipped, using the qualified Windows x64 CPython 3.14.7 and SQLite 3.54.1 configuration. The skipped tests concern opt-in real Tesseract execution. This is a documented test result for that configuration, not proof of production readiness, general platform portability, or correctness under every hostile workload.

Already implemented:

- Local document import for PDF, EPUB, DOCX, ODT, and UTF-8 TXT.
- Preservation of managed original files and SHA-256-based duplicate detection.
- Native PDF extraction, explainable text-quality checks, and selective local OCR.
- Multiple recorded OCR candidates, evidence-aware selection, and human review history.
- A shared document representation for structured formats, their sections, locators, and source units.
- SQLite-backed archival records and FTS5 lexical search.
- Human-authored Markdown notes, optional projects, and explicit note-to-source relationships.
- Controlled database migrations, transactional consistency checks, and a local browser interface.
- Semantic generation, manifest, and segment storage foundations, plus source-revision guards and transactional invalidation.

Not yet implemented as production functionality:

- A real embedding producer and semantic index construction workflow.
- Semantic or hybrid retrieval.
- Generative question answering or a local LLM interface.
- Web ingestion and external corroboration.
- Multi-vault management, an authentication system, encryption profiles, or a plugin framework.

Milestone 6.3 remains experimental and is not authorized for production implementation at this checkpoint. The experimental results should not be confused with deployed capabilities.

## 3. The principles that constrain the design

The project is not driven by a feature checklist alone. I use a set of constraints that can reject attractive ideas even when an AI assistant can implement them quickly.

### 3.1 The original source remains authoritative

A document's original bytes are preserved. Text extraction, normalization, OCR candidates, human corrections, indexes, and future AI interpretations are different layers with different evidentiary roles.

The desired provenance path for future grounded answers is:

```
Answer or interpretation
    -> retrieved passage
    -> derived segment or source unit
    -> page / section / locator
    -> document
    -> preserved original
```

That final answer layer does not exist yet. The source and evidence foundations do.

### 3.2 Deterministic before probabilistic

Hashing, database constraints, identity checks, file-format parsing, and transaction handling should not depend on a language model where ordinary software can solve the problem predictably.

AI can be useful for proposing explanations, helping investigate ambiguity, or assisting development. It should not become the authority on whether a file was preserved, a transaction committed, or an extraction matches its source.

### 3.3 Local first, with independent capabilities

The archive is designed to work locally without a cloud account, remote AI service, or network connection. Its useful core must remain operational when future semantic models or other optional components are missing.

AI capabilities should be replaceable extensions, not the canonical memory of the archive.

### 3.4 Preserve alternatives when certainty is unavailable

An OCR system can produce two plausible readings. Two publications can disagree. Two AI systems can repeat the same unsupported claim.

Those situations do not automatically justify a single canonical answer. The system should preserve evidence, expose uncertainty, and require explicit human confirmation when a decision would otherwise erase a meaningful alternative.

### 3.5 Favor understandable and reversible architecture

I prefer explicit Python, modest modules, typed data structures, ordinary SQL, and controlled boundaries over a speculative forest of abstraction layers.

When uncertain, I generally prefer the smallest reversible approach that preserves authoritative data and allows a future revision without pretending the design has already solved every problem.

## 4. The architecture that exists today

The application is a single-process, local FastAPI service with server-rendered Jinja2 pages, Python processing modules, and SQLite/FTS5 persistence. It does not use an ORM, distributed worker system, remote backend, or mandatory AI runtime.

A simplified view:

```
                        LOCAL USER
                            |
                   Browser / local UI
                            |
                  FastAPI application
                            |
           +----------------+----------------+
           |                |                |
       Archive &        Human notes      Search and
       import flow      and projects     review views
           |                |                |
           v                v                v
    Format adapters     Note/source      SQLite FTS5
           |             relationships     and evidence
           v                |                |
    Shared document          +--------+-------+
    representation                    |
           |                          v
           +--------------------> SQLite v8
           |                          |
           v                          v
    Original files             Derived records,
    retained locally           provenance and history

          [Future optional semantic and AI functions]
          [Not an operational application pipeline]
```

### 4.1 Ingestion and source identity

An import begins with a deliberate scan of the local inbox. The application validates the input and format, copies the original bytes into a managed local name, calculates its hash, and checks for duplicate content.

The document's display name is not used as an authority for filesystem placement. Imported originals are retained rather than silently rewritten. Publication of database records happens transactionally, with a second duplicate check under a write lock.

There is an important limitation: filesystem operations and SQLite transactions are not one cross-resource atomic transaction. A sudden crash may leave an unreferenced managed file. That risk is documented; a complete automatic orphan-recovery workflow does not yet exist.

### 4.2 A common document representation

PDF pages, TXT documents, EPUB chapters, DOCX paragraphs, and ODT structures are not identical objects. The architecture therefore uses format-specific adapters that converge on an application-owned intermediate representation (IR).

That representation describes documents, ordered source units, sections, locators, text evidence, warnings, and selected structural metadata. Downstream search, persistence, review, and navigation depend on this shared representation rather than on the internal objects of each document library.

This boundary is intentional: adding a new format should not require rebuilding the entire search or note system. It also prevents third-party parsers from quietly defining the application's understanding of evidence.

### 4.3 Three text layers and a human review path

The text pipeline distinguishes:

- Raw text: what the source extractor actually produced.
- Normalized text: mechanical cleanup intended to improve usability without erasing raw evidence.
- Canonical text: a selected or explicitly corrected reading, with provenance and review status.

For PDFs, text-quality heuristics may favor native extraction, selectively request OCR, or propose a reading for human review. Multiple candidates can be preserved and compared rather than spliced into an undocumented hybrid.

A material disagreement between usable readings is not resolved merely because one OCR candidate received a better heuristic score or multiple passes agreed. The application can require human confirmation, preserve review history, and distinguish accepted/corrected evidence from unreviewed suggestions.

These confidence labels are heuristics, not calibrated probabilities of factual correctness.

### 4.4 Structured documents without invented pagination

EPUB, DOCX, and ODT adapters preserve useful reading order, section and heading relationships, navigation where available, and practical table structures. They do not invent physical page numbers for formats that lack stable physical pagination.

A PDF locator can identify a physical page; a structured document locator identifies a meaningful package element or document unit. Where extraction is incomplete or a complex object cannot be faithfully interpreted, warnings should be visible rather than suppressed.

### 4.5 SQLite as durable state, FTS5 as the lexical layer

SQLite stores the archive, source units, extraction evidence, review history, notes, project associations, and derived semantic foundations. FTS5 maintains lexical indexes over effective source readings and, separately, human-authored notes.

This separation matters. A note linked to a source is not automatically proof that the source supports every claim in the note. Nor should authored commentary silently become trusted OCR vocabulary or source evidence.

Migrations are explicit, transactional, and versioned. The application refuses incompatible newer schemas, and downgrades are not implemented. An earlier migration-compatibility edge case is documented rather than handled by resetting valuable records.

### 4.6 Current trust and security boundaries

The qualified launcher checks a specified Windows Python/SQLite runtime before application imports. The application is documented for local loopback access, not remote deployment. There is no user authentication or full-vault encryption today.

Imported packages undergo bounded validation; XML parsing is constrained; untrusted source markup is not rendered as executable HTML. HTTP origin/host protections reduce exposure but do not constitute complete browser, OS, or network isolation.

Parser allocation, OCR processing, and third-party libraries still execute within an ordinary local process. Resource budgets and input checks reduce risk; they are not an operating-system sandbox.

## 5. My complete working methodology

I use the following sequence as a development discipline, not as a ritual that makes mistakes impossible.

### Step 1: Describe the problem before imagining the solution

I start with a concrete failure, user need, or question. What must become possible? What currently fails? What would count as an unacceptable outcome?

I try to separate the user's desired capability from an attractive implementation idea. An embedding model, a database table, or an agent is a possible mechanism, not a problem statement.

### Step 2: Establish the sources of authority

Before proposing code, the agent or I inspect the current repository state, the authorized milestone, architecture documentation, prior decisions, relevant source files, and tests.

Documents have distinct roles:

| Record                    | Purpose                                                               |
| ------------------------- | --------------------------------------------------------------------- |
| `MASTER_PROMPT.md`        | Product principles, scope, constraints, milestone contracts           |
| `STATUS.md`               | Current implementation, limitations, tested baseline, authorized work |
| `ARCHITECTURE.md`         | Components and behavior that actually exist                           |
| Source code and tests     | Executable implementation and observable assertions                   |
| Git history               | Traceable technical changes and review context                        |
| `PROJECT_PHILOSOPHY.md`   | Human-AI collaboration, authorship, free software, funding philosophy |
| Future architecture notes | Possible directions, not permission to implement them                 |

An AI model's recollection of a previous conversation does not supersede the repository. A plan does not become shipped functionality because it appears in a document.

### Step 3: State constraints, non-goals, and invariants

I define what must never be lost, corrupted, fabricated, or silently reclassified. For example:

- Original source bytes must remain preserved.
- A source locator must not point to a different imported document after identity changes.
- A stale write must not overwrite a later human correction.
- Optional AI failures must not disable lexical archival search.
- A failed experiment must not quietly become a successful result after its evidence is rewritten.

I also state what is out of scope. This guards against the tendency of coding agents to solve the immediate problem by expanding the application into several new subsystems.

### Step 4: Generate and criticize architectural hypotheses

I use AI dialogue as a place to propose several plausible approaches, articulate tradeoffs, and challenge assumptions. I ask what each strategy costs in complexity, data migration, performance, portability, recovery, and future maintainability.

My preferred sequence is:

```
Understand the problem
       -> formulate hypotheses
       -> compare strategies and risks
       -> choose a justified approach
       -> specify acceptance criteria
       -> only then implement
```

Where I lack technical knowledge, the answer is not to accept the model's confidence. It is to make the assumptions explicit enough to investigate.

### Step 5: Convert the decision into a bounded contract

Before implementation, I want an explicit statement of:

1. The currently authorized scope.
2. Required observable behavior.
3. Failure behavior and preservation guarantees.
4. Relevant source-of-truth documents.
5. Test fixtures and independent checks.
6. Deliverables, evidence, and a mandatory stop gate.

The person directing the project may authorize a narrowly bounded experiment without authorizing the full feature. This distinction is particularly important when an experimental writer operates on disposable databases rather than production code.

### Step 6: Implement in the smallest coherent increment

Agents can write code, propose refactoring, debug failures, and run tests when they have access. I still require them to stay inside the contract, inspect existing work first, and avoid unnecessary dependencies or speculative infrastructure.

A human remains responsible for product direction, scope expansion, architectural decisions, acceptance criteria, and final approval. This is not a claim that I personally type every line of code.

### Step 7: Test both expected behavior and meaningful failures

A passing happy-path test is necessary but insufficient. I want tests that exercise stale identities, malformed inputs, conflicting evidence, transaction rollback, concurrency, cancellation, resource limits, and failure recovery where relevant.

I distinguish three things:

- Product correctness: does the implementation preserve its contract?
- Test-harness correctness: can the test itself recognize a deliberately defective implementation?
- Evidence quality: can someone determine what was actually exercised and what remains untested?

Negative controls are particularly important: a test oracle that accepts a deliberately invalid result has not earned our trust merely because the rest of the suite reports green.

### Step 8: Prefer independent oracles over self-confirmation

Whenever practical, expected results should be derived independently of the code being tested. If a producer computes a segment layout, a validator that simply trusts that same layout is not an independent check of the declared recipe.

Tests should distinguish structural coherence from compliance with an external specification. A correct hash of the wrong result is still the hash of the wrong result.

Similarly, two AI agents agreeing does not establish truth if both are reasoning from the same unchecked premise or repeating the same mistake.

### Step 9: Preserve counterexamples and classify failures

When a test fails, the first failure is evidence. I want its fixture, observations, commands, and relevant state preserved before anyone repairs the harness or implementation.

A failure should be classified carefully. Was the architecture falsified? Did existing production code behave incorrectly? Did the experimental harness mismeasure the system? Or was the result inconclusive because the required interleaving never occurred?

A test that never exercised its claimed fault cannot be counted as a pass.

### Step 10: Apply explicit human acceptance gates

The project advances through named milestones and checkpoints. A successful agent run does not authorize the next milestone. A report marked ready for review is not equivalent to a human acceptance decision, and a human-approved design does not automatically authorize production implementation.

This separation is intentional. It slows down some work, but it prevents a confident assistant from quietly changing the project that I actually chose to build.

### Step 11: Reconcile documentation with reality

Code changes should trigger a documentation impact review. Status, architecture, usage instructions, limitations, and relevant technical records must be revised where behavior actually changed.

Historical failures stay historical. Old verdicts do not become PASS because a later correction worked. Documentation should be a record of what happened, not a promotional rewrite of it.

### Step 12: Learn from the work rather than outsourcing learning

This project is also my Python education. I use worked cases, critical questions, and explanation of execution flow to develop understanding before delegating further work.

I do not need to memorize the internals of every dependency. But I do need a usable mental model of where data enters, where it changes, which component has authority, when a transaction commits, and what happens if something fails.

Delegation should extend my reach without making my own understanding optional.

## 6. A recent case study: the oracle that trusted the wrong thing

During the laboratory-only investigation of future semantic publication, a deliberately defective synthetic candidate generated segments of the wrong width while maintaining internally consistent vectors and a digest.

The experimental validator initially accepted that representation. It verified coverage and internal integrity but did not independently establish that segment boundaries matched the declared segmentation recipe.

This was classified as a test-harness defect, not evidence of a deployed production vulnerability: the production embedding writer does not yet exist.

The laboratory oracle was revised to reconstruct expected boundaries independently, both before attempted publication and before reuse of stored work. The correction was exercised using negative controls and cases involving empty text, short final segments, Unicode boundaries, accents, and whitespace. Retained historical evidence was reassessed; the original failure remained preserved.

The important lesson was not the width of a synthetic chunk. It was the distinction between:

1. Internal consistency: the pieces fit together according to the candidate's own story.
2. Specification conformance: the pieces correspond to the contract that was supposed to govern them.

The correction was reported successful in the internal laboratory checkpoint. The complete evidence package has not yet been publicly released, and I do not present this as external audit or broad semantic-search qualification.

As of this report, subsequent Block 2 synthetic experiments covering freshness, delete/reimport identity, concurrency, corruption, rescan, and recipe separation had been authorized to resume. Their completion and outcomes are not claimed here.

## 7. M6: designing semantic capabilities without sacrificing the archive

The future local semantic layer creates specific failure risks. An embedding is derived data: it depends on exact effective text, source identity, a generation, a model, preprocessing and segmentation rules, and a representation recipe.

If any relevant input changes, an apparently valid vector may become stale or incompatible. If half of a document's segments commit and half do not, a later search could confuse incompleteness with a valid result.

M6.1 and M6.2 already provide persistent schema foundations, revision tracking, and transactional invalidation. The remaining work concerns whether a producer can safely publish complete, recipe-consistent units and recover from interruption without introducing misleading state.

The current experimental direction explores foreground, bounded processing, conservative complete-unit publication, revalidation at the commit boundary, and recoverable rescans. Those are testable hypotheses, not a declaration that the design has passed all relevant experiments or entered production.

The governing question is not merely can I produce vectors?

It is can I prove that the vectors still represent the intended source and recipe, and can the rest of the archive survive when the semantic capability fails?

## 8. What verification means, and what it does not

A passing regression suite is evidence for specified cases on a specified runtime. It is not a substitute for user trials, broad adversarial testing, real-world corpus diversity, crash durability qualification, or a public-release review.

The project uses several kinds of verification as appropriate:

| Verification level                      | Intended question                                                   |
| --------------------------------------- | ------------------------------------------------------------------- |
| Unit and regression tests               | Does known behavior remain correct?                                 |
| Integration tests                       | Do modules, migrations, database state, and local interfaces agree? |
| Adversarial cases                       | Can malformed inputs or conflicting state violate an invariant?     |
| Negative controls                       | Will the oracle reject a deliberately defective result?             |
| Controlled concurrency and interruption | Does the claimed ordering actually hold?                            |
| Runtime qualification                   | Which Python/SQLite/environment combination was exercised?          |
| Human review                            | Are the scope, residual risks, and evidence acceptable?             |
| Future release benchmarks               | Under what workloads and hardware does it remain useful?            |

I also record deliberate limitations. Local-only operation is not the same as encryption. A virtual environment is not an operating-system sandbox. An AI reviewer is not automatically independent evidence. A synthetic embedding test is not a real-model benchmark.

These distinctions matter more than a large number of green checkmarks.

## 9. Security and privacy boundaries

My long-term goal is an archive whose ordinary use does not require cloud services, telemetry, remote accounts, or disclosure of user documents. Core archival functionality presently runs locally; this does not imply that every proposed future privacy or security feature has been implemented.

The current trust assumptions remain important:

- Access to local files and the operating system must be controlled by the user and host environment.
- The application is designed for loopback, not deployment as a public web service.
- Imported files are untrusted input and can stress complex parsers.
- Human-authored notes are escaped rather than rendered as trusted executable HTML.
- Source text, including any future adversarial instructions embedded in documents, is data, not authority over an AI agent.
- There is no implemented login system, full-vault encryption, or proven process-level isolation.

Security testing is an ongoing obligation. I do not claim comprehensive resistance to prompt injection, malicious files, or local host compromise. Those claims would require much more evidence than the current project has produced.

## 10. Decisions I have deliberately deferred

One persistent temptation in AI-assisted development is to build the entire imagined future in advance. DataVaultAI intentionally avoids that.

The following remain possibilities rather than present capabilities: multiple independent vaults, configurable security profiles, encrypted vaults, model/provider replacement infrastructure, broader agents and methodology engines, advanced acceleration, distributed execution, and a dynamic plugin ecosystem.

Their future existence should not be inferred from diagrams or planning documents. Each must earn its place through a specific need, a bounded design, verified behavior, and a human decision.

The principle is simple: a boundary can be worth preserving long before the system on the other side of it is worth building.

## 11. Free software, authorship, and sustainability

DataVaultAI's development code is licensed AGPL-3.0-only, and a public source release is intended. The private development status means the public cannot yet inspect or obtain the application through this showcase.

I intend to pursue a voluntary Pay What You Want model. Contributions, if and when enabled, would primarily support me as an independent developer: development and maintenance time, self-directed education, tools, research, and related work such as Exocortex. They would not automatically be restricted to a separate project treasury.

The software's ordinary freely published capabilities should not depend on payment. Financial contributions would not buy architectural authority, priority over other users, or private control over the project's direction. No payment or sponsorship mechanism is currently operational.

The broader research question matters to me as much as the licensing model: if AI makes implementation more accessible, how do we retain meaningful human responsibility for conception, selection, criticism, and maintenance? What should count as authorship when those activities are distributed between human direction and machine assistance?

I do not claim to have settled those questions. I would rather keep the process legible enough for others to examine them.

## 12. What comes next

The next engineering priorities are to finish evaluating the M6.3 synthetic architecture, establish the limits of its evidence, and decide whether it is ready for subsequent controlled tests. A successful laboratory experiment would still not authorize deploying an embedding producer or declaring semantic search operational.

Beyond that, the intended sequence includes local semantic retrieval, optional local AI, and eventual release qualification with explicit correctness, security, capability-independence, performance, and workload limits. Those are roadmap intentions, not completion forecasts or release promises.

The immediate public priority is equally important: to provide clearer explanations of completed milestones, publish responsibly sanitized development evidence where possible, and make the project easier to scrutinize before asking anyone to trust it or support it financially.

## Closing reflection

I started this work because I wanted a tool that did not exist in the form I needed.

I continue because building it has become an experiment in intellectual autonomy: in learning the language of a discipline while using technology that can perform parts of that discipline far faster than I can.

I don't think AI-assisted development needs to be either a miracle or a fraud. It can be a demanding collaboration, provided the human involved remains willing to question the results, preserve the evidence, and accept responsibility when the system is wrong.

The vibe starts the project. Understanding and methodology allow it to endure.

Report scope and provenance

- Prepared October 21, 2026, from internal project documentation current at that checkpoint, including `STATUS.md`, `ARCHITECTURE.md`, `docs/MASTER_PROMPT.md`, `docs/PROJECT_PHILOSOPHY.md`, and the private development README.
- Internal latest recorded full regression: 675 passed, 0 failed, 3 skipped on the qualified Windows/Python/SQLite combination. Not independently reproducible by outside readers until relevant code and fixtures are released.
- M6.3 laboratory case-study outcomes are based on the development checkpoint supplied by the project owner. Unreleased evidence is not represented as public independent verification.
- No private archive, experimental database, source-document content, credentials, or local evidence paths are included in this report.
