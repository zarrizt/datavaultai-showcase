# DataVaultAI: Development Report No. 2

## Milestone 6 Progress | Turning Source Preservation into Verifiable Local Embeddings

**Reporting date:** October 26, 2026  
**Document status:** Interim development report, Milestone 6 in progress  
**Project:** DataVaultAI (provisional product name: Scrollpendium)  
**Author:** Zarrizt  
**Development model:** Independent, self-taught, human-directed and AI-assisted  
**Source availability:** Private development repository; no public software release yet

> **Evidence note.** This report is based on private source code, merged pull requests, documented laboratory results and focused qualification records. The full source, historical fixtures and raw run artifacts are not currently public; readers cannot independently reproduce the numerical claims from this showcase alone. A component demonstrated in an isolated environment is not a finished application feature, and none of the results constitutes a security certification, production benchmark or release announcement.

## 1. Why a progress report before finishing the milestone?

My first report, published October 21, 2026, described the architecture of DataVaultAI and the methodology I have been learning to use while developing it. The emphasis was on understanding the problem, documenting constraints, deciding how to assess evidence and keeping the person directing the project responsible for its direction.

Five days later, Milestone 6 has changed meaningfully. We have progressed beyond database preparations and isolated semantic-model experiments to two merged implementation slices: deterministic segmentation of actual source text and an optional, offline multilingual encoder capable of producing real numerical vectors on a CPU.

That is enough to justify a detailed report, even though semantic search is not yet an available user feature.

Waiting until the milestone was finished would risk flattening a significant learning process into a binary outcome: "the feature shipped" or "the feature did not ship." The interesting decisions happened between those endpoints. Several tests failed, assumptions had to be corrected, and one genuine input limitation was left intentionally unresolved because pretending to support it would have compromised the meaning of the resulting data.

This report answers three questions: **What has actually been built? What do the tests allow me to say about it? What remains before a reader can search an archive by meaning?**

## 2. The problem: adding meaning without replacing evidence

An ordinary text search looks for words or phrases. That is valuable, especially when a document's terminology is known. But a research question often expresses an idea using different words from the relevant source.

In a tabletop role-playing library, I might remember that a particular power changes someone's memories while forgetting the power's official name. I might also encounter two similar abilities in different editions, one with a crucial restriction. A semantic representation could help find relevant material when the words in the question do not match the words in the book.

Yet a vector cannot tell me by itself whether the retrieved passage is authoritative, whether OCR got the words right, or whether I have confused two editions. Numerical similarity is not evidence that a rule is true.

I therefore want semantic functionality to remain downstream from the original archival record. The intended relationship is:

```text
Original document and preserved reading
            |
       Searchable source unit
            |
   Exact text + revision identity
            |
     Deterministic segmentation
            |
       Optional local encoder
            |
   Derived numerical representations
            |
    [Future, not implemented]
       Durable publication
            |
    [Future, not implemented]
     Semantic search results
            |
       Back to the source
```

The last two steps are deliberately marked as future work. The project can now perform the middle computational operations when explicitly invoked; it cannot yet offer the full pipeline as a working semantic-search experience.

## 3. What the archive already does

The new work should not obscure the archival foundation that makes it worth building.

DataVaultAI already supports ingestion of PDF, EPUB, DOCX, ODT and TXT files. It has local OCR for scanned PDFs, multiple extraction readings when appropriate, reviewable corrections, original-file preservation, provenance-linked source units, SQLite/FTS5 lexical search, and research notes organized into projects and linked to sources.

Those capabilities reflect an important architectural choice: AI must remain optional. A missing model must not prevent me from opening a book, reviewing an OCR correction, searching for a literal phrase, or reading a note linked to a source.

Milestone 6.1 added database structures for semantic generations, manifests and segments. Milestone 6.2 introduced source-revision tracking and automatic invalidation of stale derived records when the selected searchable reading changes. The database reached schema version 8.

Those foundations matter, but they are not the same as an indexing worker. A database table capable of storing embeddings does not mean the application can safely populate it, and a revision token does not by itself prove that a future writer will check it at the correct moment.

### A factual checkpoint table

| Development slice | Present state | What it does not establish |
| --- | --- | --- |
| M0–M5 | Merged archival application foundation | Public release or universal input security |
| M6.1 | Derived vector storage and generation metadata | Live encoder or searchable index |
| M6.2 | Revision tracking and transactional invalidation | Complete production publication protocol |
| M6.3-A | Versioned pure-Python text segmentation | Optimal packing, real-tokenizer support for all text |
| M6.3-B | Optional offline E5 tokenizer and original ONNX CPU encoder | A production semantic indexing worker |
| M6 producer and retrieval | **Not implemented** | No semantic/hybrid search UI or active generation |

