# Workspace Tools

Two small Bash commands for creating and deleting disposable Python interview
workspaces. Each workspace is isolated, reproducible, and managed with
[uv](https://docs.astral.sh/uv/).

## Commands

### `wsc` — create a workspace

```bash
wsc interview_1  # Creates ~/workspaces/interview_1 and starts JupyterLab
wsc              # Creates ~/workspaces/workspace_1, workspace_2, ...
wsc test_1 --nj  # Creates the workspace without starting JupyterLab
```

Every workspace uses Python 3.12 and contains:

```text
interview_1/
├── .git/
├── .gitignore
├── .python-version
├── .venv/
├── pyproject.toml
└── uv.lock
```

Runtime packages:

- NumPy
- pandas
- Matplotlib
- SciPy
- Requests

Development packages:

- pytest
- Hypothesis
- pytest-timeout
- Ruff
- JupyterLab
- ipykernel
- JupyterLab LSP and Python Language Server
- Jupyter Resource Usage

Unless `--nj` is supplied, `wsc` activates the new virtual environment and
starts JupyterLab in the foreground. Stop JupyterLab with `Ctrl+C` to return to
the calling shell.

### `wsd` — delete a workspace

```bash
wsd interview_1
```

`wsd` moves the named workspace to macOS Trash. It does not permanently erase
the directory, and deleting an automatically numbered workspace does not reset
the counter.

## Installation

Requirements:

- macOS
- Bash
- [uv](https://docs.astral.sh/uv/getting-started/installation/)
- Git

Add this repository directory to your `PATH` in `~/.zshrc`:

```bash
export PATH="/path/to/workspace_tools:$PATH"
```

Then reload the shell:

```bash
source ~/.zshrc
```

Both scripts must remain executable:

```bash
chmod +x wsc wsd
```

## Configuration

By default, workspaces are stored under `~/workspaces`. Set `WORKSPACE_ROOT` to
use another parent directory:

```bash
export WORKSPACE_ROOT="$HOME/interview-workspaces"
wsc graphs
```

Automatic names are tracked in `$WORKSPACE_ROOT/.workspace_counter`, ensuring
that numbers increase even if an earlier workspace is deleted.

## Useful commands inside a workspace

```bash
uv run pytest
uv run pytest --timeout=5
uv run ruff check .
uv run ruff format .
uv run jupyter lab
```
