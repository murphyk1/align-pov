---
description: Align a POV template to a customer's requirements documentation (with optional running notes) and output a new Word document
argument-hint: <pov-template> <requirements> [running-notes] (each may be .docx, .pdf, .xlsx, .csv, .txt, or .md)
---

You are aligning a POV (Proof of Value) template to a specific customer's requirements. The goal is to produce a professional, customer-ready POV document that replaces all generic language with the customer's real context, terminology, pain points, and success criteria.

## Inputs

The user has provided two or three file paths via $ARGUMENTS:
- **Argument 1 (required)**: Path to the POV template
- **Argument 2 (required)**: Path to the customer requirements (the customer's own artifact — RFP, walkthrough notes, questionnaire, etc.)
- **Argument 3 (optional)**: Path to a running-notes file (the SE's own artifact — accumulated context from follow-up calls, internal observations, customer feedback). Layered on top of the requirements doc; see Step 4b for conflict-resolution rules.

Each input may be `.docx`, `.pdf`, `.xlsx`, `.csv`, `.txt`, or `.md`. The output is always `.docx`. Parse the file paths from $ARGUMENTS before proceeding. If exactly two paths are provided, skip Step 4b and treat the requirements doc as the sole source of customer context.

## Step 1 — Verify dependencies are available

Run:
```
python3 -c "import docx, openpyxl; print('ok')"
```

If it fails, install whichever is missing:
```
pip3 install python-docx openpyxl
```

(`python-docx` is required for output. `openpyxl` is required only when an input is `.xlsx`. PDF, CSV, and plain-text inputs are handled via the `Read` tool, which natively supports those formats — no extra dependencies.)

## Step 2 — Extract text from each document

Extract text from the template, the requirements doc, and (if provided) the running-notes file. Route by file extension:

### `.docx` — use the Python helper below

This extractor walks the document body in order and captures **both paragraphs and tables** — earlier versions missed table content, which often contains the most important customer requirements.

```python
from docx import Document
from docx.oxml.ns import qn

def extract_docx(path):
    doc = Document(path)
    lines = []
    body = doc.element.body
    para_iter = iter(doc.paragraphs)
    table_iter = iter(doc.tables)
    for child in body.iterchildren():
        if child.tag == qn('w:p'):
            try:
                p = next(para_iter)
            except StopIteration:
                continue
            text = p.text.strip()
            if text:
                lines.append(f"[{p.style.name}] {text}")
        elif child.tag == qn('w:tbl'):
            try:
                t = next(table_iter)
            except StopIteration:
                continue
            lines.append("[TABLE]")
            for row in t.rows:
                cells = [c.text.strip().replace("\n", " ") for c in row.cells]
                lines.append("  | " + " | ".join(cells) + " |")
            lines.append("[/TABLE]")
    return "\n".join(lines)

print(extract_docx("path/to/file.docx"))
```

### `.pdf` — use the `Read` tool directly

`Read` natively renders PDFs. For PDFs over 10 pages, use the `pages` parameter to read in chunks (max 20 pages per request). Read the whole document before analyzing.

### `.xlsx` — use the Python helper below

Vendor-questionnaire-style RFPs are often distributed as workbooks with one row per requirement and columns like Category / Requirement / Priority / Vendor Response. The helper walks every visible sheet, reads computed values (not formula strings), and emits each row pipe-delimited so the requirement structure is preserved.

```python
import openpyxl

def extract_xlsx(path):
    wb = openpyxl.load_workbook(path, data_only=True, read_only=True)
    lines = []
    for sheet_name in wb.sheetnames:
        ws = wb[sheet_name]
        # Skip sheets that are explicitly hidden
        if ws.sheet_state == "hidden":
            continue
        lines.append(f"[SHEET] {sheet_name}")
        for row in ws.iter_rows(values_only=True):
            cells = ["" if v is None else str(v).strip().replace("\n", " ") for v in row]
            if not any(cells):
                continue
            lines.append("  | " + " | ".join(cells) + " |")
        lines.append(f"[/SHEET] {sheet_name}")
    return "\n".join(lines)

print(extract_xlsx("path/to/file.xlsx"))
```

If the workbook is very large (thousands of rows), the output may be long — that's expected; read it all before analyzing.

### `.csv` — use the `Read` tool directly

CSV files are plain text. Read the file in full and treat the first row as the header.

### `.txt` or `.md` — use the `Read` tool directly

Read the file in full. Treat markdown headings (`#`, `##`) as section structure.

### Other formats

If a path has any other extension (e.g., `.pages`, `.gdoc`, `.rtf`, `.xls` legacy), stop and tell the user the format isn't supported and ask them to convert it to one of the supported formats.

## Step 3 — Analyze the POV template

Identify and note:
- All major sections and their purposes (Executive Summary, Objectives, Success Criteria, Timeline, Scope, Team, etc.)
- Any placeholder text (e.g., `{{CUSTOMER_NAME}}`, `[Customer Name]`, `TBD`, generic descriptions)
- The narrative structure and tone
- Any metrics or KPI placeholders
- Tables in the template — these often define the structure of Participants, Timeline, Scope, Threat Scenarios, and Success Criteria, and you should preserve them in the output

If the template is rough or unstructured (e.g., a plain-text outline or short markdown), infer reasonable section headings (Executive Summary, Goals, Scope, Success Criteria, Timeline, Appendix) rather than reproducing the rough structure verbatim.

## Step 4 — Analyze the requirements document

Extract and note:
- **Customer name** and key stakeholders / contacts (fall back to the filename if not stated in the body)
- **Business pain points** and problems they are trying to solve
- **Stated goals** and desired outcomes
- **Success criteria** or KPIs they care about
- **Technical environment** and constraints (existing tools, integrations, platforms)
- **Timeline** expectations or deadlines
- **Scope** — what is in and out of bounds
- **Any specific language or terminology** the customer uses to describe their environment or problems

If the requirements doc is rough notes (bullet points, meeting recap, email thread), still extract these dimensions — gaps become `[CONFIRM WITH CUSTOMER]` flags in Step 5, not blockers.

## Step 4b — Layer in the running-notes document (only if Argument 3 was provided)

If a running-notes file was provided, extract the same dimensions from it (customer name, stakeholders, pain points, goals, success criteria, environment, timeline, scope, terminology) and merge them with what you already extracted from the requirements doc using the following rules:

- **Additive by default.** New stakeholders, new pain points, new goals, new in-scope or out-of-scope items, new threat scenarios, and new constraints from the notes should be *added* to what came from the requirements doc, not used to replace it.
- **Notes win on direct conflicts.** When the same dimension is addressed by both files (e.g., a stakeholder's name, a target date, a coverage percentage, a specific OS version count, a deployment model preference), trust the running-notes file — it represents the latest understanding.
- **Notes can promote `[CONFIRM WITH CUSTOMER]` flags to confirmed facts.** If the requirements doc was silent on something and the notes explicitly resolve it, drop the flag and use the confirmed value.
- **Notes never invalidate the customer's stated requirements.** The customer's own RFP / requirements doc is authoritative on what they require — running notes can refine, clarify, or add context, but if a customer requirement says "must support OS X" and the SE notes say something contradictory, keep the customer's requirement and flag the discrepancy in the Appendix.
- **Preserve provenance in the Appendix.** In the final POV's Appendix or Notes section, include a brief mention that running notes were layered in, so reviewers know the document reflects post-RFP context.

If the notes file is empty, malformed, or doesn't add information, proceed as if Argument 3 had not been provided.

## Step 5 — Write the aligned POV document

Rewrite the POV template section by section:
- Replace all generic/placeholder content with customer-specific information from the requirements doc
- Use the customer's own terminology and phrasing where possible — this signals you've listened
- Map their pain points to the POV objectives directly
- Translate their stated goals into measurable success criteria
- If the template has a section the requirements don't address, make a reasonable inference and flag it with a bracketed comment like `[CONFIRM WITH CUSTOMER: assumed 30-day timeline based on Q3 mention]`
- Do NOT leave any placeholder text like `{{CUSTOMER_NAME}}` or `[Customer Name]` unfilled
- Maintain the professional tone of the original template
- Preserve the template's table structures (Participants, Timeline, Current vs. Future State, Scope, Threat Scenarios, Success Criteria, Phased Deployment) and populate them with customer-specific content

## Step 6 — Write the output `.docx` file

Determine the output filename from the requirements document filename:
- Strip the extension and any trailing descriptor (`-requirements`, `-notes`, `-rfp`, `-walkthrough`, `-questionnaire`, `-rfi`, etc.) to derive the customer slug
- e.g., `acme-requirements.docx` → `acme-pov.docx`, `contoso-notes.pdf` → `contoso-pov.docx`, `globex-questionnaire.xlsx` → `globex-pov.docx`, `expeditors.md` → `expeditors-pov.docx`
- If the filename doesn't give a clear customer name, use `aligned-pov.docx`
- Write the output to the same directory as the requirements document

Use `python-docx` to write the output. Apply these styles:
- Section titles → `Heading 1`
- Subsections → `Heading 2`
- Body text → `Normal`
- Tables → built-in style such as `Light Grid Accent 1`
- Flagged items → italicized text in brackets

```python
from docx import Document
from docx.shared import Pt

doc = Document()
# Add content section by section, including tables...
doc.save("output-path.docx")
```

## Step 7 — Report to the user

When done, tell the user:
1. The full path to the output file
2. A brief summary of the key customizations made (3–5 bullets: what pain points were mapped, what success criteria were set, any assumptions flagged)
3. Any sections where information was missing from the requirements doc and assumptions were made
4. **If a running-notes file was provided**, a short note on what came from the notes vs. the requirements doc — specifically, which `[CONFIRM WITH CUSTOMER]` flags were resolved by notes, and any direct conflicts where the notes won. This lets the SE verify the layering happened as expected.