M6.3-A and M6.3-B were merged into the private development branch through two reviewed pull requests. No additional external model backend or mandatory cloud service was introduced into the archival core.

## 4. First engineering step: the segmentation problem

Before turning a document into a numerical vector, I have to decide what exact text a vector represents.

A transformer encoder has a finite input sequence. The tested model's limit is 512 tokens, but those 512 positions include model-specific special tokens and the prefix used to indicate a passage. It would be easy to take the first 512 tokens of a long document and call the resulting vector a representation of the whole document. That would be convenient and profoundly misleading.

Instead, M6.3-A splits the original effective text into exact, ordered spans expressed as Unicode code-point offsets. Successful segmentation reconstructs the entire input in the same order with no missing characters, no overlaps, no rewriting and no hidden truncation.

The initial bounded policy permits text up to 64 KiB in UTF-8 and at most 16 segments, using deterministic halving and a small fixed number of refinement probes to find lengths that fit the tokenizer budget.

That policy is intentionally modest. It is designed so work cannot expand without a declared bound; it is **not** an optimal text-packing algorithm.

### Case study: a test that failed for the right reason

An early synthetic test assumed that 8,016 characters could be represented in exactly 16 segments of 501 characters each. In theory, this fit the synthetic tokenizer's capacity.

But the finite search sometimes selected slightly shorter segments. The final character range then required one segment too many, and the component returned an input-limit rejection. The policy had not lost data. It had refused to claim a complete representation where it could not satisfy its contract.

A more aggressive packing algorithm might succeed on that text. The project explicitly preserves this as a limitation and a regression case rather than quietly changing the test until it reports success. A separate 8,000-character fixture demonstrates a successful complete 16-segment result under the same policy.

This case changed how I think about correctness. There are at least three separate questions: Did the code stay within its promised budget? Did it preserve all of the text? Did it make the best possible use of the available capacity? The first two are current acceptance properties. The third remains an optimization question.

The pure component also has two validation perspectives. One checks source identity and the shape of the complete spans; another replays the declared policy rather than trusting the segmentation output's own counts. They share some low-level validation, so I do not call that a fully independent external audit.

## 5. Second engineering step: connecting a real local encoder

The M6.3-B slice introduces a real optional CPU encoder rather than merely synthetic token-counting.

The selected candidate is `intfloat/multilingual-e5-small`, pinned to an immutable model revision and the original ONNX export. It produces 384-dimensional embeddings. The adapter verifies the expected local model and tokenizer files, then loads them only when explicitly requested. It does not download weights, depend on a remote AI account or run whenever DataVaultAI starts.

The tested execution environment was Windows x64 with CPython 3.14.7, ONNX Runtime 1.31.0, tokenizers 0.23.3 and NumPy 2.5.4. It is a *tested configuration*, not a promise that every operating system or hardware combination has been qualified.

The adapter performs several steps that are easy to overlook in a simplified AI demo:

1. Verify the model and tokenizer bundle by expected file identities and hashes before loading it.
2. Apply the known tokenizer preprocessing policy with truncation disabled.
3. Account for the special tokens and `passage: ` prefix.
4. Preserve exact correspondence between source characters and token offsets where the tokenizer actually provides it.
5. Feed the expected tensor types and explicit attention masks into the original ONNX graph on CPU.
6. Pool the model output, normalize the vector, and serialize a 384-element little-endian float32 result.
7. Return the complete representation in memory only after every admitted segment has been encoded successfully.

The computed result still does not enter the SQLite semantic tables. Transactional publication belongs to a future slice.

### Why the precise encoder configuration matters

If two passes use different model revisions, tokenizer policies, input prefixes or segmentation parameters, the vectors might not be meaningfully interchangeable. A cosine-similarity score does not announce that hidden mismatch.

The implementation therefore records an execution identity that includes relevant model artifacts, native runtime observations, preprocessing settings and numerical rules.

The SHA-256 checks establish which local bytes were accepted. They do not magically certify the entire software supply chain, prevent every local replacement race, or give the vector the authority of the original passage.

## 6. A small scientific discipline: distinguish tests from claims

The M6.3-B results are meaningful because their scope can be stated precisely.

