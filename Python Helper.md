# Python Helper File

[Python Setup Checklist](#python-setup-checklist-on-a-new-device)
[Virtual environments with Conda and `venv`](#python-venv-and-conda---cheatsheet)

## Python Setup Checklist On a New Device

### One-time per computer

```bash
sudo apt update && sudo apt upgrade -y
```
Refresh the package index and pull in security patches before installing anything.

```bash
sudo apt install -y python3 python3-venv python3-pip
```
Install the Python interpreter, the venv module, and pip (not all distros bundle these).

```bash
sudo apt install -y python3-dev build-essential
```
Install the C headers and compiler toolchain that pip needs when a package builds a C extension.

```bash
sudo apt install -y git
```
Install Git so you can version-control projects from the start.

#### Hardware-specific apt packages (install once, share across venvs)

```bash
sudo apt install -y python3-opencv python3-picamera2 python3-libcamera python3-numpy
```
Pi-only: install the libraries that bind to the camera and GPU stack; pip's versions of these are broken or blind to the hardware.

---

### Per project / per venv

```bash
cd ~/path/to/project
```
Work from the project folder so the venv lands inside it.

```bash
python3 -m venv --system-site-packages .venv
```
Create a `.venv/` folder in this project; `--system-site-packages` lets it see apt-installed modules (needed on the Pi, skip on a plain laptop).

```bash
source .venv/bin/activate
```
Point `python` and `pip` at this venv for the current terminal (repeat in every new terminal).

```bash
pip install --upgrade pip setuptools wheel
```
Upgrade the venv's bundled pip and build tools so dependency errors are readable and modern wheels install.

```bash
pip install <package>
```
Install pure-Python packages (`imutils`, `requests`, etc.) into this venv only.

```bash
pip freeze > requirements.txt
```
Snapshot the exact versions installed so another machine can recreate the venv with `pip install -r requirements.txt`.

```bash
echo ".venv/" >> .gitignore
```
Keep the venv folder out of Git; it is large, machine-specific, and rebuildable from `requirements.txt`.

```bash
deactivate
```
Leave the venv when you are done; the terminal goes back to the system Python.

## Python venv and Conda - Cheatsheet

### venv vs conda (the one-paragraph version)

A **venv** isolates Python *packages* for one project but borrows the system's Python *version* (it can't install a different one). Packages come from pip/PyPI. Lightweight, built into Python. A **conda** env isolates the Python version *itself* plus non-Python/compiled dependencies (C/Fortran libs, CUDA, MKL) — packages come from conda channels, with pip as a fallback. Use conda for the heavy scientific/ML stack (NumPy, SciPy, GPU PyTorch, GDAL); use venv for everything else. **Both isolate package versions fully** — that's not the difference. The difference is conda *also* controls the interpreter version and compiled deps.

---

### Python (venv)

#### Create
```bash
python -m venv .venv          # creates a .venv/ folder in your project
```
Convention: name it `.venv` in the project root — VS Code auto-detects that name.

#### Activate / deactivate
```bash
source .venv/bin/activate     # macOS/Linux
.venv\Scripts\activate        # Windows (PowerShell/cmd)
deactivate                    # exit the env
```
When active, your prompt shows `(.venv)`. **No prompt marker = no env active = you're on system Python.**

#### Identify which env is active
```bash
which python                  # macOS/Linux — prints the interpreter path
where python                  # Windows
python --version
```
If the path points into your project's `.venv/`, you're good. If it points to `/usr/bin/python` or similar, you're on system Python — stop before installing anything.

#### View contents
```bash
pip list                      # all packages + versions
pip freeze                    # same, in reinstallable format
```

#### Add / remove packages
```bash
pip install pandas
pip install numpy==1.26.4     # pin a specific version
pip uninstall pandas
```
Must be done with the env **activated**, or it installs to the wrong place.

#### Limitations
- **Cannot change the Python version** — locked to whatever `python` points to. Need a different Python? Use conda, or install that Python version system-wide first.
- **Handles only Python packages** — no compiled/system libraries.
- Starts **empty** — does NOT inherit system-installed packages (only the interpreter version). This is correct behavior: everything you install is isolated and version-pinned.

#### Reproducibility
```bash
pip freeze > requirements.txt          # save current package set
pip install -r requirements.txt        # recreate elsewhere / after rebuild
```
Nothing auto-saves this — you generate it yourself. Keep it current so rebuilding a broken env is one command.

#### Rebuild (the "nuclear option")
Envs are disposable. If one gets tangled, don't debug it — delete and recreate:
```bash
rm -rf .venv
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

#### Auto-switching — READ THIS
Envs do **NOT** auto-switch the way git branches do. Git branch state lives in the folder (`.git/HEAD`), so opening a folder restores the branch. Envs have no such on-disk link. Two separate contexts:

- **VS Code UI (notebook kernel / interpreter picker):** VS Code *remembers your per-workspace choice* and reselects it when you reopen the folder. Also auto-detects `.venv/` in the root. This is the closest thing to git-like behavior — but it's VS Code saving your pick, not an intrinsic folder property. It can still guess wrong silently if multiple envs exist. **Always glance at the kernel name (top-right of the notebook) before running.**
- **Terminal (`pip install`, `python script.py`):** **NO auto-switching, ever.** Uses whatever env you last `activate`d. `cd`-ing into another project changes nothing. This is where things break:
  - Forgot to deactivate the old env → installs into the wrong project.
  - Nothing activated → installs into **system Python**, polluting it. (Most common way people wreck their setup.)

**Habit:** before any terminal install/run, check the `(.venv)` prompt marker and/or run `which python`. Terminals *opened from inside VS Code* auto-activate the workspace env (`python.terminal.activateEnvironment`, on by default); external terminals never do.

---

### Conda

#### Create
```bash
conda create -n myenv python=3.11      # -n names it; can pin Python version
conda create -n myenv python=3.11 numpy pandas   # with initial packages
```
Conda envs live centrally (`~/miniconda3/envs/`), not in your project folder.

#### Activate / deactivate
```bash
conda activate myenv
conda deactivate
```
Prompt shows `(myenv)`. `(base)` means the default conda env — treat installing into `base` like polluting system Python: avoid it.

#### List all envs / find the active one
```bash
conda env list        # lists all envs + paths; * marks the active one
```

#### Verify packages
```bash
conda list            # shows conda- AND pip-installed packages, plus source channel
python --version
which python           # confirm it points into the env
```

#### Add / remove packages
```bash
conda install scipy
conda remove scipy
pip install some-pip-only-pkg     # fallback for things not in conda channels
```
**Ordering rule (critical):** do all `conda install`s FIRST, then `pip` LAST. Don't `conda install` again after pip — conda can't track pip-installed packages and may clobber them. This is the #1 way conda envs break.

#### Limitations
- **Heavier and slower** than venv — bigger envs, slower solver. Use `mamba`/`miniforge` if the solver drags.
- The conda+pip mixing rule above is a real footgun.

#### Reproducibility
```bash
conda env export > environment.yml         # save full env spec
conda env create -f environment.yml        # recreate elsewhere
conda env remove -n myenv                   # delete, then recreate from yml
```

#### Set conda env as VS Code's default interpreter (per workspace)
1. `Ctrl/Cmd + Shift + P` → **Python: Select Interpreter**.
2. Pick your conda env from the list (it shows the env name + path).
3. VS Code saves this to the workspace — stored as `python.defaultInterpreterPath` in `.vscode/settings.json` (or the `.code-workspace` file).

To point at a **different** env later: rerun **Python: Select Interpreter** and choose another. If it's not listed, choose "Enter interpreter path" and browse to the env's `python` binary (`~/miniconda3/envs/<name>/bin/python`).

**Important:** this setting stores a *path*, not the env's contents. Commit `.vscode/settings.json` and a teammate gets pointed at the right env name — but if that env doesn't exist on their machine, the pointer is dead until they build it from `environment.yml`. The setting *selects* the env; it doesn't *create* it.

#### Make sure the correct env is loaded in VS Code
- **Interpreter (for scripts):** shown in the bottom-right status bar. Click it to switch.
- **Kernel (for notebooks):** shown top-right of the notebook. Click to change. This is separate from the interpreter — a notebook can run a different kernel than the interpreter selected for `.py` files, so check it independently.
- **Terminal:** even after selecting an interpreter in the UI, a terminal may or may not have it activated. Run `which python` / `conda env list` to confirm before installing.

---

### Jupyter notebooks — make an env available in the kernel dropdown

Two things are needed for an env to show up as a runnable kernel: (1) **`ipykernel`** installed *inside that env*, and (2) VS Code detecting the env. Installing `ipykernel` is usually enough on its own — VS Code auto-lists any env that has it. The manual "register as a kernel" step is only needed for classic Jupyter (browser) or when VS Code doesn't auto-detect the env.

#### conda env

Install jupyter + ipykernel **into the activated env** (not base):
```bash
conda activate myenv
conda install ipykernel jupyter      # or: pip install ipykernel jupyter
```

Then either:

**A) Just select it in VS Code (simplest).** Open the notebook → kernel picker (top-right) → pick `myenv`. VS Code lists it automatically once `ipykernel` is present. Done — no registration needed.

**B) Register it explicitly** (needed for classic/browser Jupyter, or if it won't appear in the dropdown):
```bash
python -m ipykernel install --user --name=myenv --display-name "Python (myenv)"
```
- `--name` = internal id (no spaces).
- `--display-name` = what shows in the dropdown.
- `--user` = installs for your user only (no admin needed).

Now "Python (myenv)" appears in the kernel dropdown in both VS Code and browser Jupyter.

#### venv

Same idea:
```bash
source .venv/bin/activate            # or .venv\Scripts\activate on Windows
pip install ipykernel jupyter
```
Then select `.venv` in the kernel picker (auto-detected), or register explicitly:
```bash
python -m ipykernel install --user --name=my-venv --display-name "Python (.venv)"
```

#### Managing registered kernels
```bash
jupyter kernelspec list              # show all registered kernels + paths
jupyter kernelspec uninstall myenv   # remove one (e.g. after deleting its env)
```
Deleting an env does **not** auto-remove its kernel entry — a stale kernel that points at a deleted env will error when selected. Uninstall the kernelspec to clean it up.

#### If the env doesn't appear in the dropdown
- Confirm `ipykernel` is in *that* env, not base: `conda list ipykernel` / `pip list | grep ipykernel` with the env active.
- Reload VS Code: `Ctrl+Shift+P` → **Developer: Reload Window**.
- Still missing → register explicitly with the `ipykernel install` command above.

---

#### Quick reference

| Task | venv | conda |
|---|---|---|
| Create | `python -m venv .venv` | `conda create -n myenv python=3.11` |
| Activate | `source .venv/bin/activate` | `conda activate myenv` |
| List packages | `pip list` | `conda list` |
| Add package | `pip install X` | `conda install X` |
| Remove package | `pip uninstall X` | `conda remove X` |
| Save env | `pip freeze > requirements.txt` | `conda env export > environment.yml` |
| Recreate | `pip install -r requirements.txt` | `conda env create -f environment.yml` |
| Which interpreter | `which python` | `which python` |
| List all envs | (they're per-folder `.venv/`) | `conda env list` |

### Things that will trip you later
- **`ipykernel` is required** for any env you want to run notebooks in. `pip install ipykernel` or `conda install ipykernel`, or VS Code will prompt you. One per env.
- **The kernel picker and the interpreter selector are different controls.** Selecting an interpreter doesn't set the notebook kernel.
- **Selecting an env in VS Code ≠ activating it in the terminal.** The UI choice doesn't guarantee your terminal is on that env.
- **Deleting a `.venv/` or `conda env remove` never touches your code** — envs are just package directories. Rebuild freely from your manifest.
- **`(base)` in conda / no marker in venv** both mean "you're about to install into a shared/system Python." Stop and activate the right env first.
  
