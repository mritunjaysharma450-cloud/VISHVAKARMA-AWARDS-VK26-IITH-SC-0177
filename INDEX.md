# How to track this repo — for the jury

Branch: main only. No other branches are used for this submission.
Commits are made one template section at a time, in template order,
so the commit history reads in the same order as this index.
Commit messages are short and named after the section they add,
for example "Add project overview, problem, and user journey".

Folder names match the institution template sections exactly,
including "&" where the template uses it. Read folders in this
numbered order — alphabetical order in GitHub will not match
template order, because the template order is not alphabetical,
so use this index, not folder sorting, to follow the repo.

| No. | Template section | Folder in this repo | What is in it | Status |
|---|---|---|---|---|
| 1 | Project Overview | Project Overview/ | Project-Overview.md | Complete, design stage |
| 2 | Problem Statement | Problem Statement/ | Problem-Statement.md | Complete, design stage |
| 3 | Problem Context | Problem Context/ | Problem-Context.md | Complete, design stage |
| 4 | User Journey | User Journey/ | User-Journey.md | Complete, design stage |
| 5 | System Architecture & Specs | System Architecture & Specs/ | Overall-System-Architecture.md, Pole-Unit-Software-Design-Refined.md, Vehicle-Module-Software-Design-Refined.md | Complete, design only, not bench-tested |
| 6 | CAD Design & Mechanical Specs | CAD Design & Mechanical Specs/ | CAD-Design-and-Mechanical-Specs.md | Pending — CAD files not yet supplied, FEA not performed |
| 7 | Electronics Schematics & BOM | Electronics Schematics & BOM/ | Pole Unit Electronic Schematic/ (KiCad source, netlist, ERC), Vehicle Module Electronic Schematic/ (KiCad source, netlist, ERC), BOM-Note.md | Schematics complete as drafts, Rev A, ERC 0/0/0. PDFs and BOM update pending, see BOM-Note.md |
| 8 | Maker Journey | Maker Journey/ | README.md with photo captions | Captions complete, photo files to be uploaded via GitHub web (text push cannot carry images), joint mentor photo pending |
| 9 | Safety & Hazard Mitigation | Safety & Hazard Mitigation/ | Safety-and-Failure-Modes.md | Complete, design stage |
| 10 | Roadmap | Roadmap/ | Roadmap.md | Complete, plan dates only |
| 11 | Technical Challenges & Risks | Technical Challenges & Risks/ | Technical-Challenges-and-Risks.md, Latency-and-Power-Budgets.md, Test-Protocols.md (results blank), Theoretical-Stress-Testing-16-Scenarios.md | Complete, theory and protocols only |
| 12 | References & Sources | References & Sources/ | Research-Plan.md (52 papers, direct links) | Complete, design research only |
| 13 | Image Sources | Image Sources/ | Image-Sources.md | Complete — AI deck images, real KiCad diagrams, real lab photos distinguished |

## Folders NOT in this index — please ignore for jury tracking
Two earlier folders were pushed before this index was made, in a
product-based layout: "Overall System Architecture/" and
"Pole Unit Electronic Schematic/" at repo root, and the root README
that lists them. Those files are the same content as sections 5 and 7
above, in different folders. They will be removed in a cleanup commit
after the template folders are complete, so the jury is left with
only the 13 template folders above plus README.md and INDEX.md.
Do not judge the repo by those duplicate folders — follow this index.

## Dates, in one place
- Schematics: made in KiCad 24 October 2026, revised 26 October 2026
  due to component changes.
- Research plan: papers analyzed before submission (Table A, 38) and
  additional literature found 26 October 2026 during submission
  preparation (Table B, 14, NOT prior use).
- All other files in this index are October 2026 design work, unless
  a file itself states a different date. No dates have been changed.
