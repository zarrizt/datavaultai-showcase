# DataVaultAI: Development Report No. 3

## From Local Embeddings to Integrated Semantic Search

**Reporting date:** November 7, 2026  
**Document status:** Development checkpoint, Milestone 6 in progress  
**Project:** DataVaultAI (provisional product name: Scrollpendium)  
**Author:** Zarrizt  
**Development model:** Independent, self-taught, human-directed and AI-assisted  
**Source availability:** Private development repository; no public software release yet

> The implementation and test results described here come from a private development repository. Many of the tests were run in late October. Because the source code and raw test records are not yet public, readers cannot independently reproduce these results from the showcase. DataVaultAI has not been released or certified for production use.

## 1. Why this update matters

On October 26, I reported that DataVaultAI could segment source text and produce real multilingual embeddings using a local CPU encoder, but could not yet turn those vectors into a complete application search experience.

That distinction was important. Computing a vector is not the same as maintaining an index, validating its relationship to an evolving source archive or returning useful results to a researcher.

Since then, several pieces have come together.

DataVaultAI now has implemented paths for bounded semantic indexing, interruption recovery, explicit generation selection and rollback, and browser-based lexical, semantic and hybrid search.

The application has therefore moved beyond isolated semantic components toward an integrated, locally operated retrieval workflow.

**That is real progress, though Milestone 6 is not finished.**

## 2. What has changed since October 26?

| Capability | Documented status as of November 7 |
| --- | --- |
| Multi-format archive, OCR, provenance, notes and FTS5 | Implemented |
| Persistent semantic storage and source invalidation | Implemented |
| Deterministic text segmentation | Implemented |
| Optional offline multilingual E5 encoder | Implemented |
| Sequential embedding producer | Implemented with bounded publication |
| Durable indexing, explicit resume and retry | Implemented |
| Generation coverage assessment | Implemented |
| Atomic generation selection and rollback | Implemented |
| Browser lexical, semantic and hybrid search | Implemented |
| Complete end-to-end M6 testing | Still in progress |
| Conversational answering and RAG orchestration | Not implemented |
| Public software release | Not available |

The SQLite schema has advanced to version 10, with additive support for durable indexing and explicit generation selection.

These features exist in the private development build, but I have not finished testing how reliably they behave across realistically sized document libraries and different environments.

## 3. A working semantic retrieval path

A semantic search system has to preserve more than numerical similarity.

The current implementation connects original documents, effective source text, deterministic segments, optional local embeddings, generation-specific storage and validated retrieval.

A simplified flow is:

```text
Preserved original document
          |
   Source units and locators
          |
  Effective, revisioned text
          |
  Deterministic segmentation
          |
 Optional local E5 encoding
          |
  Derived vector generation
          |
  Explicit indexing and resume
          |
 Generation assessment and selection
          |
 Lexical / Semantic / Hybrid search
          |
 Results linked to source evidence
```

The key design principle remains unchanged: **the archive is authoritative; embeddings are derived representations.**

An embedding does not rewrite the original document. A search score does not determine whether a statement is true. The researcher can still return to the source and its recorded reading.

### From encoded vectors to persisted representations

The sequential producer connects source discovery, the existing segmentation policy, the verified E5 adapter and SQLite persistence.

In a focused test, the real encoder processed three synthetic source units. The test covered initial persistence, reuse of a valid stored representation without recomputing it, and rebuilding that representation after a source correction.

It showed that these components can work together. It did not tell me how the system would perform on a large library or how much computing capacity it would need in everyday use.

### Indexing that can be resumed

A new indexing layer records generation-specific progress and unit outcomes.

Work can be resumed explicitly after interruption. Completed compatible representations may be reused, while failed or excluded units remain visible rather than disappearing behind a progress cursor.

A recorded "complete" state is not enough on its own: the application must still check whether the source text has changed and whether the stored representation remains valid.

