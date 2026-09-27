# Task 2 — Alternating X-Cross

Easy-to-read final HTML report: `task2_final_report.html`

Current scope: phases A–I are implemented and numerically verified under a provisional MuJoCo collision-analysis profile. The final animation remains available for user visual sign-off.

## Status

- Phase A: **PASS** — model and 37 referenced meshes load; 7 scalar joints per arm; geometry and coordinate evidence recorded
- Phase B: **PASS** — common `qpos0` reference, 14-arm-only mutation guard, provisional pair sets, and signed-distance semantics verified
- Phase C: **PASS / USER ACCEPTED** — max wrist-origin error 0.003671 m, no provisional-scope penetration
- Phase D: **PASS / USER ACCEPTED** — both mesh-derived hand points cross the centerline; left distal link is 0.062427 m above at the closest points
- Phase E: **PASS / USER ACCEPTED** — both hands cross the centerline; right distal link is 0.061796 m above at the closest points
- Phase F: **PASS / USER ACCEPTED** — 1,701 samples across two repeats; no joint-limit violation or penetration
- Phase G1: **PASS / USER ACCEPTED AS PROVISIONAL G2 INPUT**
- Phase G2 / original Task G: **PASS_TASK_LOCAL_PROVISIONAL_POLICY**
- Phase H: **PASS** — 9 parameter combinations, 3 full-path safe candidates, selected d140_v220
- Phase I: **PASS_KINEMATIC_FINAL / VISUAL REVIEW AVAILABLE** — 5 repeats and 4,251 checked samples
- Tests: **31/31 passed**
- Source model: `agibot_x2_urdf/X2_URDF-v1.3.0/x2_ultra.xml` (never modified by these tools)
- Audit configuration: `configs/task2/phase_ab.json` (`PROVISIONAL`; TBD-005/TBD-006 remain open)
- Reference state: loaded `model.qpos0`, zero `qvel`; only the configured 14 arm qpos entries may change
- Reference interpreter: `C:\Users\lyn04\anaconda3\python.exe`
- Verified environment: Python 3.13.5, NumPy 2.1.3, MuJoCo 3.14.0

## Run

From the repository root in PowerShell:

```powershell
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.audit_phase_ab
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.run_phase_c
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.run_phase_d
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.run_phase_e
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.run_phase_f
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.run_phase_g1
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.run_phase_g2
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.run_phase_h
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.run_phase_i
& 'C:\Users\lyn04\anaconda3\python.exe' -m unittest tests.test_task2_phase_ab -v
& 'C:\Users\lyn04\anaconda3\python.exe' -m unittest discover -s tests -v
```

The audit command regenerates only Task 2 phase A/B derived artifacts in this directory. It does not edit the official MJCF, URDF, or mesh assets.

## Outputs

- `environment_report.md`: evidence, pass/fail status, geometry caveats, coordinates, and Phase C readiness
- `joint_info.csv`: every loaded joint, model-derived addresses/types/limits, and Task 2 membership
- `arm_geom_info.csv`: arm geom ID ↔ body ↔ mesh trace, masks, visual/collision role
- `mesh_references.csv`: only paths actually referenced by the selected MJCF
- `distance_api_probe.csv`: installed API sign/cutoff/filter experiment
- `baseline_distances.csv`: baseline pair distances for provisional Task 2 scopes
- `baseline_validation.json`: machine-readable fixed-state, geom-set, coordinate, and minimum-distance summary
- `phase_c_report.md`: OPEN target, achieved wrist positions, pass/fail evidence, and interpretation limits
- `open_pose.csv`: OPEN pose 14-joint values and model-derived limit margins
- `open_validation.json`: machine-readable IK attempts and static validation
- `open_distances.csv`: all provisional pair distances at OPEN
- `open_front.png`, `open_side.png`: user visual-approval renders
- `phase_d_report.md`: LEFT_OVER_RIGHT targets, closest-link over/under evidence, and static validation
- `left_over_right_pose.csv`: LEFT_OVER_RIGHT 14-joint values and limit margins
- `left_over_right_validation.json`: mesh-derived hand points, IK attempts, and closest-point relation
- `left_over_right_distances.csv`: all provisional pair distances at LEFT_OVER_RIGHT
- `left_over_right_front.png`, `left_over_right_side.png`: Phase D visual-approval renders
- `phase_e_report.md`: RIGHT_OVER_LEFT targets, closest-link over/under evidence, and static validation
- `right_over_left_pose.csv`: RIGHT_OVER_LEFT 14-joint values and limit margins
- `right_over_left_validation.json`: mesh-derived hand points, IK attempts, and closest-point relation
- `right_over_left_distances.csv`: all provisional pair distances at RIGHT_OVER_LEFT
- `right_over_left_front.png`, `right_over_left_side.png`: Phase E visual-approval renders

## Phase C implementation dependencies

The existing SciPy 1.15.3 installation is used for bounded nonlinear least-squares IK; a custom damped least-squares implementation was the alternative. Existing Pillow 11.1.0 is used only to save and label PNG renders. No dependency was installed or added to the system.

## Interpretation boundary

