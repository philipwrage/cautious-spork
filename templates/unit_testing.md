You are a senior software engineer who has recently joined a software development team that has no automated unit tests for any of its microservices.  Many of the functions in this team's microservices are hundreds of lines long.  Your goal is to incrementally add Pytest unit tests to the existing code base.  You must also consider refactoring existing code into smaller, more modular and more testable functions.  Develop a set of prompts that will guide an LLM-based AI in helping you achieve your goal of improving code quality by implementing unit testing.

## Grok

Here is a structured set of high-quality, reusable prompts you can feed to an LLM (such as Grok, Claude, GPT-4o, etc.) to systematically improve the code quality of your legacy Python microservices. These prompts are designed to support an **incremental**, **safe**, **behavior-preserving** approach that prioritizes:

- Refactoring long functions into small, focused, testable units (ideally < 50 lines)
- Writing clear, maintainable Pytest unit tests (using pytest idioms, fixtures, parametrization, mocking where needed)
- Avoiding big-bang changes
- Building confidence through tests before aggressive refactoring

Each prompt is self-contained and includes placeholders (in curly braces) so you can copy-paste and fill in the specific context.

### 1. Initial Function Assessment & Refactoring Candidate Selection
Use this first when looking at a large file or function.

```
You are an expert Python refactoring and testing coach. I am incrementally adding Pytest unit tests and improving testability in a legacy microservice that currently has zero tests.

Here is the code of one function I want to improve:

```python
{entire_function_or_class_method_here}
```

Context:
- File: {file_path}
- Function name: {function_name}
- This function is called from: {brief_callers_or_entry_points}
- It depends on: {list_external_dependencies_e.g._database,_http_client,_other_services,_os,_time,_logging}
- Business purpose (in one sentence): {one_sentence_description_of_what_it_does}

Your task:
1. Assess the current testability (score 1–10) and explain the main problems (length, branches, side effects, dependencies, mutable state, etc.).
2. Identify the most promising small, high-value refactoring step(s) that would make this function significantly more testable without changing observable behavior.
   - Prefer Extract Method, Extract Parameter Object, Replace Temp with Query, Introduce Parameter Object, etc.
   - Suggest 1–3 concrete next steps, ordered by safety and leverage (highest impact first).
   - For each step, give:
     - The refactoring name/pattern
     - A short description
     - The expected new function signatures (if extracting)
     - Why this step increases testability
3. Do NOT write the refactored code yet — only recommend the next 1–2 safe moves.

Output format:
## Assessment
[testability score and bullet-point problems]

## Recommended Next Refactoring Steps
1. [Refactoring name]
   Description: ...
   Expected new signatures: ...
   Testability benefit: ...

2. ...
```

### 2. Refactoring a Single Function (Safe, Behavior-Preserving)
Use this after assessment, once you’ve chosen a refactoring step.

```
You are a senior Python developer specializing in legacy code refactoring.

Goal: Refactor the following function to make it smaller and more testable while preserving exact observable behavior (same inputs → same outputs, same side effects).

Original code:

```python
{original_function_code}
```

Specific refactoring to perform now: {describe_the_refactoring_you_chose_e.g._"Extract the validation logic into a separate function called validate_input_data()"}

Rules:
- Only perform the exact refactoring requested — do not do additional cleanups unless they are mechanical and obviously safe.
- Preserve all existing logic, order of operations, exception raising, logging, and side effects.
- Use clear, intention-revealing names.
- Add type hints where they improve clarity (use typing module).
- Keep the public interface of the original function unchanged unless the refactoring explicitly requires it.
- If extracting helper functions, place them logically (same module or new helpers module).
- Do not introduce new dependencies unless unavoidable.

Output exactly:
1. The full refactored version of the original function (including any newly extracted helpers).
2. A short explanation of what changed and why it improves testability.
3. Any new pure functions that are now easy to unit test in isolation.
```

### 3. Writing Characterization Tests (for Legacy Code Before Refactoring)
When the function is still large and messy, start with characterization tests to lock in current behavior.

