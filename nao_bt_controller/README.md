# nao_bt_controller
# nao_bt_controller — Package Overview
This package implements the **Behaviour Tree (BT) runner/controller** used in the NAO HRI demo stack.
It is responsible for orchestrating the scenario by issuing high-level requests to other subsystems and
waiting for standardized completion signals before advancing the tree.
This README provides a **high-level map**, **how to launch**, and **what must be running first**.
## Contents
- [What this package does](#what-this-package-does)
- [Interfaces and dependencies](#interfaces-and-dependencies)
- [Prerequisites](#prerequisites)
- [Build](#build)
- [Run](#run)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)
- [License](#license)
---
## What this package does
`nao_bt_controller` executes a **Behaviour Tree** that coordinates the overall HRI workflow.
The BT runner triggers actions such as:
- requesting UI updates (web/UI bridge),
- requesting speech/TTS,
- requesting pose guidance and/or evaluation,
- requesting LED signalling,
and then waits for the corresponding completion/acknowledgement from the dedicated bridge/demo nodes.
In other words, the BT is only meaningful when the rest of the stack is running, because it “ticks”
those subsystems via ROS 2 communication.
This package focuses on **deterministic orchestration** and **traceable execution** (clear state
progression driven by BT tick results). It does not provide robot drivers/simulation bringup.
## Interfaces and dependencies
The BT runner assumes the presence of the companion stack that exposes the services/actions/topics
consumed by the tree (e.g., UI bridge, speech bridge, pose bridge, evaluation node, LED bridge).
Those nodes live in the repository packages such as `nao_hri_demo_nodes` and/or the UI client stack,
depending on your workspace layout.
> If the required nodes are not running (or their interfaces are not available), the BT will block
or fail at the corresponding tick.
## Prerequisites
- ROS 2 (Rolling) installed and sourced.
- `colcon` and standard ROS 2 build tooling.
- The rest of the demo stack running (bridges/stubs/evaluation/LED/UI components that the BT ticks).
> **Platform note (Ubuntu 22.04):**
> This package was developed and validated on **Ubuntu 22.04 (Jammy)** and **ROS 2 Rolling**.
---
## Build
From the workspace root:
```bash
source /opt/ros/rolling/setup.bash
colcon build
source install/setup.bash
```
---
## Run
Before launching the BT runner, ensure that the stack components it communicates with are already up
(bridges and demo nodes providing the interfaces the BT expects).
Launch the runner with:
```bash
python3 /bt_demo/nao_bt_controller/nao_bt_controller/bt_runner.py
```
---
## Configuration
The BT runner is designed to be **configuration-driven**. In practice, this means the Behaviour Tree
definition and its runtime parameters (e.g., selected scenario/state names, timeouts/holds, profile names,
and any IDs used by the bridges) are provided via **package resources and/or runtime parameters** rather
than being hardcoded in the orchestration logic.

To adapt the behaviour to a new demo/scenario:
- Locate the BT definition(s) and any associated configuration files shipped with this package (or referenced by it).
- Adjust the parameters/IDs that the tree uses to “tick” the rest of the stack (topic/service/action names,
  expected JSON keys, profile filenames, etc.), keeping them consistent with the nodes you are running.
- Prefer changing configuration values over modifying the BT runner code, so you can iterate safely and
  keep orchestration logic stable.
If you are unsure what is configurable in your current version, start from the `bt_runner.py` entrypoint and
follow the configuration loading/parameter declarations used at startup.
---
## Troubleshooting
- **BT blocks on a step / never advances:**
 The node providing the corresponding interface is not running, is misconfigured, or is using a different
 namespace/topic/service name than expected.
- **Immediate failures at startup:**
 Verify your ROS 2 environment is sourced and the workspace overlay is active (`install/setup.bash`).
---
## License
See the repository license file (or the `package.xml` metadata, if applicable).
