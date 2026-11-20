# devalign-docs — Agent Rules

Technical documentation suite (`ARCHITECTURE.md`, `CONTRACTS.md`, `DATABASE.md`, `MODEL.md`, `PRD.md`, `ROADMAP.md`, `SCOPE.md`) and MDPI *Computers* academic paper manuscripts (`DEVALIGN_Full_Paper_Computers_EN_v*.docx`, `computers-template.dot`, `figures/`).

## Language Scope

- All academic manuscript prose (`DEVALIGN_Full_Paper_Computers_EN_*.docx`) must be written in formal academic **English** (this overrides any Spanish-UI frontend convention).

## Manuscript Safety & Word Formatting Integrity

- **Backup Protection:** Never overwrite frozen backup manuscripts (`DEVALIGN_Full_Paper_Computers_EN_v6.docx` or earlier). When editing substantive manuscript revisions, clone to a new version (or edit only the latest active version when explicitly directed).
- **Lock Files:** Never read, open, or modify Word temporary lock files (`~$*.docx`).
- **Preserve MDPI *Computers* Formatting:** When editing `.docx` files programmatically via `python-docx` or XML, modify text runs in place and preserve 100% of `MDPI_*` paragraph/character styles, tables, figures, `oMath`/`oMathPara` equations, headers/footers, and Word `REF`/`SEQ` field tokens (`fldChar`).
- **Verify Render & Fields:** After `.docx` edits, verify via Microsoft Word COM or XML inspection that all cross-reference fields update cleanly with zero broken references.

## Citation & GenAI Policy Compliance (MDPI *Computers*)

- Keep citations strictly sequential `[1..N]` in order of first appearance across body text and tables.
- Cite only Crossref/publisher-verified references; include complete author lists (no `et al.` in the final reference list), exact DOIs/ISBNs, and published conference proceedings rather than preprints whenever available.
- Maintain transparent GenAI disclosures in both **Materials and Methods** (Section 2.8) and **Acknowledgments** per `computers-template.dot`.
