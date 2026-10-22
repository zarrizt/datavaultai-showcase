# DataVaultAI
### Scrollpendium | Provisional Product Name

**Built for evidence. Designed for resilience. Grounded in privacy. Open by principle. Modular by design. Free by commitment.**

DataVaultAI aims to make trustworthy, source-grounded research accessible to everyone, without mandatory cloud services, proprietary lock-in, or financial barriers.

*These are guiding commitments. Their technical guarantees will be qualified through documented testing and release review; not all planned protections or capabilities are implemented yet.*

**Preserve the sources. Organize knowledge. Retain intellectual autonomy.**

We live in an age of unprecedented access to information, yet maintaining a trustworthy relationship with our sources remains surprisingly difficult.

Scattered documents, incompatible formats, imperfect text extraction, and growing dependence on cloud-based services can turn the simple act of finding, verifying, and connecting information into a challenge of its own.

DataVaultAI is my attempt to address that problem.

## What Is DataVaultAI?

DataVaultAI is a local-first document archiving and research application built with Python and SQLite.

Its purpose is to preserve original documents, reconstruct their contents, maintain traceable provenance, and enable meaningful exploration through searchable sources and interconnected research notes.

### Currently Implemented

- Document ingestion for PDF, EPUB, DOCX, ODT, and TXT files.
- Local OCR for scanned PDF documents.
- Preservation of original files and extraction provenance.
- Lexical search across imported content.
- Markdown research notes and project organization.
- Explicit relationships between notes and their supporting sources.
- Review history and traceable corrections.

The archival system is designed to remain useful without AI.

Future semantic capabilities are intended to extend the archive, not replace its foundations.

## Our Six Commitments

**Evidence-based results · Resilience · Privacy protection · Modularity · Open source · Free forever.**

These commitments define what DataVaultAI is designed to earn through verification, not a claim that every planned capability is already qualified. The **Product Trust Charter** in the private development repository is the current owner-approved qualification record; the public summary below does not disclose private source code or internal laboratory evidence.

- **Evidence-based results:** verify source identity and freshness, distinguish original evidence from interpretations, and plan a clearly separated diagnostic view for excluded semantic candidates.
- **Resilience:** prevent silent data loss, preserve committed state and allow only bounded, demonstrably safe automatic recovery.
- **Privacy protection:** keep core archival work local and offline, with no default telemetry and explicit boundaries for optional external data sharing.
- **Modularity:** keep the archival core useful without optional AI, GPU or cloud capabilities.
- **Open source:** provide corresponding source and AGPL-3.0-only license notices when an official software release is published; review third-party dependencies and distribution rights.
- **Free forever:** offer official **Core** and **AI** editions, if released, without mandatory payment, subscriptions or donor-exclusive ordinary software features. Model redistribution depends on compatible rights and practical constraints. Voluntary financial support sustains development without purchasing privileged access.

**Imported content safety:** PDF and other user-provided files can be damaged or malicious. DataVaultAI is **not** an antivirus scanner or OS sandbox; successful import does not certify a file as safe. The project will document security boundaries, apply hostile-input testing and provide a release notice. Warranty and liability limitations are subject to the license and applicable law; an advisory notice never replaces technical safeguards.

**Status:** No public source-code release or installable distribution is available yet. Semantic exclusion diagnostics, the planned two-profile distribution and future web ingestion are not currently shipped. These requirements will be qualified against the eventual M9 release candidate and stabilized within the scope accepted for M10.

## The First Use Case: Tabletop RPG Research

DataVaultAI began with a concrete research problem: navigating large, fragmented tabletop role-playing game libraries.

Game Masters, Storytellers, and worldbuilders often work across rulebooks, supplements, multiple editions, campaign notes, and difficult scans. In a game such as *Vampire: The Masquerade*, the same rule, character, or setting detail may appear in several publications with different contexts or interpretations.

Finding a passage is only part of the challenge. Equally important is identifying **which source says what, where it says it, and how it relates to other evidence**.

The existing archival foundation addresses parts of this problem through multi-format ingestion, local OCR, preserved originals, traceable source units, lexical search, and human-authored notes linked to documentary evidence. It does not automatically resolve disagreements between editions or determine which interpretation is correct.

### Beyond Books: Forums, Websites, and Wikis (Planned)

