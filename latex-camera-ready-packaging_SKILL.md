---
name: latex-camera-ready-packaging
description: Clean, rebuild, verify, and package LaTeX camera-ready submission materials for conference proceedings (e.g., Springer LNCS/EasyChair).
---

# LaTeX Camera-Ready Packaging

Use this skill when a user asks to prepare, clean, rebuild, verify, or package LaTeX sources and PDF for camera-ready/proceedings submission.

## Workflow

1. **Inspect the submission requirements and source bundle**
   - Read any instruction email/Markdown in the target directory.
   - List top-level files and archives.
   - If a `.zip` contains the LaTeX project, inspect its contents before extracting.

2. **Extract to a safe working directory**
   - Extract into a temporary workspace, not directly into the user's final folder.
   - If the original path contains non-ASCII or shell-sensitive characters and command tools block/quote poorly, copy/extract to a simple path such as `/tmp/camera_ready_work` for compilation.

3. **Identify the real project dependencies**
   - Determine the main `.tex` file (`\documentclass`, `\begin{document}`).
   - Parse/check actual dependencies:
     - `\includegraphics{...}` for figures.
     - `\bibliography{...}` and `\bibliographystyle{...}` for BibTeX files/styles.
     - local `.cls`, `.sty`, `.bst` files required by the document.
   - Do not keep unused template/sample files merely because they were present in the original archive.

4. **Clean and reorganize conservatively**
   - Rename ambiguous template names (e.g., `samplepaper.tex`) to `main.tex` when appropriate.
   - Put used figures under `figures/` and update `\includegraphics` paths.
   - Keep required class/style/bibliography files.
   - Exclude obvious non-submission material: template manuals, history/readme files, unused sample figures, and stale build artifacts (`.aux`, `.bbl`, `.blg`, `.fdb_latexmk`, `.fls`, `.log`, `.out`, `.synctex.gz`).
   - Do not modify paper content unless needed for compilation or explicitly requested.

5. **Compile and fix blocking build errors**
   - Prefer `latexmk -pdf -interaction=nonstopmode -halt-on-error main.tex`.
   - If TeX tools are missing and installation is allowed/available, install minimal TeX packages plus `latexmk`; otherwise report the missing dependency clearly.
   - Common fix: duplicate BibTeX keys cause BibTeX failure. Remove only the duplicate entry, preserving one canonical entry, then rebuild.

6. **Verify the final PDF**
   - Rebuild from a clean copy of the final source folder, not only from the original working directory.
   - **Check paper size**: `pdfinfo main.pdf | grep 'Page size'` — must be A4 (595.28 × 841.89 pts) for LNCS/Springer. If it shows "letter" (612 × 792 pts), add `a4paper` to `\documentclass[a4paper,runningheads]{llncs}`. A4 vs Letter is the #1 cause of "my margins look wider than the proceedings" complaints.
   - Check total page count with `pdfinfo` or equivalent.
   - For proceedings limits such as "16 pages excluding bibliography," locate where references begin using `pdftotext`/manual inspection and compute body pages.
   - Grep the final log for unresolved references/citations:
     - `undefined references`
     - `Citation .* undefined`
     - `Reference .* undefined`
     - `There were undefined`
   - Distinguish harmless layout warnings (overfull/underfull boxes) from unresolved references or fatal build errors.

7. **Springer LNCS compliance check**
   - Verify `\titlerunning{...}` is present (required for running heads).
   - Verify `\authorrunning{...}` is present.
   - Confirm `\usepackage[T1]{fontenc}` is uncommented.
   - Confirm hyperref eBook style is enabled: `\usepackage{color}`, `\renewcommand\UrlFont{\color{blue}\rmfamily}`, `\urlstyle{rm}` (must be AFTER `\usepackage{hyperref}`).
   - Remove commented template bibliography samples — they add noise to the submission source.
   - Ensure `\textsuperscript{(\Letter)}` marks corresponding author(s). Multiple co-corresponding authors can each carry the marker.
   - See `references/springer-lncs-compliance-checklist.md` for the full checklist.

