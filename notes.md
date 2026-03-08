# hap-conv-py setup notes

## where you are now

`uv init` has already been run — `pyproject.toml` and `main.py` exist.
ffmpeg must be installed on the system (see step 1).

---

## 1. install system dependencies

### ffmpeg

**macOS:**

```sh
brew install ffmpeg
```

**Linux:**

```sh
sudo apt install ffmpeg          # Debian/Ubuntu
sudo dnf install ffmpeg          # Fedora (may need RPM Fusion repo first)
```

**Windows:**

```sh
winget install ffmpeg            # or: scoop install ffmpeg
```

> Note: HapQ support depends on ffmpeg being compiled with the `hap` codec.
> Brew and most Linux package managers include it. Verify with: `ffmpeg -codecs | grep hap`

### uv

**macOS/Linux:**

```sh
curl -LsSf https://astral.sh/uv/install.sh | sh
# or on macOS: brew install uv
```

**Windows (PowerShell):**

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
# or: winget install astral-sh.uv  /  scoop install uv
```

uv manages Python itself, no separate Python install needed.

### markdownlint-cli2

**macOS:**

```sh
brew install markdownlint-cli2
```

**Linux/Windows** (requires Node.js):

```sh
npm install -g markdownlint-cli2
```

---

## 2. add dependencies

```sh
uv add typer                        # CLI arg parsing
uv add --dev ruff pyright           # linter/formatter + type checker
```

---

## 3. configure pyproject.toml

Replace the generated file with this:

```toml
[project]
name = "hap-conv-py"
version = "0.1.0"
description = "Convert video files to HapQ .mov + .mp3"
requires-python = ">=3.13"
dependencies = [
    "typer",
]

[project.scripts]
hap-conv = "main:app"

[dependency-groups]
dev = [
    "pyright",
    "ruff",
]

[tool.ruff]
line-length = 100
indent-width = 4

[tool.ruff.format]
indent-style = "space"
quote-style = "double"

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "RUF"]

[tool.pyright]
strict = true
pythonVersion = "3.13"
```

---

## 4. restructure files

```sh
mkdir src
mv main.py src/main.py
```

Update `pyproject.toml` scripts entry to `"src.main:app"` if you move to src/.

---

## 5. write the program

`src/main.py` skeleton:

```python
import subprocess
from pathlib import Path
import typer

app = typer.Typer()

@app.command()
def convert(files: list[Path]) -> None:
    for file in files:
        convert_to_hapq(file)
        convert_to_mp3(file)

def convert_to_hapq(input: Path) -> None:
    output = input.with_suffix(".mov")
    subprocess.run(
        ["ffmpeg", "-i", str(input), "-vcodec", "hap", "-format", "hap_q", str(output)],
        check=True,
    )

def convert_to_mp3(input: Path) -> None:
    output = input.with_suffix(".mp3")
    subprocess.run(
        ["ffmpeg", "-i", str(input), "-q:a", "0", "-map", "a", str(output)],
        check=True,
    )

if __name__ == "__main__":
    app()
```

The ffmpeg flags above are a starting point — tweak as needed.

---

## 6. run and type check

```sh
uv run src/main.py video.mp4       # run directly
uv run pyright                     # type check
uv run ruff check src/             # lint
uv run ruff format src/            # format
```

---

## 7. optional: pre-commit hooks

```sh
uv add --dev pre-commit
```

Create `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.9.0
    hooks:
      - id: ruff
      - id: ruff-format
  - repo: https://github.com/igorshubovych/markdownlint-cli2-action
    rev: v0.17.0
    hooks:
      - id: markdownlint-cli2
```

```sh
uv run pre-commit install
```

---

## 8. optional: build a single executable

```sh
uv build
```

Or for a self-contained binary (no Python needed on target machine), look into `pyinstaller`:

```sh
uv add --dev pyinstaller
uv run pyinstaller --onefile src/main.py
```

---

## workflow summary

| task | command |
| --- | --- |
| run | `uv run src/main.py <file>` |
| type check | `uv run pyright` |
| lint | `uv run ruff check src/` |
| format | `uv run ruff format src/` |
| lint markdown | `markdownlint-cli2 "**/*.md"` |
| add dep | `uv add <package>` |
