# Product Requirements Document: Workspace Tools

## Product summary

Workspace Tools is a set of dependency-free Bash commands that create and
delete isolated Python and Node.js environments for coding-interview
preparation. Python runtimes and dependencies are managed through uv. Node.js
runtimes are pinned through mise, and npm dependencies remain workspace-local.

## Goals

- Create a ready-to-use Python interview workspace with one short command.
- Create a ready-to-use Node.js interview workspace with one short command.
- Keep every Python workspace isolated in its own `.venv` and every Node.js
  workspace isolated through its local `node_modules`.
- Make environments reproducible through `pyproject.toml`, `.python-version`,
  and `uv.lock`.
- Provide testing, property-testing, formatting, linting, notebook, numerical,
  plotting, and HTTP tooling out of the box.
- Delete workspaces safely through macOS Trash.
- Keep the implementation understandable as two standalone Bash scripts.

## Commands

### Create

```text
wscp [name] [--nj]
wscn [name]
```

- With `name`, create `$WORKSPACE_ROOT/name`.
- Without `name`, create `workspace_N`, where `N` increases monotonically.
- `wscp` creates a Python workspace; `--nj` prevents JupyterLab from starting.
- `wscn` creates a Node.js workspace without interacting with Jupyter.
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
- Ignore a virtual environment inherited by `wscp`.
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
- jupyterlab-lsp
- python-lsp-server
- jupyter-resource-usage
- jupyterlab-code-formatter
- black
- isort

Node.js development dependencies, stored in the workspace's `devDependencies`:

- typescript
- @types/node
- tsx
- vitest
- fast-check
- @biomejs/biome

### Generated files

Each Python workspace must include:

- `.gitignore` covering Python, uv, pytest, Ruff, Hypothesis, and Jupyter
  generated files.
- `.python-version`, `pyproject.toml`, and `uv.lock` managed by uv.
- A newly initialized local Git repository.

Each Node.js workspace must include:

- An exact Node 24 release pinned in `mise.toml`.
- `package.json` and `package-lock.json` managed by npm.
- A workspace-local `node_modules`; never install npm packages globally.
- `tsconfig.json`, `biome.json`, `.gitignore`, and a local Git repository.
- No generated solution, test, notes, or Jupyter files.

### Startup behavior

- By default, `wscp` activates the new `.venv` and launches JupyterLab Desktop
  with the workspace directory and `.venv/bin/python` selected.
- Detect `jlab` on `PATH` first, then the standard macOS application launcher.
- Fall back to browser-based JupyterLab when Desktop is unavailable.
- With `--nj`, complete Python setup without starting JupyterLab and print the
  command needed to activate the environment.
- `wscn` must run setup through `mise exec` so shell activation is not required.

### Deletion safety

- Require exactly one validated workspace name.
- Resolve deletion targets only as direct children of `WORKSPACE_ROOT`.
- Reject missing targets.
- Use the macOS `trash` command so deletion is recoverable.

## Non-goals

- Supporting Windows batch files or PowerShell.
- Supporting operating systems without the macOS `trash` command.
- Managing global Python installations without uv.
- Activating a workspace in the caller's shell after a creator exits; child
  processes cannot mutate their parent shell environment.
- Providing a general-purpose project scaffolding framework.

## Acceptance criteria

- `wscp interview_1 --nj` creates a complete, isolated workspace at
  `~/workspaces/interview_1` without launching JupyterLab.
- `wscp` and `wscn` share sequential `workspace_N` numbering without reusing
  deleted numbers.
- `uv run pytest --version` and `uv run ruff --version` work in a newly
  generated workspace.
- All declared runtime and development packages can be imported or invoked.
- JupyterLab Code Formatter is enabled with Black and isort available as
  formatter backends.
- `wscp interview_1` opens JupyterLab Desktop using the workspace's `.venv`, or
  browser-based JupyterLab if Desktop is unavailable.
- `wscn interview_node` creates a Node 24 workspace whose TypeScript, testing,
  property-testing, formatting, and linting tools run successfully.
- `wsd interview_1` moves only that workspace to Trash.
- Invalid names and existing destinations fail without overwriting data.
- The repository contains only executable `wscp`, `wscn`, and `wsd` scripts
  plus project documentation and repository metadata.
