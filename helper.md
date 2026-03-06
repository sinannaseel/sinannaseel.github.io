---
layout: default
title: Helper
---

<style>
.helper-page h1 {
  font-size: 1.4rem;
  font-weight: 600;
  margin-bottom: 0.25rem;
  color: var(--text-primary);
}
.helper-page .subtitle {
  color: var(--text-secondary);
  font-size: 0.9rem;
  margin-bottom: 2rem;
}
.helper-section {
  margin-bottom: 0.75rem;
  border: 1px solid var(--border);
  border-radius: 8px;
  overflow: hidden;
}
.helper-section > summary {
  list-style: none;
  padding: 1rem 1.25rem;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: var(--bg-card);
  font-weight: 600;
  font-size: 0.95rem;
  color: var(--text-primary);
  user-select: none;
}
.helper-section > summary::-webkit-details-marker { display: none; }
.helper-section > summary::after {
  content: '+';
  font-size: 1.1rem;
  color: var(--text-secondary);
  font-weight: 300;
}
.helper-section[open] > summary::after { content: '−'; }
.helper-section > summary:hover { background: var(--bg-secondary); }
.helper-section-body {
  padding: 0.5rem 0.75rem 0.75rem;
  background: var(--bg-primary);
}
.helper-article {
  margin-bottom: 0.4rem;
  border: 1px solid var(--border);
  border-radius: 6px;
  overflow: hidden;
}
.helper-article > summary {
  list-style: none;
  padding: 0.65rem 1rem;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: var(--bg-secondary);
  font-size: 0.88rem;
  color: var(--text-secondary);
  user-select: none;
}
.helper-article > summary::-webkit-details-marker { display: none; }
.helper-article > summary::after {
  content: '›';
  font-size: 1rem;
  color: var(--text-muted);
}
.helper-article[open] > summary {
  color: var(--accent);
  border-bottom: 1px solid var(--border);
}
.helper-article[open] > summary::after { content: '›'; transform: rotate(90deg); display: inline-block; }
.helper-article > summary:hover { color: var(--text-primary); }
.helper-article-body {
  padding: 1rem 1.25rem;
  background: var(--bg-primary);
  font-size: 0.88rem;
  line-height: 1.7;
  color: var(--text-secondary);
}
.helper-article-body pre {
  background: var(--bg-secondary);
  border: 1px solid var(--border);
  border-radius: 6px;
  padding: 0.75rem 1rem;
  overflow-x: auto;
  margin: 0.75rem 0;
  font-size: 0.82rem;
}
.helper-article-body code {
  font-family: 'SF Mono', 'Fira Code', monospace;
}
.helper-article-body p { margin-bottom: 0.6rem; }
.helper-article-body ul, .helper-article-body ol {
  padding-left: 1.25rem;
  margin-bottom: 0.6rem;
}
.helper-article-body strong { color: var(--text-primary); }
.helper-article-body table {
  width: 100%;
  border-collapse: collapse;
  margin: 0.75rem 0;
  font-size: 0.85rem;
}
.helper-article-body th, .helper-article-body td {
  padding: 0.5rem 0.75rem;
  border: 1px solid var(--border);
  text-align: left;
}
.helper-article-body th { background: var(--bg-secondary); color: var(--text-primary); }
</style>

<div class="helper-page">
<h1>Helper</h1>
<p class="subtitle">Personal cheatsheet — click to expand.</p>

<details class="helper-section">
<summary>Simulators</summary>
<div class="helper-section-body">

<details class="helper-article">
<summary>PyBullet</summary>
<div class="helper-article-body">

Lightweight physics engine, fast to get running. Good for quick prototyping and RL environments.

**Install**
```bash
pip install pybullet
```

**Basic usage**
```python
import pybullet as p
import pybullet_data

physicsClient = p.connect(p.GUI)
p.setAdditionalSearchPath(pybullet_data.getDataPath())
p.setGravity(0, 0, -9.81)
planeId = p.loadURDF("plane.urdf")
robotId = p.loadURDF("r2d2.urdf", [0, 0, 1])

for _ in range(1000):
    p.stepSimulation()

p.disconnect()
```

**RL**
- Wrap in `gymnasium.Env` — override `reset()`, `step()`, `render()`
- Control: `applyExternalForce`, `setJointMotorControl2`

</div>
</details>

<details class="helper-article">
<summary>CoppeliaSim</summary>
<div class="helper-article-body">

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

**RL**
- Headless: `coppeliaSim -h`
- Wrap in `gymnasium.Env`, `step()` calls `sim.step()`
- Reset: `sim.stopSimulation()` → `sim.startSimulation()`

</div>
</details>

<details class="helper-article">
<summary>MuJoCo</summary>
<div class="helper-article-body">

High-accuracy physics, standard for robotics research. Maintained by Google DeepMind.

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

**RL**
- Built-in envs: `gymnasium.make("HalfCheetah-v4")`
- Custom: define `.xml` (MJCF), wrap in `gymnasium.Env`
- Read: `data.qpos`, `data.qvel`; control: `data.ctrl`

</div>
</details>

<details class="helper-article">
<summary>MuJoCo XLA (MJX)</summary>
<div class="helper-article-body">

Runs MuJoCo on GPU/TPU via JAX. Massively parallel simulation for fast RL training.

```bash
pip install mujoco-mjx
```

```python
import mujoco
from mujoco import mjx
import jax, jax.numpy as jnp

model = mujoco.MjModel.from_xml_path("model.xml")
mx = mjx.put_model(model)
dx = mjx.make_data(mx)

@jax.jit
@jax.vmap
def step(data):
    return mjx.step(mx, data)

batch = jax.vmap(lambda _: mjx.make_data(mx))(jnp.arange(4096))
batch = step(batch)
```

