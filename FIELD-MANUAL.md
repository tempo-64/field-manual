# The Solo Builder's Field Manual

Practical engineering with AI coding agents — grounded in evidence, not vibes.

## What This Is

This is a field manual for software engineers who build real products with AI coding agents. Not a tutorial on which buttons to click. Not a methodology framework with templates and ceremonies. A concise set of practices that actually work, drawn from research, practitioner experience, and pattern observation across thousands of projects.

These practices are **agent-agnostic** — they apply whether you're using Claude Code, Cursor, Copilot, Codex, Windsurf, or whatever ships next month. They're also **stack-agnostic** — the principles transfer across languages, frameworks, and domains.

**Who this is for:** A software engineer (any level) working with one or more AI coding agents to build software that other people will use. "Solo builder" means you're the human — you might have multiple agents, but you're the one with the vision, the taste, and the accountability.

**Who this is not for:** Teams building AI agents themselves, managers evaluating AI tools, or people looking for a specific tool tutorial.

## Where These Practices Come From

Every practice in this guide meets at least one of these bars:

- **Independently converged** — multiple practitioners discovered it separately (strongest signal)
- **Empirically researched** — backed by studies with data, not just anecdotes
- **Observed pattern** — consistent across many projects but not yet formally studied

Opinions are labeled as opinions. Preferences are left out. When evidence is thin, that's noted.

Key sources: Anthropic's engineering research and agentic coding studies (~400K sessions analyzed), CodeScene's peer-reviewed code health research (39 production codebases), Simon Willison's Agentic Engineering Patterns guide, Addy Osmani's practitioner workflow documentation, and field experience across production projects. Full references at the end.

---

## 1. The Mental Model

### You're the architect. The agent is the builder.

Anthropic's study of ~400,000 coding sessions found that in a typical session, **the human makes most of the planning decisions** (what to do) and **the agent makes most of the execution decisions** (how to do it). This isn't a limitation to work around — it's the model that works.

The same study found that **domain expertise, not coding background**, is the primary predictor of session success. Every major occupation succeeds at nearly the same rate as software engineers. The more domain expertise you bring, the more work the agent does per instruction, and the more likely the session succeeds.

What this means in practice:

- **Your value is in the what and the why** — which features matter, what the user needs, what trade-offs to make, when to say no
- **The agent's value is in the how** — consistent implementation, breadth of knowledge, tireless execution, catching what you skip
- **Neither side is a rubber stamp** — the agent should push back when your idea has problems, and you should push back when the agent's implementation has problems

### The product gets better because both sides push back

This is not "human supervises bot" or "tool assists human." The best results come from honest friction. If the agent thinks an approach is wrong, it should say so. If you think the agent over-engineered something, say so. Polite agreement produces mediocre work.

Anti-pattern to watch for: **performative agreement**. When you give feedback, some agents will immediately agree and implement without evaluating whether the feedback is technically correct. Good agents push back when your feedback would make things worse.

---

## 2. Set Up for Success

### The project operating manual

