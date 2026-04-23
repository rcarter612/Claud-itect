---
name: submittal-review
description: Review construction submittals against project specs and QAP for compliance. Use when asked to review submittals, check submittal compliance, or process incoming submittals.
---

# Submittal Compliance Review

## Your Role

You are performing a first-pass compliance check on a construction submittal. Flag every discrepancy, cite specific clause references, and never assert compliance without evidence in the submitted documents. Your job is to IDENTIFY issues, not resolve them. Do not recommend approval or rejection.

## Inputs

You will receive:

1. A submittal PDF in submittals/incoming/.
2. A spec section — retrieve the matching PDF from specs/ by matching the CSI division number in the filename.
3. The QAP — retrieve from qap/.
4. BABA reference documents — retrieve from baba/.

## Review Process

1. **Review Log Check**: Before reviewing any submittal, read reviews/review-log.csv.
    - Generate MD5 hash of each submittal file before reviewing.
    - If filename exists in log but hash differs, treat as resubmission and review again.
    - Log the new hash alongside the original entry.
    - Skip any file already listed in the log with a matching hash.
    - After completing each review, append a row:
        - submittal_file: filename of the submittal
        - spec_section: CSI number reviewed against
        - reviewed_date: current date (YYYY-MM-DD)
        - reviewer: claude-code
        - md5_hash: hash of the submittal file
    - Never overwrite or modify existing log entries.

2. **Spec Lookup**: Search specs/ for a .txt file matching the submittal's CSI division number. If no .txt file exists, fall back to the PDF. If no matching spec is found at all, follow the **No Matching Specification** protocol below before proceeding.

3. **Manufacturer Check**: Is the submitted manufacturer named in the spec? If not, flag as not a specified manufacturer.

4. **Spec Compliance**: Compare every quantitative requirement in the spec against the submittal data. For each requirement:
    - If the submittal meets it, note as compliant.
    - If the submittal deviates, flag with the spec paragraph number, the required value, and the submitted value.
    - If the submittal does not address it at all, flag as "NOT ADDRESSED IN SUBMITTAL."
    - If the submittal uses a different test standard or nomenclature than the spec, flag as "TEST STANDARD MISMATCH" — do not attempt equivalency unless the submittal explicitly provides it.
    - If the spec language appears to reference a different product type than what was submitted, flag as "SPEC COORDINATION ISSUE."

5. **QAP Compliance**: Retrieve the QAP plaintext file (.txt) from qap/. Do not attempt to read QAP PDFs. Check the submittal against QAP requirements. Flag any deviations or gaps using QAP section references.

6. **Reference Standards and Tests**: Identify all referenced standards, test methods, and regulatory citations listed in Part 1, in-line references throughout the spec, and in the QAP. Search the submittal for evidence of compliance with each. Account for formatting variations in standard names (e.g., SCAQMD / S.C.A.Q.M.D., ASTM E 283 / ASTM E283, AAMA 1503-09 / AAMA 1503). If a referenced standard appears in the spec but has no corresponding data or certification in the submittal, flag as "REFERENCED STANDARD NOT ADDRESSED."

7. **Missing Items**: Check the spec's submittal requirements paragraph (typically 1.05 or similar) and list any required submittal items not provided.

8. **Build America, Buy America Compliance**:
    - Retrieve applicable BABA guidance documents from baba/.
    - Cross-reference the submittal against BABA requirements in the reference documents.
    - If the submittal specifically says "BABA" compliant or "Build America, Buy America" certified, note the page reference.
    - If the submittal says "Made in America," look for any standard that supports the claim.
    - Categorize the submittal into the correct compliance category:
        - **Iron/Steel**: Items consisting wholly or predominantly (more than 50% of the total component cost) of iron, steel, or both.
        - **Manufactured Products**: Items processed into a specific shape or combined with other materials to create a product with different properties.
        - **Construction Materials**: An article, material, or supply — other than an item of primarily iron or steel; a manufactured product; cement and cementitious materials; aggregates such as stone, sand, or gravel; or aggregate binding agents or additives — that is or consists primarily of: non-ferrous metals; plastic and polymer-based products (including polyvinylchloride, composite building materials, and polymers used in fiber optic cables); glass (including optic glass); lumber; or drywall. Items that consist of two or more of the listed materials combined through a manufacturing process, and items that include at least one listed material combined with an unlisted material through a manufacturing process, should be treated as manufactured products rather than construction materials.
        - **Excluded Materials**: Items not falling into the above categories.

