# VscodeConfig

Shared VS Code configuration for Python projects: pytest test discovery, Ruff lint/format tasks, and keybindings for code jumping, block selection, and multi-cursor editing.

## Required extensions

Install these before using this configuration:

- **[Ruff](https://marketplace.visualstudio.com/items?itemName=charliermarsh.ruff)** (`charliermarsh.ruff`) — powers the `ruff check` / `ruff format` tasks in [tasks.json](.vscode/tasks.json). See the [Ruff editor integrations docs](https://docs.astral.sh/ruff/tutorial/#integrations) for setup details.
- **[Space Block Jumper](https://marketplace.visualstudio.com/items?itemName=jmfirth.vsc-space-block-jumper)** (`jmfirth.vsc-space-block-jumper`) — powers the `spaceBlockJumper.*` commands in [keybindings.json](keybindings.json).
- **Python** (`ms-python.python`) — required for `python.testing.*` settings in [.vscode/settings.json](.vscode/settings.json).

## settings.json

[.vscode/settings.json](.vscode/settings.json) configures pytest as the test runner:

- `python.testing.pytestEnabled: true`, `python.testing.unittestEnabled: false` — use pytest, not unittest.
- `python.testing.pytestArgs: ["tests"]` — discover tests under the `tests/` directory.
- `python.defaultInterpreterPath` — points at the project-local `.venv`, so a virtualenv must exist at `${workspaceFolder}/.venv` (e.g. created via `uv sync`).

## tasks.json

[.vscode/tasks.json](.vscode/tasks.json) defines a **"Ruff: Fix & Format"** task that runs against the current file:

1. `ruff-fix` — `ruff check --fix --extend-select I` (lints and applies auto-fixes, including import sorting).
2. `ruff-format` — `ruff format` (formats the file).

Both steps run sequentially via `${workspaceFolder}/.venv/bin/ruff`, so Ruff must be installed in the project's virtualenv (it's included as a dev dependency in the example `pyproject.toml` below). Run the task from the Command Palette (`Tasks: Run Task` → `Ruff: Fix & Format`) or bind it to a key.

## keybindings.json

[keybindings.json](keybindings.json) adds shortcuts for code navigation and multi-cursor editing (active while the editor has focus):

| Shortcut | Command | Action |
|---|---|---|
| `Ctrl+Up` | `spaceBlockJumper.moveUp` | Jump cursor to the previous code block |
| `Ctrl+Down` | `spaceBlockJumper.moveDown` | Jump cursor to the next code block |
| `Ctrl+Shift+Up` | `spaceBlockJumper.selectUp` | Select up to the previous code block |
| `Ctrl+Shift+Down` | `spaceBlockJumper.selectDown` | Select down to the next code block |

To install: copy the contents into your user `keybindings.json` (Command Palette → `Preferences: Open Keyboard Shortcuts (JSON)`), or use `File > Preferences > Keyboard Shortcuts` and import individual bindings.

## Example pyproject.toml

A companion `pyproject.toml` for a `src`-layout project using the [hatchling](https://hatch.pypa.io/) build backend, [uv](https://docs.astral.sh/uv/) for dependency management, and default pytest/Ruff config matching the settings above:

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "my-project"
version = "0.1.0"
description = ""
readme = "README.md"
requires-python = ">=3.12"
dependencies = []

[tool.hatch.build.targets.wheel]
packages = ["src/my_project"]

[dependency-groups]
dev = [
    "pytest>=8.0",
    "ruff>=0.6",
]

[tool.pytest.ini_options]
pythonpath = ["."]
markers = ["unit", "integration"]

[tool.ruff]
src = ["src"]
line-length = 88

[tool.ruff.lint]
select = ["E", "F", "I"]

[tool.uv]
package = true
```

Adjust `name`, `packages`, and `requires-python` to match your project. After adding this file, run `uv sync` to create the `.venv` that `settings.json` and `tasks.json` expect.
