## Poetry Common Commands

Here are the most commonly used Poetry commands:

**update Poetry to the latest stable version:**

- `poetry self update`

**Project Management:**

- `poetry new <project-name>` - Create a new Poetry project with basic structure
- `poetry init` - Initialize Poetry in an existing project (interactive setup)
- `poetry install` - Install all dependencies from pyproject.toml
- `poetry update` - Update all dependencies to their latest compatible versions

**Dependency Management:**

- `poetry add <package>` - Add a new dependency
- `poetry add <package> --group dev` - Add a development dependency
- `poetry remove <package>` - Remove a dependency
- `poetry show` - List all installed packages
- `poetry show <package>` - Show details about a specific package
- `poetry show --tree` - Display dependency tree

**Environment Management:**

- `poetry shell` - Activate the virtual environment
- `poetry run <command>` - Run a command in the virtual environment
- `poetry env info` - Show virtual environment information
- `poetry env list` - List available virtual environments

**Build and Publishing:**

- `poetry build` - Build the project (creates wheel and tar.gz)
- `poetry publish` - Publish to PyPI
- `poetry version` - Show current version
- `poetry version <new-version>` - Bump version (patch, minor, major, etc.)

**Lock File Management:**

- `poetry lock` - Generate/update poetry.lock file
- `poetry lock --no-update` - Regenerate lock file without updating dependencies
- `poetry export -f requirements.txt --output requirements.txt` - Export to requirements.txt

**Configuration:**

- `poetry config --list` - Show all configuration settings
- `poetry config repositories.<name> <url>` - Add a custom repository
- `poetry cache clear <cache> --all` - Clear Poetry cache

These commands cover the essential workflow for managing Python projects with Poetry, from initial setup through
dependency management to publishing.