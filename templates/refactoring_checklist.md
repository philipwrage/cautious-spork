To avoid being overwhelmed by "technical debt paralysis," you need a workflow that treats refactoring like a surgical strike rather than a massive invasion. This workflow follows the **Red-Green-Refactor** cycle but adapted for legacy code (often called the **"Boy Scout Rule"**—leave the campground cleaner than you found it).

### **The "Surgical Strike" Refactoring Workflow**

Follow these steps for **one** function or **one** small module per day.

---

#### **Step 1: The "Box-In" (Morning)**

Before changing a single line of code, create a safety net so you know if you've broken the "contract" of the function.

* [ ] **Identify the Target:** Pick one long function (100+ lines) that is currently being modified for a business task.
* [ ] **Generate Characterization Tests:** Use the LLM to generate a Pytest file that passes in various inputs and asserts the *current* outputs.
* [ ] **Run the Tests:** Ensure they pass. If they fail, your understanding of the function is wrong, or the function is non-deterministic.
* [ ] **Commit:** Commit these "locking" tests to a temporary branch.

#### **Step 2: The Decomposition (Mid-Day)**

Now that you are safe, break the "God Function" into smaller pieces.

* [ ] **Extract Pure Logic:** Identify parts of the function that only do math, data transformation, or validation. Extract these into standalone functions.
* [ ] **Apply the Refactoring Cheat Sheet:** Pass these new small functions through the LLM with your **"Enterprise Refactoring Standard"** prompt.
* [ ] **Inject Dependencies:** If the function hard-codes a DB call in the middle, move that call to a parameter or a high-level wrapper.

#### **Step 3: The Unit Test Swap (Afternoon)**

Replace the coarse "Characterization" tests with precise Unit Tests.

* [ ] **Test the "Leaf" Functions:** Write specific Pytest `@pytest.mark.parametrize` tests for the new, small helper functions you extracted.
* [ ] **Mock the "Orchestrator":** In the main function (the one that now calls your helpers), mock the helpers and the database to test the *coordination* logic.
* [ ] **Verify Coverage:** Run `pytest --cov` to ensure the logic you touched is fully green.

#### **Step 4: The Peer Review & Polish (End of Day)**

* [ ] **Run the Linter:** Run `black`, `flake8`, or `pylint` to catch PEP-8 violations.
* [ ] **The "Human" Read Test:** Read the code aloud. If you stumble over a variable name or a nested loop, it's still too complex.
* [ ] **Submit PR:** Your PR should now contain:
1. The refactored code.
2. The new unit tests.
3. (Optional) Documentation updates.



---

### **Daily Progress Tracker (Example)**

| Day | Module/Function | Status | Tests Added |
| --- | --- | --- | --- |
| **Mon** | `auth_service.py` -> `validate_user()` | ✅ Done | 12 |
| **Tue** | `payment_gateway.py` -> `process_invoice()` | 🚧 In Progress | 5 |
| **Wed** | `utils/email.py` -> `send_notification()` | 📅 Scheduled | 0 |

---

### **The "Mental Guardrails"**

* **Don't fix bugs while refactoring:** If you find a bug in the legacy code, write a test that *proves* the bug exists, then fix it in a separate commit. Never mix "fixing" and "cleaning."
* **Stop at the "Good Enough" point:** You don't need 100% coverage on day one. 100% coverage on the **new** code you wrote is the goal.
* **The "Rule of Three":** If you've refactored three similar functions, it's time to create a shared utility or base class.

### **Next Step**

To help you track this, would you like me to generate a **GitHub Issue Template**? You can use it to create "Refactoring Tasks" for your team so everyone follows the same "Surgical Strike" format.


## Grok

Here is a realistic, sustainable **daily/weekly workflow checklist** tailored for a senior engineer working **incrementally** on a legacy Python microservices codebase with **zero initial tests**, long functions, and tight coupling.

The design draws from proven patterns:

- Michael Feathers' *Working Effectively with Legacy Code* core loop (Identify change points → Break dependencies → Test → Refactor → Repeat)
- Characterization testing first
- Test → Refactor → Test again rhythm
- Small, shippable increments (ideally daily PRs)
- "Boy Scout Rule" micro-cleanups whenever you touch code
- Time-boxing to prevent burnout and scope creep
- Regular coverage check-ins to guide prioritization (not blind 100% chasing)

### Core Principles Before the Checklist
- **Goal is not 100% coverage quickly** — it is **risk-reducing coverage on business-critical & frequently changed paths** first.
- **One small win per day** is far better than heroic multi-day sessions.
- **Always leave the code slightly more testable** than you found it (even if only +2–5% coverage).
- **PR every day** (or at worst every 2 days) — even if it's just tests + tiny refactor.
- **Time-box aggressively** — 45–90 min focused sessions.

### Daily Workflow Checklist (≈45–120 min / day)