Another goal is to bring **user-selected information from forums, websites, and community or official wikis** into the same research workflow.

The intended approach is to extract relevant knowledge, classify its provenance and context, and cross-reference it with documents and notes. The researcher could organize that information **within the relevant project or in the broader archive**, rather than leaving useful discoveries scattered across browser tabs.

For example, a rulebook passage, an official clarification, a community-wiki explanation, a forum discussion, and a personal house rule should remain distinguishable. They may be related, but they do not carry the same evidentiary authority.

Future web references should retain source links and available metadata, identify uncertainty, and be reviewed before becoming durable archive knowledge. The goal is **traceable knowledge extraction, not indiscriminate website mirroring**: respect applicable rights and access restrictions, and prefer structured facts, genuinely transformed notes, and permitted short excerpts over copying entire pages. Web access should be explicit and optional; the local archive should remain useful offline.

**This web-ingestion and classification workflow is not implemented yet.** Existing project and note features operate on locally imported documents. Website, wiki, and forum integration belongs to the future roadmap.

Tabletop RPG research is the original niche, not the intended limit. Similar source-provenance problems arise in historical research, technical reference collections, and other evidence-intensive fields.

## Development Status

**Active development. Private source repository. No public release available yet.**

The core archival, document-processing, lexical-search, and note-management foundations have been implemented.

Current M6 work focuses on the persistence foundations and experimental validation needed for future local semantic representations.

Real embedding production and semantic retrieval are not yet integrated into the application.

This public repository is a project showcase and development record.

**It does not currently contain the application's source code or an installable release.**

## The Experiment Behind the Software

I'm developing DataVaultAI as a self-taught amateur, with substantial assistance from AI tools.

I'm not a formally trained software engineer, and I have no intention of pretending otherwise.

What interests me is how far disciplined curiosity, independent learning, and increasingly capable cognitive tools can take someone whose technical ambitions exceed their initial expertise.

The interesting question isn't simply whether AI can write code.

**It's whether we can expand our ability to build without surrendering our responsibility to understand what we're building.**

This is why the project emphasizes explicit hypotheses, human architectural decisions, reproducible tests, critical review, and evidence preservation.

Including when our experiments fail.

An unsuccessful experiment is not necessarily wasted work. Sometimes it reveals that the thing we trusted most was our own verification process.

## Beyond DataVaultAI: Exocortex

DataVaultAI is also connected to a broader independent research initiative: **Exocortex**.

Exocortex explores extended and distributed cognition, human-machine collaboration, and the gradual development of symbiotic cognitive systems.

The objective isn't to automate human judgment out of existence.

It's to explore how our tools might increase our capacity to learn, reason, remember, and act without taking ownership of our decisions.

Exocortex remains exploratory research, not a completed technological system.

## Free Software and Sustainability

DataVaultAI is developed under the GNU Affero General Public License v3.0 only (AGPL-3.0-only), with a public source release planned.

I want the resulting software to remain freely accessible regardless of a user's financial means.

At the same time, developing, maintaining, studying, and researching software requires real time and resources.

My intended funding approach is **Pay What You Want**.

Voluntary contributions may eventually support my independent development activity, continued education, DataVaultAI maintenance, and broader research efforts, including Exocortex.

Financial support will not purchase exclusive access to ordinary free-software features, architectural authority, or privileged control over the project's direction.

No sponsorship or payment mechanism is currently operational.

## Development Reports

I document the project's technical progress, architectural decisions,
engineering methodology, and experimental findings.

- [Development Report No. 1 | Architecture, Engineering Methodology, and AI-Assisted Development (October 21, 2026)](reports/2026-10-21-development-methodology.md)

Reports distinguish implemented capabilities, experimental findings,
and future objectives. Internal test results are identified as such
until their evidence becomes publicly reproducible.

## The Road Ahead

The immediate priority is to consolidate a dependable local archival foundation before introducing increasingly sophisticated AI capabilities.

Future work will be guided by reproducible evidence, clearly documented limitations, and explicit decisions about what the software should and should not become.

I don't expect this project to revolutionize software development.

But I do think the process of building it can help investigate some genuinely interesting questions about technical learning, human agency, and AI-assisted creation.

---

**The vibe starts the project. Understanding and methodology allow it to endure.**

*Independently developed by [Zarrizt](https://github.com/zarrizt).* 
