# Author Response — ESOP 2027 Paper #31

We thank all three reviewers for the careful reading. The reviews converged on one
real problem: an unsound equality axiom on distributed pairs. Reviewers B and C are
right about it, and we have fixed it directly in the mechanized development. The fix
removes the triviality that undermined the composition results and restores the
consistency of the theory. The remaining comments led to concrete improvements, and
all of them are now in the repository and the anonymous artifact. We answer by theme,
so that a fix shared by more than one review appears only once.

## 1. The equality axiom and the fault model (Reviewers B and C)

You are both right, and this is the central issue. The old axiom forced built-in `=`
between distributed pairs with equal values regardless of location. Athena's logic is
first-order, so `=` carries Leibniz congruence, and this made `alive` insensitive to
location. That is exactly Reviewer C's "all replicas alive or none" collapse. Together
with the generated selector axioms it also made the assumption base inconsistent. This
explains Reviewer B's unease it is doing no useful work in the paper.

We change it. Equality on `DPr` is now structural. Two pairs are equal only when they
agree on both values and locations, so the location is part of a pair's identity. This
answers your question. `alive` looks at the location. Equal pairs now agree on their
location, they behave equivalently under `alive`. This definition makes equality well-defined.
The location-transparent comparison that we actually need is a separate predicate, `pr-equiv`
and `dpr-equiv`, which compares values only and claims no congruence with `alive`. The
two notions are now kept apart in the development. This removes the collapse, and with
it the triviality of `fault_tolerance_correctness`, and the theory is consistent again.

On the replicated-function theorem (Reviewer C, Section 3.3): under the corrected
equality the reading "a function is alive if at least one of its locations is alive" no
longer implies "all or none". The theorem now states: a composition survives as long as
each component keeps at least one live replica. We acknowledge that
this first theorem is simple, and that is deliberate. It introduces the language and
shows how the framework is built from the ground up.

In the same topic, reviewer A asks (p. 5) what `alive` buys us if a real detector can be
wrong in both directions, since an alive location may die immediately or answer too slowly.
`alive` is not a runtime failure detector. It is an abstract predicate in the model that
records whether a location's replica is reachable for the step of reasoning at hand.

## 2. Athena syntax and the length of `rep_e` (Reviewer B)

Agreed, and this is an easy fix. Section 3.3 will introduce the syntax it relies on. We
add a short primer that covers `assert` versus `assert*`, the implicit universal
quantifier over the free variables of an `assert`, and the `mp` and `uspec` rules used
later in the paper.

On `rep_e`, agreed as well. We will compressed it to the statement and a short remark,
which makes room for Section 4.

## 3. The DHT: availability, the wrap-around axioms, and `values` (Reviewer B)

**Wrap-around axioms.** Your analysis is correct and your merged rule is the one we now
use. It replaces the three empty-bucket clauses and fixes both problems: the clause that
returned `false` instead of wrapping, and the boundary step that made bucket indices
behave modulo one too many. The only thing we add is keeping the empty-original-list
clause first, so the wrap still terminates when the table is genuinely empty. The
dependent lemmas, `dht_replication_correctness` among them, re-check unchanged. We also
make the bound on hash values and `rv` an explicit hypothesis, and we correct the prose
bound to "any `r` consecutive bucket nodes", matching the abstract.

**What `available` means, and whether `query_functional_correctness` needs it.**
`available` is about stored data: it says every value inserted so far still has a
reachable replica. This is intentionally weaker than the sliding-window property you
describe, which is `no_consecutive_failures`, and the implication from the second to the
first is proved as `dht_replication_correctness`. On the hypothesis you may well be
right. The per-key premise `available_key (hf at v)` is what drives the query, while the
table-wide premise only supports the recursive case. We will either drop it or explain
why the recursion needs it.

**The `values_functional_correctness` contradiction.** You were right, and we fixed it
without weakening the statement. The public `values` folds the buckets with the flag set
to `true`, and the fold only returns `NONE` in its false-flag empty case. We now prove
that with the flag `true` the fold is always defined (`values_true_some`), that a fresh
table returns the empty set (`values_init_some`), and therefore that an empty, and hence
available, table returns `SOME null` rather than `NONE` (`values_fc_empty`). That last
theorem is exactly the sentence your example showed was violated. With these lemmas
`values_functional_correctness` is now a real proof rather than an assertion, its empty
case fully discharged and its insert case resting on one remaining lemma,
`values_after_insert`.

**Returning an `Option` from `values`.** We prefer to keep the `Option`, now that it has
a proved meaning. `NONE` marks "no reachable replica", and the work above shows this case
cannot occur for an available table, so an available DHT always returns `SOME` of its
reachable set. Your concern that partial unavailability quietly returns a subset is
handled by a separate completeness result: under `no_consecutive_failures` the returned
set equals the set of all inserted values. This keeps the distinction between "nothing
reachable" and "some reachable" in the type instead of dropping it.

## 4. The case study, Figure 1, and modularity (Reviewers B and C)

Wee agreed on the setup of Section 4. The revision describes the node and replica layout,
that each service is replicated at two locations, that the metadata DHT uses `r = 1`,
and that the telemetry theorems range over any two Grow-Only-Set replicas. This also
fixes what the arrows of Figure 1 mean. With this section we want to show (as Reviewer C
pointed) the service-level guarantees are direct applications of the previously verified
results.

We also will supply the statement of `avgByType_fault_tolerance` (Listing 1.5), which was
proved in the artifact but stated only implicitly in the paper.

On modularity (Listing 1.6): fair point that `f x = f y` from `x = y` is not a feat of
modular reasoning. The stronger example you ask for is already in the artifact; we simply
did not use it in the paper. `avgTemp_eventual_consistency` combines two different
modules, CRDT convergence on the data side and `DFunction` fault tolerance on the
function side, and concludes that two service replicas give the same answer.

## 5. Scope of the framework (Reviewer B's questions)

**Proving fault tolerance and EC in one framework.** Nothing prevents
assumming EC for known CRDTs. We
chose the stronger path because we want a self-contained theory built from the ground up.
Proving each guarantee inside the theory that uses it checks every assumption against the
rest, and ties the CRDT delivery model to the same `alive` failure model as everything
else. This round is a case in point: that discipline is what surfaced the equality
assumption we corrected, which the assume-and-compose route would have left unexamined.
It closes a formalization gap rather than opening one.

**A path to deployable code.** Extraction from the Athena proofs to running code is
planned as future work in a later paper. The current development is formal only, and we
say so plainly.

## 6. Grammar, typography, and small fixes (all reviewers)

All three reviews flag wording, spelling, and typography, and Reviewer A lists these most
fully. We are grateful the reviews isolated the equality problem so precisely.
The revision it forced is the single change that most strengthens the paper, and it is done.
