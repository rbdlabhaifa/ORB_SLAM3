# Demos & usage notes

This fork builds **`libORB_SLAM3.so` only** — there is no `Examples/` directory in the repository (removed in commit `3b5d324`). Demo executables from upstream ORB-SLAM3 are **not** included.

Use this repo as a **shared library** consumed by lab projects (e.g. [RBD-SLAM](https://github.com/rbdlabhaifa/RBD-SLAM)) or your own C++ driver.

---

## Demo 1 — Build the library

```bash
git clone https://github.com/rbdlabhaifa/ORB_SLAM3.git
cd ORB_SLAM3
chmod +x build.sh
./build.sh
```

**Outputs:**

| Artifact | Path |
|---|---|
| Shared library | `lib/libORB_SLAM3.so` |
| Vocabulary (after untar) | `Vocabulary/ORBvoc.txt` |
| Thirdparty libs | `Thirdparty/DBoW2/lib/`, `Thirdparty/g2o/lib/` |

Minimal driver invocation pattern:

```cpp
#include <System.h>

ORB_SLAM3::System SLAM(
    "Vocabulary/ORBvoc.txt",
    "config/your_settings.yaml",
    ORB_SLAM3::System::MONOCULAR,
    "" /* atlas load path */,
    true /* viewer */
);

cv::Mat pose = SLAM.TrackMonocular(frame, timestamp).matrix();
```

Copy and edit `config/settings.monocular.example.yaml` for pinhole monocular setups.

---

## Demo 2 — Pangolin viewer: save destination

When the viewer is enabled, the Pangolin menu includes **SaveDestination**. It calls `Tracking::SaveDestination()` and appends world-frame points to:

- `drone_destinations.txt` (CWD)
- `System::destinations` vector

Useful for marking waypoints during interactive SLAM sessions.

---

## Demo 3 — Map drawer overlays (RRT / exit)

RBD extensions on `MapDrawer`:

```cpp
auto *drawer = SLAM.get_map_drawer();
drawer->set_draw_RRT(true);
drawer->set_draw_exit(true);
drawer->RRT_path = { /* Eigen::Vector3f waypoints */ };
drawer->Exit_point = Eigen::Vector3f(x, y, z);
```

Overlays render in the Pangolin 3D view when flags are set.

---

## Demo 4 — Atlas save / load

**Via YAML** (`System.SaveAtlasToFile` / `System.LoadAtlasFromFile` in settings file).

**Programmatic:**

```cpp
SLAM.SaveAtlasAsOsaWithTimestamp("/path/to/atlas_dump");
// or toggle map merging:
SLAM.ChangeMapMerging(false);
```

Atlas files use the `.osa` extension internally.

---

## Demo 5 — Trajectory evaluation (EuRoC ground truth)

Bundled scripts under `evaluation/`:

```bash
pip3 install -r requirements.txt

python3 evaluation/associate.py groundtruth.txt CameraTrajectory.txt > associated.txt
python3 evaluation/evaluate_ate_scale.py groundtruth.txt CameraTrajectory.txt
```

Ground-truth transforms for EuRoC left camera are in `evaluation/Ground_truth/`.

Requires a trajectory file produced by your consumer application — this repo does not ship a dataset runner.

---

## Demo 6 — Visualize point cloud (`plot_keyframes.py`)

Lab script for Open3D inspection of exported PCD files:

```bash
pip3 install -r requirements.txt
# Edit my_pcd3 at top of plot_keyframes.py to match your .pcd basename
python3 plot_keyframes.py
```

Expects `my_pcd3.pcd` in the working directory (customize the `file` variable in the script).

---

## Restoring upstream Examples (optional)

If you need RealSense / EuRoC demo binaries, copy the `Examples/` tree from [UZ-SLAMLab/ORB_SLAM3](https://github.com/UZ-SLAMLab/ORB_SLAM3) and extend `CMakeLists.txt` to add the example targets. This fork intentionally omits them to keep the repo library-focused.