8. **Export final deliverables**
   - Create a versioned folder such as `v1/` under the user's submission directory.
   - Include:
     - cleaned LaTeX source folder,
     - final PDF,
     - zipped LaTeX source archive for upload,
     - change note Markdown.
   - The change note should state what was kept, removed/excluded, any build fixes, compile command, page-count verification, and upload recommendation.

9. **If requested, split the main file by chapter/section**
   - Detect top-level `\section{...}` boundaries and split section bodies into a `sections/` directory with stable numbered filenames (e.g., `01_introduction.tex`).
   - Keep `main.tex` as the project entry point containing the preamble, `\begin{document}`, title/author/abstract/front matter, `\input{sections/...}` lines, bibliography commands, and `\end{document}`.
   - Do not leave backup copies such as `main_before_section_split.tex` inside the upload source archive unless the user explicitly asks; extra `.tex` files can confuse publisher build systems.
   - Rebuild from a clean copy after splitting, refresh the final PDF and source zip, and append the split to the change note.

9. **Handle Springer/copyright forms when requested**
   - Download the form into the versioned submission folder, but verify the actual file type rather than trusting the URL or requested filename.
   - Springer form endpoints may return a Word document (`application/vnd.openxmlformats-officedocument.wordprocessingml.document`, ZIP header `PK...`) even when the URL has no extension or the email calls it a form; save/rename it as `.docx`, not `.pdf`.
   - Extract `word/document.xml` from the `.docx` to identify fillable placeholders and produce a short filling guide for the user. Use `scripts/inspect_springer_docx.py` for a repeatable read-only inspection of visible field text, embedded media, and signature-block anomalies.
   - When checking a filled DOCX, distinguish visible text (`<w:t>`) from Word field-code instructions (`<w:instrText>`); otherwise placeholders such as `FORMTEXT` can be falsely reported as visible errors.
   - For Springer LNCS/SNCS proceedings forms, flag uncertain fields such as `Volume Editor(s) Name(s)`, `Edition ID`, and `PS` as conference/Springer-provided rather than guessing.
   - Inspect the signature area specifically: partial stray text (e.g., `[Ha`) is not a valid signature and should be removed/replaced with a handwritten signature or signature image before PDF export.

10. **Plan reviewer-comment revisions when reviews are provided**
   - Read the review Markdown/export and the relevant current LaTeX chapter files before writing the plan; do not rely only on the review text.
   - Create a next-version planning file such as `v2/review_action_plan.md`.
   - For each reviewer point, list: issue summary, current paper location, concrete edit(s), and unsupported claims to avoid.
   - If the user says not to run extra experiments, convert requests for extra comparisons/ablations into textual clarifications, limitations, threats-to-validity notes, or future work; never invent results.
   - See `references/reviewer-action-planning.md` for an output template and non-experimental revision-planning rules.

11. **Apply reviewer-comment revisions when requested**
   - Start from a copied/versioned next-source tree such as `v2/latex`; do not overwrite the previously verified camera-ready source.
   - Apply edits chapter-by-chapter from the action plan: running-example clarifications, formal-definition symbols, algorithm pseudocode fixes, evaluation/RQ explanations, threats, related work, future work, and obvious reviewer minor issues.
   - When the user has ruled out extra experiments, convert requests for cross-model comparisons, success rates, validity rates, and broader SOTA comparisons into explicit methodology clarification, limitations, or future work; do not invent numbers.
   - If the paper is at the page limit, compact after adding required clarifications: shorten repeated prose, reduce float/caption spacing conservatively, and use local small algorithm fonts only when readability remains acceptable.
   - Create a separate actual-change log such as `v2/actual_changes_v2.md` that records completed edits, not just planned edits.
   - If the user needs auditability or response-letter material, generate a paragraph-level before/after Markdown grouped by LaTeX file.
   - Also produce a remaining-gap report (e.g., `remaining_review_gaps_v2.md`) that distinguishes not-run experiments/statistics from partially satisfied optional presentation suggestions.
   - Refresh the PDF and source zip, then verify by compiling from the unzipped source archive in a clean `/tmp/...` directory.
   - See `references/tase-lncs-v2-review-revisions.md` and `references/reviewer-revision-implementation.md` for concrete patterns, page-limit tactics, before/after reporting, remaining-gap reporting, and clean source-zip verification.

