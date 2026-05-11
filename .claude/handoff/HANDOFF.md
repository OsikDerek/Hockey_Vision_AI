# Hockey_Vision_AI — Handoff context

If you're a Claude Code session landing in this repo, read this first.

## What this repo is

**Body-mechanics half** of Derek Osik's larger hockey AI project.
Single-skater computer-vision pipeline: pose estimation → joint angles
→ YAML-defined technique rubric → annotated video + coaching report
with drill recommendations.

## Origin

Spun out of [OsikDerek/Hockey_AI](https://github.com/OsikDerek/Hockey_AI)
on 2026-05-11. The decision-simulator half (3D viewer, Quiz Mode,
game-film tactical analysis, VR-sim roadmap) stayed in that repo.

The initial commit here (`8887f95`) preserved a clean copy of the body
mechanics code. Earlier git history is in the Hockey_AI repo for
anyone who wants to see the original development timeline.

## What ships

- `main.py` — CLI entry point. `--technique <name>` runs analysis
  against `knowledge_base/techniques/<name>.yaml`. `--mode general` or
  `--mode crossover` runs the legacy mechanics engine.
- 10 technique rubrics (`knowledge_base/techniques/*.yaml`):
  forward_stride, crossover, wrist_shot, snap_shot, one_timer,
  stickhandling, hockey_stop, backwards_skating, transitions, edge_work
- Pose backends: MediaPipe (default) or YOLOv8-pose
- Auto-crop preprocessing (YOLO person detect → crop + upscale to
  fixed height) for distant / wide-angle clips
- Kalman landmark smoothing
- Optional puck tracking via `src/object_tracker.py` for stickhandling
- Coaching report writers (text + JSON)

## Status / known issues

- Functional end-to-end on practice / drill footage at the time of
  the split.
- Detection quality depends heavily on camera angle and skater size
  in frame — use `--auto-crop` on distant clips.
- Tests under `tests/` cover smoothing and the mechanics engine
  specifically; not a full integration test suite.

## Companion repo

Decision-simulator side: [OsikDerek/Hockey_AI](https://github.com/OsikDerek/Hockey_AI).
That repo has its own HANDOFF.md / NEXT.md / memory layer for the
viewer + Quiz Mode + simulator work.

## Author

Derek Osik — professional hockey player, skating coach, software
engineer. Personal-development tool first. Be willing to skip skating
/ hockey fundamentals in coaching feedback you write into YAMLs or
reports.
