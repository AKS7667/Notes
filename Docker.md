# RobotArm-sim

A ready-made Docker setup for the robot arm simulation. It packages ROS 2 Jazzy, Gazebo Harmonic, MoveIt 2 and OpenCV into one container, so everyone gets exactly the same software on Windows or Linux, without installing ROS on their own computer.

## Key ideas

| Term | What it is | Analogy |
|---|---|---|
| **Dockerfile** | A text file listing every step needed to set up the software | A recipe |
| **Image** | The result of running those steps once, saved as a snapshot | A frozen meal cooked from the recipe |
| **Container** | A running copy of the image: a small, isolated Linux computer | Heating up one portion |
| **Bind mount** | A folder shared between your computer and the container | A shared drive |

**Why this matters:** the image is built once and reused. Containers are disposable, so you can delete one and start a fresh one in seconds. Your code survives because it lives in the shared folder, not inside the container.

## Folder contents

| File | Purpose |
|---|---|
| `Dockerfile` | The recipe for the image |
| `docker-compose.yml` | How to run the container: display, shared folder, settings (Windows/WSL defaults) |
| `docker-compose.linux.yml` | Extra settings added automatically on Linux (real display, graphics chip) |
| `sim.sh` | One command that builds, starts and opens a terminal in the container |
| `ws/src/` | Your ROS 2 packages go here (created on first run) |

---

## 1. Install Docker

### Windows (WSL 2)

1. Open **PowerShell as administrator** and run `wsl --install`. Restart the computer.
   *Why:* containers are Linux programs. WSL 2 gives Windows a real Linux system, and its display feature (WSLg) lets Linux windows such as Gazebo appear on your Windows desktop.
2. Open **Ubuntu** from the Start menu and create a username and password.
3. Download and install **Docker Desktop** from docker.com. Keep **"Use WSL 2"** ticked during install.
4. Open Docker Desktop → **Settings** (gear icon) → **Resources → WSL integration** → turn on **Ubuntu** → **Apply & restart**.
   *Why:* this lets the `docker` command inside Ubuntu use Docker Desktop's engine.
5. Open a new Ubuntu terminal and test: `docker run --rm hello-world`. It should print "Hello from Docker!".

> Do **not** run the `get.docker.com` install script inside WSL when using Docker Desktop. Two Docker installs on one system clash.
> Docker Desktop must be **open** whenever you use the sim.

### Linux (Ubuntu)

1. Install Docker Engine: `curl -fsSL https://get.docker.com | sh`
   *Why:* the official script installs Docker and the `docker compose` tool in one step.
2. Let your user run Docker without `sudo`: `sudo usermod -aG docker $USER`
3. Log out and log back in.
   *Why:* group changes only apply to new login sessions.
4. Test: `docker run --rm hello-world`

> Use Docker **Engine** (step 1), not Docker Desktop for Linux. Docker Desktop runs containers inside a virtual machine that cannot reach your screen or graphics chip.

---

## 2. Get the project

1. Put the `RobotArm-sim` folder in your Linux home folder, so it is at `~/RobotArm-sim`.
   On Windows, copy it from Windows into Ubuntu: `cp -r /mnt/c/Users/YOURNAME/Downloads/RobotArm-sim ~/`
   *Why:* files on the C: drive are slow to reach from Linux and lose Linux file permissions.
2. `cd ~/RobotArm-sim`
3. `chmod +x sim.sh`
   *Why:* marks the script as a program you are allowed to run.

---

## 3. How the image is built: the Dockerfile, step by step

Docker reads the `Dockerfile` top to bottom. Each instruction adds one **layer** to the image.

```dockerfile
FROM osrf/ros:jazzy-desktop-full
```
Start from the official ROS image: Ubuntu 24.04 with ROS 2 Jazzy, RViz and the standard tools already installed.
*Why:* installing ROS from scratch takes many steps; starting from a tested base skips all of them.

```dockerfile
ARG USERNAME=dev
ARG USER_UID=1000
ARG USER_GID=1000
```
Settings that can be changed at build time. `sim.sh` fills in your real user ID and group ID.
*Why:* files the container creates in `ws/` are then owned by **you**, not by the administrator account (root), so you can edit and delete them normally.

```dockerfile
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y \
      ros-jazzy-ros-gz ... \
    && rm -rf /var/lib/apt/lists/*
```
Download the package list, install the simulation tools, then delete the list to keep the image smaller. `-y` and `noninteractive` answer every installer question automatically, because nobody is there to type during a build.