12. **Final submission checklist**
   - After packaging, remind the user of remaining non-source tasks: manual PDF review, reviewer-comment coverage, Springer/copyright form completion and signature, EasyChair metadata consistency, and upload of both source zip and PDF.
   - If reviewer comments or copyright forms are not present in the working folder, call them out as missing materials rather than implying the submission is complete.

## Pitfalls

- **A4 vs Letter paper size**: TeX Live defaults to US Letter on many systems (especially WSL). LNCS/Springer requires A4. Check with `pdfinfo main.pdf | grep 'Page size'` — it must say "595.28 × 841.89 pts (A4)". If it says "letter" or "612 × 792 pts", add `a4paper` to `\documentclass[a4paper,runningheads]{llncs}`. This is the #1 cause of "my margins look wider than the proceedings" complaints (the proceedings PDFs are cropped to the text block by Springer, so the difference is even more visible).

- Some terminal tooling may block workdirs containing non-ASCII characters. Use a simple `/tmp/...` workspace and copy results back.
- A successful first `pdflatex` pass can still have unresolved citations; always run BibTeX/latexmk to completion.
- Removing an unused-looking `.bst`, `.cls`, or `.bib` can break publisher compilation; confirm references from the `.tex` before deletion.
- Camera-ready uploads often require both PDF and source archive; provide both unless the user explicitly asks otherwise.
- Avoid deleting or overwriting the user's original source archive. Create versioned outputs instead.
- Do not include backup/source-history `.tex` files in the final source archive after reorganizing; publisher tooling may compile the wrong file or complain about ambiguous sources.
- Re-zipping the source after later edits (e.g., section splitting) is easy to forget; always refresh both the PDF and the source zip after structural changes.
- Reviewer-action plans should be grounded in the current source files, not just the review text. When experiments are out of scope, explicitly mark experiment requests as limitations/future work rather than fabricating evidence.
- After applying reviewer-comment revisions, record both what changed (`actual_changes_*.md`) and what remains only partially addressed (`remaining_review_gaps_*.md`) so the user can judge whether optional tables/extra experiments are still needed.
- After applying review-driven revisions, do not trust the working-tree build alone: create the final source zip, unzip it to a clean temp directory, rebuild, and re-check page count, bibliography start page, and unresolved references.
- If review additions push an LNCS paper over the body-page limit, compact prose and float/algorithm spacing before removing substantive reviewer clarifications.
- `grep -c` exits 0 only with matches; with zero matches it prints "0" and exits 1, so `grep -c pattern file || echo "not found"` prints both "0" and "not found". Use `grep -q` for boolean checks, or use Python for reliable undefined-reference detection.
- **Conference system builds can differ from local**: EasyChair and other submission systems run their own TeX Live (often in Docker) with potentially different font metrics, `llncs.cls` versions, or default paper sizes. A paper that fits in 16 body pages locally may render as 17 body pages on the system. Mitigation: (a) always add `a4paper` to `\documentclass` explicitly, (b) use Tier 1 spacing as baseline even if local build fits, (c) keep `\linespread{0.95}` in the preamble as insurance — it can be removed if the system produces correct page counts on first submission.
- A `main.synctex(busy)` file (with parentheses in the name) is a stale lock file from an interrupted compilation. Delete it along with other build artifacts.
- On WSL with files on the Windows filesystem (`/mnt/c/...`, `/mnt/d/...`), an open PDF viewer locks the file on the NTFS side. `rm`, `cp -f`, `mv -f`, and Python `os.remove` all fail with "Permission denied". Workaround: compile to a different filename (e.g., `main_v3.pdf`) and tell the user to close the viewer and rename manually.
- To force References to start on a clean page (e.g., page 17 top), add `\newpage` before `\bibliography{references}`. This is cleaner than spacing hacks and is acceptable for LNCS/Springer proceedings.
- When enabling Springer's eBook URL style, the `\renewcommand\UrlFont{\color{blue}\rmfamily}` line MUST come AFTER `\usepackage{hyperref}`. `\UrlFont` is defined by `hyperref`, not `url`. Placing it before causes `! LaTeX Error: Command \UrlFont undefined`. The correct order is: `\usepackage{hyperref}`, then `\usepackage{color}`, `\renewcommand\UrlFont{...}`, `\urlstyle{rm}`.
- After adding `\titlerunning{}` or enabling eBook URL style, always do a clean rebuild — a stale auxiliary file can mask the error in a quick `pdflatex`-only test.

