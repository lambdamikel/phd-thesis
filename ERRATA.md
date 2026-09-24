# Errata

Defects found during a partial re-reading of the dissertation in 2026, eighteen
years after the defence. **This is not an exhaustive audit.** Chapter 2, the
tableau rules and blocking conditions of Chapter 6, the query subsumption and
satisfiability algorithms of Chapter 5, the normal-form transformations and the
algebraic semantics of Chapter 4 were not systematically checked.

The thesis contains no theorems — its formal results were published separately
and are referenced rather than re-derived. The exposed surface is therefore
definitions and specifications, and that is where all of these sit. None affects
the implementation.

**Provenance.** This list came out of an AI-assisted re-reading of the thesis in
2026 (Claude, working from the original LaTeX sources and the `nRQL-9-9-9`
code). Every item below is stated so that it can be checked directly against the
cited definition or source file, which is how it should be used — including
item 1, where the initial reading of the specification turned out to be right
about the definition and wrong about the implementation.

---

## 1. The QBox definition is not acyclic as written

**Chapter 4, Definition (QBox).** A QBox is defined as a *directed acyclic graph*
$(V,E)$ over a set of queries, with $(q_2,q_1) \in E$ iff $q_2 \sqsubseteq q_1$
and every $q_3$ with $q_2 \sqsubseteq q_3 \sqsubseteq q_1$ is equivalent to one
of them. Three consequences follow that contradict acyclicity:

- **Self-loops.** $q \sqsubseteq q$ always holds and the side condition is
  satisfied vacuously, so $(q,q) \in E$ for every query.
- **2-cycles.** If $q_1 \equiv q_2$ with $q_1 \neq q_2$, both $(q_1,q_2)$ and
  $(q_2,q_1)$ satisfy the condition.
- **Quantifier scope.** The condition is written `aus ∃q₃.(q₂ ⊑ q₃, q₃ ⊑ q₁)
  folgt stets q₃ ≡ q₁ oder q₃ ≡ q₂`, where `q₃` in the consequent falls outside
  the scope of the ∃ binding it. The intent is plainly ∀ over the whole
  implication.

**Correction.** Nodes are equivalence classes under $\equiv$, and edges use
*strict* subsumption.

### The implementation does not have this bug

This was checked against `nRQL-9-9-9`, the source the thesis describes. Both
problems are handled, in two places in `repository-9-9-9.lisp`:

```lisp
;; in CLASSIFY — if parents and children are the same single node,
;; the query is equivalent to it and gets no DAG edges at all
(if (and (set-equal parents children)
         parents (not (cdr parents)) (not (cdr children)))
    (let ((equi-node (car parents)))
      (unless (eq equi-node query)
        (setf (slot-value query 'equivalents) (list equi-node))
        (pushnew query (slot-value equi-node 'equivalents))))
  (setf (dag-node-parents query) parents
        (dag-node-children query) children))

;; in REGISTER-QUERY — only non-equivalent queries become DAG nodes
(unless equivalents
  (insert-dag-node qbox query))
```

Self-loops are impossible by ordering: `compute-node-parents` classifies against
`(dag-nodes qbox)`, and the query is not inserted until afterwards, so it is
never compared with itself.

So the real defect is narrower than it first appears. The implementation carries
an `equivalents` structure beside the DAG that the thesis never describes, and
**that omitted structure is exactly what makes the definition well-formed.** The
specification describes a simplification of the implemented design.

---

## 2. The DDB description in Chapter 6 (a description issue, not a claim)

**Chapter 6, \"Potentials opened up by the command history\", item 2.**

First, what this is *not*. The thesis makes no claim that dependency-directed
backtracking (DDB) was implemented. The section is titled **Potentials**, says
the architecture permits their evaluation **\"in principle\"**, introduces the
mechanism as something that **\"can optionally\"** be recorded **\"if DDB is
desired\"**, argues in the subjunctive throughout, and closes by stating that
these potentials were **\"not all realised\"** in the work. It is a design
sketch, presented as one.

As a description, though, it has one defect worth recording.

### The traversal is described in the wrong direction

