# Flexible and Configurable Software Architectures for Data-Intensive Ontology-Based Information Systems

PhD thesis by **Michael Wessel**, Technische Universität Hamburg-Harburg.
Submitted **2005**, defended **14 April 2008**. Supervisor: Prof. Dr. Ralf Möller,
Institute for Software Systems (STS).

> **The thesis is in German. [An extended English summary is in this repository](english-summary.pdf) (21 pp).**

This repository is a hub. It connects the dissertation to the systems it
describes — [Racer and nRQL](https://github.com/lambdamikel/Racer),
[MiDeLoRa](https://github.com/lambdamikel/MiDeLoRa) and
[DLMAPS](https://github.com/lambdamikel/DLMAPS) — and to the published papers,
because until now none of those pointed at each other.

---

## Citing

The dissertation is open access at TUHH with a DOI. **Please cite that, not this repository:**

> Wessel, M. (2008). *Flexible und konfigurierbare Software-Architekturen für
> datenintensive ontologiebasierte Informationssysteme.* Dissertation,
> Technische Universität Hamburg-Harburg. Logos Verlag Berlin.
> ISBN 978-3-8325-2162-2. DOI [10.15480/882.832](https://doi.org/10.15480/882.832)

Repository record: [tore.tuhh.de/handle/11420/834](https://tore.tuhh.de/handle/11420/834)

> **Note on dates.** The dissertation was **submitted in 2005** and **defended in
> 2008**; the three-year gap was examination time, not work. The 2005 date is the
> one that matters for questions of precedence — the content predates the SPARQL 1.0
> Recommendation (January 2008) entirely.

The copy here (`diss-wessel-2008.pdf`) is the *corrected* version deposited with
the university: identical in content to the Logos edition apart from minor
typographic corrections.

---

## What the thesis is about

How do you build the software substrate for an **ontology-based information
system** — one whose behaviour is driven by a formal ontology rather than a fixed
database schema? Three problems arise at once and cannot be solved separately:

1. **Representation** — in what data model are the data held?
2. **Query answering** — how are they retrieved?
3. **Inference** — how are the deductions that (2) already requires carried out?

The thesis answers with a Common Lisp framework in three layers that share one
data model, evaluated in two deliberately dissimilar domains: a deductive
geographic information system over a real Hamburg city map, and Semantic Web
retrieval benchmarked on LUBM.

Its organising claim is that such a framework should cover **regions of the
design space, not single points** — which is why it declines to commit to any one
W3C standard and defines its own data model instead.

### Three ideas worth knowing about

**The substrate** — a node- and edge-labelled graph with *pluggable* label
languages and a separate first-order background theory. Instantiate the label
languages differently and the same model becomes a description logic ABox, an RDF
graph, an RCC network, or a map of spatial objects. Its *data substrate* variant
— nodes and edges each carrying key–value property maps — is in substance a
**property graph**, implemented around 2005 and queryable, though [not the first
and with no documented influence on later graph
databases](https://github.com/lambdamikel/shacl-nrql-comparison).

**nRQL** — an expressive conjunctive ABox and Semantic Web query language with
negation as failure, negated roles, constraint atoms and projection operators,
plus *incomplete* and *two-phase* modes that trade completeness for resources. It
shipped inside RacerPro and was used in production. It also has a real
**cost-based query optimiser**: plans assign each atom a role (generator,
successor generator, predecessor generator, or tester), and choosing badly costs
a measured factor of up to **10,000**.

**MiDeLoRa** — a **construction kit for description logic reasoners**, where a
prover is a point in a space of *task × logic × ABox class*. Its central move:
the knowledge base and the tableau are **the same object**, and every mutation is
logged as a command in an explicit history. That makes the *proof process itself*
an inspectable data structure — which opens the way to explanation, true
dependency-directed backtracking, database-style checkpointing and recovery, and
in principle proof migration. **Chapter 6, which describes it, was never
published anywhere.**

---

## Where to read what

| Chapter | Topic | Available in English? |
| --- | --- | --- |
| 1 | Introduction; the three problems | the thesis's own English abstract |
| 2 | Logic, knowledge representation, description logics | standard background |
| 3 | The two guiding domains; requirements | the DL workshop papers |
| 4 | The substrate data model and query language | the JAL article |
| 5 | Realization; nRQL, DISKQL, the optimiser | the JAL article |
| **6** | **MiDeLoRa — reasoner construction kit** | **nowhere — never published** |
| 7 | Evaluation | partly the JAL article and the scalability papers |

**The single best entry point in English** is the journal article, which covers
Chapters 3, 4, 5 and parts of 7:

> Wessel, M., Möller, R. (2009). *Flexible software architectures for
> ontology-based information systems.* Journal of Applied Logic 7(1), 75–99.
> DOI [10.1016/j.jal.2007.07.006](https://doi.org/10.1016/j.jal.2007.07.006)

Chapter 6 does not appear in it beyond a mention. If you are interested in
reasoner architecture, that chapter is the reason this repository exists.

---

## The systems

| Repository | What it is |
| --- | --- |
| [Racer](https://github.com/lambdamikel/Racer) | The RacerPro reasoner, and the home of **nRQL** — the query language of Chapters 4–5 |
| [MiDeLoRa](https://github.com/lambdamikel/MiDeLoRa) | Michael's Description Logic Reasoner — the Chapter 6 construction kit |
| [DLMAPS](https://github.com/lambdamikel/DLMAPS) | Ontology-based spatial queries over digital city maps — the Chapter 5 GIS, and an early predecessor of GeoSPARQL |
| [alcircc5](https://github.com/lambdamikel/alcircc5) | **Current work.** Decidability of ALCI_RCC5, open since 2002–03: a Lean 4 machine-checked decision procedure for the ∀PO-free fragment, full logic still open |
| [shacl-nrql-comparison](https://github.com/lambdamikel/shacl-nrql-comparison) | Retrospective analysis of nRQL against SHACL, SPARQL 1.1 and property graphs |
| [OntoLisp](https://github.com/lambdamikel/OntoLisp) | A Common Lisp framework for the Semantic Web |
| [RacerPorter](https://github.com/lambdamikel/RacerPorter) | Ontology visualization and authoring workbench for KRSS/OWL reasoners |
| [Common-Lisp-Persistency-Manager](https://github.com/lambdamikel/Common-Lisp-Persistency-Manager) | The serializer behind the thesis's *persistent* substrates and ABoxes — the nRQL sources declare their classes with `PERSISTENCE-MANAGER:DEFCLASS` |

---

## Contents of this repository

| File | |
| --- | --- |
| [`english-summary.pdf`](english-summary.pdf) | Extended English summary, 21 pp — chapter by chapter |
| `english-summary.tex` | LaTeX source of the summary |
| `diss-wessel-2008.pdf` | The dissertation (German, 586 pp) |
| [`ERRATA.md`](ERRATA.md) | Known defects in the thesis, and one that turned out not to be one |

---

## Keywords

Description logics · ontology-based information systems · nRQL · MiDeLoRa ·
RacerPro · DLMAPS · substrate data model · property graph · ABox query answering ·
conjunctive queries · query optimisation · tableau calculi · dependency-directed
backtracking · LUBM · RCC · qualitative spatial reasoning · GeoSPARQL · SHACL ·
SPARQL · OWL · Semantic Web · Common Lisp
