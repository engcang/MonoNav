# MonoNav with AirStack

RGB → ZoeDepth → TSDF → motion primitives. Based on
[natesimon/MonoNav](https://github.com/natesimon/MonoNav).
The AirStack worker adds the HTTP interface and flight-control integration;
network weights are not retrained.

## Installation

Use Linux, Docker, NVIDIA Container Toolkit and the AirStack Isaac/PX4 setup.
Keep `AirStack/` and `MonoNav/` beside each other.

```bash
git clone --recursive --branch eungchang/adv-ws2 git@github.com:castacks/AirStack.git
git clone --recursive git@github.com:engcang/MonoNav.git
cd MonoNav
./docker/build_image.sh
```

Follow `AirStack/tools/ws2_bench/README.md` to prepare the simulator/robot
containers and bridge. ZoeDepth downloads its pretrained weights on first use;
the bench retains them in Docker volume `mononav-torch-cache`. Do not commit
that cache or generated experiment data.

## Run

From `AirStack/`, open the viewer in one terminal and run a scenario in another:

```bash
source .venv-ws2/bin/activate
python3 tools/ws2_bench/dashboard.py --port 8892
python3 tools/ws2_bench/episode.py tools/ws2_bench/scenarios/corridor_mononav.yaml \
  --no-bag --camera follow --record-video
```

View `http://127.0.0.1:8892`. The bench handles takeoff, worker startup,
execution, evaluation and landing. MP4 recording needs `ffmpeg` or the
`imageio-ffmpeg` Python package. Omit `--no-bag` to also record a ROS bag.
Only one executing planner may use the bridge at a time.

For the common maps and recorded clean/attack suite:

```bash
python3 tools/ws2_bench/corridor_suite.py --targets mononav --tiers easy \
  --output robot/ros_ws/ws2_runtime/campaigns/mononav_recorded
```

This runs clean checks before sensor attacks. A completed result in the same
output directory is reused on resume, not rerun. Change the output directory
when changing settings. The viewer's Start tests form runs Office campaigns;
corridor suites start through the CLI above.

## Configuration

- `AirStack/tools/ws2_bench/scenarios/corridor_mononav.yaml`: target settings,
  including `velocity`, `zoe_depth_scale`, `minimum_planning_clearance`,
  `tsdf_local_radius`, altitude and timeout.
- `AirStack/tools/ws2_bench/scenarios/corridor_suite.yaml`: shared corridor
  scale/width, lighting, sensor noise/delay and observer camera. The suite
  replaces each target YAML's scene with this common map.
- `config.yml`: underlying ZoeDepth/MonoNav parameters and primitive library.
- Standalone worker options: `python mononav_airstack.py --help` inside the
  `mononav-demo:1.0` image. `--headless --execute` is the unattended mode;
  `--wait-for-start` is only for the interactive GUI.

The corridor bench supplies explicit corner task goals; MonoNav computes its
own avoidance path. Kim/PULP have different task interfaces and receive no
route goals. `--depth-source ground-truth` is a privileged diagnostic, not
normal RGB operation. Rui's FCRN patch is not supported on ZoeDepth.
Actual flight evidence is in AirStack `tools/ws2_bench/CORRIDOR_VALIDATION.md`.
