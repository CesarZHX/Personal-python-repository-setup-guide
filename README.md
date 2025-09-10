# **My personal python project management guide**

This repo documents **how I set up and maintain Python projects**: layout, tooling, dependency management, testing, typing, packaging, and small tips I use day-to-day. It’s opinionated — copy what helps you and ignore the rest.

---

## Table of contents

1. Overview — my goals
2. Conventions & layout
3. Tooling
4. Dependency management
5. Testing & typing
6. Packaging & release notes
7. Security & troubleshooting
8. Snippets.

---

# 1 — Overview (what I want from a project)

My projects aim to be:

* **Reproducible**
* **Easy to install for development**
* **Typed enough** for useful static checking
* **Clear layout** so tests, sources, and packaging work without surprises

---

# 2 — Conventions & layout

**Directory layout I use (src-layout)**:

```
/ (repo root)
├── pyproject.toml
├── README.md
├── src/
│   └── your_package/
│       └── __init__.py
├── tests/
│   └── test_*.py
└── docs/ (optional)
```

* `src/` layout avoids accidental imports of local code during tests. (src layout — *estructura src*)
* Add `src/your_package/py.typed` to indicate typed package. (py.typed — *archivo py.typed*)

**Python version**

I recommend to use the [latest stable Python version](https://devguide.python.org/versions/). I’d like to use Python 3.13, but the current stable release is 3.12.

---

# 3 — Tooling I use (quick list)

* Dependency manager: **PDM** (subsection below). (dependencies — *dependencias*).
* CLI tool isolation: **pipx** for installing PDM.
* Formatter: **black** (run via vscode extension).
* Import sorter: **isort** (configured to work together with black, also via VSCode).
* Type checking: **mypy** (run via vscode extension with mypy installed in global environment).
* Test runner: **pytest** with `pytest-asyncio` for async tests.
* Autocompletion / AI help: **Windsurf** (free AI tool that suggests code as I type).
* Icons & file visuals: **Material Icons Theme** extension for VSCode (improves readability in file explorer).
  
---

# 4 — Dependency management

PDM is the dependency manager I prefer — but it’s just one part of the workflow. It manages the virtual environment, dependency resolution, and `pyproject.toml`/lockfile handling for reproducible installs.

### Why I like PDM (short)

* Automatic environment management (I don’t worry about `.venv`).
* `pdm init` scaffolds projects quickly.
* Works well for libraries (packaging + distribution via `pyproject.toml`).

### How I install PDM on Windows 11 (my preferred method)

I keep PDM isolated with `pipx`:

```powershell
pip install pipx
pipx ensurepath
# restart terminal if required
pipx install pdm
```

You can also install PDM in [other ways](https://pdm-project.org/latest/#installation).

### Common PDM commands (summary)

```bash
# initialize
pdm init

# project installation
pdm install                                     # use lockfile and install project's deps
pdm install -d                                  # include dev dependencies as well

# Dependency management commands
pdm add package-name
pdm add -d pytest mypy                          # add to dev
pdm add -G web flask                            # add to specific optional group

# Update dependencies
pdm update                                      # update all dependencies according to lockfile strategy
pdm update package-name                         # update only a specific package
pdm update -G web                               # update all packages in the optional group "web"
pdm update -d                                   # update all dev dependencies

# remove dependencies
pdm remove package-name
pdm remove -G web flask                         # remove from optional group "web"
pdm remove -d pytest                            # remove from dev dependencies


# run commands within project env
pdm run python main.py
pdm run python -m your_package
pdm run pytest

# inspect / export
pdm list
pdm info
pdm export -f requirements > requirements.txt
pdm lock                                        # Updates the lock file
```

---

# 5 — Testing & typing

### Pytest (I use asyncio often)

Add to `tool.pytest.ini_options` in `pyproject.toml`:

```toml
[tool.pytest.ini_options]
asyncio_default_fixture_loop_scope = "session"
asyncio_default_test_loop_scope = "session"
asyncio_mode = "auto"
```

* `asyncio_mode = "auto"`: pytest-asyncio auto-detects async tests.
* `session` scope for the event loop (`asyncio_default_fixture_loop_scope` / `asyncio_default_test_loop_scope`):
  I always leave it set to `session` to ensure that a **single event loop is reused across all tests**.
  Using multiple event loops (e.g., default `function` scope) can sometimes cause errors, especially with long-lived fixtures like databases, browser instances, or network clients.
* Reusing a session-scoped loop reduces repeated setup/teardown costs for expensive fixtures and prevents conflicts between loops.

More info [in the official pytest-asyncio documentation](https://pytest-asyncio.readthedocs.io/en/stable/reference/configuration.html).

### mypy

I use `mypy` with selective overrides when third-party packages lack stubs:

```toml
[[tool.mypy.overrides]]
module = ["twocaptcha.*", "pytesseract.*", "thefuzz.*", "google_auth_oauthlib.flow.*", "pygsheets.*"]
ignore_missing_imports = true
```

I often install `mypy` globally (for quick checks) and rely on local config for project rules.

---

# 6 — Packaging & releasing

My preferred `pyproject.toml` snippets for a library (src layout + distribution):

```toml
[tool.pdm]
# By default, PDM assumes your package has the same name as the project and lives in the root or in a folder matching that name.
# package-dir = "src" tells PDM: “look for all Python packages inside the src/ directory instead of the root”.
# This is useful when you want a custom folder structure or a different package name, and it avoids accidental imports from the repo root during development or testing.
package-dir = "src"
```

---

# 7 — Security & troubleshooting

### Security

To reduce risks from accidental dependency upgrades or malicious updates, I always **pin exact versions** in my projects.  

You can do it in two ways:

1. **Specify the version when adding a package:**

```bash
pdm add my-library==1.0.0
````

2. **Configure PDM to always save exact versions automatically:**

```bash
# Local project-only setting
pdm config --local strategy.save exact

# Global setting for all projects
pdm config strategy.save exact
```

This ensures that `pdm add` writes **exact versions** in both the `pyproject.toml` and the lockfile, improving **reproducibility** and **security** by avoiding unintended upgrades.

### Common troubleshooting example

If you see:

```
ValueError: invalid pyproject.toml config: project.dependencies[2].
configuration error: project.dependencies[2] must be pep508
WARNING: Add '-v' to see the detailed traceback
```

→ Check that dependency entries are PEP 508-compliant. For private git dependencies use:

```bash
pdm add "your-lib-name @ git+https://github.com/your-username/your-lib-name.git"
```

For private repos prefer SSH access or token-based solutions; avoid embedding tokens in plain text. If `pdm add` fails, run with `-v` for a detailed traceback and verify URL/auth.

---

# 8 — Snippets I copy into new projects

**py.typed**
Create an empty file at `src/your_package/py.typed` to mark package as typed.
