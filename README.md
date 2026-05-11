# Hockey Vision AI

Single-skater computer-vision pipeline that turns drill / practice /
shooting footage into a biomechanics-graded coaching report and an
annotated video. Built by a professional hockey player and skating
coach.

> Spun out of [OsikDerek/Hockey_AI](https://github.com/OsikDerek/Hockey_AI)
> on 2026-05-11 — that repo retained the game-analysis + decision-simulator
> half of the original project. **This repo is the body-mechanics half.**

## What it does

Drop in a video of yourself (or a skater you're coaching) doing one
skill. The system:

1. Runs pose estimation (MediaPipe or YOLO-pose) to find joint landmarks.
2. Computes joint angles + spatial metrics (knee bend, hip hinge, hand
   position, head pitch, blade engagement, etc.).
3. Evaluates each frame against a YAML-defined technique rubric.
4. Emits an annotated video + a coaching report with specific drill
   recommendations from the drill library.

## Supported techniques

10 skills, each with detection rules + thresholds + coaching cues +
drill recommendations defined in YAML under
`knowledge_base/techniques/`:

| Technique | Key checks |
|-----------|-----------|
| `forward_stride` | Knee bend · hip hinge · forward lean · ankle dorsiflexion · stride symmetry |
| `crossover` | Knee drive · internal rotation · step-out explosiveness |
| `wrist_shot` | Knee bend · hip rotation · weight transfer · hands out front · follow-through |
| `snap_shot` | Knee bend · hip rotation · hands from body |
| `one_timer` | Knee bend · hip rotation · hand separation |
| `stickhandling` | Athletic stance · head/eyes up · hand spacing · puck control zone |
| `hockey_stop` | Knee bend · hip position · edge engagement |
| `backwards_skating` | Knee bend · hip position · posture |
| `transitions` | Knee bend · hip position · balance |
| `edge_work` | Knee bend · ankle engagement · balance |

## Quick start

```bash
# 1. Install
pip install -r requirements.txt

# 2. (Optional) for stickhandling puck tracking, download the puck model
#    into models/puck_yolov8n.pt — set in the technique YAML's
#    detection.params.puck_model field.

# 3. Run technique analysis
python main.py -i drill.mp4 --technique forward_stride --auto-crop
python main.py -i shot.mp4  --technique wrist_shot   --auto-crop
python main.py -i edge.mp4  --technique crossover    --auto-crop --mode crossover
```

`--auto-crop` runs YOLO person detection on the first frame to find the
skater, then crops + upscales every subsequent frame to the skater's
bounding box. Use it for distant / wide-angle clips where the skater
is small in frame.

## Output

Every run produces (next to `output/<name>_analyzed.mp4`):

- `<name>_analyzed.mp4` — annotated video with skeleton, angle labels,
  HUD feedback panel, and drill recommendations
- `<name>_report.txt` — human-readable coaching report
- `<name>_report.json` — structured data for further analysis

## Pose backends

| Backend | Landmarks | Best for | Speed |
|---------|-----------|----------|-------|
| `mediapipe` (default) | 33 (incl. feet) | Single skater, CPU | ~5 fps |
| `yolo` | 17 (COCO) | Multi-person, GPU | ~15 fps (GPU) |

Switch via `--backend yolo`.

## Project structure

```
Hockey_Vision_AI/
├── main.py                          # CLI entry point
├── requirements.txt
├── config/
│   ├── skating_mechanics.yaml       # Angle thresholds (coach-tunable)
│   └── drill_library.yaml           # Drill recommendations
├── knowledge_base/
│   └── techniques/                  # 10 YAML technique rubrics
│       ├── forward_stride.yaml
│       ├── crossover.yaml
│       ├── wrist_shot.yaml
│       └── ...
├── src/
│   ├── pose_estimator.py            # MediaPipe + YOLOv8-pose backends
│   ├── angle_calculator.py          # Joint angle geometry + spatial metrics
│   ├── smoothing.py                 # Kalman landmark smoothing
│   ├── technique_engine.py          # YAML-driven technique evaluation
│   ├── mechanics_engine.py          # Legacy general-mechanics engine
│   ├── stride_detector.py           # Stride event detection
│   ├── crossover_analyzer.py        # Legacy crossover detector
│   ├── object_tracker.py            # Puck detection (for stickhandling)
│   ├── annotator.py                 # Video overlay rendering
│   ├── video_preprocessing.py       # Auto-crop to skater ROI
│   ├── report_generator.py          # Text + JSON report writers
│   ├── video_io.py                  # OpenCV wrappers
│   ├── utils.py                     # Small helpers
│   ├── detectors/                   # Per-frame + temporal detector registry
│   ├── pose_estimation/             # Pose backend internals
│   ├── mechanics_analysis/          # Stride symmetry, knee-drive scoring
│   ├── video_processing/            # Frame-pipeline utilities
│   └── visualization/               # HUD widgets
├── tests/
└── output/                          # Generated annotated videos + reports (not in git)
```

## Adding your own technique

Drop a YAML file in `knowledge_base/techniques/`:

```yaml
technique:
  name: my_technique
  detection:
    detector: FrameByFrame
  checks:
    knee_angle:
      angle_function: knee_angle
      good_range: [95, 125]
      feedback:
        good: "Great knee bend"
        poor: "Bend your knees more"
      drills:
        poor:
          - name: "Wall Sits"
            description: "Hold for 30 seconds against a wall"
            cue: "Tail down, head up"
```

The `detection.detector` field can be `FrameByFrame` (evaluate every
frame independently) or a temporal detector like
`CrossoverDetector` that emits discrete events.

## Requirements

- Python 3.10+ (3.12 recommended)
- OpenCV, MediaPipe, Ultralytics, NumPy, SciPy, PyYAML, lapx
- Optional: NVIDIA GPU for faster YOLO inference

## About

Built by Derek Osik — professional hockey player, skating coach, and
software engineer. This is the **biomechanics half** of a larger
hockey AI project; the **decision-simulator** half (3D viewer,
Quiz Mode, VR-sim roadmap, game-film tactical analysis) lives at
[OsikDerek/Hockey_AI](https://github.com/OsikDerek/Hockey_AI).

Both projects are personal-development tools first.
