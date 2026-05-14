# align-pov

A Claude Code plugin for sales engineers and solutions architects who write Proof of Value (POV) documents. Point it at a generic POV template, the customer's requirements file, and (optionally) your own running notes from the engagement and it produces a polished, customer-ready Word document with the customer's terminology, stakeholders, pain points, timeline, scope, and success criteria threaded through every section.

## What it does

`/align-pov` takes two required inputs and one optional input:

1. **A POV template** — your standard, reusable POV deck or document (with placeholders like `{{CUSTOMER_NAME}}`, generic objectives, empty success-criteria tables, etc.).
2. **A customer requirements document** — anything from a formal RFP to rough discovery-call notes. This is the customer's own artifact and should stay clean (e.g., as-received from the customer).
3. **A running-notes file (optional)** — your own running document of follow-up context: notes from subsequent calls, internal observations, customer feedback, decisions you've reached together. Layered on top of the requirements doc so the customer's RFP stays untouched.

It produces a new `.docx` in the same directory as the requirements file, with:

- All placeholders replaced with the customer's actual context.
- Objectives mapped to the customer's stated pain points.
- Success criteria derived from the customer's stated requirements (with priority preserved when present).
- Timeline aligned to the customer's stated dates (or flagged for confirmation when missing).
- Stakeholders populated from the requirements doc (or filename) where possible.
- All template tables preserved and populated — Participants, Mutual Activity Plan, Current vs. Future State, Deployment Scope, Demonstrable Threat Scenarios, Success Criteria, Phased Deployment.
- `[CONFIRM WITH CUSTOMER: ...]` flags anywhere an assumption was made.

The result is a doc you can take into a customer kickoff with minimal editing.

## Supported input formats

Each input can be any of these formats — the plugin routes to the right extractor automatically:

| Format | Typical use case |
|---|---|
| `.docx` | Formal RFP, vendor questionnaire, standard POV template |
| `.pdf` | Scanned RFP, board-distributed evaluation document |
| `.xlsx` | Vendor-questionnaire workbook with one row per requirement (multi-sheet supported, hidden sheets skipped) |
| `.csv` | Flat requirements export from a spreadsheet or ticketing system |
| `.txt` | Plain-text notes or transcript |
| `.md` | Discovery call notes, meeting recaps, email threads |

The output is always `.docx`.

## Installation

In Claude Code, run these commands:

```
/plugin marketplace add murphyk1/align-pov
/plugin install align-pov@murphyk1-align-pov
/reload-plugins
```

Verify the install with `/plugin` and look for `align-pov` under the **Installed** tab.

## Usage

```
/align-pov:align-pov <path-to-template> <path-to-requirements> [path-to-running-notes]
```