## Verification commands

```bash
latexmk -pdf -interaction=nonstopmode -halt-on-error main.tex
pdfinfo main.pdf | grep '^Pages:'
pdfinfo main.pdf | grep 'Page size'     # must be A4 (595.28 x 841.89 pts), not letter
grep -E "undefined references|Citation .* undefined|Reference .* undefined|There were undefined" main.log || true
```

For Springer copyright-form DOCX inspection:

```bash
python scripts/inspect_springer_docx.py /path/to/Springer_SNCS_ProceedingsPaper_Copyright_Form.docx
```

For bibliography page detection (prefer the Python script in `references/pdf-body-page-detection.md` for ambiguous cases):

```bash
for p in $(seq 1 $(pdfinfo main.pdf 2>/dev/null | grep '^Pages:' | awk '{print $2}')); do
  echo "---PAGE $p---"
  pdftotext -f "$p" -l "$p" main.pdf - | grep -n -E 'References|Bibliography|Conclusion' | head
done
```

## References

- See `references/tase-lncs-v1-packaging.md` for a concrete session pattern: cleaning an LNCS package, fixing a duplicate BibTeX key, rebuilding, and verifying a 16-page body + references.
- See `references/springer-copyright-form.md` for Springer LNCS/SNCS copyright form download, DOCX verification, field-filling guidance, and filled-form review pitfalls.
- See `references/reviewer-action-planning.md` for reviewer-comment action-plan structure, especially non-experimental camera-ready revisions grounded in the current LaTeX source.
- See `references/tase-lncs-v2-review-revisions.md` for applying a reviewer action plan into a concrete v2 LaTeX/PDF/zip, including actual-change logging, page-limit compaction, and clean source-zip verification.
- See `references/reviewer-revision-implementation.md` for reusable implementation steps for applying review plans, generating before/after and remaining-gap reports, and verifying source-zip rebuildability.
- See `references/page-compaction-techniques.md` for tiered spacing adjustments that recover 0.5–1.5 pages without touching prose. Start with Tier 1 (float spacing: textfloatsep/floatsep/intextsep 4pt, abovecaptionskip 2pt) which alone recovers ~1 page on a typical LNCS paper.
- See `references/springer-lncs-compliance-checklist.md` for a comprehensive Springer LNCS formatting checklist: mandatory fields, package ordering, source cleanliness, and common fixes (`\titlerunning`, hyperref eBook style, `\UrlFont` ordering).
- See `references/pdf-body-page-detection.md` for a Python pdftotext loop that precisely identifies body/reference boundaries when `pdfinfo` is unavailable or the bash `grep` approach is ambiguous.
- Use `scripts/inspect_springer_docx.py` for repeatable read-only DOCX form inspection that ignores non-visible Word field-code instructions.