- Copy back to CPU for visualisation: `mjx.get_data(model, dx)`
- Brax is built on MJX — good reference for RL loops

</div>
</details>

</div>
</details>

<details class="helper-section">
<summary>Hardware</summary>
<div class="helper-section-body">

<details class="helper-article">
<summary>Robotiq 2F-85 Gripper</summary>
<div class="helper-article-body">

| Spec | Value |
|---|---|
| Max opening | 85 mm |
| Payload | 5 kg |
| Power | 24 V DC |
| Communication | Modbus RTU over RS-485 |

- DC only — never AC
- Single cable carries power + Modbus RTU
- Object detection via embedded indirect sensing (0/1 bit)

</div>
</details>

<details class="helper-article">
<summary>TCP vs UDP</summary>
<div class="helper-article-body">

| | TCP | UDP |
|---|---|---|
| Connection | Handshake | Connectionless |
| Reliability | Guaranteed, ordered | No guarantee |
| Use when | Commands, control | Sensor streams, high-freq |

```python
# TCP client
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("192.168.1.10", 502))
s.send(b"data")

# UDP sender
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s.sendto(b"data", ("192.168.1.10", 5005))
```

</div>
</details>

<details class="helper-article">
<summary>Wrist Connector (UR)</summary>
<div class="helper-article-body">

- D-sub connector on UR flange exposes tool I/O port
- 2F-85 single cable plugs in directly — carries 24 V + RS-485
- Pin config: 2× 24 V DC, 2× RS-485 A/B, 1× GND
- Connector type: M8 or D-sub depending on adapter

</div>
</details>

<details class="helper-article">
<summary>Intel RealSense D435</summary>
<div class="helper-article-body">

| Spec | Value |
|---|---|
| Type | Active IR stereo |
| Depth range | 0.2 – 10 m |
| Depth resolution | Up to 1280 × 720 |
| RGB resolution | Up to 1920 × 1080 |
| Interface | USB 3.1 Gen 1 |

```bash
pip install pyrealsense2
```

```python
import pyrealsense2 as rs
import numpy as np

pipeline = rs.pipeline()
config = rs.config()
config.enable_stream(rs.stream.depth, 640, 480, rs.format.z16, 30)
config.enable_stream(rs.stream.color, 640, 480, rs.format.bgr8, 30)
pipeline.start(config)

frames = pipeline.wait_for_frames()
depth  = np.asanyarray(frames.get_depth_frame().get_data())
color  = np.asanyarray(frames.get_color_frame().get_data())
pipeline.stop()
```

</div>
</details>

</div>
</details>

<details class="helper-section">
<summary>How Tos</summary>
<div class="helper-section-body">

<details class="helper-article">
<summary>DH Calibration</summary>
<div class="helper-article-body">

Corrects nominal DH parameters to match the real robot. Achieves sub-mm repeatability.

**Process**
1. Move robot to known configs, record end-effector positions externally
2. Minimise `||p_measured - FK(q, θ_DH)||` with DH params as free variables
3. Free variables: `a`, `d`, `α`, `θ_offset` per joint — start with `θ_offset` only
4. Apply calibrated params to URDF or robot config

**Tips**
- Use diverse workspace poses — avoid collinear configurations
- At least 3× poses vs free parameters
- Calibrate base outward, lock distal joints first
- Re-calibrate after any mechanical change or crash

</div>
</details>

</div>
</details>

<details class="helper-section">
<summary>Dev Tools</summary>
<div class="helper-section-body">

<details class="helper-article">
<summary>Git</summary>
<div class="helper-article-body">

```bash
# Daily
git status
git add .
git commit -m "message"
git push origin main

# Branching
git checkout -b feature/name
git merge feature/name
git branch -d feature/name

# Undo
git restore <file>             # discard unstaged
git restore --staged <file>    # unstage
git reset --soft HEAD~1        # undo last commit, keep staged

# Useful
git log --oneline
git stash / git stash pop
git pull --rebase origin main
```

</div>
</details>

<details class="helper-article">
<summary>Docker</summary>
<div class="helper-article-body">

```bash
# Images
docker pull ubuntu:22.04
docker build -t my-image:latest .

# Containers
docker run -it ubuntu:22.04 bash
docker run -d --name myapp -p 8080:80 -v /host:/container ubuntu:22.04
docker ps / docker ps -a
docker stop myapp && docker rm myapp
docker exec -it myapp bash
docker logs -f myapp
```

**Minimal Dockerfile**
```dockerfile
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y python3 python3-pip \
    && rm -rf /var/lib/apt/lists/*
WORKDIR /workspace
COPY requirements.txt .
RUN pip3 install -r requirements.txt
COPY . .
CMD ["python3", "main.py"]
```

</div>
</details>

<details class="helper-article">
<summary>PyCharm</summary>
<div class="helper-article-body">

**Setup**
- Interpreter: `Settings → Project → Python Interpreter → Add`
- Run config: `Run → Edit Configurations → + → Python`
- Shortcuts: `Shift+Shift` search, `Ctrl+B` go to definition, `Ctrl+Alt+L` reformat

**Remote via SSH**
1. `Settings → Deployment → +` → SFTP, enter host/port/user
2. `Settings → Interpreter → SSH Interpreter` — pick deployment config
3. For access anywhere: use Tailscale (`tailscale up` on both machines)

</div>
</details>

</div>
</details>

</div>
