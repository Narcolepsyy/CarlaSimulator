CARLA Simulator (Binary Relese 0.8.4)
=====================================
[Demo](https://youtu.be/eKuQxochj_k) 
-------------
This repository contains the Linux binary distribution of [CARLA](https://carla.org/), an open-source autonomous driving simulator built on Unreal Engine 4.18. It includes the precompiled simulator (`CarlaUE4`), the complete Python API (`PythonClient`), final project materials that rely on CARLA, and tooling for running the simulator locally or from a Docker container.

Highlights
----------
- Prebuilt UE4 city maps (Town01, RaceTrack, Course4, etc.) with vehicles, pedestrians, traffic lights, and weather profiles ready to stream sensor data.
- Python APIs for synchronizing with the simulator, driving agents, benchmarking, lidar and camera sensors, and logging utilities such as `live_plotter`.
- Instructor materials for Coursera/Udacity-style course projects (`Course1FinalProject`, `Course4FinalProject`), including waypoint-following and behavioral planning stacks.
- Headless/server workflows through `CarlaUE4.sh`, configurable `CarlaSettings.ini`, automated tests, and an NVIDIA-enabled Docker recipe.

Repository layout
-----------------
| Path | Description |
| --- | --- |
| `CarlaUE4/` | Unreal project, assets, saved configs (`Config/CarlaSettings.ini`) and runtime logs under `Saved/Logs/`. |
| `CarlaUE4.sh` | Launcher script that makes `CarlaUE4/Binaries/Linux/CarlaUE4` executable and forwards CLI flags to the simulator. |
| `Engine/` | UE4 runtime libraries that ship with the binary release. |
| `PythonClient/` | Python API package (`carla` module), sample clients, plotting utilities, Course 1 & 4 capstone code, and test suites. |
| `PythonClient/test/` | Console harnesses, acceptance tests, and regression suites for stressing the simulator. |
| `Dockerfile` | NVIDIA OpenGL–based container recipe for headless rendering. |
| `CHANGELOG` | History up to release 0.8.4 (Tesla Model 3, ROS bridge, lidar improvements, etc.). |
| `requirements.txt` | Minimal Python dependency set for the bundled clients. |
| `LICENSE` | MIT license governing the contents of this repository. |

Prerequisites
-------------
### Hardware and OS
- 64-bit Linux (Ubuntu 16.04 or newer) with an NVIDIA GPU and drivers >= 390.
- ≥16 GB RAM and ≥20 GB free disk space for maps, assets, and logs.
- Display server or the ability to run SDL in offscreen mode (`SDL_VIDEODRIVER=offscreen`) for headless setups.

### Python environment
Use Python ≥3.8 (the repo currently runs against Python 3.12) and install the client dependencies inside a virtual environment.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
# Install the API package and examples in editable mode so `import carla` works everywhere.
pip install -e PythonClient
```

Quick start
-----------
### 1. Launch the simulator
The binary ships with compiled maps, so running the server is simply:

```bash
chmod +x CarlaUE4.sh
./CarlaUE4.sh -carla-server -quality-level=Epic -fps=30 -benchmark -world-port=2000 -windowed
# To load a specific map: ./CarlaUE4.sh /Game/Maps/RaceTrack -carla-server ...
```

Key command-line switches you can pass via `CarlaUE4.sh`:
- `-carla-server` – enables the networking layer (defaults to port 2000).
- `-world-port=<port>` – changes the RPC port so multiple instances can run in parallel.
- `-quality-level={Low,Epic}` – controls rendering fidelity and performance.
- `-fps=<n>` and `-benchmark` – set fixed simulation rate (useful for deterministic testing).
- `-windowed`, `-ResX`, `-ResY`, `-opengl` – configure display resolution or force OpenGL.
- Map path (`/Game/Maps/Town01`, `/Game/Maps/Course4`, etc.) – choose the environment at launch.

Simulator behavior (weather presets, agent counts, sensors) can also be configured by editing `CarlaUE4/Config/CarlaSettings.ini`, or by passing a custom file via the Python clients’ `--carla-settings` flag.

### 2. Drive interactively or run sample clients
With the server running, open another shell in the repo and activate your Python environment.

- **Manual driving demo**
  ```bash
  python3 PythonClient/manual_control.py --quality-level Epic --autopilot
  ```
  Arrow keys / WASD control the ego car, `P` toggles autopilot, `R` restarts the episode, and `--lidar` adds a 32-channel lidar sensor.

- **Simple autopilot client**
  ```bash
  python3 PythonClient/client_example.py --host 127.0.0.1 --port 2000 --autopilot
  ```
  Shows the bare-bones API for spawning sensors, retrieving measurements, and handing over control to CARLA’s built-in autopilot.

- **Driving benchmark**
  ```bash
  python3 PythonClient/driving_benchmark_example.py --config PythonClient/driving_benchmark_config.ini
  ```
  Executes repeatable benchmark routes with pause/resume support and logs metrics under `PythonClient/_out/`.

- **Waypoint follower / Course 1 project**
  ```bash
  python3 PythonClient/Course1FinalProject/module_7.py --quality-level Epic --carla-settings CarlaUE4/Config/CarlaSettings.ini
  ```
  Replays the classical longitudinal/lateral controller pipeline, records plots under `Course1FinalProject/controller_output/`, and uses `live_plotter` for on-screen telemetry.

- **Behavioral planning stack / Course 4 project**
  ```bash
  python3 PythonClient/Course4FinalProject/module_7.py --quality-level Epic
  ```
  Or run the individual notebooks (`local_planner.ipynb`, `velocity_planner.ipynb`, etc.) to experiment with each planning component before executing the full stack.

- **Utilities**
  - `PythonClient/view_start_positions.py` lists spawn transforms for the current map (useful for deterministic tests).
  - `PythonClient/point_cloud_example.py --save-ply` converts depth images into colored point clouds.
  - `PythonClient/live_plotter.py` embeds reusable matplotlib widgets used by the course projects; import it to create dynamic XY plots even on headless servers (falls back to Agg if Tk is unavailable).

Docker workflow
---------------
The `Dockerfile` builds a headless CARLA server on top of `nvidia/opengl:1.0-glvnd-runtime-ubuntu16.04`. This is useful for CI or remote GPU machines.

```bash
docker build -t carla-simulator:0.8.4 .
docker run -it --rm \
  --gpus all \
  -p 2000-2002:2000-2002 \
  -e SDL_VIDEODRIVER=offscreen \
  carla-simulator:0.8.4 \
  ./CarlaUE4.sh -carla-server -world-port=2000 -quality-level=Low
```

Make sure the host has the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html). Clients can then connect from the host or other machines by pointing to the exposed port(s).

Configuration tips
------------------
- **`CarlaSettings.ini`** (`CarlaUE4/Config/`): tweak default weather, traffic density, sensor blueprints, and synchronous mode. Python clients accept a `--carla-settings` argument to load alternative files.
- **Course configs**: `PythonClient/Course1FinalProject/options.cfg` and `Course4FinalProject/options.cfg` hold controller/planner gains and scenario toggles; editing these files is the recommended way to tune the projects.
- **Logging**: simulator logs live under `CarlaUE4/Saved/Logs/CarlaUE4.log`. Python clients emit CSVs/PNGs inside their respective `controller_output` folders.
- **Headless plotting**: `live_plotter` auto-detects Tkinter; on minimal systems install `python3-tk` or rely on the built-in Agg fallback.

Testing and validation
----------------------
- **Client stress test**  
  ```bash
  python3 PythonClient/test/test_client.py --synchronous --images-to-disk
  ```
  Randomizes spawn points, toggles autopilot/manual control, and validates the sensor APIs (images saved under `PythonClient/test/_out/`).

- **Repeatability tests** (`PythonClient/test/test_repeatability.py`) and acceptance suites in `PythonClient/test/acceptance_tests` help detect determinism regressions when running scripted agents.

Resources
---------
- Official documentation: <http://carla.readthedocs.io>
- Forum and Q&A: <https://github.com/carla-simulator/carla/discussions>
- Release notes: see the bundled `CHANGELOG` for features introduced in 0.7–0.8.4.

License
-------
CARLA and the Python client code in this repository are distributed under the MIT License (see `LICENSE`). Please keep the notice intact when redistributing or embedding parts of this release.