For now, indexing is started manually and works in limited batches. It does not yet run as a background service.

## 4. Choosing which semantic generation to use

A research archive changes.

A source may be corrected. A document may be deleted and imported again. A model or preprocessing recipe may be replaced.

Those changes create a difficult question: when is a semantic index still an appropriate representation of the current archive?

The new schema records an explicit selected generation and a retained rollback candidate.

Selection is validated against current source and representation state. Concurrent changes are fenced so that a generation cannot be accepted solely because it looked complete before a conflicting update.

The application also supports an explicit lexical-only state.

This means that semantic functionality is not an unavoidable condition for using DataVaultAI. The document archive and literal search remain available independently of a valid semantic generation or working model environment.

Controlled tests exercised selection and rollback when operations fail or overlap. Sudden power loss and failures deep inside native libraries remain outside what those tests established.

## 5. Three search modes in the browser

The application's existing local browser interface now provides distinct **Lexical**, **Semantic** and **Hybrid** search modes.

Lexical remains the default.

**Lexical search** uses the established SQLite FTS5 literal-phrase behavior. It is useful when a researcher knows a term, quotation or other distinctive wording.

**Semantic search** uses a selected, compatible E5 generation to rank validated source-unit representations by meaning-related similarity. Its results still expose source provenance, and its availability depends on the optional local model and the selected generation.

**Hybrid search** combines lexical and semantic rankings using deterministic reciprocal rank fusion. It retains information about the ranking branches that contributed to each result.

Hybrid search is not assumed to be better than either individual method. Controlled overlap tests include cases where fusion improves the ranking, makes no difference or makes it worse.

The interface also distinguishes a failed semantic execution from a successful lexical result. A hybrid request that falls back to lexical-only results is labelled accordingly rather than being silently reported as a successful hybrid search.

Importantly, the browser does not automatically download a model, start indexing or activate a generation when someone submits a search.

### A small but meaningful multilingual result

A focused real-E5 search qualification used a fixed seven-unit corpus with conceptual and bilingual examples. Expected relevant units ranked first in those controlled cases.

That is encouraging for a multilingual research tool, particularly one intended to help when a query does not share exact vocabulary with a source.

It does not establish general retrieval accuracy, robustness across arbitrary writing styles or superiority to lexical search on a realistic library.

## 6. What the internal tests actually establish

Several substantial regressions were recorded during the late-October integration work.

| Qualification checkpoint | Recorded result |
| --- | --- |
| Durable indexing | 921 passed, 13 skipped, 37 subtests passed |
| Generation selection and schema v10 | 992 passed, 13 skipped, 37 subtests passed |
| Integrated browser search | 1,014 passed, 13 skipped, 37 subtests passed |
| Final focused application search checks | 50 passed |
| Generation selection and migration focused checks | 71 passed |

These are **separate runs at different checkpoints**, not additive test totals. They also contain overlapping tests.

The skipped cases include optional real-model and OCR execution paths. Real E5 integration was assessed separately in bounded native tests. Not every full regression repeated native inference.

The tests provide evidence for the examined configurations, including Windows x64, the controlled Python and SQLite environment, and the specific experimental or application scenarios exercised.

They do not establish supported performance across every computer, document collection or operating system.

## 7. Recoverability, provenance and honest failure reporting

One of the most useful lessons from the project has been that a system can produce a correct-looking answer for the wrong reason.

A successful test may conceal a defective assumption. A process may write durable data but fail before acknowledging its commit. An index may contain mathematically valid vectors tied to obsolete text.

The newer integration work addresses some of those problems explicitly.

Generation-specific indexing records interruptions and incomplete work. Selection uses transactional validation. Search checks current evidence within a consistent database snapshot. A missing or incompatible model does not invalidate the archival foundation.

Controlled tests also examine actual child-process exits around database commits and inspect persisted state using fresh connections.

These mechanisms strengthen the project's ability to distinguish what was attempted, what was observed and what can be safely concluded.