The current arm collision geoms mostly reuse the detailed visual STL meshes, and wrist-yaw collision geoms are absent/commented. MuJoCo's mesh collision representation is therefore useful for model-relative screening, but these numbers are not certified physical clearances or exact raw STL triangle distances. Any later geometry/pair-set change requires Silver Robotics reprocessing under `docs/07_AGENT_RULES.md`.

Before making safety-grade clearance claims, the recommended follow-up is a separate, versioned analysis MJCF containing arm-link capsule/compound-convex proxies and the missing wrist-yaw surfaces, with proxy error checked against the source meshes. The official v1.3.0 model should remain unchanged.

## Completed gate history

Phase H selected d140_v220 and Phase I completed the final repeated kinematic validation.
## Phase F — continuous trajectory

- Status: `PASS / USER ACCEPTED`
- Sequence: `OPEN → LEFT_OVER_RIGHT → OPEN → RIGHT_OVER_LEFT → OPEN`
- Sampling: 100 Hz; 1.5 s minimum-jerk transitions; 0.5 s holds; 2 repeats
- Validation: 1,701/1,701 samples within joint limits and without penetration
- Animation: `task2_episode.gif` (640×480, 25 fps, one 8.5 s episode)

Generate and validate:

```powershell
python -m scripts.task2.run_phase_f
```

Interactive MuJoCo playback:

```powershell
python -m scripts.task2.play_trajectory --repeats 2 --loop
```

Phase F outputs: `trajectory.csv`, `trajectory_validation.csv`,
`trajectory_summary.json`, `phase_f_report.md`, and `task2_episode.gif`.
The user accepted the animation before Phase G.

## Phase G1 — provisional collision calibration

- Status: `PROVISIONAL_CALIBRATION_COMPLETE_VISUAL_REVIEW_REQUIRED`
- 1,111 deterministic calibration poses: 171 accepted-trajectory samples and 940 synthetic probes
- 89 penetrating probes retained as negative/evaluation evidence, never ACT action targets
- 478 raw geometry pairs evaluated; 53 pairs entered the provisional under-100-mm review shortlist
- TBD-006 (pair set) and TBD-009 (threshold) remain open pending review

Run:

```powershell
python -m scripts.task2.run_phase_g1
```

Phase G1 outputs: `g1_pairwise_distances.csv.gz`, `g1_sample_minima.csv`,
`g1_pair_statistics.csv`, `g1_pair_review_tiers.csv`,
`g1_threshold_scenarios.csv`, `g1_calibration_summary.json`, and
`phase_g1_report.md`.

## Phase G2 — separated states, recovery, and full-path Task G analysis

- Working profile: COLLISION if any provisional-scope distance is below 0 m; otherwise NEAR if arm↔arm is below 0.020 m; otherwise SAFE
- Labeled probes: 977 SAFE, 45 NEAR, 89 COLLISION
- 89 COLLISION states are negative/evaluation-only and never ACT action targets
- 11 recovery episodes, 561 rows, and zero penetration
- Accepted Task 2 episode: 851 samples across nine contiguous segments
- Episode arm↔arm minimum: 0.073224888 m at 2.950 s
- Closest pair: `left_elbow_link ↔ right_wrist_roll_link`
- TBD-006 and TBD-009 remain project-wide OPEN

Outputs: `g2_labeled_states.csv`, `g2_negative_collision_states.csv`,
`g2_safe_demonstration.csv`, `g2_recovery_demonstrations.csv`,
`g2_recovery_manifest.csv`, `g2_task_g_timestep_analysis.csv`,
`g2_task_g_segment_summary.csv`, `g2_task_g_worst_pose.csv`,
`g2_arm_arm_distance.png`, `g2_worst_arm_arm_frame.png`,
`g2_dataset_summary.json`, and `phase_g2_report.md`.

Synthetic SAFE/NEAR probes are context/evaluation states, not demonstrations.
Only the accepted trajectory and fully revalidated recovery paths carry next-q
action targets.

## Phase H — parameter exploration

- Grid: cross depth 0.11/0.14/0.17 m × vertical separation 0.16/0.22/0.28 m
- Results: 3 SAFE_CANDIDATE, 2 CROSS_RELATION_FAIL, 4 IK_FAIL
- Selected: d140_v220, preserving the approved visual shape
- Selected 100 Hz arm↔arm minimum: 0.070935214 m
- Outputs: phase_h_sweep.csv, both heatmaps, selected pose files/renders, phase_h_selected.json, and phase_h_report.md

## Phase I — final demonstration

- Five explicit repeats at 100 Hz; 4,251 checked samples over 42.5 s
- Penetrations: 0; joint-limit violations: 0
- Arm↔arm minimum: 0.070935214 m in every repeat
- Global minimum: 0.005653412 m at left_shoulder_roll_link ↔ torso_link
- Outputs: final_trajectory.csv, final_distance_records.csv, final_episode_summary.csv, task2_final.gif, final_manifest.json, phase_i_report.md, and task2_field_contract.md

All requested phases are implemented. The remaining non-code gate is user visual review of task2_final.gif. Project-wide collision geometry, pair set, and positive threshold remain TBD-005/006/009.

