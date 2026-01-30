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
