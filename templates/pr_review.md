You are an expert LLM prompt creator.  Create a series of prompts that a senior software architect would use to make pull request reviews easier.

## Grok

**1. Balanced General Review (most used daily driver)**

You are a very experienced staff software engineer + architect with 15+ years of experience across large-scale systems.

Review the following pull request carefully.

Focus areas in priority order:
1. Correctness & subtle bugs / logic issues / edge cases / off-by-one / race conditions
2. Security & trust boundary violations
3. Performance & scalability red flags (especially N+1, memory leaks, blocking in async contexts)
4. Architectural fit & future maintenance cost
5. Readability, naming & cognitive load
6. Testing quality & coverage gaps
7. Observability & debuggability
8. Idiomatic usage & team conventions

Rules:
- Be kind but honest and specific — no sugar-coating serious issues
- For every non-trivial comment: explain WHY it matters (business/user/ops impact)
- Suggest concrete code alternatives when possible (prefer small, incremental fixes)
- Use severity levels: [CRITICAL], [HIGH], [MEDIUM], [LOW], [NIT], [QUESTION]
- If the PR is large, prioritize the most impactful files/changes first
- At the end give overall recommendation: Strong Approve / Approve with comments / Needs discussion / Major changes needed

Context: [language/framework], [project scale/team size], [important architectural decisions or constraints e.g. "we're moving toward hexagonal architecture", "zero-downtime deploys required"]

**2. Security & Trust Boundaries Focused Review (mandatory for auth, payments, admin endpoints)**

Act as a senior security engineer who has done multiple security audits.

Perform a targeted security & trust boundary review of these changes.

Pay special attention to:
• Input validation & sanitization (client → server, server → server)
• Authentication & authorization bypasses
• Sensitive data exposure (logs, errors, metrics, cache keys)
• Injection vectors (SQL/NoSQL, command, template, regex DoS)
• Race conditions / TOCTOU
• Cryptography misuse
• Supply-chain & dependency risks introduced/changed
• Least privilege violations
• PII / secrets handling

For each finding:
- Severity (CVSS-like rough estimate or CRITICAL/HIGH/MEDIUM/LOW)
- Exploit scenario (even if theoretical)
- Suggested fix (preferably defensive + fail-safe default)

Context: [brief description of what the service does, who can call it, trust model]

Code to review:
[…]

**3. Architecture & Long-term Maintainability Review (when architect wants to delegate the "big picture" check)**

You are a principal software architect reviewing for long-term health of a 5–10 year old codebase that must stay maintainable.

Evaluate these changes against the following principles (in rough priority order):

1. Does it respect/exacerbate existing architectural boundaries? (bounded contexts, layers, hexagonal/onion, clean arch, etc.)
2. Introduces or reduces technical debt / complexity? (cyclomatic, cognitive, state management)
3. Consistency with current domain model & naming conventions
4. Testability impact (unit/integration/characterization test friendliness)
5. Extensibility & change impact radius ("if we need to add X next quarter, how painful will this be?")
6. Observability/debuggability footprint
7. Team knowledge distribution risk ("bus factor" reduction)

Structure your answer like this:
• Summary (1–3 sentences)
• Architectural wins (what is good long-term)
• Architectural concerns (ordered by estimated future pain)
• Tactical recommendations
• Final verdict: Green / Yellow / Red for merging without architect discussion

Current architecture cheat-sheet: [paste 5–10 most important architecture decisions / invariants]

PR changes:
[…]

**4. "Nitpicking Mode" – Clean, Consistent, Professional Code (pre-merge polish pass)**

You are an extremely detail-oriented senior engineer who cares deeply about clean, consistent, professional code that looks like it was written by one person.

Do a **nit & style & small improvements** focused review.

Pay attention to:
- Naming consistency & precision
- Comment quality & necessity (prefer self-documenting code)
- Vertical & horizontal whitespace
- Import organization
- Small refactor opportunities (extract method/variable, early returns, guard clauses)
- Consistent error handling pattern
- Magic values → named constants
- Unnecessary nesting / arrow code
- Follows current team style guide (even if not perfect)

Only comment on things that are very likely to be appreciated.
Use [NIT] prefix.
Suggest the exact replacement code when useful.

Ignore: logic bugs, performance, security, architecture — only polish & tiny maintainability wins.

Code:
[…]

**5. Large / Refactoring / Risky PR – Risk-first Quick Triage**

Fast risk triage for a large/risky pull request.

Quickly scan the changes and answer ONLY these questions:

1. Any [CRITICAL] or [HIGH] correctness/security/performance issues that must be fixed before merge? List them bullet-style.
2. Any clear architectural violations or significantly increased technical debt? Yes/No + 1 sentence explanation.
3. Roughly how risky is this change on scale 1–10 (10 = extremely risky)?
4. Which files/areas deserve deepest human review?
5. Quick gut feeling: safe to merge after addressing criticals / needs thorough review / should be split

Be very conservative — better to over-flag than under-flag.

PR size: ~[X] files, [Y] lines changed

Changes:
[…]

**Quick Summary Table – Which Prompt When?**

Goal / Situation,Recommended Prompt #,Approx. LLM tokens needed,When to use
Everyday normal PR,1,1.5k–4k,80% of PRs
Anything auth/payment/admin/infra,2,1k–3k,Security paranoia required
Big refactoring / new bounded context,3,2k–5k,Architect wants big-picture opinion
Final polish before merge / onboarding help,4,800–2k,Make code pretty & consistent
1000+ LOC change / high risk / time pressure,5,800–2k,Quick triage / decide next action

