# Why PR Signals?

If your team is using AI coding agents, you've probably noticed something:

**the bottleneck isn't writing code anymore — it's reviewing it.**

We hit this wall hard. A small team, 15+ PRs a day, most of them agent-generated. Code was landing faster than anyone could review it. PRs piled up. Reviews got thinner. Bugs slipped through.

It's easy to frame this as "PRs take too long to review."

That's not quite right.

**The real issue is that the volume and frequency of PRs have fundamentally changed.**
Agentic coding dramatically increases how much code gets produced, and the traditional review model wasn't designed for that level of throughput.

At the same time, most PRs — especially AI-generated ones — are **hard to review efficiently**. They show what changed, but not:

* why it changed
* where the risk is
* what was validated
* where a reviewer should actually focus

So every review starts from scratch. Reviewers — human or agent — have to reconstruct intent from raw diffs. That doesn't scale.

We realized the solution wasn't to make reviews faster.

**It was to make PRs more focused for humans to review.**

Here's the other thing we noticed: if you're building an agentic SDLC pipeline, your agents are already handling a huge chunk of what used to eat human review time. Policy adherence. Code architecture. Test coverage gaps. Documentation. Code quality heuristics. All the mechanical stuff that used to generate 80% of PR comments is now caught before a human ever sees the code.

That changes what human review is *for*.

If the machine concerns are handled upstream, human reviewers shouldn't be re-checking them. They should be doing the thing only humans are good at: evaluating whether the change makes sense in the domain, whether the mental model holds, whether the design decision is right for this system at this moment. That's judgment work, not inspection work.

PR signals exist to support exactly that shift. They give reviewers the context they need to focus on domain correctness and design intent — not formatting nits or missing test cases that an agent already caught.

---

# The shift: PRs as signals, not diffs

A PR should carry enough structure that a reviewer can quickly answer:

1. What changed?
2. Why did it change?
3. Where is the risk?
4. How was it validated?
5. What kind of review is actually needed?

The core idea:

> A PR should expose **review signals**, not just a diff.


## 1. Intent signals

Every reviewer's first question is "what is this even trying to do?" You'd be surprised how many PRs — especially agent-generated ones — don't answer it clearly.

### Signals to include

* **Purpose**: the problem being solved
* **Outcome**: expected result or user-visible/system-visible change
* **Change type**:
  * bug fix
  * feature
  * refactor
  * cleanup
  * infra
  * test-only
* **Scope**:
  * narrow/local
  * cross-cutting
  * architectural

### Why this matters

Agents are bad at inferring intent from raw diffs when the code is noisy. If the PR states intent explicitly, the agent can compare:

* stated purpose
* actual changed files
* actual behavior implied by code/tests

That makes it much easier to detect mismatch.

---

## 2. Mental-model signals

A list of changed files isn't a story. Reviewers need to understand *how* the change works, not just *where* it lives.

### Signals to include

* **How the change works at a system level**
* **Entry points affected**
* **Primary modules/components touched**
* **What is intentionally unchanged**
* **Control/data flow summary**

### Example

Instead of:

> Updated service and controller

Use:

> Validation now happens before request persistence, so invalid submissions fail early and never reach downstream processing.

### Why this matters

This lets an agent:

* map changed files into a coherent story
* spot missing layers
* notice when the implementation does not match the described flow

---

## 3. Risk signals

This is the big one. If your team only adopts one category of PR signals, make it this one.

A PR should declare its own risk — because the alternative is hoping the reviewer catches it by reading every line. At 15 PRs a day, that's not realistic.

### Signals to include

* **Risk level**: low / medium / high
* **Risk categories**:

  * business logic
  * data model
  * migration
  * API contract
  * performance
  * auth/security
  * concurrency
  * infra/deployment
  * backward compatibility
* **Blast radius**
* **Failure modes**
* **Edge cases**
* **Known limitations**

### Why this matters

You want agents to help route attention, not just summarize code. Risk signals let an agent ask:

* does this really look low risk?
* are there files touched that suggest hidden risk?
* are the declared edge cases actually covered?

This is where "avoid rubber stamp" becomes real.

---

## 4. Review-targeting signals

Most PRs bury the important stuff in a sea of noise. Generated types, lockfile changes, formatting diffs — they all compete for attention with the actual logic change. The PR should tell reviewers exactly where to look and what to skip.

### Signals to include

* **Review hotspots**: specific files/functions/modules that deserve attention
* **Mechanical/noise areas**: generated files, renames, formatting, lockfiles
* **Suggested review order**
* **Questions for reviewer**

### Example

* Focus on `EligibilityService.evaluate()`
* Skim `api-types.ts` because it is generated
* Review tests after reading the service logic
* Need reviewer judgment on whether fallback behavior is acceptable

### Why this matters

This allows agent reviewers to:

* prioritize deeper analysis on important areas
* skip low-value areas
* produce more targeted comments
* avoid wasting tokens and time on noise

---

## 5. Validation signals

