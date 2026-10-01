# BRD Knowledge Gap Explorer

A prototype that builds a knowledge graph from a regulation and the BRD written from it, and flags requirements whose needed knowledge (a definition, data source, mapping, calculation rule, system capability or business decision) is written in neither document. For each gap it produces a question for an expert and the type of internal source most likely to hold the answer.

**Demo:** the page in `index.html` (hosted with GitHub Pages)
**Notebook:** `brd_gap_prototype.ipynb`

> All data (regulation, BRD, internal documents, answer key) is synthetic and was written for this prototype. Extraction rules and prompts were tuned on the same 14 requirements used for scoring, so the scores are optimistic. They show that the method works end to end, not how accurate it is on real documents.

## Idea

A gap is a requirement that depends on something neither document defines or sources.

- **Graph:** regulation clauses, BRD requirements, and the things requirements depend on (definitions, data elements, calculations, mappings, reference data, system capabilities, rules). Edges are `derived_from` and `requires`.
- **Provenance label per requirement:** *regulatory* (linked to a clause, parameters match), *interpreted* (linked, but a BRD parameter differs from the clause) or *business-added* (no clause).
- **Rule-based checks:** required attributes per item type (for example a calculation needs a formula and a rounding rule), requirements with no clause and no rationale, and BRD times/durations/currencies/versions that differ from the clause.
- **LLM checks:** (1) structured extraction where every attribute must carry a verbatim quote from the BRD, otherwise it is rejected; (2) a "needs clarification?" check that returns the single most important question.
- **Combination:** a requirement is a high-confidence gap when the extraction-based check and the clarification check agree. If only one flags it, it goes to a human review queue.
- **Gap to source:** gaps are matched against a small synthetic inventory of internal documents using text retrieval plus a prior by gap type.

## Results (14 synthetic requirements, 10 with planted gaps; recall 1.00 for every method in both runs)

The notebook was run twice. LLM output varies between runs, so both are shown.

| Method | Run 1: flagged / precision | Run 2: flagged / precision |
|---|---|---|
| Flag every requirement (baseline) | 14 / 0.71 | 14 / 0.71 |
| LLM: list of missing knowledge | 14 / 0.71 | 14 / 0.71 |
| Extraction-based check | 12 / 0.83 | 11 / 0.91 |
| LLM: needs-clarification check | 12 / 0.83 | 14 / 0.71 |
| Either of the last two | 13 / 0.77 | 14 / 0.71 |
| **Both of the last two agree** | 11 / 0.91 | 11 / 0.91 |

Requiring agreement gave the same result in both runs. In run 2 the clarification check flagged every requirement, so the agreement result matched the extraction-based check alone. The main finding is that single LLM checks are unstable and that agreement between independent checks is steadier.

## Running the notebook

1. Upload `brd_gap_prototype.ipynb` to Kaggle (or any Python 3 notebook environment). No GPU is needed.
2. Cells 1 to 9b use only the Python standard library plus `networkx` and `matplotlib`, and need no internet. They produce the rule-based results, graph (`graph.png`, `graph.json`) and routing test.
3. Cells 10 to 17 call a hosted LLM through the Groq API. Turn Internet on, then store your key as a notebook secret named `GROQ_API_KEY` (Kaggle: Add-ons, Secrets). Never paste a key into the notebook or commit it to the repository.
4. Run the cells top to bottom once. Afterwards a single cell can be re-run on its own.
5. Cell 17 writes `gap_report.csv` and `gap_report.xlsx` with a blank `review_status` column for a reviewer.

To test your own text, replace `REAL_REG` and `REAL_BRD` in cell 11. The scoring cells assume the synthetic answer key, so with other text read the flagged gaps and questions directly.

## Limits

- 14 synthetic requirements; one requirement moves precision by about seven points.
- Prompts and extraction rules were tuned on the test set. A fresh set with a locked answer key is needed.
- The answer key is subjective in places; a requirement can be flagged for a different reason than the planted one.
- Real PDFs (tables, cross-references, annexes) and real BRDs are untested.
- Source suggestions come from a small synthetic document set, not real internal sources.
- If no clause is linked to a requirement, giving the model all clauses can pull in unrelated content.
- Outputs are a prioritised review list, not an automatic verdict.

## Next steps

- Score a fresh set of requirements with the answer key locked before any run, and repeat runs to measure stability.
- Test on a real regulation and BRD pair, including PDF parsing.
- Use existing review comments, change requests and defects as ground truth.
- Express the completeness rules as SHACL shapes and align data elements with a regulator's published taxonomy where one exists.

## Related work (quick scan)

- Graph-RAG for requirement traceability and compliance checking (arXiv 2412.08593)
- LLM-based trace-link prediction between requirements and legal provisions (arXiv 2502.04916)
- Regulatory knowledge graphs for compliance question answering (arXiv 2508.09893)
- Standards considered: XBRL taxonomies, SBVR, DMN, SHACL, W3C PROV-O, FIBO