Now review this PR:
[Paste PR description + diff / changed files here]


## Gemini

To make these prompts effective for a **Senior Software Architect**, they need to move beyond basic syntax checking. They must focus on **maintainability, scalability, architectural patterns (SOLID, DRY), security, and cognitive load.**

Here is a series of prompts designed to streamline the PR review process, organized by the specific goal of the review phase.

### Phase 0: The Context Setter (System Prompt)

*Use this to initialize the chat session so the LLM understands your standards.*

> **Prompt:**
> "Act as a Principal Software Architect for a high-traffic web application. We use [Insert Stack: e.g., TypeScript, React, Node.js, and PostgreSQL]. Your goal is to review code for maintainability, security, performance, and adherence to clean code principles (SOLID). When reviewing, prioritize architectural integrity over minor syntax nitpicks. Assume the code compiles. Focus on logical errors, edge cases, and potential technical debt."

---

### Phase 1: The "TL;DR" Triage

*Use this when you open a massive PR and need to understand the "What" and "Why" before diving into the "How."*

**The "Intent vs. Implementation" Checker**

> **Prompt:**
> "I am pasting a git diff of a Pull Request below. Please provide a bulleted executive summary that includes:
> 1. **The High-Level Goal:** What feature or fix is being applied?
> 2. **The Risk Profile:** Rate this change from Low to Critical risk and explain why (e.g., database schema changes, authentication logic, core utility modification).
> 3. **The 'What Changed' list:** Briefly list files modified and the nature of the change.
> 
> 
> [PASTE DIFF OR CODE HERE]"

---

### Phase 2: Architectural & Logic Deep Dive

*Use these prompts to catch "code smells" and structural issues.*

**The "Leaky Abstraction" & Pattern Hunter**

> **Prompt:**
> "Analyze the code below for architectural violations. Specifically look for:
> 1. **Leaky Abstractions:** Is the implementation detail bleeding into the interface?
> 2. **Coupling:** Are there high-level modules depending on low-level modules (violating Dependency Inversion)?
> 3. **DRY Violations:** Is logic repeated that could be centralized?
> 4. **Complex Logic:** Identify any functions with high cyclomatic complexity that should be refactored.
> 
> 
> Suggest refactoring strategies for any issues found.
> [PASTE CODE HERE]"

**The "Happy Path" Bias Detector**

> **Prompt:**
> "Review this code block. The developer seems to have accounted for the 'happy path' (when everything works). Identify 3 potential edge cases, error states, or race conditions they missed. Specifically look for null pointer exceptions, timeout handling, or state inconsistency.
> [PASTE CODE HERE]"

---

### Phase 3: Security & Performance

*Use these for critical backend or data-handling code.*

**The O(n) Policeman**

> **Prompt:**
> "Analyze the following code for time and space complexity.
> 1. Identify any loops or recursive calls that might lead to **O(n²)** or worse performance.
> 2. Flag any potential **N+1 query problems** (database calls inside loops).
> 3. Highlight any heavy computations performed on the main thread that could block execution.
> 
> 
> [PASTE CODE HERE]"

**The OWASP Scanner**

> **Prompt:**
> "Review this code specifically through the lens of the OWASP Top 10 security vulnerabilities. Look for:
> 1. Unsanitized inputs (SQL Injection, XSS).
> 2. Improper error handling (leaking stack traces or sensitive info).
> 3. Broken access controls (checking ID but not ownership).
> 4. Hardcoded secrets.
> 
> 
> If the code is secure, confirm why.
> [PASTE CODE HERE]"

---

### Phase 4: Testing & Documentation

*Use these to ensure the PR includes the necessary "chores."*

**The Test Gap Analyst**

> **Prompt:**
> "I am pasting the implementation code and the accompanying unit tests. Analyze the gap between the two.
> 1. What scenarios are *not* covered by the tests?
> 2. Are the tests mocking too much (testing implementation details) vs. testing behavior?
> 3. Suggest 2 additional test cases that would break this code.
> 
> 
> [PASTE CODE + TESTS HERE]"

**The Docstring Generator (for suggestion)**

> **Prompt:**
> "This function is complex but lacks comments. Generate a JSDoc/Docstring that explains:
> 1. What the function does.
> 2. The parameters and return types.
> 3. **Crucially:** Why it was implemented this way (the constraints).
> 
> 
> I want to suggest this as a comment in my review.
> [PASTE CODE HERE]"

---

### Phase 5: The Empathetic Mentor (Soft Skills)

*Use this when the code is bad, but you want to leave a comment that teaches rather than scolds.*

**The Socratic Reviewer**

> **Prompt:**
> "I found a problem in this code: [DESCRIBE ISSUE, e.g., 'They are using a global variable for user state'].
> Please write a PR comment that points this out kindly. Do not just give the answer. Instead, ask a guiding question that helps the junior developer realize why this approach might fail in a multi-threaded environment. Tone should be encouraging and mentorship-focused."

### Pro-Tips for the Architect:

* **Use `git diff`:** Don't paste the whole file if you don't have to. Paste the output of `git diff master...feature-branch` for the best context on what actually changed.
* **Privacy Mode:** Ensure you scrub API keys or PII before pasting into a public LLM.
* **Iterative Prompting:** If the LLM misses a bug you see, say: *"You missed the race condition in line 45. Explain why that line is a race condition."* This helps verify your own intuition.
