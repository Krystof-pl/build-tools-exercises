# Copilot Instructions

## Build, run, and test

This project uses `uv` and targets Python 3.10 or newer. From the repository root:

```bash
uv venv
source .venv/bin/activate
uv pip install -e .
```

Start the MCP server with:

```bash
uv run main.py
```

Run the complete test suite with:

```bash
uv run pytest
```

Run one test module, class, or test function with standard pytest selectors, for example:

```bash
uv run pytest tests/test_document.py
uv run pytest tests/test_document.py::TestBinaryDocumentToMarkdown
uv run pytest tests/test_document.py::TestBinaryDocumentToMarkdown::test_binary_document_to_markdown_with_pdf
```

No lint or format command is configured in `pyproject.toml`; do not introduce a new tool configuration unless the task calls for it.

## Architecture

- `main.py` creates a `FastMCP("docs")` server and registers exposed tools. A function in `tools/` is not available through MCP until it is imported and passed to `mcp.tool()`.
- `tools/math.py` contains the currently registered `add` MCP tool.
- `tools/document.py` contains the document conversion helper. `binary_document_to_markdown` wraps `markitdown.MarkItDown`, feeds it an in-memory `BytesIO` stream, and supplies the file extension through `StreamInfo`.
- `tests/test_document.py` tests the document helper directly using the checked-in DOCX and PDF files under `tests/fixtures/`. The current tests validate successful, non-empty Markdown conversion and basic Markdown structure.

When adding a new MCP capability, keep its implementation in `tools/`, import it in `main.py`, and register it there. Keep conversion/business logic separate from MCP server setup so it remains directly testable.

## Repository conventions

- Always annotate function arguments with appropriate, specific Python types; avoid untyped parameters.
- Define tool parameters with `pydantic.Field(description=...)` so MCP clients receive useful parameter metadata.
- MCP tool docstrings should start with a one-line summary, then describe behavior, when to use or not use the tool, and include input/output examples. Follow the examples in `README.md`.
- Keep tool functions small and return their typed result directly. The document helper returns `result.text_content` rather than the full MarkItDown result object.
- For binary document tests, read fixture bytes in the test and pass the matching extension (`"docx"` or `"pdf"`) to `binary_document_to_markdown`.
- Keep dependency versions and resolution changes synchronized through `pyproject.toml` and `uv.lock`.