| Internal observation | Recorded result | What this means |
| --- | --- | --- |
| Historical tokenizer reference cases | **117/117 matching token-ID streams** | Agreement with recovered frozen token IDs, not proof of universal text support |
| Independent frozen reference-vector comparisons | **3**, maximum component difference **7.45 × 10⁻⁸** | Numerical agreement on three preserved examples only |
| Native CPU test groups | **8 passed** | Local targeted execution of the real model |
| Pure adapter tests | **19 passed** | Offline logic and negative controls |
| Segmentation regression tests | **34 passed** | Complete-span and policy behavior |
| Process-supervisor reporting tests | **12 passed** | Mocked failure/cleanup paths, not actual native crashes |
| Pure tests at the repaired checkpoint | **65 passed** | A focused combined count, not full application regression |

The last full-application regression separately documented for an earlier qualified application baseline was 675 passed and three skipped. That earlier result is not being passed off here as a new full-suite run after M6.3-B.

The recovered historical reference assets provided two useful checks: first, that the inputs belonged to the intended prior experiments; second, that the repaired preprocessing reproduced their token-ID sequences. The three vector comparisons used actual retained reference vectors. I would be overstating the result if I called those three measurements a 117-vector validation or described the suite as a general test of answer relevance.

During one real CPU qualification, model initialization was measured at 2.86 seconds, and the six short first inferences took roughly 6.74 to 13.32 milliseconds apiece. Sampled private memory rose to about 1.32 GB. Those are concrete observations from one host, not an upper memory bound, a minimum hardware requirement or expected performance for an entire library.

## 7. Three kinds of error and what they taught me

### 7.1 A compatibility check that assumed too much

An initial check required the tokenizer's declared 250,002 IDs to equal the model's 250,037 embedding rows. That premise was incorrect: an embedding matrix may contain additional positions beyond the tokenizer's declared vocabulary.

The first qualification stopped at initialization. Instead of treating this as proof that E5 was incompatible, the project corrected the check to validate the IDs that may actually be emitted against the model's valid range. The failed result remains part of the internal record.

The broader lesson is that **a strict check is not automatically a correct check**. Sometimes a well-intentioned invariant tests an invented requirement.

### 7.2 A mistaken assumption about native library loading

Another early gate expected a particular ONNX Runtime DLL to be loaded separately. The actual tested wheel executed through a verified native extension without requiring that separate DLL to appear as a loaded module.

Once again, the test's assumption was wrong, not necessarily the runtime. The implementation moved toward checking the actual loaded execution identity.

A test can accidentally become more restrictive than the genuine security or correctness requirement. I want to preserve that distinction.

### 7.3 An actual limitation: Unicode combining accents

Not every failure can be fixed by correcting a bad assertion.

The tokenizer may visually combine a base letter with a following accent. For one tested decomposed input, it returned offsets covering only the base letter, leaving the combining mark uncovered. A matching token-ID stream did not answer the harder question: *Can this vector still be traced honestly to every character of the original text?*

The adapter currently answers conservatively. It rejects those inputs for semantic encoding with an explicit unsupported-input outcome, while leaving the archived source untouched. Precomposed accented letters and several other tested Unicode inputs were accepted; this is not a claim of universal Unicode coverage.

The cost is significant: a source unit can be readable and valid yet receive no semantic representation because of one combining character. I am willing to document that exclusion rather than let a future search claim complete coverage it cannot demonstrate. A later solution must be evaluated separately and must not silently normalize the authoritative archive.

## 8. Testing the test harness, not only the application

A further defect appeared in the runner that collects native qualification results. If the model process stopped responding, the runner could attempt to kill it and then wait for its output. But if that second output-drain attempt timed out too, the runner could fail before recording its own supervisory report.

That would compromise the evidence of precisely the kind of failure the project most needs to understand.

The runner was therefore changed to distinguish three outcomes: process termination with complete output; process termination with incomplete output; and cleanup that could not be confirmed. Output snapshots are bounded, missing data is never invented, and a report from the child claiming success cannot override the supervisor's knowledge that a deadline expired.

Twelve pure tests use simulated process failures to exercise that bookkeeping, including double timeouts and failed cleanup calls. Those tests do **not** prove the implementation can safely terminate an actually hung native library under every Windows condition. Nor does a Python audit callback constitute an operating-system sandbox.

I consider this an important methodological advance. Reproducibility depends not only on the code under test, but on the honesty of the mechanism reporting what happened to it.

## 9. Why the existing laboratory results are not a product launch

Before these two implementation slices, I had explored retrieval quality and vector storage through isolated experiments.