| Package | What it adds |
|---|---|
| `ros-jazzy-ros-gz` | Gazebo Harmonic, plus the bridge that passes messages between Gazebo and ROS |
| `ros-jazzy-gz-ros2-control` | Lets ros2_control drive the arm's joints inside Gazebo |
| `ros-jazzy-ros2-control`, `ros-jazzy-ros2-controllers` | Standard joint controllers |
| `ros-jazzy-moveit` | Motion planning for the arm |
| `ros-jazzy-joint-state-publisher-gui` | Sliders to move each joint by hand when testing a robot model |
| `ros-jazzy-xacro` | Shortcuts (macros) for writing robot model (URDF) files |
| `ros-jazzy-cv-bridge`, `python3-opencv` | Turns ROS camera images into OpenCV images |
| `mesa-utils` | `glxinfo`, for checking how 3D graphics are being drawn |
| `sudo nano git bash-completion` | Everyday tools |

```dockerfile
RUN userdel -r ubuntu 2>/dev/null || true \
 && (getent group ${USER_GID} >/dev/null || groupadd --gid ${USER_GID} ${USERNAME}) \
 && useradd --uid ${USER_UID} --gid ${USER_GID} --create-home --shell /bin/bash ${USERNAME} \
 && echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/${USERNAME} \
 && chmod 0440 /etc/sudoers.d/${USERNAME}
```
Create a normal user called `dev` with your IDs and give it `sudo` without a password. The Ubuntu 24.04 base already has a user called `ubuntu` holding ID 1000, the ID most people have, so it is removed first.
*Why:* working as a normal user is safer than working as root, and matching IDs keeps file ownership correct.

```dockerfile
USER ${USERNAME}
ENV QT_X11_NO_MITSHM=1 \
    XDG_RUNTIME_DIR=/tmp/runtime-${USERNAME}
```
Everything from here on runs as `dev`. The two settings make Gazebo and RViz windows draw correctly through the shared display and silence a startup warning.

```dockerfile
RUN mkdir -p -m 0700 /tmp/runtime-${USERNAME} \
 && mkdir -p /home/${USERNAME}/ws/src \
 && rosdep update
```
Create the folders above and download `rosdep`'s list, which maps ROS package dependencies to installable packages.
*Why:* later, `rosdep install` can automatically fetch whatever a new package needs.

```dockerfile
RUN cat >> /home/${USERNAME}/.bashrc <<'EOF'
source /opt/ros/jazzy/setup.bash
...
alias build='cd ~/ws && colcon build --symlink-install && source install/setup.bash'
EOF
```
Add lines to `.bashrc`, the file every new terminal runs at startup. Each terminal then loads ROS and your built packages automatically, and gets a `build` shortcut.
*Why:* forgetting to load (`source`) ROS is the most common beginner error.

```dockerfile
WORKDIR /home/${USERNAME}/ws
CMD ["bash"]
```
Terminals open in the workspace folder; the default program is a terminal.

### Building the image by hand

`./sim.sh` builds the image for you, but this is the underlying command:

```bash
docker build -t robotarm-sim:jazzy --build-arg USER_UID=$(id -u) --build-arg USER_GID=$(id -g) .
```

| Part | Meaning |
|---|---|
| `docker build` | Follow the Dockerfile and produce an image |
| `-t robotarm-sim:jazzy` | Name the image `robotarm-sim`, with the label `jazzy` |
| `--build-arg ...` | Fill in the `ARG` values with your user and group IDs |
| `.` | The build context: the current folder, where Docker finds the Dockerfile |

Run `docker images` to see the finished image.

**Layer caching:** the first build takes 10–20 minutes and downloads several GB. Docker saves every layer, so a later build only redoes the steps from the first changed line downward. That is why slow, rarely changed steps (installing packages) sit near the top.

---

## 4. How the container is run: docker-compose.yml

Docker Compose stores all the run settings in one file, so nobody has to type a long `docker run` command.

