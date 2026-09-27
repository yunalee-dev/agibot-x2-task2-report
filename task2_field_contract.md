# Task 2 Field Contract

Schema: task2-phase-i-final-v1

## final_trajectory.csv

- sample_index: monotonically increasing row index across explicit repeats
- time_s: global kinematic playback time in seconds
- repeat_index: zero-based repeat identity; dataset splits must keep an episode/repeat intact
- episode_time_s: time within the repeat
- phase, segment, segment_progress: semantic motion state and interpolation progress
- fourteen named arm-joint columns: model-derived qpos values in radians

## final_distance_records.csv

- sample identity/time fields match the trajectory
- minimum_joint_limit_margin_rad, joint_limit_violation: model-limit checks
- left_arm_right_arm_min_distance_m: raw group minimum, not a thresholded feature
- left_arm_other_robot_min_distance_m, right_arm_other_robot_min_distance_m: provisional scope minima
- global_min fields: raw closest scope, geom IDs, and body names
- penetration: true only when the raw signed global minimum is below zero
- hand xyz fields: mesh-derived distal point positions used by Task 2 validation

## Downstream boundary

- Pairwise/minimum distances are reusable Silver Robotics-like evidence.
- SAFE/NEAR/COLLISION and any normalized score are experimental derived features and must remain versioned separately.
- The current collision geometry, pair set, and positive clearance threshold remain TBD-005/006/009.
- This contract contains no camera frames and is not yet a final ACT Gold dataset.
