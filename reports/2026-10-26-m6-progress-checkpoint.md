# DataVaultAI: Development Report No. 2

## Milestone 6 Progress Checkpoint | From Archival Foundations to a Local Embedding Prototype

**Reporting date:** October 26, 2026  
**Document status:** Intermediate development report  
**Project:** DataVaultAI (provisional product name: Scrollpendium)  
**Author:** Zarrizt  
**Development model:** Independent, self-taught, human-directed and AI-assisted  
**Source availability:** Private development repository; no public software release yet

> This report documents a completed intermediate checkpoint, not the completion of Milestone 6. Source code, internal test fixtures and full experimental reports are not publicly available at this time. Test counts and numerical results below are internal observations, not independently reproducible public benchmarks, security certifications or general performance guarantees.

## 1. Why publish before the end of Milestone 6?

The first report, dated October 21, described DataVaultAI's architecture, learning methodology and the experimental foundations of future semantic indexing.

Five days later, the project has reached a useful new boundary: it can now segment source text deterministically and, in an explicitly invoked optional path, use a real multilingual CPU model to compute embeddings.

This does **not** mean a user can search their archive semantically yet. It means the project has moved from preparing the conditions for semantic indexing to exercising the model component against those conditions. An intermediate report records that distinction while the evidence is fresh, rather than retroactively compressing every step into a final milestone announcement.

## 2. What exists today

The local archival foundation already supports imports from PDF, EPUB, DOCX, ODT and TXT, local OCR for scanned PDFs, preserved originals and extraction provenance, lexical search, human review, and research notes with source relationships.

Milestone 6.1 added persistent, derived semantic-storage foundations. Milestone 6.2 added source-revision and invalidation safeguards. Those mechanisms do not make embeddings authoritative and do not yet constitute a functioning semantic-search service.

Two further implementation slices have now been merged internally:

| Slice | What was implemented | What remains unclaimed |
| --- | --- | --- |
| M6.3-A | Versioned, deterministic text segmentation preserving full original-source character spans | No claim of optimal segmentation or coverage of every input |
| M6.3-B | Offline, explicitly invoked multilingual tokenizer and CPU ONNX encoder adapter, with pinned artifact identity checks | No semantic indexing worker, live retrieval or production release qualification |

The archival application remains useful without the optional AI runtime.

## 3. Making a machine-readable representation accountable

A numerical embedding is not a source document. It is a derived representation of specific text processed with a specific model and preprocessing policy.

The segmentation component therefore works with exact original-text positions, enforces a token budget that includes model prefixes and special tokens, and returns a complete sequence of spans or an explicit rejection. It never treats a successfully encoded opening fragment as complete coverage of a longer source.

Its algorithm is deliberately bounded and deterministic. Bounded search can reject some texts that a more expensive packing strategy might accommodate. This is documented as a coverage limitation, not concealed as a successful indexing outcome.

The encoder component uses a pinned version of `intfloat/multilingual-e5-small`, original ONNX and CPU-only inference. It runs only when explicitly requested and does not change archival startup or lexical search dependencies. The program verifies expected model files before loading and records execution identity so future representations can be tied to the process that produced them.

## 4. What the tests actually showed

Internal focused qualification reported:

- 117 matching historical tokenizer token-ID cases after correcting a discovered preprocessing mismatch.
- Three reference-vector comparisons with maximum reported per-component difference `7.45 × 10⁻⁸`.
- Eight passing groups of local CPU model tests.
- 65 passing pure-Python adapter, segmentation and process-reporting tests at the latest internal checkpoint.

These are narrow checks, not measures of semantic-search quality on a large real-world library. The model was not evaluated here as a full retrieval product. The last process-reporting fix used synthetic process failures and did not repeat native inference.

The experiments also recorded a practical deployment consideration: in one Windows CPU run, sampled private memory rose to approximately **1.32 GB**. That is a single-machine observation, not an upper limit or recommended minimum hardware specification.

## 5. An uncomfortable but useful Unicode finding

A particularly instructive failure concerned visually similar accented text.

Some characters can be represented as one precomposed Unicode code point or as a base character followed by a combining accent. For certain decomposed forms, the tested tokenizer returned offsets that did not cover every character of the original input.

The adapter now **rejects those inputs explicitly** rather than silently treating its offsets as complete or rewriting archived text to make the test pass.

That choice is conservative, but it carries a cost: an otherwise readable source unit may not receive semantic vectors. It is not a claim that all Unicode text is supported, and it is not a data-loss event in the authoritative archive. A future indexing feature must report such exclusions honestly.

This is the sort of limitation that matters more to a research tool than a polished demo might suggest.

## 6. Preserving failed tests as engineering evidence

Several early checks failed for reasons in the test or adaptation assumptions rather than because the selected model had been proven incompatible. One check incorrectly required the tokenizer's declared vocabulary size to equal the number of rows in the model embedding table. Another assumed a particular native runtime library had to be loaded separately.

Those assumptions were corrected, and the failures were retained as part of the internal historical record.

The qualification runner was also changed to distinguish a completed child process from incomplete output collection or unconfirmed termination. Synthetic tests exercise the reporting behavior. They do not establish a general operating-system sandbox or complete native crash containment.

For me, this is a methodological result as much as a programming result: a trustworthy PASS depends on asking the correct question, not merely producing green tests.

## 7. What has not been completed

Milestone 6 still needs a controlled producer that checks source identity and freshness before publishing complete derived vectors; durable publication and recovery rules; generation lifecycle and activation; and actual semantic and hybrid retrieval with source-grounded results.

There is no automatic semantic indexing or semantic-search interface to announce today. No source documents, model weights, private test corpora or native runtime packages are being published with this report. The private repository remains private.

Later qualification must test the integrated system, not assume independent component successes automatically establish end-to-end correctness.

## 8. What comes next

The next engineering challenge is no longer whether a pinned local model can produce a plausible vector. It is whether DataVaultAI can persist a complete vector representation **only for the correct current source and recipe**, survive interruption and concurrent changes, and tell the truth about partial coverage.

That work will be authorized and reviewed in bounded slices, just as the earlier implementation was.

**Checkpoint assessment:** significant implementation progress within Milestone 6; Milestone 6 itself remains **in progress**.

The guiding principle has not changed: preserve the source, make derived claims inspectable, and keep human judgment in charge.
