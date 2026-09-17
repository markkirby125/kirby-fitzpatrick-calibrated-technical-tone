# Calibrated Technical Tone — Technical Operational Dispatcher

**Framework Author**: William Fitzpatrick (*Writer Science*)  
**Source Lecture**: [The EXACT System to Transform Messy Drafts into Clear Writing](https://www.youtube.com/watch?v=6HPNb0tiDNg)  
**Parent Collection**: [Master Collection](../../kirby-fitzpatrick-writers-collection/SKILL.md) | [Global Help](../../../kirby-help/SKILL.md)  

---

## 1. Cognitive Foundation: Calibration as the Evidence–Confidence Ratio

**The concept.** Every technical sentence carries two independent payloads: a **claim**, and a **confidence signal about that claim**. In a messy draft those two payloads are allowed to drift apart. The author reaches for adverbs to do work that evidence is supposed to do — `obviously`, `clearly`, `trivially`, `of course` on one side; `might`, `perhaps`, `maybe`, `arguably`, `I think` on the other. Neither adverb tells the reader anything about the claim. Both are *tone leaking around the edges of missing evidence*.

Fitzpatrick's calibration move is a substitution, not a softening: **delete the marker, keep the fact, and let a verifiable observation carry the confidence the marker was faking.** A review comment that says `obviously this races` has asserted consensus the author has not earned. A comment that says `line 41 reads b->len after free(b) on line 39; TSAN reports the race under -race` needs no adverb and admits no argument. The first invites the reader to argue about tone. The second forces them to argue about a fact — which is the only conversation that can terminate.

**Why this is a software-engineering problem, not a style preference.** Technical prose in this domain is an **action-selection interface**. Reviewers, on-call engineers, incident commanders, and security triagers read it to decide what to do next, how fast, and who owns it. Both failure modes corrupt that decision:

* **Overclaim (dogmatism).** `obviously`, `clearly`, `trivially`, `just`, `simply`, `everyone knows`. The reader hears *authority*. If the claim is wrong, the cost is a bad merge, a false alarm, or a disputed priority. If the claim is right but unsupported, the cost is that the reviewer cannot audit it and must re-derive it — the exact labour the comment was supposed to save.
* **Underclaim (hedging).** `might perhaps maybe`, `it seems like`, `potentially`, `possibly a concern`. The reader hears *noise*. A real defect delivered in a modal stack gets triaged below an echo of its true severity. In security disclosure the hedge is worse than the dogmatism: minimizing language is how a live vulnerability gets buried in a backlog.

Both are the same error with opposite signs: **the sentence is not falsifiable by its reader.** Calibration restores falsifiability — every assertion arrives with the evidence status that decides how far it can be trusted.

| Marker class | Draft example | What the reader actually hears | Engineering cost |
|---|---|---|---|
| Dogmatic authority | "Obviously this is the wrong abstraction." | *I have not verified this but want it to be true.* | Argument about design taste; no diff, no defect, no owner. |
| Crowd assertion | "Everyone knows shared mutable state is bad here." | *An appeal to consensus substitutes for the failing case.* | Reviewer must supply the reproduction to justify a block. |
| Minimizing hedge | "There might possibly be a small race here." | *Low severity; probably not actionable.* | Live data race triaged below its real priority. |
| Epistemic fog | "I think maybe we could consider caching this." | *No proposal, no measurement, no decision.* | RFC deliberation loops; the thread ends without a decision record. |

```text
[BEFORE — TONE DECOUPLED FROM EVIDENCE]

  claim ──▶ [ adverb layer ] ──▶ reader ──▶ ??? action
             "obviously"          ├─ verified, or assumed?
             "clearly"            ├─ defect, or preference?
             "might perhaps"      └─ act now, or never?
  evidence: (absent)                  the reader guesses the author's confidence

        asserted confidence
                ▲
   dogmatism    │   ✗ "obviously broken, clearly wrong"      ← unearned certainty
   (overclaim)  │
   ─────────────┼──────────────── evidence ─────────────────▶
                │   ✗ "it might possibly perhaps be an issue" ← unearned doubt
   hedging      │
   (underclaim) │
                ▼
   the calibration line — asserted tone == provable evidence — is empty
```

```text
[AFTER — TONE BOUND TO EVIDENCE]

  claim ──▶ [ verifiable fact + evidence pointer ] ──▶ reader ──▶ action

             "line 41 reads b->len after free(b) at line 39"
             "repro 3/3 · seed 7 · fuzz run #2214 · exit 137"
             "not tested: Windows path, no CI runner"
             "confidence: high (measured) — cost: medium (1 migration)"

  → the reader now decides only: severity, owner, urgency. Tone is not a variable.
```

**The mental model, in one line.** A calibrated sentence states *what was observed*, *how it was observed*, and *what remains unknown* — and nothing else. Confidence is a property of the evidence, so the writer's job is to report the evidence, not to perform the confidence.

---

## 2. Core Transformation Protocols

**Rule 1 — Assertion Ownership: every claim ships with its evidence status.** Each assertion is one of four states, stated explicitly: **Verified** (reproduction, command, measurement), **Observed** (seen once, no reproduction), **Reasoned** (inferred from code, not executed), **Unknown** (no signal). If a sentence cannot be labelled, it is not a technical assertion — it is a preference, and it must be marked as one.

**Rule 2 — Purge the dogmatic markers.** Eradicate markers that assert certainty the writer has not demonstrated: `obviously`, `clearly`, `trivially`, `needless to say`, `of course`, `everyone knows`, `it goes without saying`, `just`, `simply`, `plainly`, `as anyone can see`. Deletion always beats replacement; when the claim is genuinely trivially true, the reader will find it trivial without being told. If deletion leaves a sentence that now looks unsupported, that is the protocol working — the unsupported sentence was the defect.

**Rule 3 — Purge the modal fog, but never delete real uncertainty.** Slash `might perhaps maybe`, `it seems like`, `sort of`, `kind of`, `arguably`, `somewhat`, `I could be wrong but`, and stacked modals (`could potentially possibly`). Real uncertainty is *not* removed — it is **promoted to a named gap**: `untested`, `unmeasured`, `not reproduced on Linux`, `assumes <condition>`. Fog is replaced by a boundary, never by false confidence.

**Rule 4 — Convert speculation into a testable proposition.** A hedged guess is an unfinished experiment. `This might possibly leak on reconnect` becomes `Hypothesis: reconnect leaks one fd per attempt. Test: 1k reconnects, read /proc/<pid>/fd. Result: pending.` Every speculation either becomes a checkable test or drops out of the document.

**Rule 5 — The falsifiability gate.** Before shipping a sentence, ask: *could a competent reader prove this wrong using resources named in the sentence?* If no — either add the evidence pointer or mark the sentence as opinion. Applied mechanically, this gate kills both marker classes at once, because neither `obviously` nor `might perhaps` survives a requirement for a reproduction step.

**Rule 6 — Collapse to a single confidence channel.** Numbers carry the calibration so adjectives do not have to: `3/3 reproductions`, `p99 480ms → 2.1s`, `1 of 12,400 requests`, `8 of 8 fuzz seeds`, `not measured`. When a number exists, delete every confidence adjective next to it.

**Rule 7 — Escalation without assertion: name the consequence, not the outrage.** Dogmatism in review usually disguises a real concern. State the mechanism and its blast radius instead: not `this is clearly a disaster`, but `this drops the unique index; during the rebuild, writes to orders fail with 23505 for ~4 min`.

### Transformation table

| # | Anti-pattern (uncalibrated) | Calibrated replacement | Mechanism |
|---|---|---|---|
| 1 | "This is **obviously** a race condition." | "TSAN reports a write/read pair on `state.ready` at `worker.go:88` / `:112`; `go test -race` fails 3/3." | Marker → tool + location + repro rate. |
| 2 | "**Clearly** the cache invalidation is wrong." | "`users:42` stays stale for TTL=300s after `PATCH /users/42`; the invalidation call is absent from `updateUser()`." | Assertion → mechanism + observable symptom. |
| 3 | "It's **just** a one-line fix." | "One-line change + 2 tests; blast radius: auth middleware only; no migration." | Minimizer → explicit scope and cost. |
| 4 | "**Everyone knows** this pattern doesn't scale." | "At 40k rps this path allocates 1.2 GB/s; the p99 we measured was 480ms at 12k rps (bench `load-2214`)." | Consensus appeal → measurement. |
| 5 | "There **might possibly perhaps** be a memory leak." | "RSS grows 12 MB/min under 50 rps for 30 min; not reproduced at 5 rps. Hypothesis: unbounded `pending` map." | Modal fog → measured slope + boundary of the claim. |
| 6 | "I think maybe we should consider batching." | "Proposal: batch writes in groups of 500. Trade-off: +180 ms p50, −94% syscalls. Not yet benchmarked end-to-end." | Opinion → proposal + trade-off + unknown. |
| 7 | "This **should be fine** now." | "Verified: 3 integration suites green. Not verified: concurrency under replica failover." | Blanket reassurance → verified / not-verified split. |
| 8 | "**Arguably** the ordering is wrong." | "Ordering is `write → ack → persist`, so a crash between ack and persist loses the acknowledged write." | Vague critique → falsifiable consequence. |
| 9 | "The migration **could potentially** be risky." | "Migration takes an `ACCESS EXCLUSIVE` lock; on the 400 GB `events` table that is ~3 min of writes blocked. Reversible: yes. Rollback: `downgrade()` verified." | Risk adjective → lock mode, duration, reversibility. |
| 10 | "This is **clearly** exploitable." | "`POST /import` writes user-controlled `path` into `os.Open` without `filepath.Clean`; local file read confirmed on `v2.7.1`. Remote impact: unassessed." | Severity claim → exact sink, one confirmed impact, one open question. |
| 11 | "I **might be wrong**, but the query looks slow." | "`EXPLAIN ANALYZE`: seq scan, 2.4M rows, 3.1s. Missing index on `(tenant_id, created_at)`." | Self-doubt → plan output. |
| 12 | "Nothing is **obviously** broken here." | "Reviewed `diff` + tests passed. Not reviewed: the Terraform change in this PR." | Absence-of-proof → explicit coverage boundary. |

**Pre-flight pass (run in this order):** (1) strip dogmatic markers; (2) strip modal stacks; (3) attach an evidence pointer or a `not measured` label to every surviving assertion; (4) convert every remaining speculative sentence into a named test or delete it; (5) verify no severity adjective remains that a number could replace.

---

## 3. Engineering Application Scenarios

### 3.1 Code Reviews

The review comment is the highest-frequency calibration surface in the codebase, and the one where tone drift costs the most trust per token.

* **Dogmatic comment (rejected):** `This is clearly wrong — obviously you need a lock here.`
* **Calibrated comment:** `updateCounter() is called from both the HTTP handler (handler.go:64) and the cron worker (jobs.go:203) with no shared lock. Concurrent increments lost: reproduced 200 lost writes in 10k under -race-free load. Recommend sync/atomic or a mutex on Counter.`
* **What changed:** the marker was replaced by a caller list, an observed loss rate, and a bounded recommendation. The author can now agree or produce a counter-example.
* **Hedged comment (rejected):** `I think this might possibly be a bit of a problem maybe?`
* **Calibrated comment:** `Concern (unverified): the retry loop has no cap — `for { send(); }` at client.go:151. I did not test a permanently failing endpoint. If intended, add a note; if not, treat as P1.`

Review-body discipline:
1. **Label every comment with its state** — `Defect (verified)`, `Concern (unverified)`, `Nit`, `Question`, `Preference`. The label replaces the adverb entirely.
2. **Never escalate with intensifiers.** `Critical`, `blocking`, and `must` are reserved for items whose blast radius is stated in the same sentence.
3. **Do not hedge a real defect into a suggestion.** If a data-loss path is confirmed, `data loss on concurrent PATCH; repro 200/10k` is the minimum. Tone softening here is a disclosure hazard, not politeness.
4. **Preference must be named as preference.** Design taste asserted with `obviously` is the single largest source of review friction; `Preference (style): I would extract this; no functional impact. Non-blocking.` ends the thread.
5. **Close the loop with the calibration.** When resolving, report status in the same currency: `Fixed in 9f3c1a2; the same repro now passes 3/3.`

### 3.2 PR / Merge-Request Descriptions

A PR description is a calibrated summary of what is *known* about a change. The standard failure is a body that reads `Refactored the auth flow, should be fine, tested locally` — hedging on risk, dogmatic on coverage.

Template (calibrated):

```markdown
## What changed
Rotates the signing key on a 24h schedule instead of on restart. (auth/keys.go, +112 −38)

## Evidence
- `go test ./auth/... -race` — pass, 3/3 runs
- Staging soak: 6h, 0 auth errors, 41k sign-ins, key rotated 6× (dashboard link)
- Load test: p99 92ms → 96ms at 12k rps (bench `load-2218`)

## Not verified
- Multi-replica rollover timing (single-replica staging only)
- Clients with cached old keys older than 24h — assumed zero; not measured

## Risk & reversibility
Blast radius: auth issuance for all users. Reversible: yes (config flag `KEY_ROTATE=off`).
```

Rules: **the "Not verified" section is mandatory** — its absence is itself a dogmatic claim of total coverage; **no "should work"** — replace with the command and its result; **verify claims carry the command that produced them**; **quantify performance**, never `faster now`.

**Security-disclosure variant.** Disclosures fail by underclaim, and `might`, `possibly`, and `in some cases` are how a P1 reads as a P4. Write: the exact sink (`path` → `os.Open`, `import.go:88`), the confirmed impact on a named released version, the reproduction, and the unassessed scope as a **named** gap (`remote exploitability: unassessed, no network-exposed call path found in 30 min of review`). Never use minimizing framers (`only`, `just`, `low-impact`) without the measurement that establishes them.

### 3.3 Architecture RFCs and ADRs

Design documents have a longer half-life than the decision, so an uncalibrated RFC (either `obviously the right choice` or `we could maybe consider possibly X`) becomes a permanent record of a non-decision.

Rules:
1. **Separate decision from evidence in the document structure.** Blocking: *Decision*, *Status* (`Proposed` / `Accepted` / `Superseded`), *Evidence* (measurements, prototypes, incident counts), *Open questions* (each with an owner), *Reversibility*.
2. **Every comparative claim carries a number or a citation.** Not `option B is clearly faster`; instead `option B: p99 96 ms vs A 480 ms at 12k rps (bench load-2218); cost: +1 service, +2 on-call rotations`.
3. **Do not hedge a decision into vapour.** `We might maybe use Postgres` is not an ADR. Write `Decision: Postgres 16. Constraint: single-writer throughput ≤ 8k tps measured. Revisit if write load exceeds 6k tps.` If the decision is genuinely deferred, write `Status: Deferred — blocked on X, owner @name, revisit by <date>`.
4. **Calibrate estimates explicitly.** `Unknown (no data)`, `Modelled (similar migration took 3 weeks)`, `Measured (prototype: 4 days)`. An unlabelled estimate reads as a commitment.
5. **Record the rejected option with its disconfirming evidence**, not with a verdict adjective. `Rejected A: measured 5× tail latency; the failure mode at 2× current load was not tested.` — this is what prevents the same debate from restarting a quarter later.
6. **Translate dogmatic design language into blast radius.** `Obviously we need an event bus` → `Fan-out to 3 consumers now, 7 projected by Q4; a direct call crosses a team boundary that has no on-call coupling. Trade-off accepted: eventual consistency, ≤ 2 s lag.`

---

## 4. Verification Checklist

- [ ] **Dogmatic markers removed:** `rg -i "obviously|clearly|trivially|just |simply|of course|everyone knows|needless to say"` over the artifact returns no surviving claim-carrying instance.
- [ ] **Modal fog removed, real uncertainty preserved:** no stacked hedges (`might perhaps maybe`, `could potentially possibly`); every remaining unknown is expressed as a named, testable gap (`not measured`, `not reproduced`, `assumed X`).
- [ ] **Every assertion carries its state and pointer:** each claim is labelled Verified / Observed / Reasoned / Unknown and names its evidence (file:line, command, exit code, measurement, or `no data`).
- [ ] **Falsifiability gate passed:** for each sentence — could a reader disprove it with the resources named in it? If not, the sentence has been softened, labelled as preference, or deleted.
- [ ] **Severity is quantified, not asserted:** every `critical` / `should be fine` / `low impact` is backed by a number or a stated blast radius, and disclosure-grade text contains no minimizing framer without its measurement.