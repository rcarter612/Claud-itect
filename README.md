# Claud-itect

AI-powered construction document review skills for [Claude Code](https://claude.ai/claude-code). These skills automate first-pass compliance checking of specification books and construction submittals against project requirements.

## Skills

### Spec Reviewer (`/spec-reviewer`)

A multi-agent specification coordination review system. It reads a full CSI MasterFormat spec book (`.docx` files) and checks every section against project requirements, flagging discrepancies, contradictions, outdated standards, and non-compliance.

**Review agents (user-selectable per project):**

| Agent | What It Checks |
|---|---|
| Internal Consistency | Duplicate products, contradictory requirements, orphaned references, submittal gaps |
| Cross-Section Coordination | Conflicts between spec sections, edition mismatches, missing cross-references |
| Referenced Standards | Cited standard editions vs. a maintained current-standards reference file |
| QAP Compliance | Qualified Allocation Plan requirements (materials, warranties, energy, accessibility, VOC) |
| Geotechnical | Geotech report findings mapped to affected spec sections (sulfates, bearing capacity, water table, etc.) |
| Client Requirements | Owner standards and client-specific requirements |
| BABA | Build America, Buy America domestic content categorization and compliance |
| Additional Requirements | Per-project programs from a drop-zone folder (NGBS, Enterprise Green Communities, Green Built Homes, etc.) |

**Workflow:**

1. **Setup** -- Select reviewers, confirm section-to-reviewer mapping, parse table of contents (XML-aware: handles both Word tables and plain-text entries)
2. **Internal Consistency** -- Parallel agents review sections for logic errors and outdated standards
3. **Requirement Agents** -- Parallel agents check each section against QAP, geotech, client, BABA, and additional program requirements
4. **Cross-Section Coordination** -- Fingerprint index built across all sections; conflicts and duplicates identified
5. **Coordination** -- All findings merged, deduplicated, and cross-source conflicts resolved using a Most Stringent algorithm with a defined hierarchy (Building Code > QAP > Additional Requirements > Geotech > Client)
6. **Document Output** -- Changes applied as tracked revisions to `.docx` copies; reports generated

**Outputs (saved to `9.0 Output/A. Reviews/A1. Spec Reviews/[date]_Review/`):**

| Output | Description |
|---|---|
| `specs/` | Copies of `.docx` files with tracked changes and Word comments -- designer accepts/rejects in Word |
| `coordination-report.md` | Full findings report organized by section with compliance status tables |
| `coordination-report.html` | Interactive HTML dashboard with filtering, pure-CSS stacked bar charts, localStorage-persisted correction tracking, and TSV export/import |
| `speclink-reentry-guide.md` | Step-by-step guide for re-entering accepted changes into BSD SpecLink Cloud |
| `review-log.csv` | Per-section review log with dates, reviewer list, change counts, and file hashes |
| `findings/` | Intermediate JSON findings from each agent (used by the coordination phase) |

---

### Submittal Review (`/submittal-review`)

A first-pass compliance checker for incoming construction submittals. It reads a submittal PDF and compares it against the matching spec section, QAP, BABA requirements, and referenced standards.

**What it checks:**

- Manufacturer listed in spec (or substitution required)
- Every quantitative spec requirement vs. submitted values
- QAP compliance with specific section references
- All referenced standards and test methods cited in the spec
- BABA compliance categorization (Iron/Steel, Manufactured Product, Construction Material, Excluded)
- Accessibility compliance (ADA / ANSI 117.1) for applicable product types
- Missing submittal items per the spec's submittal requirements paragraph

**Protocols:**

- **No Matching Specification** -- When no spec section exists for the submittal's CSI division, the review proceeds against industry standards, drawings, and QAP only
- **Cross-Section Deferral** -- When a product belongs to a different spec section, the review spawns additional review parts under the correct sections
- **Resubmission Detection** -- MD5 hashing skips already-reviewed files and detects resubmissions

**Output:** A markdown review file saved to the project's reviews folder, following a strict template with tables for deviations, QAP flags, reference standards, BABA compliance, accessibility, and a summary with critical issues.

---

## Project Folder Structure

Both skills operate on a standardized OneDrive project folder. Each project lives under `OneDrive - ASK Studio` with this layout:

```
[Project Number] - [Project Name]/
  1.0 Drawings/              -- Drawing set (PDF)
  2.0 Specs/                 -- Spec sections (.docx, CSI MasterFormat)
  3.0 References/
      A. Building Code/
      B. QAP/                -- Qualified Allocation Plan (.txt preferred)
      C. Client/             -- Owner standards
      D. Energy/             -- Energy docs (not a review source)
      E. BABA/               -- Buy America guidance
      F. Geotech/            -- Geotechnical report
      G. Referenced Standards/  -- current-standards.md
      H. Other/              -- Drop-zone for additional programs (NGBS, EGC, etc.)
  4.0 Addenda/
  5.0 RFI/
  6.0 ASI/
  7.0 Communications/
  8.0 Submittals/
      A. Incoming/            -- Submittal PDFs from contractor
  9.0 Output/
      A. Reviews/
          A1. Spec Reviews/   -- Spec reviewer output
          A2. Submittal Reviews/  -- Submittal reviewer output
```

## Dependencies

- **OneDrive sync** -- Project folders must be synced locally via OneDrive for ASK Studio
- **Python packages** -- `python-docx` and `lxml` (spec reviewer uses these to write tracked changes to `.docx` files)
- **Referenced standards file** -- `3.0 References/G. Referenced Standards/current-standards.md` must be maintained with current standard editions (ASTM, AAMA, ANSI, NFPA, etc.)
- **QAP as plaintext** -- `.txt` format preferred for QAP documents; PDF fallback is supported by spec-reviewer but not submittal-review
- **Claude Code** -- Both skills run as Claude Code agent skills

## Key Design Principles

- **Never modify originals** -- All edits go to copies in the output folder
- **Cite everything** -- Every finding includes specific paragraph and requirement references
- **Most stringent governs** -- When multiple sources apply, the value satisfying all sources is used
- **VERIFY over speculation** -- When intent is ambiguous, flag for designer review rather than guessing
- **Full context in comments** -- Designers can accept or reject changes without re-doing research

## License

See [LICENSE](LICENSE) for details.
