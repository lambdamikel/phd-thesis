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

## 2. `L_V(i)` is typed inconsistently in Chapter 4

The definition gives $L_V : V \rightarrow \mathcal{L}_V$ — a node maps to a
*single* expression, and the chapter uses it that way when introducing
propositional substrates: `L_V(i) = C ∧ (D ∨ ¬E)`.

The illustration of atomic query semantics, later in the same chapter, writes
`C ∧ (D ∨ ¬E) ∈ L_V(i)` for the same example — treating $L_V(i)$ as a *set*.
Under the definition this does not typecheck.

(Chapter 6's `L_V/unexpanded(i)` genuinely is set-valued, but that is a
separately defined structure, not this slip.)

---

## 3. The substrate model definition is over-determined

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

## 4. Figure 1.2 c) carries the wrong title

All three sub-figures of Figure 1.2 are titled *Geschichtete Architektur*
(layered architecture), including c), which the accompanying text explicitly
describes as the **integrated** approach. A copy-paste error in the UMLet source.

---

## 5. "RDMS" for "RDBMS"

Throughout the Figure 1.2 sub-figures.

---

## Where to look next

If anyone wants to continue this, the **dependency-directed backtracking marking
algorithm in Chapter 6** is the place to start. It is a two-phase mark/unmark
fixpoint over a causal graph of command objects — exactly the shape where an
off-by-one in the unmark condition hides silently and still behaves correctly on
small examples.