9. **Accessibility Compliance**: For plumbing, doors, windows, and door hardware submittals, look for compliance with ANSI 117.1 and ADA. Review submittals for correct threshold height (1/2" max) and for certifications like "ADA", "ANSI 117.1", or similar.

---

## No Matching Specification Protocol

If no spec section exists in specs/ for the submittal's CSI division:

1. State the finding prominently in the Submittal Information section: "**NO MATCHING SPECIFICATION SECTION FOUND.** The project manual does not contain a specification for [CSI number]. This review is conducted against applicable industry standards, drawing references, and QAP requirements only."
2. Search ASI and Addenda folders for any supplemental spec issuances covering this division.
3. Identify the CSI divisions this submittal would typically fall under.
4. **Still use the standard output template below.** Do not change the format.
5. In the Deviations table, use the Spec Reference column for drawing references (e.g., "Drawing C-101"), industry standards (e.g., "AWWA C515"), or "No spec basis" as applicable.
6. In the Manufacturer Check section, state: "No spec section exists to verify manufacturer listing."
7. In the Missing Submittal Items section, state: "No submittal requirements paragraph exists. Items listed are based on standard industry practice for this product type."
8. Add a row to the Summary table: `Spec Coordination Issues | 1` with the critical issue: "No specification section exists for this work. Recommend issuance of applicable spec sections via ASI or confirmation that site work is governed by drawings and local utility standards only."

---
## Cross Section Deferral Protocol

During review, a product may belong to a different spec section than the one the submittal was filed under. Indicators include:
- The product type does not match the spec section scope (e.g., a firestopping sealant submitted under Joint Sealants).
- The spec explicitly defers a requirement to another section (e.g., "See Section 28 10 00" or "Per Section 09 29 00").
- A product's function, classification, or CSI category clearly falls outside the current spec section.

When this occurs:
1. In the current spec section's review, flag the product with status SPEC COORDINATION ISSUE and a note: "Product deferred to Section [XX XX XX]. See PART [N] below."
2. Do not review the deferred product against the current spec section beyond that flag.
3. Retrieve the matching spec PDF from specs/ for the deferred section.
4. Create a new ## PART [N]: Section [XX XX XX] — [Title] heading and produce a full review of the deferred product using the complete output template (Submittal Information through Summary).
5. If no spec exists for the deferred section, apply the No Matching Specification Protocol within that PART.
6. If the spec itself defers a requirement to another section (e.g., §2.03.A says "VOC limits per §01 61 16"), retrieve that referenced spec and evaluate the requirement. Do not flag a deferral-to-deferral as a separate PART — resolve it inline and cite both section references in the Spec Reference column (e.g., "§2.03.A / §01 61 16").

Number PARTs sequentially: PART 1 is always the primary spec section the submittal was filed under. PART 2+ are deferred sections, in the order encountered during review.

---

## Status Vocabulary

Use ONLY these statuses. Do not invent alternatives, shorthand, or narrative tags.

| Status | Use When |
|---|---|
| DEVIATION | Submittal value conflicts with a stated spec/QAP requirement |
| NOT ADDRESSED | Submittal is silent on a spec/QAP requirement |
| TEST STANDARD MISMATCH | Submittal references a different test method than the spec |
| SPEC COORDINATION ISSUE | Spec language contradicts the basis-of-design product, references wrong product type, or internal inconsistency |
| REFERENCED STANDARD NOT ADDRESSED | A standard cited in the spec has no evidence in the submittal |
| VERIFY | Item is not clearly non-conforming, but requires confirmation from architect, engineer, contractor, or further documentation before compliance can be determined |
| COMPLIANT | Submittal demonstrates conformance with the requirement (use in tables only — never assert compliance without evidence) |

In the Deviations table, only include rows with statuses DEVIATION, NOT ADDRESSED, TEST STANDARD MISMATCH, SPEC COORDINATION ISSUE, or VERIFY. Do not include COMPLIANT rows in the Deviations table.

In the Reference Standards table, use COMPLIANT, NOT ADDRESSED, REFERENCED STANDARD NOT ADDRESSED, or VERIFY.

In the QAP Flags table, use DEVIATION, NOT ADDRESSED, or VERIFY.

---

## Output Format

Save the review as a markdown file in reviews/ named: `YYYY_MM_DD - [submittal filename].md`

**You must follow the template below exactly.** Reproduce the markdown structure, headings, and table column headers as shown. Do not rename sections, reorder them, add narrative-style subsections, or use non-conformance numbering schemes (e.g., NC-01). If a submittal contains multiple products, repeat the Deviations table under a #### subheading per product — do not restructure into a different layout.

---

### OUTPUT TEMPLATE — BEGIN

```markdown
# Submittal Review: [Submittal Filename]

## Submittal Information

| Field | Value |
|---|---|
| **Project** | [Project number and name] |
| **Submittal** | [Submittal filename] |
| **Spec Section** | [CSI number — Section title] |
| **MD5 Hash** | [Hash] |
| **Review Date** | [YYYY-MM-DD] |
| **Reviewer** | claude-code |
| **Product(s)** | [List of products and manufacturers] |
| **Prior Review** | [If engineer/other reviewer stamps exist, note them. Otherwise "None observed."] |

---

## Manufacturer Check

[For each manufacturer in the submittal, state whether it is listed in the spec. If not listed, state that a substitution request is required per §01 60 00. If no spec exists, state "No spec section exists to verify manufacturer listing."]

---

## Product-to-Spec Mapping

[If the submittal contains multiple products, state which spec article each product is intended for. If the submittal does not identify this mapping, state that it was inferred and flag the absence. If only one product is submitted, this section may read: "Single product submittal. No mapping required."]

---

## Deviations from Specification

[If multiple products, use a #### subheading per product. If single product, no subheading needed.]

#### [Product Name] → [Spec Article]

| Item | Spec Reference | Requirement | Submitted Value | Status |
|---|---|---|---|---|
| [Item description] | [§X.XX.X] | [What the spec requires] | [What the submittal says, or "Not addressed"] | [STATUS] |

---

## QAP Flags

| Item | QAP Reference | Requirement | Submitted Value | Status |
|---|---|---|---|---|
| [Item description] | [QAP §X.X.X.X] | [What the QAP requires] | [What the submittal says, or "Not addressed"] | [STATUS] |

[If no QAP flags: "No QAP deviations identified."]

---

## Reference Standards

| Standard | Spec Location | Evidence in Submittal | Status |
|---|---|---|---|
| [Standard name/number] | [§X.XX or "QAP §X.X"] | [What the submittal shows, or "None"] | [STATUS] |

---

## BABA Compliance

**Category:** [Iron/Steel | Manufactured Product | Construction Material | Excluded Material]

[For each product: state BABA category, evidence found (with page references), country of manufacture if stated, and whether a formal BABA compliance letter is provided. If no documentation exists, state "No BABA compliance documentation provided."]

---

## Accessibility Compliance

[State findings if applicable to the submittal type (plumbing, doors, windows, door hardware). If not applicable: "N/A — not an applicable submittal type."]

---

## Research Needed

[List items where online research might resolve ambiguity. For each, state what specifically needs to be looked up. If none: "No research items identified."]

---

## Missing Submittal Items

[List items required by the spec's submittal requirements paragraph (typically §1.04 or §1.05) that were not provided. Use a bulleted list. If no spec exists, list items based on standard industry practice and note the basis.]

---

## Summary

| Metric | Count |
|---|---|
| Deviations | [n] |
| Not Addressed | [n] |
| Test Standard Mismatches | [n] |
| Spec Coordination Issues | [n] |
| Verify Items | [n] |
| QAP Flags | [n] |
| Referenced Standards Not Addressed | [n] |
| Missing Submittal Items | [n] |

**Critical issues requiring immediate attention:**

- [List each critical finding as a concise bullet. Critical = missing products for entire spec articles, wrong spec section, BABA non-compliance, missing required certifications, or items that would prevent approval regardless of other corrections.]
```

### OUTPUT TEMPLATE — END

---

## Rules

- Always prefer .txt files over PDFs for specs and QAP documents. Only fall back to PDFs if no .txt exists.
- For submittals, read the PDF (these are the incoming documents being reviewed).
- Cite specific spec paragraph numbers (e.g., 2.04.D.1) for every flag.
- Cite specific QAP section references for every QAP flag.
- Never speculate about product intent or assume compliance.
- If data is missing, say it is missing.
- Do not soften findings. Flag everything.
- When you are uncertain whether something is compliant, flag it with status VERIFY and a note explaining the uncertainty.
- Do not recommend approval or rejection. Do not include a "Recommendation" section.
- Do not use non-conformance numbering (NC-01, NC-02, etc.) or narrative-style finding blocks. Use the tables.
- Do not use ad-hoc status tags like [FLAG], [INFO], [ACCEPTABLE], [CRITICAL], [NON-CONFORMANCE], or any other tag not in the Status Vocabulary.
- If a submittal contains products belonging to a different spec section, follow the Cross-Section Deferral Protocol. This includes cases where the spec explicitly defers to another section (e.g., "See Section 28 10 00") and cases where a product's type or classification does not match the filed spec section.