The slot is introduced as recording **\"which other command objects are causal
for the executed action\"** — *backward* pointers, an action's causes. Phase 1
then says that traversing **those same associations** identifies *\"the ends of
the command-**effect** causal chain\"*. Effects run forward; causes run
backward. Following an action's causes cannot reach its effects. Either the slot
is forward-pointing and the gloss is wrong, or phase 1 should traverse
`postcondition-…`. Both cannot hold.

Relatedly, the AND/OR semantics of multiple supports is left open. Phase 2
unmarks any command having an unmarked support, so an action survives if *any*
support survives — right for alternative justifications, wrong for the
conjunctive premises of a single rule application. For a sketch that is a
reasonable omission; for anyone implementing it, it is the first thing to pin
down.

### What the implementation actually does

The fine-grained causal machinery is indeed unused. `precondition-for-actions`
and `postcondition-for-actions` are declared on `abox-item` and occur in exactly
two places across the whole repository — that declaration and `copy-node`'s copy
list. Never written, never read. The two-phase marking algorithm has no
implementation.

**But the principle behind DDB is implemented, by another route.** Under
*retention of deterministic assertions*, MiDeLoRa deliberately does **not**
`rollback-to` the start of a tableau expansion after a satisfiability test.
Instead it **selectively removes only the assertions `A` with
`choice_points(A) ≠ ∅`** — the nondeterministic ones — and keeps the
deterministic remainder, which is logically implied and therefore still valid.

Given `{i : ∃R.C, j : A ⊔ B}` with completion
`{…, (i,k):R, k:C, j:A}`, only `j:A` depends on a choice. It is removed; the
rest survives, so the next satisfiability test skips re-expanding `i : ∃R.C`
entirely.

That *is* dependency-directed selective retraction — keep what does not depend on
a choice, discard what does. It operates at tableau-expansion boundaries using
choice-point sets rather than per clash using a causal chain. Chapter 7 measures
it: the average ABox unsatisfiability test drops from **12 s to about 3.7 s**.

So the accurate summary is narrower than \"DDB was never built\": the *coarse*
dependency-directed retention ships and is measured; the *fine-grained* per-clash
variant remained a sketch, and it is that sketch which carries the direction
error above.

*(Minor, same area: the thesis calls the inverse slot `postcondition-of-actions`;
the code calls it `postcondition-for-actions`.)*

---

## 3. `L_V(i)` is typed inconsistently in Chapter 4

The definition gives $L_V : V \rightarrow \mathcal{L}_V$ — a node maps to a
*single* expression, and the chapter uses it that way when introducing
propositional substrates: `L_V(i) = C ∧ (D ∨ ¬E)`.

The illustration of atomic query semantics, later in the same chapter, writes
`C ∧ (D ∨ ¬E) ∈ L_V(i)` for the same example — treating $L_V(i)$ as a *set*.
Under the definition this does not typecheck.

(Chapter 6's `L_V/unexpanded(i)` genuinely is set-valued, but that is a
separately defined structure, not this slip.)

---

## 4. The substrate model definition is over-determined

**Chapter 4, Definition (model relation for substrates).** Conditions 1–2 require
*membership* ($i^\mathcal{I} \in (L_V(i))^\mathcal{I}$, reading labels as
set-denoting); conditions 3–4 require *satisfaction* ($\mathcal{M} \models
\bigcup \{L_V(i)\}$, reading the same labels as formulae).

Under the φ-translation route given immediately afterwards, $\Phi(L_V(i))$
already contains the constant $i$, so 3–4 entail 1–2. The conditions overlap.

Condition 1 also typechecks only if nodes are identified with their own
interpretations, Herbrand-style. The surrounding prose implies this; the
definition does not state it.

---

## 5. Figure 1.2 c) carries the wrong title

All three sub-figures of Figure 1.2 are titled *Geschichtete Architektur*
(layered architecture), including c), which the accompanying text explicitly
describes as the **integrated** approach. A copy-paste error in the UMLet source.

---

## 6. "RDMS" for "RDBMS"

Throughout the Figure 1.2 sub-figures.

---

## Where to look next

The DDB algorithm was the obvious candidate and is now item 2 above. The
remaining unchecked areas are the query subsumption and satisfiability algorithms
of Chapter 5, the normal-form transformations and algebraic semantics of
Chapter 4, and Chapter 2 throughout.