**First run** (just the template and the customer's RFP):

```
/align-pov:align-pov templates/standard-pov.docx customers/acme/acme-rfp.pdf
```

**Subsequent run** (layering in your running notes from follow-up calls):

```
/align-pov:align-pov templates/standard-pov.docx customers/acme/acme-rfp.pdf customers/acme/acme-running-notes.md
```

The plugin will:

1. Read each file (using the right extractor for each format).
2. Analyze the template structure (sections, tables, placeholders).
3. Analyze the requirements doc (customer name, stakeholders, pain points, goals, scope, timeline).
4. **If a running-notes file was provided**, layer its content on top of the requirements doc — adding new context where the requirements doc was silent, and trusting the notes on direct conflicts (since they represent the latest understanding). The customer's requirements doc remains the authority on what they actually require; notes refine, clarify, and add.
5. Generate the aligned POV.
6. Save it next to the requirements file with a derived filename — `acme-rfp.pdf` becomes `acme-pov.docx`. The running-notes file does not affect the output filename.
7. Report back a summary of what was customized, any assumptions flagged, and (if notes were provided) which `[CONFIRM WITH CUSTOMER]` flags the notes resolved.

## Don't have a template yet?

A vendor-neutral starter template is bundled at [`templates/generic-pov-template.docx`](templates/generic-pov-template.docx). It includes all the sections the plugin knows how to align (Executive Summary, Participants, Goals, Timeline, Current vs. Future State, Scope, Threat Scenarios, Success Criteria, Phased Deployment, Appendix), priority tiers for success criteria (Must Have / Nice to Have / Out of Scope), and inline guidance notes coaching you on what each section should contain.

You can use it directly:

```
/align-pov:align-pov templates/generic-pov-template.docx customers/acme/acme-rfp.pdf
```

Or copy it, swap in your own branding/structure, and use the customized version as your standing template going forward.

## Not sure what running notes should look like?

A generic example is at [`examples/example-running-notes.md`](examples/example-running-notes.md). It shows the shape of a useful running-notes file — dated entries for stakeholder confirmations, timeline updates, scope refinements, new requirements that emerged post-RFP, priority changes, and an open-items list. Copy it as a starting point and replace the bracketed placeholders with your customer's actual context.

The plugin doesn't require any specific format — plain Markdown with dated section headings is enough. The example just shows what tends to work well.

## One-time setup nuance

The plugin uses two Python libraries to read and write Word documents and Excel workbooks:

- `python-docx` — required for the output `.docx` (and for reading `.docx` inputs).
- `openpyxl` — required only when you give it an `.xlsx` requirements file.

The plugin will check for these on first run and prompt you to install whatever's missing:

```
pip3 install python-docx openpyxl
```

That's a one-time thing per machine. After that, all subsequent runs are dependency-free.

PDF, CSV, and plain-text inputs need no extra dependencies — Claude Code's built-in `Read` tool handles those natively.

## What the output looks like

The plugin doesn't invent a structure — it follows your template's section order and table layouts. If your template has six section headings and seven tables, the output will have six section headings and seven tables, populated with customer-specific content.

Common sections it knows how to align:

- **Executive Summary** — reframed around the customer's strategic drivers.
- **Participants** — populated with named stakeholders from the requirements doc.
- **PoV Goals + Business Objectives** — mapped from customer pain points and stated goals.
- **Mutual Activity Plan & Timeline** — aligned to customer-stated dates (or flagged for confirmation).
- **Current vs. Future State** — framed around what the customer is replacing or improving.
- **Deployment Scope** — explicit in-scope and out-of-scope items from the customer doc.
- **Demonstrable Threat Scenarios** — built around the customer's specific stated concerns.
- **Success Criteria** — derived from the customer's stated requirements with priority preserved.
- **Activity Plan: Phased Deployment** — populated with the integrations and platforms the customer named.
- **Appendix and Notes** — open items and assumptions to confirm at kickoff.

## Tips for best results

- **Use the customer's actual document.** A two-page email thread produces a better POV than a synthetic summary you wrote yourself — the plugin uses their terminology and quotes their stated language.
- **Don't pre-edit the template.** Let the plugin see the placeholders (`{{CUSTOMER_NAME}}`, generic table rows, etc.) — that's how it knows what to fill in.
- **Review the `[CONFIRM WITH CUSTOMER: ...]` flags.** Anywhere the requirements doc was silent, the plugin makes a reasonable inference and flags it. Walk those flags into your kickoff agenda.
- **Keep the customer's requirements doc clean — put your follow-up context in a running-notes file.** Each run regenerates the POV from scratch and overwrites the previous output, so the way to incorporate new information is to give the plugin all of it at once. The recommended pattern: leave the customer's RFP as-received (don't edit it), and maintain a `<customer>-running-notes.md` alongside it where you capture context from follow-up calls, decisions, and clarifications as they happen. Pass that as the optional third argument. When new context comes in, append to the notes file and re-run.

## Limitations

- The output uses standard Word styles (Calibri, default Heading 1/2, the `Light Grid Accent 1` table style). It does not preserve custom branding, fonts, or theme colors from your source template.
- Images and embedded objects in the inputs are not extracted.
- Legacy `.xls` workbooks are not supported — convert to `.xlsx` first.
- `.pages`, `.gdoc`, and `.rtf` are not supported — convert to one of the supported formats first.

## License

MIT — see [LICENSE](LICENSE).

## Contributing

Issues and pull requests welcome. If you have a customer-requirements format that isn't well-handled, open an issue with a sanitized example and the expected output.
