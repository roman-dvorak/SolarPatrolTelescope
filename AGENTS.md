# AGENTS.md

## What this is
ROS1 (Kinetic, **Python 2**) catkin package `solar_patrol_telescope`: control nodes + web config for the solar patrol telescope at the Astronomical Institute of CAS. Built on the AROM framework (`arom`, `arom-web_ui` repos) and pymlab/MLAB hardware. Deploy target is an Odroid C2 at the observatory.

Keep code Python 2 compatible — the runtime is ROS Kinetic's rospy. Do not "modernize" (e.g. `print` statements) as a side effect.

## Build / run
- No tests, no lint, no CI. Build is plain catkin inside a workspace, e.g.:
  `cd ~/arom_ws && catkin_make`
- Launch with `roslaunch solar_patrol_telescope patrola.launch` (needs `arom` and `rosbridge_server` packages).
- `src/scripts/start_*.sh` hardcode obs-deployment paths (`/home/odroid/arom_ws/...`); they document the production layout, don't treat them as portable.

## Repo-specific gotchas
- Nodes are **extensionless executables** in `src/`: `mount`, `telescope_ctrl`, `telescope_caps`, `io_keyboard`. The launch file references them via `type=...` — never rename or add a `.py` suffix.
- Each ROS node pairs with a handlebars web-UI template loaded as `__file__ + '.feature.hbs'` → `src/<name>.feature.hbs`. When changing a node's published features/UI, update its sibling `.feature.hbs` (and the SVG assets in `src/`) too.
- `config/*.json` are two kinds: pymlab hardware-tree configs for `pymlab_bridge` (e.g. `patrola.json`, `demo.json`) and **live state files** (`mount.json` — contains positions with unix timestamps rewritten at runtime). Exercise care with state files.
- Messages: add new `.msg` files to `add_message_files()` in `CMakeLists.txt` (uses `message_generation`, ROS 1 style).
- I2C/SPI mapping (from README): 0x28 RA motor, 0x2C DEC motor, 0x2D TimeMachine, 0x2E focuser (CLARK+ZEISS).
- `patrola.launch` instantiates the same `telescope_ctrl` node twice (`telescope_1`, `telescope_2`; `telescope_3` commented out). Nodes load per-instance config **by ROS node name** from the hardcoded deploy path `/home/odroid/config/<name>.json` (e.g. `mount`, `telescope_1`). The repo's `config/*.json` files are the source templates for that deployed directory; `mount.json` also stores live positions rewritten at runtime.

## Layout notes
- `cad/` – OpenSCAD/STL for the MLAB control-box enclosure; separate from the ROS build.
- `doc/` – Czech user manual sources (`.odt`, DIA schema, SVG panel art).
- README is the only user-facing doc; keep I2C address list in sync if hardware mapping changes.
