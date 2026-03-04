# nao_hri_demo_nodes
# nao_hri_demo_nodes — Package Overview
This package contains the **demo nodes and bridge scripts** used by the NAO HRI demo stack.
Its purpose is to provide the “glue” between the BT runner (`nao_bt_controller`) and the
external subsystems (speech/TTS, pose guidance, OpenCV-based evaluation, UI/web bridge, LEDs, etc.).
The BT runner “ticks” these components via ROS 2 communication. Therefore, this package is typically
launched **before** starting the BT so that all expected interfaces are available.
## Contents
- [What this package does](#what-this-package-does)
- [Repository layout](#repository-layout)
- [Prerequisites](#prerequisites)
- [Build](#build)
- [Run](#run)
- [How to verify and simulate a tick](#how-to-verify-and-simulate-a-tick)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)
- [License](#license)
---
## What this package does
`nao_hri_demo_nodes` provides:
- **Bridge nodes** that translate high-level BT requests into concrete actions (speech, pose, UI, etc.).
- **Evaluation nodes** that compute and publish results (e.g., Mediapipe/OpenCV-based exercise evaluation).
- **Configuration assets** used by the nodes (e.g., exercise catalog, OpenCV templates).
This package is integration-focused: it does not replace robot drivers/simulation bringup; it enables
repeatable HRI scenarios by exposing stable ROS 2 interfaces that the BT can call.
## Repository layout
- `scripts/`
- `speak_bridge.py` — Speech/TTS bridge. Receives a high-level “speak” request and delegates it
to a **ROS 2 service provided by an external package** (`simple_hri`), then reports completion
back to the requester.
- Upstream service provider: https://github.com/rodperex/simple_hri
- `nao_pos_bridge.py` — Pose/position bridge. Receives pose-related requests (e.g., guidance,
pose selection, start/stop), forwards them to the relevant subsystem, and reports progress/results.
- `opencv_eval_node.py` — Vision/evaluation node. Uses **MediaPipe** for landmark extraction and
OpenCV utilities for evaluation logic. Loads templates from `config/opencv_templates/` when needed,
and publishes standardized evaluation outputs.
- `config/`
- `exercises_catalog.yml` — Catalog describing the available exercises/scenarios and their parameters.
- `opencv_templates/` — Templates/resources used by the vision/evaluation node.
## Prerequisites
- ROS 2 (Rolling) installed and sourced.
- `colcon` and standard ROS 2 build tooling.
- Runtime dependencies:- **MediaPipe** (for pose/landmark extraction): https://google.github.io/mediapipe/
- **Google Cloud** (for Text-to-Speech, when using the cloud TTS backend):
- Google Cloud Text-to-Speech: https://cloud.google.com/text-to-speech
- Python client library: https://cloud.google.com/python/docs/reference/texttospeech/latest
- The external speech service package used by `speak_bridge.py`:
- `simple_hri`: https://github.com/rodperex/simple_hri
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
> Recommended order: bring up the **backend servers** first (the ones these scripts connect to), then
> launch the bridge/demo nodes from this package, and finally start the BT runner.
### 0) Start required backend servers (must be running first)
These scripts are **clients/bridges**: they only work correctly if the corresponding server-side nodes
are already up (service/action/topic providers).
Typical examples:
- **Speech/TTS backend server**: provides the ROS 2 service that `speak_bridge.py` calls.
- **LED action server**: provides the `leds_play` action if your flow uses LED profiles/actions.
- **Any pose/execution server**: if your pose bridge forwards requests to another node.
- **Any evaluation dependencies**: camera/streams or producers required by the evaluation pipeline.
> If a required server is not running, the bridge will either block waiting, time out, or drop requests.
> Use `ros2 node list`, `ros2 service list`, and `ros2 action list` to confirm providers exist.
### 1) Speech/TTS bridge (`speak_bridge.py`)
```bash
python3 ./src/thirdparty/bt_demo/nao_hri_demo_nodes/scripts/speak_bridge.py
```
This node forwards high-level speak requests to an external **ROS 2 service provider**.
In **my setup**, the service provider comes from the `simple_hri` stack (and may use Google TTS), but
**in your setup it may be a different node/service** depending on the speech pipeline you use.
- My provider: https://github.com/rodperex/simple_hri
- If you use a different TTS node, update the bridge configuration or service name accordingly.
### 2) Pose bridge (`nao_pos_bridge.py`)
```bash
python3 ./src/thirdparty/bt_demo/nao_hri_demo_nodes/scripts/nao_pos_bridge.py
```
This node forwards pose/execution requests. Ensure any **pose server** it depends on is already running
(if your deployment uses one), otherwise requests will not complete.
### 3) Vision/evaluation node (`opencv_eval_node.py`)
```bash
python3 ./src/thirdparty/bt_demo/nao_hri_demo_nodes/scripts/opencv_eval_node.py
```
This node runs the evaluation loop (MediaPipe/OpenCV). Ensure that all runtime inputs it needs (camera,
video source, templates under `config/opencv_templates/`, etc.) are available on your machine.
> If you use a different workspace layout, adjust the script paths accordingly.
> If these scripts are installed as ROS 2 executables in your environment, you can also run them via
> `ros2 run <package> <executable>` (check with `ros2 pkg executables nao_hri_demo_nodes`).
### Quick sanity checks (recommended)
After launching each node, verify the graph:
```bash
ros2 node list
ros2 node info <node_name>
ros2 service list
ros2 action list
ros2 topic list
```
If something does not appear, you are either in the wrong environment (not sourced) or a required server
is missing.
## How to verify and simulate a tick
Because the BT runner calls into these nodes through ROS 2 interfaces, you can validate everything
without the BT by following a three-step approach:
### Step A — Confirm the nodes are running
In a new terminal (with the workspace sourced):```bash
ros2 node list
```
Then inspect each node:
```bash
ros2 node info <node_name>
```
### Step B — Discover the exposed interfaces
List available interfaces and identify which ones correspond to each script:
```bash
ros2 service list
ros2 action list
ros2 topic list
```
To understand what types are used:
```bash
ros2 service type <service_name>
ros2 action info <action_name>
ros2 topic info <topic_name>
```
### Step C — Simulate a tick (realistic examples)
Below are **real commands used during development** to validate end-to-end behaviour before integrating
the full BT logic. Adapt topic names, message types, and payloads to your own programs and use cases.
#### 1) Low-level effector sanity check (LEDs)
```bash
ros2 topic pub --once /effectors/right_eye_leds \
nao_lola_command_msgs/msg/RightEyeLeds \
"{colors: [{r: 0.0, g: 1.0, b: 0.0, a: 1.0}, ...]}"
```
This allowed verifying that the (virtual) hardware responded correctly before introducing action logic
and higher-level profiles.
#### 2) High-level HRI commands (namespace `/hri`)
Trigger an exercise evaluation:
```bash
ros2 topic pub --once /hri/eval_cmd std_msgs/msg/String \
"{data: '{\"id\":\"rightarmUp\"}'}"
```
Execute a concrete pose for a given duration:
```bash
ros2 topic pub -1 /hri/nao_pos_cmd std_msgs/msg/String \
"{data: '{\"file\":\"rightarmUp\",\"duration\":4.0}'}"
```
Request speech output:
```bash
ros2 topic pub -1 /hri/speak_cmd std_msgs/msg/String \
"{data: 'Hola, soy NAO'}"
```
To observe results and debug the dataflow, echo the relevant outputs/topics used by your nodes:
```bash
ros2 topic echo <topic_name>
```
Useful debug tools:
- `rqt_graph` (visualize connections)
- `ros2 topic hz <topic>` (rate)
- `ros2 topic echo --once <topic>` (single sample)
> Replication tip: if you are unsure which interface belongs to which node, use:
> `ros2 node info <node_name>` and read the “Services / Actions / Subscriptions / Publications” sections.
---
## Configuration
- `config/exercises_catalog.yml` defines the available exercises and scenario parameters.
Update this file to add or tune exercises without modifying the Python scripts.
- `config/opencv_templates/` stores templates/resources used by the vision/evaluation node.
If you need to locate package share/config paths at runtime, prefer ROS 2 conventions
(e.g., `ament_index_python` in Python nodes) over hard-coded relative paths.
---
## Troubleshooting
- **Nothing happens when calling an interface:**
Verify the target node is running and the interface name/type matches (`ros2 node info`, `ros2 service type`).
- **BT runner blocks waiting for a result:**
The corresponding bridge node is not running, is using different names/namespaces, or returned an error.
- **Vision node fails at startup:**
Check that MediaPipe/OpenCV are installed and that templates/config paths exist and are readable.
- **Audio/TTS issues:**
Confirm that `simple_hri` is running and its service is available, then validate Google Cloud credentials
and TTS configuration, and finally re-run `speak_bridge.py` while inspecting logs.
---
## License
See the repository license file (or the `package.xml` metadata, if applicable).