Some of those results were encouraging. Others were important warnings. In a 28-query synthetic retrieval test, the vector approach found the expected result first in 23 cases. But the multilingual subset performed poorly, and distinctions involving negation, editions and exceptions remained difficult. Another particular experiment combining lexical and vector rankings did **worse** than the vector-only version.

That is useful evidence against the temptation to assume that hybrid search must always be an improvement.

Storage comparisons also suggested tradeoffs among ordinary SQLite BLOB vectors, a vector-search extension and duplicated representations. A dual approach consumed appreciably more disk in the isolated experiment. At this stage there is no reason to enlarge the initial producer with a second vector backend simply because one measured search path was faster in a synthetic benchmark.

Earlier Windows SQLite testing explored WAL, transactions and interruption in isolated databases. Later controlled experiments examined stale data, ownership and publication-order hypotheses. Their evidence guides design. It does not mean a real integrated producer has already been implemented or qualified.

## 10. What remains before a real semantic-search milestone

This is the central unfinished work, not an appendix.

**First: selecting work honestly.** A future producer must discover the source units that actually need embeddings, including units with missing or invalidated representations. It must not count only entries that already have manifests and then declare full coverage.

**Second: protecting authority at commit time.** Computing vectors takes time. During that interval, the user may correct OCR, change the selected reading, delete a document or reimport content. Before publishing any representation, the producer must prove that the source and declared recipe are still the intended ones, under a transaction that serializes the relevant mutations.

**Third: recovering from uncertainty.** A worker can die after SQLite commits but before receiving acknowledgment. A safe implementation cannot blindly assume success or blindly retry. It needs a way to reconcile durable state without inventing a clean historical sequence.

**Fourth: keeping failure contained.** Native model execution uses substantial resources. A missing model, an inference crash, a cancelled run or an exhausted resource budget must not disable ordinary archival import, lexical search, notes or review. Partial, skipped, rejected and genuinely complete outcomes must remain distinguishable.

**Fifth: making results meaningful to users.** Only after complete, current and compatible representations can be published should the application activate a semantic generation and implement semantic or hybrid search. A lexical-only result must never be mislabeled as semantic because the model happened to be missing.

None of these steps has been completed just because the E5 adapter can produce a vector today.

## 11. Methodology: I do not want AI to become my replacement for thinking

One reason this project exists is to learn Python and systems development while building something I genuinely want to use.

My role is not to pretend that I personally hand-wrote every line of the implementation. Coding agents can perform large parts of that labor. My responsibility is to understand the constraints, challenge assumptions, decide which tradeoffs are acceptable, insist on evidence, and stop when an operation goes beyond the scope I approved.

This milestone has made that relationship more concrete. When a test failed, the next question was not merely "How do we make it green?" It was "What property was the test supposed to establish, and did the test even measure that property?" When the tokenizer behaved unexpectedly, the answer was not to silently repair the source text. It was to distinguish information loss in the original archive from rejection of a derived computation.

I use a sequence that has become increasingly useful: understand the problem, form hypotheses, justify an approach, implement a bounded slice, test both ordinary and adverse cases, preserve failures, and reconcile the documentation with what actually happened.

The human-versus-AI division is not an argument that human judgments are infallible. It is an insistence that there be an accountable place where decisions are made, questioned and revisited.

## 12. Current verdict and next report

**Milestone 6 remains in progress.** Its storage foundations, source-invalidation safeguards, deterministic segmentation and real offline CPU encoder constitute a meaningful implementation advance. They do not yet constitute a semantic-search product.

The next report should follow a new observable boundary, not just the passage of time: ideally a guarded, explicitly invoked producer that can publish complete, source-current derived representations in a disposable or suitably controlled environment, with transparent outcomes under interruption and concurrent change. That work requires its own authorization and qualification.

I also intend to make appropriate development evidence easier to inspect when it can be shared without disclosing private source data or distributing unreviewed third-party model artifacts. Today, numerical results in this report remain internally documented rather than publicly reproducible.

I began this project wanting a better way to remember where knowledge comes from. This checkpoint has reinforced a related idea: I also need to remember **where my confidence in the software comes from**, and exactly where that confidence stops.

The original sources remain authoritative. The code has to earn every further claim.

---

### Reporting provenance and exclusions

This report was prepared October 26, 2026, using internal project status and architecture records, merged M6.3-A/B source and PR evidence, the documented M6 laboratory record and focused local qualification reports. It neither executed new tests nor inspected unreleased third-party model distribution rights. No private archive data, local paths, raw corpus fixtures, private GitHub source, model weights or internal evidence packages are included here. The previous Development Report No. 1 remains unchanged.
