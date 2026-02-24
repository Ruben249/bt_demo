# bt_demo
# BT_DEMO — Repository Overview
This repository groups several ROS 2 packages used to build a Human–Robot
Interaction (HRI) demo stack around NAO.
Each package is documented separately; this README provides a **high-level map** and
**where to look next**.
## Contents
- [Packages](#packages)
- [Prerequisites](#prerequisites)
- [Build](#build)
- [Run](#run)
- [License](#license)
---

## What this repository does (and does not)

This repository provides a **ROS 2 HRI demo stack for NAO**: a Behaviour Tree runner plus “glue” nodes that translate high-level BT commands into **ROS 2 services/actions** (web UI, speech/TTS, pose/evaluation, LEDs) and report standardized results back to the BT.  
It **does** focus on integration, traceability, and configuration-driven behaviour (e.g., YAML LED profiles) so the scenario can be iterated without recompiling everything.  
It **does not** implement the full robot hardware stack end-to-end (drivers/simulation bringup are out of scope here and live elsewhere), nor does it provide a single monolithic launcher for every deployment.  
It **does not** define the semantics of UI/LED “emotions” inside the LED server; those remain **client-side policies** expressed via profiles and bridges.  


## Packages
> Links below point to the package directories 


- [`nao_bt_controller`](./nao_bt_controller) — **Behaviour Tree (BT) runner/controller**  
  Orchestrates the overall HRI workflow. It publishes high-level commands (e.g., UI, speech, pose, evaluation) and waits for standardized results, so the interaction logic stays deterministic and easy to trace.

- [`nao_hri_demo_nodes`](./nao_hri_demo_nodes) — **Demo ROS 2 nodes and helper scripts (stubs/bridges/evaluation)**  
  Provides the “glue” components that connect the BT to external subsystems: the web UI service, TTS/audio playback, pose/exercise evaluation nodes, and other integration helpers. These nodes translate BT commands into concrete ROS 2 service/action calls and report completion back to the BT.

- [`nao_led_profiles`](./nao_hri_demo_nodes/nao_ui_client/nao_led_profiles) — **YAML-based LED state/emotion profiles**  
  Contains declarative LED patterns (mode, timing, colors/intensities, targeted effectors) that can be loaded at runtime. This keeps LED behaviour configurable (no recompilation) and ensures consistent visual signalling across nodes and hardware setups.

- [`nao_ui_utils`](./nao_hri_demo_nodes/nao_ui_client/nao_ui_utils) — **Shared UI utilities**  
  Small reusable helpers for UI-related state/message handling used across multiple nodes (e.g., stubs/bridges). This avoids duplicated parsing/formatting logic and keeps UI integration consistent across the system.


---
## Prerequisites
- ROS 2 (Rolling) installed and sourced.
- `colcon` and standard ROS 2 build tooling.
- A working workspace (example name: `nao_ws`) with this repository placed under
`src/`.
> Any extra dependencies that are package-specific are documented in each package
README.
---
## Build
From the workspace root (e.g., `nao_ws`):
```bash
source /opt/ros/rolling/setup.bash
colcon build
source install/setup.bash
```
---
## Run
There is no single “one command” launcher in this top-level README because execution
is typically split across multiple terminals (or tmux).
Start with the package READMEs, in this order:
1. `nao_hri_demo_nodes` (brings up bridges/stubs and demo nodes used by the BT)
2. `nao_led_profiles` (ensure profiles are discoverable; required by the UI/LED bridge)
3. `nao_bt_controller` (BT runner that drives the scenario once subsystems are up)
---

> **Platform note (Ubuntu 22.04):**  
> These packages were developed and validated on **Ubuntu 22.04 (Jammy)** and ROS2 rolling. They can be executed either **natively** on Ubuntu 22.04 or inside an **Ubuntu 22.04 Distrobox** container (recommended when isolating dependencies or keeping the host clean).


## License
See the repository license file (or the per-package `package.xml` metadata, if
applicable).