They are not substitutes for power-loss testing, operating-system isolation or full end-to-end resource qualification.

## 8. A separate investigation: dependency qualification

The last phase of development also included independent laboratory investigations into selected dependency and runtime candidates.

The experiments examined document-processing packages, Python runtime configurations, ONNX Runtime behavior and saved package metadata.

I kept the original lab reports and their associated files in a separate private evidence collection. Reviewing them together helped clarify what the records supported and which questions remained open.

None of these experiments led to changing the application's approved dependencies.

Some comparisons were successful within their tested conditions. Others remained inconclusive or lacked important reference material. Security reproduction, real OCR coverage, historical vector comparisons and broader runtime guarantees were not established by those limited experiments.

One later package investigation remained at the metadata-inspection stage, with further qualification pending.

The findings remain experimental. They are not a recommendation to use those alternatives in the application.

This distinction matters. Preserving a test report is not the same as being able to reproduce its environment, and reproducing a narrow result is not the same as proving a dependency suitable for deployment.

The original evidence remains private; this public summary explains the conclusions and their limits without presenting them as independently verified.

## 9. The development methodology is changing too

DataVaultAI remains an experiment in human-directed, AI-assisted software development.

I began with limited formal programming experience and an ambition to build a research tool whose scope exceeded my initial expertise.

AI assistance has made implementation and technical investigation more accessible. It has also exposed a methodological problem: producing code faster is not useful if I cannot explain the assumptions that hold it together or recognize when a confident answer is unsupported.

My working process increasingly follows a deliberate rhythm:

**Understand the problem → formulate hypotheses → compare approaches → choose a bounded strategy → test → examine the results → refine.**

I remain responsible for the project's goals, the risks I'm willing to take and the architectural decisions I approve. AI tools help me investigate alternatives, review code and test assumptions, but agreement between models isn't independent evidence.

Some of the most valuable moments have occurred when an apparently reasonable test was contradicted by actual execution.

An unsuccessful experiment can reveal a software defect, but it can also reveal an error in the reasoning or instrumentation used to assess the software.

I am interested in preserving that distinction because the development process is itself part of the research.

The broader question is whether human adaptability and machine-assisted formalization can become mutually corrective without weakening human judgment.

DataVaultAI is a practical case study for exploring that question, not proof that the hypothesis has already been validated.

## 10. What remains before Milestone 6 is complete?

The remaining challenge is less about adding another isolated component and more about qualifying the complete search product and its lifecycle.

Before calling M6 complete, I still need to test the entire search experience under realistic conditions: how well it finds relevant passages, what happens when indexing is interrupted, whether replacement and rollback work as expected, how incomplete coverage is shown, and what the system costs in memory, CPU time and storage. Offline operation, dependency integrity and failure isolation also need to be checked together.

Meaningful limits must be measured rather than inferred from synthetic test guards. Correctness in a small controlled corpus cannot automatically be extrapolated to a large personal library.

I am not treating the current build as a publicly supported release.

Conversational answers, retrieval-augmented generation and more autonomous functions are future ambitions, not features of the current application.

## 11. Closing reflection

On October 26, the important achievement was a real, locally executed multilingual encoder connected to source-preserving text segmentation.

By November 7, DataVaultAI has progressed to explicit semantic indexing, generation management and three browser search modes.

The project is closer to its original purpose: helping a researcher find relevant material without losing track of where that material came from.

But progress is not the same thing as completion.

I want the eventual application to be useful, inspectable and trustworthy precisely because it remains willing to say when evidence is incomplete.

The software has advanced. The methodology has advanced alongside it.

**The next milestone is not merely making semantic search work. It is establishing where, how and under which conditions it can be trusted.**

---

*DataVaultAI is independently developed by [Zarrizt](https://github.com/zarrizt). Its source remains private during development, with a public release planned. This report reflects the project's documented state as of November 7, 2026.*