```
You are an expert in characterization testing for legacy code.

I want to add Pytest unit tests for this function before I refactor it. The tests should document and lock in the current behavior (characterization tests) so I can refactor safely later.

Function under test:

```python
{full_function_code_including_signature}
```

Context:
- Dependencies to mock: {list_e.g._requests,_psycopg2,_os.environ,_datetime}
- Known input examples (if any): {paste example calls or payloads}
- Known outputs / side effects (if any): {what it returns, what it writes to DB, what HTTP calls it makes, etc.}

Your task:
1. Write a Pytest test file (or tests to add to an existing file) that exercises the function with realistic and edge-case inputs.
2. Use @pytest.mark.parametrize for multiple cases.
3. Mock external dependencies aggressively (use unittest.mock, pytest-mock, or responses as appropriate).
4. Name tests descriptively: test_{function_name}_does_X_when_Y
5. For each test, assert on return value AND any important side effects (calls, state changes).
6. Include at least:
   - Happy path(s)
   - Error cases (invalid input, downstream failures)
   - Edge cases (empty, max size, None, special characters)
7. If the function has many branches, add tests that cover the main paths even if you don't understand every detail yet.

Output only the test code (complete, runnable file snippet) and a short comment block at the top explaining coverage intent.
```

### 4. Writing Unit Tests for a Small, Pure Function
Use this after extracting a small, focused helper.

```
You are a Pytest expert writing clean, maintainable unit tests.

Write comprehensive Pytest tests for this small function:

```python
{full_small_function_code_with_type_hints}
```

Rules for excellent tests:
- Use descriptive test names (test_{function}_returns_X_when_Y)
- Use @pytest.mark.parametrize for input/output pairs
- Cover happy path, edge cases, invalid inputs, boundary values
- Use pytest.raises for expected exceptions
- Prefer assert over assertEqual etc.
- Keep tests independent (no shared state)
- Mock only if the function has unavoidable external calls (this function should be pure if possible)
- Aim for >90% branch coverage if feasible

Output:
- The complete test function(s) or test module snippet
- A short comment block listing the cases covered
```

### 5. Reviewing & Improving Existing Tests
Use this after generating tests to make them better.

```
Review these Pytest tests I wrote for {function_name}:

```python
{paste_test_code_here}
```

Critique them strictly from the perspective of a senior engineer focused on maintainability, readability, and robustness.

For each issue you find, explain:
- The problem
- Why it matters
- Suggested fix (with code snippet if helpful)

Also rate overall:
- Readability (/10)
- Coverage of happy/edge/error cases (/10)
- Mocking quality (if applicable)
- Brittleness risk

Finally, provide an improved version of the test file incorporating your fixes.
```

### 6. Planning the Next Increment (Big-Picture Guidance)
Use this periodically to decide what to tackle next.

```
I am incrementally adding tests and refactoring a legacy Python microservice.

Current state:
- Files/functions already under test: {list}
- Functions refactored: {list}
- Coverage so far: roughly {percent} (or qualitative)

Remaining large/complex functions in priority order:
1. {function_name_1} - {why_important}
2. {function_name_2} - {why_important}
...

Dependencies / shared concerns:
- Database access via {ORM/tool}
- HTTP clients
- Message queues
- Authentication / authorization
- Logging / metrics

Recommend the next 1–2 highest-value increments. For each:
- Which function/file to tackle
- Whether to start with characterization tests or refactor first
- Rough steps (e.g., "1. Extract validation → 2. Write tests for validator → 3. Test main path → 4. Extract business logic")
- Any cross-cutting concerns to address (fixtures, conftest.py, mocking strategy)

Output in prioritized order with clear rationale.
```

### Recommended Workflow Using These Prompts
1. Pick one large function → Prompt 1 (assessment)
2. Choose the first refactoring → Prompt 2 (perform refactoring)
3. If the function is still hard to test → Prompt 3 (characterization tests first)
4. After extracting small pure functions → Prompt 4 (write proper unit tests)
5. After writing tests → Prompt 5 (review & polish)
6. Every few functions → Prompt 6 (plan next increment)

This prompt set creates a tight feedback loop that lets you move fast while keeping changes safe and tests trustworthy. Over time you’ll build a regression suite, confidence to refactor aggressively, and much cleaner code.

## Gemini

Here is a strategic set of prompts designed for a Senior Software Engineer tackling a legacy Python codebase. These prompts follow a safe, iterative workflow: **Understand  Lock Behavior (Safety Net)  Refactor  Unit Test.**

### **Phase 0: The System Context (Set the Stage)**

*Use this prompt once at the start of your session to establish the rules of engagement.*

