---
layout: default
title: Helper
---

# Helper Reference

Personal cheatsheet for tools, hardware, and workflows.

---

- [Simulators](#simulators)
- [Hardware](#hardware)
- [How Tos](#how-tos)
- [Dev Tools](#dev-tools)

---

## Simulators

<img src="/assets/images/notes/simulators.png" alt="Simulators overview" style="width:100%; max-height:220px; object-fit:cover; border-radius:8px; margin-bottom:1.5rem;">

### PyBullet

Lightweight physics engine, fast to get running. Good for quick prototyping and RL environments.

**Install**
```bash
pip install pybullet
```

**Basic usage**
```python
import pybullet as p
import pybullet_data

physicsClient = p.connect(p.GUI)          # or p.DIRECT for headless
p.setAdditionalSearchPath(pybullet_data.getDataPath())
p.setGravity(0, 0, -9.81)
planeId = p.loadURDF("plane.urdf")
robotId = p.loadURDF("r2d2.urdf", [0, 0, 1])

for _ in range(1000):
    p.stepSimulation()

p.disconnect()
```

**Learning / RL**
- Use with `gymnasium` via custom `Env` wrappers — override `reset()`, `step()`, `render()`
- PyBullet has built-in `pybullet_envs` (legacy), but wrapping your own URDF is better practice
- Control loop: `applyExternalForce`, `setJointMotorControl2` for torque/velocity/position control

---

### CoppeliaSim

Feature-rich simulator (formerly V-REP). Good for complex scenes and hardware-in-the-loop.

**Remote API (ZMQ — CoppeliaSim 4.3+)**
```bash
pip install coppeliasim-zmqremoteapi-client
```

```python
from coppeliasim_zmqremoteapi_client import RemoteAPIClient

client = RemoteAPIClient()
sim = client.require('sim')

sim.startSimulation()
handle = sim.getObject('/your_robot')
pos = sim.getObjectPosition(handle, sim.handle_world)
sim.stopSimulation()
```

**Learning / RL**
- Run CoppeliaSim headless: `coppeliaSim -h`
- Connect via ZMQ, wrap in a `gymnasium.Env` — `step()` calls `sim.step()` to advance one frame
- Use `sim.setJointTargetVelocity` / `sim.setJointTargetPosition` for joint control
- Scene state reset via `sim.stopSimulation()` → `sim.startSimulation()`

---

### MuJoCo

High-accuracy physics, the standard for robotics learning research. Maintained by Google DeepMind.

**Install**
```bash
pip install mujoco
pip install gymnasium[mujoco]
```

**Basic usage**
```python
import mujoco
import mujoco.viewer

model = mujoco.MjModel.from_xml_path("model.xml")
data  = mujoco.MjData(model)

with mujoco.viewer.launch_passive(model, data) as viewer:
    while viewer.is_running():
        mujoco.mj_step(model, data)
        viewer.sync()
```

**Learning / RL**
- Gymnasium wraps MuJoCo environments directly: `gymnasium.make("HalfCheetah-v4")`
- For custom robots: define a `.xml` (MJCF) model, wrap in `gymnasium.Env`
- `mujoco.mj_step` advances physics; read `data.qpos`, `data.qvel`, `data.sensordata`
- For manipulation: use `data.ctrl` to set actuator values

**MuJoCo XLA (MJX)**

MJX runs MuJoCo physics entirely on GPU/TPU via JAX. Enables massively parallel simulation for fast RL training.

```bash
pip install mujoco-mjx
```

```python
import mujoco
from mujoco import mjx
import jax
import jax.numpy as jnp

model = mujoco.MjModel.from_xml_path("model.xml")
mx    = mjx.put_model(model)        # move model to device
dx    = mjx.make_data(mx)           # create data on device

# JIT-compile and vmap for parallel envs
@jax.jit
@jax.vmap
def step(data):
    return mjx.step(mx, data)

# Batch N environments in one forward pass
batch = jax.vmap(lambda _: mjx.make_data(mx))(jnp.arange(4096))
batch = step(batch)
```

**Key points:**
- Only differentiable/JAX-compatible operations — no OpenGL rendering in MJX itself
- Use standard MuJoCo viewer for visualisation by copying data back to CPU: `mjx.get_data(model, dx)`
- Brax (DeepMind) is built on MJX — check their RL training loops for reference
- Not all MuJoCo features supported yet (check MJX docs for support table)

---

## Hardware

### Robotiq 2F-85 Gripper

<img src="/assets/images/notes/gripper_image.png" alt="Robotiq 2F-85 Gripper" style="width:100%; max-height:220px; object-fit:cover; border-radius:8px; margin-bottom:1.5rem;">

| Spec | Value |
|---|---|
| Max opening | 85 mm |
| Payload | 5 kg |
| Grasp types | External and internal |
| Power | 24 V DC |
| Communication | Modbus RTU over RS-485 |
| Cable | Single device cable (power + comms) |
| Object detection | Embedded — object detected bit (0/1) via indirect sensing |

**Key notes**
- Never supply with AC — DC only (24 V)
- Always respect the 5 kg payload limit
- For welding environments: ensure gripper is not on the ground path of the welding power source
- The single cable carries both power and Modbus RTU — no separate comms cable needed
- Controller model K1995 has limited documentation — check the [official manual](https://assets.robotiq.com/website-assets/support_documents/document/2F-85_2F-140_Instruction_Manual_e-Series_PDF_20190206.pdf)

---

### TCP and UDP

| | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented (handshake) | Connectionless |
| Reliability | Guaranteed delivery, ordered | No guarantee, no order |
| Speed | Slower (overhead) | Faster (low overhead) |
| Use when | Commands, config, control data | Sensor streams, high-freq data |

**In robotics:**
- Use **TCP** for gripper/robot commands — you need to know the command arrived
- Use **UDP** for high-frequency sensor data (camera frames, IMU) — speed matters more than guarantee
- Modbus RTU (gripper) runs over RS-485 serial, not TCP/UDP — separate layer

**Quick Python sockets**
```python
# TCP client
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("192.168.1.10", 502))
s.send(b"data")
response = s.recv(1024)

# UDP sender
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s.sendto(b"data", ("192.168.1.10", 5005))
```

---

### Wrist Connector

The wrist connector on the UR arm carries power and signal between the robot flange and end-effector (gripper, sensor, tool).

- Connection is **port-based** — the D-sub connector on the flange exposes the tool I/O port
- The 2F-85 uses the **single device cable** plugged into this port — carries 24 V + RS-485
- **Efficient connection tip:** Use the UR tool I/O port to power the gripper directly — avoids external power supply wiring
- Pin config: 2 pins for 24 V DC, 2 pins for RS-485 A/B, 1 GND — check UR e-Series manual for exact pinout
- Connector type: M8 or the D-sub depending on the adapter used

---

### Intel RealSense D435

Stereo depth camera with RGB. Used for scene understanding and object detection.

| Spec | Value |
|---|---|
| Type | Active IR stereo |
| Depth range | 0.2 – 10 m |
| Depth resolution | Up to 1280 × 720 |
| RGB resolution | Up to 1920 × 1080 |
| Depth FPS | Up to 90 fps |
| Interface | USB 3.1 Gen 1 |

**Install SDK**
```bash
# Ubuntu — add Intel repo first
sudo apt install librealsense2-dkms librealsense2-utils
pip install pyrealsense2
```

**Basic usage**
```python
import pyrealsense2 as rs
import numpy as np

pipeline = rs.pipeline()
config   = rs.config()
config.enable_stream(rs.stream.depth, 640, 480, rs.format.z16, 30)
config.enable_stream(rs.stream.color, 640, 480, rs.format.bgr8, 30)
pipeline.start(config)

try:
    frames      = pipeline.wait_for_frames()
    depth_frame = frames.get_depth_frame()
    color_frame = frames.get_color_frame()
    depth_image = np.asanyarray(depth_frame.get_data())
    color_image = np.asanyarray(color_frame.get_data())
finally:
    pipeline.stop()
```

**Align depth to colour**
```python
align     = rs.align(rs.stream.color)
frames    = align.process(pipeline.wait_for_frames())
depth     = frames.get_depth_frame()   # now aligned to RGB frame
```

---

## How Tos

### DH Calibration Pointers

DH (Denavit-Hartenberg) calibration corrects the nominal DH parameters of a robot to match the real physical robot.

**Why it matters**
- Nominal URDF/DH params have manufacturing tolerances (~1–2 mm error)
- After calibration: sub-mm repeatability is achievable
- Essential before any precision task (insertion, grasping, confined-space work)

**Process overview**

1. **Measure reference poses** — move robot to known joint configurations, record end-effector position with an external measurement tool (e.g. tracking system, CMM, or laser tracker)
2. **Set up optimisation** — minimise error between measured poses and forward kinematics (FK) using the DH params as free variables
3. **Optimisation target** — minimise `||p_measured - FK(q, θ_DH)||` over a set of N configurations
4. **Free variables** — typically `a`, `d`, `α`, `θ_offset` per joint — start by only freeing `θ_offset` (joint zero offsets) as they have the most impact
5. **Apply calibrated params** — update URDF or robot config file with corrected values

**Practical tips**
- Use diverse configurations — spread across the workspace, avoid collinear poses
- At least **3× the number of poses as free parameters** for a well-conditioned problem
- Lock distal joints first, calibrate from base outward
- Use `scipy.optimize.least_squares` or a dedicated tool like `robot_calibration` (ROS) or `DQ Robotics` calibration module
- Re-calibrate after any mechanical change or crash

**Dual quaternion approach (DQ Robotics)**
```python
# FK residual using DQ Robotics
from dqrobotics import *
from dqrobotics.robot_modeling import DQ_SerialManipulatorDH

# Define robot with current DH params
robot = DQ_SerialManipulatorDH(dh_matrix)

# Residual function for optimiser
def residual(params):
    # update robot DH params from 'params'
    # compute FK at each measured config
    # return vector of position errors
    ...
```

---

## Dev Tools

### Git Basics

```bash
# Setup
git config --global user.name  "Name"
git config --global user.email "email@example.com"

# Daily workflow
git status                         # what changed
git add <file>                     # stage specific file
git add .                          # stage everything
git commit -m "message"
git push origin main

# Branching
git checkout -b feature/my-feature # create + switch
git checkout main                  # switch back
git merge feature/my-feature       # merge into current
git branch -d feature/my-feature   # delete branch

# Undo
git restore <file>                 # discard unstaged changes
git restore --staged <file>        # unstage
git reset --soft HEAD~1            # undo last commit, keep changes staged

# Useful
git log --oneline                  # compact log
git diff                           # see unstaged changes
git stash                          # stash uncommitted work
git stash pop                      # restore stash
git pull --rebase origin main      # rebase instead of merge on pull
```

---

### Docker

**Key concepts**
- **Image** — blueprint (read-only)
- **Container** — running instance of an image
- **Dockerfile** — script to build an image
- **Volume** — persistent storage mounted into container

**Essential commands**
```bash
# Images
docker pull ubuntu:22.04
docker images
docker rmi image_name

# Containers
docker run -it ubuntu:22.04 bash         # interactive
docker run -d --name myapp ubuntu:22.04  # detached
docker run -v /host/path:/container/path # mount volume
docker run -p 8080:80 nginx              # port mapping

docker ps                                # running containers
docker ps -a                             # all containers
docker stop myapp
docker rm myapp
docker exec -it myapp bash               # shell into running container

# Build from Dockerfile
docker build -t my-image:latest .
docker build --no-cache -t my-image .

# Logs
docker logs myapp
docker logs -f myapp                     # follow
```

**Minimal robotics Dockerfile**
```dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    python3 python3-pip \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /workspace
COPY requirements.txt .
RUN pip3 install -r requirements.txt

COPY . .
CMD ["python3", "main.py"]
```

**Docker Compose (multi-container)**
```yaml
# docker-compose.yml
services:
  robot:
    build: .
    volumes:
      - .:/workspace
    ports:
      - "8080:8080"
    environment:
      - ROBOT_IP=192.168.1.10
```
```bash
docker compose up         # start
docker compose down       # stop + remove containers
docker compose up --build # rebuild before starting
```

---

### PyCharm

#### Basic Setup

- Set interpreter: `Settings → Project → Python Interpreter → Add Interpreter`
- Run config: `Run → Edit Configurations → + → Python`
- Useful shortcuts: `Shift+Shift` (search everywhere), `Ctrl+Alt+L` (reformat), `Ctrl+B` (go to definition)

---

#### Dev Containers (Docker inside PyCharm)

Requires **PyCharm Professional**.

1. Install Docker Desktop and the Docker plugin in PyCharm
2. Open project → `Settings → Build, Execution, Deployment → Docker` → connect to Docker daemon
3. Create a `Dockerfile` or `docker-compose.yml` in the project root
4. `Settings → Project → Python Interpreter → Add Interpreter → On Docker` — point to your image
5. PyCharm syncs files into the container and runs code inside it
6. Alternatively use `devcontainer.json` (VS Code format — PyCharm also supports it): `File → Open → Select devcontainer.json`

**devcontainer.json example**
```json
{
  "name": "Robotics Dev",
  "dockerFile": "Dockerfile",
  "mounts": ["source=${localWorkspaceFolder},target=/workspace,type=bind"],
  "postCreateCommand": "pip install -r requirements.txt",
  "customizations": {
    "jetbrains": {
      "backend": "PyCharm"
    }
  }
}
```

---

#### Remote Development via SSH (from anywhere)

**Step 1 — Set up SSH on the remote machine**
```bash
# On the remote machine (Linux)
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh

# Check it's running
sudo systemctl status ssh
```

**Step 2 — Connect with PyCharm**

1. `Settings → Build, Execution, Deployment → Deployment → +` → SFTP
2. Enter host IP, port (default 22), username, password or SSH key
3. Set **Root path** to your project folder on the remote
4. Set **Mappings**: local path → remote path
5. `Settings → Project → Python Interpreter → Add Interpreter → SSH Interpreter` — pick the deployment config
6. PyCharm uploads changes and runs code on the remote machine

**Step 3 — Access from anywhere (outside local network)**

Option A — **SSH port forwarding through a jumphost / VPN**
```bash
# On your laptop — forward local port 2222 to remote machine's port 22 via a jumphost
ssh -L 2222:remote_machine_ip:22 user@jumphost_ip
# Then connect PyCharm to localhost:2222
```

Option B — **Direct SSH with public IP** (if remote machine has a public IP)
```bash
# Ensure firewall allows port 22 (or custom port)
sudo ufw allow 22
# Use the public IP directly in PyCharm deployment config
```

Option C — **Tailscale (recommended for lab machines)**
```bash
# Install on both machines
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
# Tailscale assigns a stable private IP — use that in PyCharm from anywhere
```

**SSH key setup (avoid password every time)**
```bash
# On your laptop
ssh-keygen -t ed25519 -C "your_email"
ssh-copy-id user@remote_ip          # copies public key to remote
# Test
ssh user@remote_ip                  # should not ask for password
```
