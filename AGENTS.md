# Product Requirements Document: Workspace Tools

## Product summary

Workspace Tools is a pair of dependency-free Bash commands that create and
delete isolated Python environments for coding-interview preparation. The
commands themselves remain simple filesystem scripts; Python runtimes,
environments, packages, and lockfiles are managed exclusively through uv.

## Goals

- Create a ready-to-use Python interview workspace with one short command.
- Keep every workspace isolated in its own `.venv`.
- Make environments reproducible through `pyproject.toml`, `.python-version`,
  and `uv.lock`.
- Provide testing, property-testing, formatting, linting, notebook, numerical,
  plotting, and HTTP tooling out of the box.
- Delete workspaces safely through macOS Trash.
- Keep the implementation understandable as two standalone Bash scripts.

## Commands

### Create

```text
wsc [name] [--nj]
```

- With `name`, create `$WORKSPACE_ROOT/name`.
- Without `name`, create `workspace_N`, where `N` increases monotonically.
- `--nj` prevents JupyterLab from starting after setup.
- `-h` and `--help` print usage without creating anything.

### Delete

```text
wsd <name>
```

- Move `$WORKSPACE_ROOT/name` to macOS Trash.
- Never permanently delete the workspace.
- Never decrement or reset the automatic-name counter.

## Functional requirements

### Workspace location and naming

- Default `WORKSPACE_ROOT` to `$HOME/workspaces`.
- Allow `WORKSPACE_ROOT` to be overridden through the environment.
- Accept names containing letters, numbers, periods, underscores, and hyphens.
- Require names to begin with a letter or number.
- Reject paths, traversal sequences, whitespace, and names outside the root.
- Refuse to overwrite an existing file, directory, or symbolic link.
- Persist the last automatic number in
  `$WORKSPACE_ROOT/.workspace_counter` so deleted numbers are not reused.

### Python and uv

- Use Python 3.12.
- Ignore a virtual environment inherited by `wsc`.
- Use `uv init`, `uv python pin`, and `uv add` for project and dependency setup.
- Create and select a workspace-local `.venv`.
- Record resolved dependencies in `uv.lock`.

### Dependencies

Runtime dependencies:

- numpy
- pandas
- matplotlib
- scipy
- requests

Development dependencies, stored in uv's `dev` dependency group:

- pytest
- hypothesis
- pytest-timeout
- ruff
- jupyterlab
- ipykernel

### Generated files

Each workspace must include:

- `.gitignore` covering Python, uv, pytest, Ruff, Hypothesis, and Jupyter
  generated files.
- `.python-version`, `pyproject.toml`, and `uv.lock` managed by uv.
- A newly initialized local Git repository.

### Startup behavior

- By default, activate the new `.venv` and replace the `wsc` process with
  JupyterLab running in the foreground.
- With `--nj`, complete setup without starting JupyterLab and print the command
  needed to activate the environment.

### Deletion safety

- Require exactly one validated workspace name.
- Resolve deletion targets only as direct children of `WORKSPACE_ROOT`.
- Reject missing targets.
- Use the macOS `trash` command so deletion is recoverable.

## Non-goals

- Supporting Windows batch files or PowerShell.
- Supporting operating systems without the macOS `trash` command.
- Managing global Python installations without uv.
- Activating a workspace in the caller's shell after `wsc` exits; child
  processes cannot mutate their parent shell environment.
- Providing a general-purpose project scaffolding framework.

## Acceptance criteria

- `wsc interview_1 --nj` creates a complete, isolated workspace at
  `~/workspaces/interview_1` without launching JupyterLab.
- `wsc` creates sequential `workspace_N` directories without reusing deleted
  numbers.
- `uv run pytest --version` and `uv run ruff --version` work in a newly
  generated workspace.
- All declared runtime and development packages can be imported or invoked.
- `wsc interview_1` starts JupyterLab using the workspace's `.venv`.
- `wsd interview_1` moves only that workspace to Trash.
- Invalid names and existing destinations fail without overwriting data.
- The repository contains only executable `wsc` and `wsd` scripts plus project
  documentation and repository metadata.