| Setting | Why |
|---|---|
| `build` + `args` | Build from the Dockerfile in this folder, using your IDs |
| `command: sleep infinity` | Keeps the container running in the background, so you can open as many terminals as you like |
| `init: true` | Lets the container shut down instantly when stopped |
| `DISPLAY` + the `.X11-unix` volume | Tells programs where to draw windows: the display connection shared by WSLg |
| `QT_QPA_PLATFORM: xcb` | Makes Gazebo and RViz use X11, the most reliable display method inside containers |
| `LIBGL_ALWAYS_SOFTWARE: "1"` | On WSL, 3D drawing is done by the CPU, because passing the graphics chip into containers is unreliable there |
| `ROS_AUTOMATIC_DISCOVERY_RANGE: LOCALHOST` | ROS programs only find each other on this computer, not on other machines on the same Wi-Fi |
| `GZ_IP: 127.0.0.1` | Keeps Gazebo's internal messages on this computer, which stops the window losing contact with the simulator when the network changes |
| `./ws:/home/dev/ws` | Shares your workspace: edit code with any editor on your computer, run it in the container |

On Linux, `docker-compose.linux.yml` is layered on top. It swaps in the normal display connection (`/tmp/.X11-unix`), passes in the graphics chip (`/dev/dri`) with the permissions to use it, and turns software drawing off.

### What sim.sh does

1. Reads your user and group IDs.
2. Detects Linux or WSL. On Linux it adds `docker-compose.linux.yml` and runs `xhost +local:`, which allows local programs, including containers, to open windows on your screen.
3. Creates `ws/src` if it is missing.
4. `docker compose up -d --build`: builds the image if it is missing or the Dockerfile changed, then starts the container in the background (`-d`).
5. `docker compose exec sim bash`: opens a terminal inside the running container.

---

## 5. Opening GUI apps (Gazebo, RViz)

There is nothing extra to set up. Windows opened inside the container appear on your desktop like normal apps; on Windows they also show up in the taskbar.

1. In your Ubuntu (WSL) terminal, run `./sim.sh`. The prompt changes to `dev@...`.
2. Run `gz sim shapes.sdf`. A Gazebo window opens on your desktop.
3. That terminal stays busy while Gazebo runs. To type more commands, open a new Ubuntu window and run `./sim.sh` again.
4. Close Gazebo with the window's X button, or press `Ctrl+C` in its terminal.

**How the window gets from the container to your screen**

- **Windows (WSL):** Windows runs a built-in display server (WSLg) that draws Linux windows on the Windows desktop. Linux programs reach it through a connection file, `/mnt/wslg/.X11-unix/X0`. `docker-compose.yml` shares that file into the container at `/tmp/.X11-unix/X0`, and `DISPLAY=:0` tells programs to use it. No separate display app (such as VcXsrv or Xming) is needed.
- **Linux:** the same idea, using your own desktop's connection file in `/tmp/.X11-unix`.

---

## 6. Daily use

| Task | Command | Run it on |
|---|---|---|
| Start and open a terminal | `./sim.sh` | Your computer, in `~/RobotArm-sim` |
| Open another terminal | `./sim.sh` in a new window | Your computer |
| Build your packages | `build` | Inside the container |
| Test Gazebo | `gz sim shapes.sdf` | Inside the container |
| Test RViz | `rviz2` | Inside the container |
| Stop everything | `./sim.sh stop` | Your computer |

A prompt starting with `dev@` means you are inside the container.

**What is kept and what is lost**

- **Kept:** everything in `ws/`, because it lives on your computer.
- **Lost when the container stops:** any other change made inside it, such as `sudo apt install something`.
  To keep a new package, add it to the `apt-get install` list in the Dockerfile and run `./sim.sh`, which rebuilds the image automatically.

---

## 7. Troubleshooting

| Problem | Fix |
|---|---|
| `docker` not found (WSL) | Turn on WSL integration (Section 1, Windows step 4) |
| "Cannot connect to the Docker daemon" (WSL) | Open Docker Desktop and wait until it says the engine is running |
| "permission denied … docker.sock" (Linux) | Log out and back in after the `usermod` step |
| No window appears (WSL) | Run `ls /mnt/wslg/.X11-unix/X0`. If it is missing, run `wsl --update` then `wsl --shutdown` in PowerShell, and reopen Ubuntu |
| No window appears (Linux) | Run `./sim.sh` again from a terminal on the desktop, so `xhost` runs again |
| Gazebo is slow | Inside the container run `glxinfo -B` and read the "OpenGL renderer" line. `llvmpipe` means the CPU is drawing (expected on WSL); on Linux it should name your graphics chip |