Every project needs a file that tells the agent how to work in this specific codebase. Most tools call this `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, or similar — the name varies but the purpose is universal. By early 2026, this format is read natively by Claude Code, OpenAI Codex CLI, Cursor, Aider, Devin, GitHub Copilot, Gemini CLI, Windsurf, and Amazon Q.

**What makes a good operating manual:**

Think of it as what you'd tell a smart new team member on their first day. Not everything about the project — just what they need to make good decisions without asking you every five minutes.

Include:
- **Environment** — language versions, build tools, how to run tests, how to run the app
- **Conventions** — code style, commit format, file organization, naming patterns
- **Architecture** — where things live, how the system is structured, key relationships
- **Security policy** — non-negotiable rules as specific constraints, not vague principles
- **Domain context** — what the project does, who it's for, key terminology

Leave out:
- Things the agent can derive from reading the code
- Git history and recent changes (that's what `git log` is for)
- Ephemeral state or in-progress work
- Every convention ever discussed — keep it to the 20 that matter most

**Keep it concise.** A 500-line file with every rule ever discussed is worse than a 100-line file with the rules that actually matter. The agent reads the entire file on every interaction — noise dilutes signal.

**It's a living document.** Update it when you discover a new pattern that matters, and remove things that no longer apply. A stale operating manual gives the agent wrong instructions with high confidence.

**Context over rules.** "Always use semicolons" is a rule. "This project uses ESLint with the Airbnb config; semicolons are required" is context. Context helps the agent make judgment calls about cases you didn't anticipate. Rules only cover the exact cases you thought of.

### Security defaults from the start

Multiple sources converge on this: security cannot be bolted on later when working with agents.

Willison describes the **"Lethal Trifecta"** — agent access to private data + processing untrusted external content + ability to communicate externally = zero-click attack surface. Any single component is manageable, but all three together create real risk.

Anthropic's 2026 Agentic Coding Report makes it concrete: an agent writing 1,000 PRs/week with a 1% vulnerability rate creates 10 new vulnerabilities weekly.

What this means for your operating manual — encode security as **specific constraints**, not principles:

```
# Security (non-negotiable)
- subprocess: always list form, never shell=True
- TLS: never verify=False — fail hard when certs are missing
- Paths: validate user-supplied components before joining (traversal prevention)
- Secrets: never hardcode — use env vars or secret management
- YAML: always safe_load(), never load()
- SQL: always parameterized queries, never string interpolation
- User input: validate at system boundaries
```

Specific rules are enforceable. "Be secure" is not.

### Git as your safety net

Agents can generate a lot of code fast. Without commits as checkpoints, a wrong turn means losing everything since the last save.

- **Commit before asking the agent to make changes** — if the change goes wrong, you can revert
- **One logical change per commit** — makes rollback granular
- **Branch per feature or task** — isolates experimental work
- **Conventional commits** (`feat:`, `fix:`, `refactor:`) — not ceremony, rollback indexing

---

## 3. Before You Write Code

### Specs before code

This is the single most universally agreed-upon practice across every source. Osmani, Willison, Anthropic, CodeScene, Donner — everyone who builds seriously with agents converges here.

The reasoning is empirical: NxCode found that **10 minutes spent writing a clear spec saves hours of rework.** Anthropic's harness research found that agents given only high-level prompts try to "one-shot" complex projects and fail. Agents given structured specs build incrementally and succeed.

A spec doesn't need to be long. It needs to answer:

1. **What** — exactly what should this do, from the user's perspective?
2. **Where** — which files, modules, or systems does this touch?
3. **How to verify** — how will you know it works?
4. **What it doesn't do** — explicit scope boundaries prevent agent drift

The spec exists so that both you and the agent are building toward the same target. Without it, the agent builds what it thinks you meant, and you discover the gap after the work is done.

### First slice before breadth

Prove the architecture works with one vertical slice before expanding features. One scenario, one user journey, end to end. This practice appears independently in proof-driven development, lean product thinking, and Anthropic's harness architecture (which implements one feature per session, verifying before moving on).

The instinct to build broadly first is strong — "let me set up the whole project structure, all the models, all the routes." Resist it. Breadth before proof is noise. An agent will happily scaffold an impressive-looking shell that avoids the hardest product question.

### Design the verification alongside the feature

Don't plan the feature first and figure out testing later. Design them together. Specify what "done" looks like — in terms of tests, in terms of observable behavior, in terms of what a user should see — before writing the implementation.

This front-loads the thinking about edge cases, failure modes, and acceptance criteria. It also gives you a concrete stopping point, which matters with agents that will happily keep building forever if you don't define "done."

---

## 4. The Build

### Small, verifiable steps

Every source converges here. Osmani: "LLMs do best when given focused prompts — implement one function, fix one bug, add one feature at a time." Anthropic's harness research: agents that try to do too much at once fail. CodeScene: smaller changes in healthy code produce fewer defects.

The practical heuristic: **if you can't verify a change in under two minutes, break it smaller.**

This also protects you from the most expensive failure mode — the agent builds something large, you don't verify along the way, and you discover at the end that the foundation was wrong. Small steps make wrong turns cheap.

### Plan when choices matter, just build when they don't

Most builders either never use plan mode (and waste time on wrong approaches) or always use plan mode (and waste time planning things that should just be done).

The heuristic: **use plan mode when there are multiple reasonable approaches and the wrong choice would be expensive to reverse.** "Fix the typo on line 42" doesn't need a plan. "Add authentication" needs alignment on the approach before code starts.

Osmani's workflow makes this explicit: Explore → Plan → Code → Commit. But the Explore and Plan steps scale with the ambiguity of the task. A clear, scoped task can skip straight to Code.

### TDD works better with agents, not worse

Willison advocates TDD specifically adapted for agent workflows. The pattern:

1. Write the test (or have the agent write it from your spec)
2. Verify the test captures the behavior you actually want
3. Let the agent implement until the test passes
4. Review the implementation

This works better with agents for a specific reason: **the test is an unambiguous, machine-checkable definition of "done."** Without a test, "done" is the agent's judgment — and Anthropic's research shows agents reliably skew positive when evaluating their own work. With a test, "done" is a pass/fail that neither side can talk their way around.

Write tests that verify **behavior**, not implementation details. Tests written to hit a coverage number are paperwork. Tests that describe what the system should do from the outside are specifications.

### Build complete, not fast

When requirements come from real experience — months of validation, caught bugs, proven workflows — they aren't hypothetical. They're the spec. Build them complete.

The anti-pattern: agents (and some methodologies) will suggest trimming scope to a "minimal" version. This is appropriate when you're exploring an unknown problem space. It's inappropriate when you know exactly what's needed and the requirements are the distilled result of real-world use.

Build each feature completely — with error handling, edge cases, tests, documentation. "We'll polish it later" is a fiction. The polish phase never arrives. Ship the feature done or don't ship it.

### Commit as checkpoints

Commits aren't ceremony. They're save points.

- **Commit after each working step** — before asking the agent to make the next change
- **Meaningful messages** — "feat: add user authentication with JWT" tells you what to revert to. "WIP" tells you nothing.
- **One logical change per commit** — a commit that mixes a feature, a refactor, and a bug fix is three commits that can't be independently reverted

---

## 5. Working Together

### Give full errors, not summaries

When something fails, paste the full error message and stack trace. Don't say "it didn't work" or "there's an error." The single biggest time-waster in agent sessions is vague error descriptions that force the agent to guess at what went wrong.

The agent can process a 50-line stack trace in milliseconds. You summarizing it to one sentence loses the diagnostic signal.

### Feedback shapes future behavior

When the agent does something wrong, say **why**, not just "no." When it does something right in a non-obvious way, say so — validated approaches are as important to capture as corrections.

Specific, directional feedback: "Don't add a try/catch here — this function should let errors propagate to the caller because the caller has the context to handle them meaningfully."

Vague feedback: "That's wrong." (Forces the agent to guess what's wrong and why.)

If you find yourself giving the same feedback twice, put it in your operating manual. The correction should persist, not repeat.

### When to review closely vs. trust

Not every line needs the same scrutiny. Calibrate your review to the risk:

- **New territory** (unfamiliar library, security-sensitive code, complex business logic) — review every line
- **Established patterns** (another CRUD endpoint matching five existing ones) — spot-check
- **Pure boilerplate** (test scaffolding, import reordering, formatting) — skim

The calibration should shift over a project's lifetime: tight review early (while patterns are being established) → relaxed in the middle (patterns are proven) → tight again when entering new territory.

The one constant: **always review the diff, never just the agent's summary.** The summary describes intent. The diff describes reality. They diverge more often than you'd expect.

### Show, don't tell

When you want the agent to follow a pattern, pointing at an existing example in the codebase is 10x more effective than describing the pattern in prose.

"Follow the pattern in `src/api/users.ts`" beats a paragraph explaining the API pattern. The agent will read the file, internalize the structure, and replicate it. This works because agents are excellent pattern matchers — showing them a concrete example is playing to their strength.

### Don't accumulate decisions in conversation

If something important is decided — an architectural approach, a naming convention, a scope boundary — put it in a file. Conversation context gets compressed, summarized, or lost across sessions. Files are permanent.

The agent can always re-read a file. It can't always recall a decision from 50 messages ago, especially across session boundaries. If a decision matters tomorrow, write it down today.

---

## 6. Quality and Verification

### Automated gates are force multipliers

CodeScene's peer-reviewed research (39 production codebases) found that **healthy code enables 124% faster development and contains 15x fewer defects.** When AI agents operate on unhealthy code, defect risk increases by at least 60%. Agents need code health above 9.4/10 to keep bug rates in check; the industry average is 5.15.

What this means in practice: **invest in your automated quality pipeline before investing in agent sophistication.** Tests, linters, type checkers, and formatters provide backpressure that catches agent mistakes automatically. The loop becomes: agent writes code → automated tools catch issues → agent fixes them → you oversee the direction.

Osmani describes this as "having an extremely fast junior dev whose work is instantly checked by a tireless QA engineer."

### Separate who builds from who evaluates

Anthropic's harness research found a critical pattern: **agents reliably praise their own work**, even when quality is mediocre. Without a separate evaluation signal, you get agents that ship at 30% complete with full confidence.

Practical applications:

- **Code review**: the agent that wrote the code should not be the sole reviewer. Use a separate review pass — whether that's a different agent, a code review bot, or your own eyes on the diff.
- **Test evaluation**: if the agent wrote both the code and the tests, verify the tests actually test something meaningful. Tests written to make the suite pass are paperwork.
- **Completion assessment**: don't ask the agent "are you done?" It will say yes. Define done as observable criteria (tests pass, feature works in the browser, deployment succeeds) and check those criteria yourself.

### Live verification is the definition of done

Type checking and test suites verify code correctness. They do not verify feature correctness. A feature is done when you've seen it work — in a browser, in a terminal, against real data.

This is the gap where the most bugs hide: code that passes all tests but doesn't actually do what the user needs. The agent can't close this gap for you. You have to use the thing.

### Coverage honesty

"Found nothing" is not the same as "didn't look." When running quality checks:

- **Full coverage**: all expected checks ran and completed
- **Partial**: some checks ran, gaps are identified
- **Degraded**: checks failed to run, coverage gaps explain what wasn't checked

A clean report with degraded coverage is less trustworthy than a report with findings but full coverage. Track what you checked, not just what you found.

---

## 7. Scaling Across Sessions

### The session continuity problem

Anthropic's harness research names this directly: "The core challenge of long-running agents is that they must work in discrete sessions, and each new session begins with no memory of what came before."

Every session start is a cold start. The agent doesn't remember what you discussed yesterday, what approach you agreed on, or what's half-finished. Without a bridging mechanism, you waste the first 10 minutes of every session re-establishing context — or worse, the agent proceeds from stale assumptions.

### Structured artifacts bridge sessions

Anthropic's solution: a progress file alongside git history. The agent reads the progress file, checks recent git log, and understands current state without being told.

The general principle: **anything the agent needs to know across sessions should be in a file, not in your memory.** This includes:

- Current state of work (what's done, what's in progress, what's next)
- Decisions made and why
- Known issues and workarounds
- The operating manual (CLAUDE.md / AGENTS.md) itself

### Docs in the repo, not in conversation

For multi-session projects, write reference documentation in the repo before building. Context windows compress and sessions end. A spec that exists only in conversation will be lost or distorted.

Chunked docs in the repo enable progressive disclosure — load only what's needed for the current task, not the entire project history. This matters more as projects grow: a 20-file project can fit in context, a 200-file project cannot.

### Re-orient before acting

Every session should start with orientation: read the current state, check what's changed, verify the baseline still works. Anthropic's harness architecture makes this explicit — every session follows: **Orient** (read progress, task list, recent git history) → **Verify baseline** (test that existing functionality still works) → **Build** (implement the next thing).

Skipping orientation is how you get an agent that "fixes" something that was already working, or builds on top of a broken foundation from the previous session.

---

## 8. Working with Multiple Agents

As projects grow, you'll use multiple agents — a builder, a reviewer, parallel investigators, specialized subagents. These patterns, drawn from multi-agent production work and Anthropic's harness research, help you coordinate them.

### Builder/reviewer separation

The agent that wrote the code should not be the sole evaluator of the code. This is Anthropic's most consistent finding: agents reliably praise their own work. The fix is structural, not prompting — use a separate context, separate prompt, and separate evaluation criteria for review.

The cycle: **build** (agent implements a slice) → **self-test** (agent runs tests) → **review** (different agent or human reviews the diff against the spec) → **human stop** (you decide whether to continue).

### Narrow review scopes

Broad review prompts ("review the whole codebase") time out or produce shallow results on any non-trivial project. Each review should name the exact changed files, the spec they should match, and the specific concerns to check.

Good: "Review the diff in `src/auth.py` and `tests/test_auth.py` against the JWT spec in the design doc."

Bad: "Review all the changes and make sure everything looks good."

### Human stop points

Without explicit stops between slices, agents chain work indefinitely. Each chained step inherits and amplifies any drift from the plan. Build natural stopping points where you evaluate direction, not just correctness.

The stop isn't "does the code work?" — it's "is this still building toward what I want?"

### AI can veto but cannot approve

Agents can reject work that fails invariants — a test suite, a linter, a security check. But final approval to merge and ship requires human judgment. This is the authority boundary: agents enforce constraints, humans make acceptance decisions.

### Drafts vs. canonical artifacts

When agents produce work products (not just code — reports, analyses, decisions), treat them as drafts until a human or policy engine promotes them. The boundary should be structural, not just social: agent output is labeled as draft, and promotion to canonical status is a separate, auditable step.

Without this boundary, AI can effectively self-certify its own work. The draft/canonical split makes human review a part of the system, not just a good intention.

### Parallel investigation, not parallel implementation

Multiple agents investigating different hypotheses simultaneously (debugging, research, code archaeology) is a superpower. Multiple agents implementing code in parallel usually causes problems — merge conflicts, inconsistent patterns, duplicate work.

Use parallelism for divergent work (investigation). Use sequential work for convergent work (implementation).

---

## 9. Anti-Patterns

These are specific failure modes observed across many projects. Each one is tempting, common, and expensive.

### The "just fix it" instinct

When something breaks, the instinct is to tell the agent "fix this error." The agent will apply a patch. The patch will address the symptom. The root cause will remain, and you'll see a different symptom ten minutes later.

**Instead:** investigate why it broke. Have the agent explain the error, trace the cause, and propose a fix that addresses the root. This takes three minutes longer and saves thirty.

### Rubber-stamping

Reading the agent's summary of what it did and saying "looks good." The summary describes intent. The code describes reality. Always read the diff.

This gets harder as you trust the agent more — which is when it matters most. The agent is most likely to introduce subtle bugs in areas you've stopped closely reviewing because "it always gets this right."

### One-shotting complex work

Asking the agent to build an entire feature, page, or system in one request. Anthropic's research documented this failure mode explicitly: agents given high-level prompts try to one-shot and produce incomplete, low-quality results.

**Instead:** break work into steps. Each step should be small enough to verify before moving to the next.

### Narrating instead of doing

"I'm going to read the file, then I'll check the tests, then I'll look at the configuration..." The agent (or you) spending time describing what will be done instead of doing it. Bias toward action.

### Suggesting easier alternatives when things get hard

When the agent hits infrastructure friction — a test that's hard to write, a configuration that's complex, a dependency that's difficult — it may suggest skipping it or doing something simpler. Often the hard thing is the right thing. Push through.

### Silent scope reduction

Cutting features, removing edge cases, or simplifying requirements without explicitly acknowledging the reduction. Deferral is fine — silence about it is not. If scope changes, say what changed and why. This applies to both you and the agent.

### Over-engineering for hypothetical futures

Building abstractions, helpers, or frameworks for requirements that don't exist yet. Three similar lines of code is better than a premature abstraction. The right amount of complexity is what the task actually requires, not what it might someday need.

### Using the agent as a search engine

Don't ask "what does this function do?" when you can read it yourself in 30 seconds. Use the agent for synthesis (processing many files), implementation (generating code), investigation (tracing through a codebase), and transformation (refactoring) — things where the agent's ability to process in parallel provides real leverage. Simple lookups are faster with your own eyes.

### Not establishing patterns early

The first component, module, or API endpoint you build with the agent becomes the implicit template for everything that follows. If the first one is sloppy, everything downstream inherits that sloppiness. Make the first one excellent. Every subsequent one will follow its shape.

### Ignoring the agent's pushback

When the agent says "this approach has a problem," evaluate it on its merits. The agent isn't being difficult — it may have caught something you missed. Dismissing pushback because you want to move fast is how design mistakes get baked in.

The flip side: when the agent pushes back with generic caution ("are you sure you want to..."), that's not the same as specific technical pushback. Generic caution can be overridden. Specific warnings should be investigated.

---

## 9. The Practices at a Glance

A one-page reference for the practices in this guide, ordered by when they matter.

### Before you start
- [ ] Write your operating manual (CLAUDE.md / AGENTS.md) — concise, specific, context over rules
- [ ] Encode security as specific constraints, not principles
- [ ] Set up automated quality gates (tests, linter, type checker, formatter)
- [ ] Establish git workflow (branch per feature, conventional commits)

### Before each feature
- [ ] Write a spec — what, where, how to verify, what it doesn't do
- [ ] Design verification alongside the feature, not after
- [ ] Plan when choices matter, just build when they don't
- [ ] If multi-session: write reference docs in the repo, not in conversation

### During the build
- [ ] Small, verifiable steps — verify each before moving on
- [ ] TDD: write the test, verify it captures intent, let the agent implement
- [ ] Commit after each working step — commits are save points
- [ ] Give full errors, not summaries
- [ ] Show examples, don't describe patterns
- [ ] Put decisions in files, not conversation
- [ ] Build complete — no placeholder anything

### After each step
- [ ] Review the diff, not the summary
- [ ] Run the feature yourself — type checks verify code, not features
- [ ] Separate who builds from who evaluates
- [ ] Update progress artifacts for session continuity

### When using multiple agents
- [ ] Separate who builds from who reviews — different context, different prompt
- [ ] Narrow review scopes — name exact files and spec, not "review everything"
- [ ] Build in human stop points between slices
- [ ] AI enforces constraints; humans make acceptance decisions
- [ ] Use parallel agents for investigation, sequential for implementation

### Every session start
- [ ] Orient: read progress, task list, recent git history
- [ ] Verify baseline: confirm existing functionality still works
- [ ] Then build

---

## Sources

### Research
- [Anthropic — Agentic coding and persistent returns to expertise](https://www.anthropic.com/research/claude-code-expertise) — ~400K sessions, domain expertise as primary success predictor
- [Anthropic — Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) — session continuity, two-agent harness, progress files
- [Anthropic — How AI is transforming work at Anthropic](https://www.anthropic.com/research/how-ai-is-transforming-work-at-anthropic) — internal study, 200K transcripts, supervision paradox
- [Anthropic — 2026 Agentic Coding Trends Report](https://resources.anthropic.com/2026-agentic-coding-trends-report) — industry trends, security at scale
- [CodeScene — Agentic AI Coding: Best Practice Patterns](https://codescene.com/blog/agentic-ai-coding-best-practice-patterns-for-speed-with-quality) — peer-reviewed code health research, 39 codebases, 60% defect increase in unhealthy code

### Practitioner Guides
- [Simon Willison — Agentic Engineering Patterns](https://simonwillison.net/2026/Feb/23/agentic-engineering-patterns/) — TDD with agents, lethal trifecta, domain knowledge hoarding
- [Addy Osmani — My LLM coding workflow going into 2026](https://addyosmani.com/blog/ai-coding-workflow/) — iterative chunks, review discipline, automated gates
- [Nx Blog — A Practical Guide on Effective AI Use](https://nx.dev/blog/practical-guide-effective-ai-coding) — AI as peer programmer
- [Edward Donner — AI Coder: Vibe Coder to Agentic Engineer](https://edwarddonner.com/2026/02/17/ai-coder-vibe-coder-to-agentic-engineer/) — progression framework, multi-agent patterns

### Community Resources
- [BuildBetter — AGENTS.md Complete Guide for Engineering Teams](https://blog.buildbetter.ai/agents-md-complete-guide-for-engineering-teams-in-2026/) — operating manual as universal standard
- [Wes McKinney (MotherDuck) — Vibe Coding Is Dangerous, Agentic Engineering Isn't](https://motherduck.com/blog/vibe-coding-dangerous-agentic-engineering-wes-mckinney/) — scope, design, taste, conceptual integrity
- [Andrej Karpathy](https://x.com/karpathy) — original "vibe coding" and "agentic engineering" distinctions

---

## License

This work is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Share it, adapt it, use it. Attribution appreciated.