**Preparation phase (5–10 min)**  
☐ Open your "Legacy Testing Dashboard" document / Notion / Markdown file  
   (tracks: covered files/functions, coverage trend, high-risk areas list, blocked items)  
☐ Check yesterday's PR was merged (or at least approved)  
☐ Run quick `pytest --cov` on the current test suite → glance at overall % and uncovered lines in files you're tracking  
☐ Decide today’s **single focus area** (choose **one**):

   Option A: Continue yesterday's function/file  
   Option B: Pick highest-priority item from coverage analysis (Prompt 7)  
   Option C: Characterization tests on a hot/change-prone function  
   Option D: Refactor a recently-tested small piece (clean + extract)

**Main work block — time-box 45–90 min (core loop)**

☐ **Step 1 – Characterize or Protect (if < 10–20% coverage on target)**  
   ☐ Paste function → use Prompt 3 (Characterization tests) or Prompt 1 (assessment)  
   ☐ Write 3–8 characterization / high-level path tests (mock heavy dependencies)  
   ☐ Get them green → commit

☐ **Step 2 – Tiny Refactor to Enable Better Testing**  
   ☐ Identify **one** extractable piece (validation, transformation, condition block, side-effect block)  
   ☐ Use Prompt 1 → choose safest next refactoring  
   ☐ Use Prompt 2 → perform **one** refactoring (Extract Method / Parameter Object / etc.)  
   ☐ Keep public interface unchanged  
   ☐ Run tests → green? → commit (even if coverage gain is small)

☐ **Step 3 – Test the New Small Pieces**  
   ☐ For every newly extracted pure(ish) function → use Prompt 4  
   ☐ Add parametrized tests covering main cases + edges  
   ☐ Aim: new helpers ≥ 80–90% branch coverage  
   ☐ Re-run full function tests → still green?

☐ **Step 4 – Micro Boy-Scout cleanups (only if time & tests green)**  
   ☐ Rename 1–3 confusing variables/parameters  
   ☐ Add 1–3 type hints (especially on extracted functions)  
   ☐ Remove dead code / commented blocks you now understand  
   ☐ Extract tiny repeated expression to named variable/function

☐ **Step 5 – Coverage & Commit Check**  
   ☐ Run `pytest --cov --cov-report=term-missing` on changed files  
   ☐ Did coverage go up (even 3–8%) on the targeted area?  
   ☐ Write meaningful commit message:  
     `test(users): characterize process_user_payment happy + 3 error paths`  
     `refactor: extract validate_payment_input() from process_user_payment()`  
     `test: add unit tests for validate_payment_input (12 cases)`  
   ☐ Create / update PR (or amend yesterday's if very small)

**Wrap-up phase (5–10 min)**

☐ Update Legacy Testing Dashboard:  
   - New coverage numbers  
   - Functions newly under test / refactored  
   - Blockers discovered (e.g. "can't mock X without ugly hacks")  
   - Next candidate suggestion (1–2 lines)

☐ (Once or twice a week) Run full Prompt 7 (Coverage Report Analysis) → paste output into dashboard  
☐ Decide tomorrow's focus (write 1 sentence)

### Weekly Rhythm Additions (pick 1–2 per week)

- **Coverage Review Friday** (15–30 min)  
  Run Prompt 7 → prioritize next week's 3–5 focus areas

- **Mikado / Dependency-breaking session** (if stuck)  
  Use pen/paper or draw.io: goal at top, break dependencies downward

- **Fixture / conftest.py investment** (30–60 min)  
  Create reusable mocks/fixtures for common dependencies (DB, HTTP client, auth, settings)

- **Test review & polish**  
  Use Prompt 5 on tests written Mon–Thu → improve names, parametrization, remove duplication

- **Show & Tell** (team sync / async update)  
  Demo one small win: "This function went from 0% → 42% and is now 40% smaller"

### Long-term Progress Tracking (monthly view)

Track in dashboard:

| Month     | Approx. files with ≥1 test | Business-critical coverage | Avg function length trend | Open high-risk areas left |
|-----------|-----------------------------|-----------------------------|----------------------------|---------------------------|
| Jan 2026  | 2 / 45                      | ~8%                         | 320 → 290 LOC              | 38                        |
| Feb 2026  | 9 / 45                      | ~22%                        | 290 → 210 LOC              | 32                        |

Aim for:

- +4–10 functions under meaningful test per month
- +10–20% coverage on critical paths every 4–6 weeks
- Average function length dropping ~20–40 LOC per quarter

This rhythm is intentionally **conservative and burnout-resistant**. If followed 4–5 days/week, you'll see visible, compounding progress within 6–10 weeks — enough to start gaining team trust and momentum for bigger refactorings.

Adjust time-box and daily goal size based on your actual meeting load and energy. The key sentence to repeat to yourself: **"Today I make tomorrow's change 5–15% safer."**
