# Maos Agent Gate

**Stop shipping lobotomized agents.**

`agentgate` is a Drop-in CI/CD Action that acts as a quality firewall for your AI Agents. It runs your System Prompt against a "Golden Dataset" of test cases using an LLM-as-a-Judge (GPT-4o) to ensure every pull request maintains intelligence standards.

If your agent's quality score drops below the threshold (default **90%**), the Pull Request is **blocked**.

---

## 🚀 Usage

Add this to your repository in `.github/workflows/agent-check.yml`:

```yaml
name: Agent Quality Gate

on: [pull_request]

jobs:
  maos-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # The Maos Gate
      - uses: maosproject-dev/agentgate@v1
        with:
          openai_api_key: ${{ secrets.OPENAI_API_KEY }}
          system_prompt: 'prompts/finance_agent.txt'
          test_cases: 'tests/golden_dataset.json'
          threshold: 95

```

---

## ⚙️ Inputs

| Input | Description | Required | Default |
| --- | --- | --- | --- |
| **`openai_api_key`** | Your OpenAI API Key (for the Judge). | **Yes** | N/A |
| **`system_prompt`** | Path to the text file containing your agent's system instructions. | **Yes** | `prompts/system.txt` |
| **`test_cases`** | Path to the JSON file containing input/output pairs. | **Yes** | `tests/expected.json` |
| **`threshold`** | Minimum passing score (0-100). Fails build if lower. | No | `90` |

---

## 📂 Prerequisites

To use this action, your repository must contain two files:

### 1. The System Prompt (`.txt`)

A standard text file with your agent's instructions.
*Example: `prompts/finance_agent.txt*`

```text
You are a financial analyst. You must always answer in JSON format.
If you don't know the answer, say "UNKNOWN".

```

### 2. The Golden Dataset (`.json`)

A JSON file containing the test cases. We support strict equality checks and "LLM Rubrics" (fuzzy matching).
*Example: `tests/golden_dataset.json*`

```json
[
  {
    "description": "Recipe 1: Strict Tool Usage Check",
    "vars": {
      "user_input": "Book a flight to Paris for tomorrow"
    },
    "assert": [
      {
        "type": "is-json",
        "value": {
          "required": ["tool", "arguments"],
          "properties": {
            "tool": { "const": "book_flight" },
            "arguments": {
              "required": ["destination"],
              "properties": {
                "destination": { "const": "Paris" }
              }
            }
          }
        }
      }
    ]
  },
  {
    "description": "Recipe 2: Fuzzy Logic / Tone Check",
    "vars": {
      "user_input": "Why is Maos better than AWS?"
    },
    "assert": [
      {
        "type": "llm-rubric",
        "value": "The response should be professional and mention 'Spot Instance Savings'. It must NOT be rude to competitors."
      }
    ]
  },
  {
    "description": "Recipe 3: Efficiency Check",
    "vars": {
      "user_input": "Hi"
    },
    "assert": [
      { "type": "cost", "threshold": 0.0005 },
      { "type": "latency", "threshold": 2000 }
    ]
  }
]

```

---

## 🛠️ How it works

1. **Injects** your `system_prompt` into a simulated conversation.
2. **Runs** every test case defined in `test_cases`.
3. **Grades** the output using GPT-4o ("The Judge").
4. **Calculates** a final percentage score.
5. **Blocks** the PR if `Score < Threshold`.

---

**Part of the [Maosproject Platform](https://www.google.com/search?q=https://maosproject.io) — The Control Plane for Autonomous Compute.**

---

### Deployment Checklist

1. Push `action.yml` (from previous step).
2. Push this `README.md`.
3. Tag the release:
```bash
git tag -a v1 -m "Launch v1"
git push origin v1

```
