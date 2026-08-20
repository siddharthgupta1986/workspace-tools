# Workspace Tools

Small Bash commands for creating and deleting disposable Python and Node.js
interview workspaces. Python environments are managed with
[uv](https://docs.astral.sh/uv/), while Node.js runtimes are pinned with
[mise](https://mise.jdx.dev/) and dependencies are installed locally with npm.

## Commands

### `wscp` — create a Python workspace

```bash
wscp interview_1  # Creates ~/workspaces/interview_1 and starts JupyterLab
wscp              # Creates ~/workspaces/workspace_1, workspace_2, ...
wscp test_1 --nj  # Creates the workspace without starting JupyterLab
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
- JupyterLab Code Formatter with Black and isort

Unless `--nj` is supplied, `wscp` activates the new virtual environment and
opens the workspace in JupyterLab Desktop using its `.venv`. If the Desktop app
is unavailable, it falls back to browser-based JupyterLab. Use `--nj` to create
the workspace without launching either interface.

### `wscn` — create a Node.js workspace

```bash
wscn interview_node  # Creates ~/workspaces/interview_node
wscn                 # Uses the next shared workspace_N number
```

Every Node workspace pins an exact Node 24 release in `mise.toml` and contains
`package.json`, `package-lock.json`, `tsconfig.json`, `biome.json`, a local
`node_modules`, and a newly initialized Git repository. No Jupyter settings or
Python environments are touched.

The following development packages are installed locally:

- TypeScript, `tsx`, and Node.js type definitions
- Vitest and fast-check
- Biome

No npm packages are installed globally. Useful commands include:

```bash
npm start
npm run watch
npm test
npm run test:watch
npm run typecheck
npm run check
npm run format
```

If mise is not activated in your shell, prefix a command with `mise exec --`,
for example `mise exec -- npm test`.

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
- [mise](https://mise.jdx.dev/getting-started), installable with
  `brew install mise`
- Git

Optional but recommended:

- [JupyterLab Desktop](https://github.com/jupyterlab/jupyterlab-desktop),
  installable on macOS with `brew install --cask jupyterlab-app`

Add this repository directory to your `PATH` and activate mise in `~/.zshrc`:

```bash
export PATH="/path/to/workspace_tools:$PATH"
eval "$(mise activate zsh)"
```

Then reload the shell:

```bash
source ~/.zshrc
```

All scripts must remain executable:

```bash
chmod +x wscp wscn wsd
```

## Configuration

By default, workspaces are stored under `~/workspaces`. Set `WORKSPACE_ROOT` to
use another parent directory:

```bash
export WORKSPACE_ROOT="$HOME/interview-workspaces"
wscp graphs
```

Automatic names are shared by `wscp` and `wscn` and tracked in
`$WORKSPACE_ROOT/.workspace_counter`, ensuring that numbers increase even if an
earlier workspace is deleted.

## Useful commands inside a workspace

```bash
uv run pytest
uv run pytest --timeout=5
uv run ruff check .
uv run ruff format .
uv run jupyter lab
```
