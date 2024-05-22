## Overview

| Developed by | Guardrails AI |
| Date of development | Feb 15, 2024 |
| Validator type | Format |
| Blog | |
| License | Apache 2 |
| Input/Output | Output |

## Description

### Intended Use

This validator ensures that any URL generated by an LLM is an endpoint that can be reached. In order to validate this, the validator makes a request to the URL and expects a 200 response.

### Requirements

* Dependencies:
	- guardrails-ai>=0.4.0
    - `requests`

## Installation

```bash
$ guardrails hub install hub://guardrails/endpoint_is_reachable
```

## Usage Examples

### Validating string output via Python

In this example, we apply the validator to a string URL generated by an LLM.

```python
# Import Guard and Validator
from guardrails import Guard
from guardrails.hub import EndpointIsReachable


# Setup Guard
guard = Guard().use(EndpointIsReachable, on_fail="exception")

response = guard.validate("https://www.guardrailsai.com/")  # Validator passes

try:
    response = guard.validate("https://www.guardrailsai.co")  # Validator fails
except Exception as e:
    print(e)
```
Output:
```console
Validation failed for field with errors: URL https://www.guardrailsai.co could not be reached
```

### Validating JSON output via Python

In this example, we apply the validator to a URL that is a field within a Pydantic object.

```python
# Import Guard and Validator
from pydantic import BaseModel, Field
from guardrails.hub import EndpointIsReachable
from guardrails import Guard

val = EndpointIsReachable(on_fail="exception")


# Create Pydantic BaseModel
class PaperCitations(BaseModel):
    paper_name: str
    paper_url: str = Field(description="URL at which to find paper", validators=[val])


# Create a Guard to check for valid Pydantic output
guard = Guard.from_pydantic(output_class=PaperCitations)

# Run LLM output generating JSON through guard
guard.parse(
    """
    {
        "paper_name": "Attention Is All You Need",
        "paper_url": "https://arxiv.org/abs/1706.03762"
    }
    """
)

try:
    # Run LLM output generating JSON through guard
    guard.parse(
        """
        {
            "paper_name": "Attention Is All You Need",
            "paper_url": "https://arxiv.org/abs/1706.0376234"
        }
        """
    )
except Exception as e:
    print(e)
```
Output:
```console
Validation failed for field with errors: URL https://arxiv.org/abs/1706.0376234 returned status code 404
```

# API Reference

**`__init__(self, on_fail="noop")`**
<ul>

Initializes a new instance of the Validator class.

**Parameters:**

- 
- **`on_fail`** *(str, Callable):* The policy to enact when a validator fails. If `str`, must be one of `reask`, `fix`, `filter`, `refrain`, `noop`, `exception` or `fix_reask`. Otherwise, must be a function that is called when the validator fails.

</ul>

<br>

**`__call__(self, value, metadata={}) → ValidationResult`**

<ul>

Validates the given `value` using the rules defined in this validator, relying on the `metadata` provided to customize the validation process. This method is automatically invoked by `guard.parse(...)`, ensuring the validation logic is applied to the input data.

Note:

1. This method should not be called directly by the user. Instead, invoke `guard.parse(...)` where this method will be called internally for each associated Validator.
2. When invoking `guard.parse(...)`, ensure to pass the appropriate `metadata` dictionary that includes keys and values required by this validator. If `guard` is associated with multiple validators, combine all necessary metadata into a single dictionary.

**Parameters:**

- **`value`** *(Any):* The input value to validate.
- **`metadata`** *(dict):* A dictionary containing metadata required for validation. No additional metadata keys are needed for this validator.

</ul>
