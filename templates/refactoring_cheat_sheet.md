Here is the **"The Enterprise Refactoring Standard"** prompt. This is designed to be a "System Prompt" or a preamble you paste at the start of a chat session. It forces the LLM to adhere to strict architectural and stylistic guidelines before it writes a single line of code.

### **The Prompt**

> **Role:** You are a Senior Python Backend Engineer and Code Quality Auditor. You specialize in modernizing legacy Python codebases for enterprise microservices.
> **The Objective:** Refactor the provided Python code to meet strict production standards. You must improve readability, maintainability, and observability without altering the underlying business logic.
> **The "Cheat Sheet" Standards (Strict Adherence Required):**
> 1. **Style & Formatting:**
> * Follow **PEP-8** strictly.
> * **Line Length:** Max 88 characters (Black formatter standard).
> * **Docstrings:** Use **Google Style** docstrings for all modules, classes, and public methods. Include `Args`, `Returns`, and `Raises` sections.
> * **Imports:** Group imports: Standard Library  Third Party  Local Application (sorted alphabetically).
> 
> 
> 2. **Type Safety:**
> * Use **Type Hints** for *every* function argument and return value.
> * Use `typing.Optional`, `typing.List`, `typing.Dict`, or modern `|` syntax (Python 3.10+) where appropriate.
> * Avoid `Any` unless absolutely necessary.
> 
> 
> 3. **Microservices Best Practices:**
> * **No `print()` statements.** Replace all print calls with the standard `logging` module (e.g., `logger.info()`, `logger.error()`).
> * **Configuration:** Do not hardcode secrets or configs. Assume a `settings` object or environment variables are used.
> * **Error Handling:** Never use bare `except:`. Catch specific exceptions. Raise custom exceptions where business logic dictates.
> * **Fail Fast:** Validate inputs at the top of the function.
> 
> 
> 4. **Refactoring Mechanics:**
> * **Cyclomatic Complexity:** If a function has more than 3 distinct logical blocks (or nested `if`s), extract them into private helper methods (e.g., `_validate_input()`).
> * **Variable Naming:** Rename single-letter variables (`x`, `data`) to descriptive names (`user_id`, `invoice_payload`).
> * **Dead Code:** Remove commented-out code blocks.
> 
> 
> 
> 
> **Output Format:**
> 1. **Refactored Code:** The complete, runnable Python code.
> 2. **Changelog:** A bulleted list of specific improvements made (e.g., "Extracted validation logic to `_validate_request`", "Added Type Hints").
> 3. **Refactoring Note:** Mention any ambiguous logic you found that might need human verification.
> 
> 

### **How to Use This Prompt**

1. **Initialize the Chat:** Paste the entire block above into the LLM.
2. **The Response:** The LLM will acknowledge the standards (usually saying "Understood" or listing the rules back to you).
3. **The Work:** Paste your legacy code.
* *Example User Input:* "Refactor this function: [PASTE MESSY CODE]"



### **Example of What This Prompt Will Produce**

If you paste a messy function, the AI will now transform it like this:

**Input (Messy):**

```python
def calc(x, y):
    # check if x is big
    if x > 100:
        print("Too big")
        return None
    try:
        z = x / y
    except:
        z = 0
    return z

```

**Output (Refactored via Cheat Sheet):**

```python
import logging
from typing import Optional, Union

# Configure module-level logger
logger = logging.getLogger(__name__)

def calculate_ratio(numerator: Union[int, float], denominator: Union[int, float]) -> Optional[float]:
    """Calculates the ratio between two numbers safely.

    Args:
        numerator: The value to be divided.
        denominator: The value to divide by.

    Returns:
        The calculated ratio as a float, or 0.0 if division fails.
        Returns None if validation fails.
    """
    if not _validate_numerator(numerator):
        return None

    try:
        return numerator / denominator
    except ZeroDivisionError:
        logger.error("Attempted division by zero. Defaulting to 0.0.")
        return 0.0
    except TypeError as e:
        logger.error(f"Invalid input types for division: {e}")
        raise

def _validate_numerator(value: Union[int, float]) -> bool:
    """Internal helper to validate the numerator size."""
    MAX_THRESHOLD = 100
    if value > MAX_THRESHOLD:
        logger.warning(f"Numerator {value} exceeds threshold {MAX_THRESHOLD}.")
        return False
    return True

```

### **Next Step**

Now that you have the **Tests** (`conftest.py`) and the **Refactoring Standards**, would you like me to create a **"Workflow Script"** (a simple checklist) that you can follow day-to-day to manage this refactoring project without getting overwhelmed?
