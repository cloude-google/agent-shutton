# Blogger Agent Evaluation

This directory contains the evaluation framework for the `blogger-agent`.

## How to Run

To run the evaluations, you first need to install the required dependencies by running `uv pip install -r requirements.txt` from the root of the project.

Once the dependencies are installed, you can run the evaluation using `uv run pytest` from the root of the project:

```bash
pytest eval/test_eval.py
```

## Test Scenarios

The evaluation is designed to test the `blogger-agent` by running it through a predefined conversation. The test scenario is defined in the `data/blog_eval.test.json` file.

The current test case is:
*   **Blog post about Gemini 2.5 Flash Preview:** The user asks the agent to write a blog post about the "Google Gemini 2.5 Flash Preview" model.
