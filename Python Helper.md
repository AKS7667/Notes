# Python Setup Checklist

## One-time per computer

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

### Hardware-specific apt packages (install once, share across venvs)

```bash
sudo apt install -y python3-opencv python3-picamera2 python3-libcamera python3-numpy
```
Pi-only: install the libraries that bind to the camera and GPU stack; pip's versions of these are broken or blind to the hardware.

---

## Per project / per venv

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