"Does this actually work?" It's a simple question, but most PRs make you dig for the answer. A PR should make it obvious how correctness was established — and be honest when it wasn't.

### Signals to include

* **Tests added/changed**
* **Kinds of validation performed**

  * unit
  * integration
  * end-to-end
  * manual
  * lint/typecheck
  * benchmark/perf
* **How to test locally**
* **Expected observable behavior**
* **Screenshots / sample requests / before-after examples**, when useful

### Why this matters

Agents reviewing PRs should not just inspect code. They should inspect whether the validation story matches the change.

For example, an agent can detect:

* behavioral change with no behavioral tests
* API change with no contract examples
* migration with no rollback plan
* UI change with no screenshots or interaction steps

---

## 6. Traceability signals

Why does this PR exist? What ticket, bug report, or design decision spawned it? Without this link, reviewers are flying blind — and agents have no way to check whether the implementation actually matches the original ask.

### Signals to include

* **Linked issue / ticket / spec**
* **Acceptance criteria**
* **Related design decision**
* **Follow-up work intentionally deferred**
* **Non-goals**

### Why this matters

Agents can compare the PR against the original requirement and identify:

* incomplete implementation
* scope drift
* unrelated changes
* hidden future work not called out

This is especially useful when AI-generated code tends to "keep going" beyond the intended scope.

---

## 7. Change-shape signals

Some of the best mismatch detectors are purely structural. These signals can be derived automatically from the PR itself — no author input needed.

### Signals to include

* size of diff
* number of files changed
* number of modules/systems touched
* test-to-code ratio
* deletion vs addition ratio
* config/schema/migration presence
* public API changes
* dependency changes

### Why this matters

Agents can use these as heuristics:

* "small PR" but touches 14 subsystems
* "refactor" but adds behavior and migration
* "cleanup" but changes API schema
* "low risk" but modifies auth, config, and persistence in one PR

These are excellent mismatch detectors.

---

## 8. Operational signals

Code that passes tests can still break production. If a change affects real running systems — deployments, configs, migrations, feature flags — the PR needs to say so explicitly.

### Signals to include

* rollout plan
* feature flag status
* migration sequencing
* monitoring/logging changes
* rollback path
* config/env var changes
* downstream dependency impact

### Why this matters

This lets agents review not only code correctness, but deployability and operational safety.

A lot of AI-generated PRs may look correct locally but be weak operationally.

---

## 9. Confidence and uncertainty signals

This might be the most underused signal category — and one of the most valuable. AI-generated code tends to project confidence it hasn't earned. PR authors (human or agent) should be explicit about what they don't know.

### Signals to include

* **Confidence level**
* **What is assumed**
* **What is not fully validated**
* **What needs human judgment**

### Why this matters

Agents should be encouraged to surface uncertainty, and PR authors should too.

Examples:

* "Validated on happy path, not load tested"
* "Backward compatibility assumed; not tested with older clients"
* "Need reviewer opinion on whether this belongs in service or domain layer"

This avoids fake certainty, which is a big failure mode in AI-heavy workflows.

---

# Why we built this skill

This is exactly why we created **ver-pr-writer**.

We took everything above — the nine signal categories, the mismatch heuristics, the five core buckets — and distilled it into an agent skill that does the work for you. When you run it against a diff, it inspects the changes, classifies risk, surfaces uncertainty, and produces a structured PR description that both humans and agents can act on immediately.

We built it because we needed it. Our team was drowning in PRs and reviews were becoming the weakest link in an otherwise fast workflow. Once we started using structured signals, reviews got faster, more targeted, and more honest. Rubber stamps dropped. Real bugs got caught earlier.

We're sharing it because we know we're not the only team hitting this wall. If you're shipping AI-generated code at any meaningful pace, you're probably feeling it too. And if your pipeline already has agents enforcing formatting, linting, and code quality — you've already done the hard part. The next step is making sure human review time is spent on the things agents *can't* judge: domain fit, design intent, and whether the mental model actually holds.

The five buckets that matter most:

### A. Intent

What is this PR trying to do?

### B. Risk

What could go wrong, and how serious is it?

### C. Evidence

What tests or validation support the change?

### D. Focus

Where should review attention go?

### E. Operations

How does this behave when merged and deployed?

If a PR carries all five, an agent can do much more than summarize:

* classify the PR
* route it
* challenge weak claims
* identify missing evidence
* focus human review on the right places

---

# Start here

You don't have to adopt everything at once. This is the smallest set of signals that makes a real difference:

## Required author-provided signals

* Purpose
* Mental model
* Main changes
* Risk level
* Risk areas
* Review hotspots
* How to test locally
* Expected behavior
* Non-goals
* Rollout / migration notes if applicable

## Auto-derived signals

* Diff size
* Files/modules touched
* Test changes present or absent
* Public API/schema/config changes
* Migration/dependency changes
* Generated/mechanical files
* Ownership/domain areas touched

That combination is enough to transform your review process — for both humans and agents. The days of reading every line of every PR don't scale. Better signals do.