> **Prompt:**
> "Act as a Senior Python Engineer and Refactoring Expert. I am working on a legacy microservice codebase with **zero** unit tests. The code uses Python 3.9+. My goal is to add **Pytest** coverage incrementally without breaking existing functionality.
> **Constraints:**
> 1. Do not suggest a full rewrite. We must refactor incrementally.
> 2. Prioritize **Pure Functions** (deterministic, no side effects) when extracting code.
> 3. Use `unittest.mock` or `pytest-mock` for all external dependencies (DB, APIs, S3).
> 4. Focus on high-value, high-risk logic first."
> 
> 

---

### **Phase 1: The "Code Archaeology" (Understanding the Monster)**

*Before you touch a 500-line function, use these prompts to map its behavior.*

**The Logic Mapper**

> **Prompt:**
> "Analyze the Python function pasted below.
> 1. List every **external dependency** (database calls, API requests, global variables) that needs to be mocked.
> 2. Map the **Control Flow**: Identify the main decision points (if/else blocks) and return states.
> 3. Identify **isolated logic chunks** (e.g., 'lines 10-50 validate input', 'lines 51-90 transform JSON') that could be extracted into helper functions.
> 
> 
> [PASTE CODE HERE]"

**The "What Breaks?" Predictor**

> **Prompt:**
> "I am about to refactor this function. Based on the code below, list 3-5 specific **edge cases** or error states (e.g., 'database returns None', 'malformed JSON') that I must account for in my tests to ensure I don't introduce regressions.
> [PASTE CODE HERE]"

---

### **Phase 2: The "Safety Net" (Characterization Tests)**

*Use this to write a coarse-grained test that "locks in" the current behavior (even the bugs) before you change anything.*

**The "Gold Master" Generator**

> **Prompt:**
> "Create a 'Snapshot Test' (or Characterization Test) for this function using Pytest.
> 1. Mock all the external dependencies listed previously so the test runs locally.
> 2. The test should simply assert that for a given input, the output matches the *current* behavior of the function (even if that behavior seems wrong).
> 3. Use `pytest.fixture` for the mocks.
> 
> 
> [PASTE CODE HERE]"

---

### **Phase 3: The Surgical Refactor (Decomposition)**

*Now that you have a safety net, use these prompts to break the code apart.*

**The "Pure Function" Extractor**

> **Prompt:**
> "Refactor the selected code block below into a separate, **pure** function.
> 1. The new function should take inputs and return outputs only (no side effects, no database calls).
> 2. Name the function descriptively (e.g., `calculate_tax_breakdown`).
> 3. Show me how to call this new function inside the original legacy function so the logic is preserved.
> 
> 
> [PASTE SPECIFIC LOGIC BLOCK HERE]"

**The Dependency Inversion Assistant**

> **Prompt:**
> "This function hard-codes a database connection (e.g., `db = get_db_client()`). Refactor the function signature to use **Dependency Injection**, allowing me to pass in a mock database client during testing, while defaulting to the real client for production use.
> [PASTE CODE HERE]"

---

### **Phase 4: The "Real" Unit Tests (Testing the New Units)**

*Now you test the small, clean functions you just created.*

**The Pytest Parametrization Pro**

> **Prompt:**
> "Generate a Pytest test file for the new `calculate_tax_breakdown` function we just extracted.
> 1. Use `@pytest.mark.parametrize` to cover 5 scenarios: Happy path, Null input, Negative numbers, Float precision edge cases, and Invalid types.
> 2. Ensure the test fails with clear messages.
> 
> 
> [PASTE NEW EXTRACTED FUNCTION HERE]"

**The Mocking Strategist**

> **Prompt:**
> "I need to test a function that calls an AWS S3 bucket.
> 1. Write a `conftest.py` fixture using `moto` or `unittest.mock` that simulates the S3 bucket.
> 2. The fixture should spin up the mock state before the test and tear it down after.
> 3. Write a test case that uses this fixture to verify the function handles a 'File Not Found' error gracefully.
> 
> 
> [PASTE CODE HERE]"

---

### **Phase 5: The Code Review (Quality Check)**

*Before you commit, check your work.*

**The "Testable Code" Auditor**

> **Prompt:**
> "Review this refactored code and the accompanying tests.
> 1. Are there any **'Code Smells'** remaining that make testing difficult?
> 2. Is the Cyclomatic Complexity low enough (< 10)?
> 3. Did I mock too much? (i.e., am I testing the mock implementation rather than the business logic?)
> 
> 
> [PASTE REFACTORED CODE + TESTS]"

---

### **Next Step**

Would you like me to generate a specific **`conftest.py` template** with common fixtures (like a mock generic DB session or API client) to get your team started?
