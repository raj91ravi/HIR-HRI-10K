# HIR-HRI-10K: Synthetic Multimodal Human Intention Recognition Dataset

## Overview
- 10,000 simulated observations, 1,000 independent episodes, 10 consecutive frames per episode.
- 12 approximately balanced intention classes (830–840 observations/class).
- Frame interval: 0.2 seconds; episode duration between first and last frame: 1.8 seconds.
- 36 CSV columns including IDs, split, labels, simulated robot-relative kinematics, pose, gaze, hand/object interaction.
- Split by episode (and synthetic subject): 7,000 train, 1,500 validation, 1,500 test rows.
- Fixed generation seed: 20261010.

## Files
- hir_multimodal_all.csv: full dataset, with split column
- hir_multimodal_train.csv: training split
- hir_multimodal_validation.csv: validation split
- hir_multimodal_test.csv: held-out test split
- label_map.csv: label definitions and class frequencies

## Label taxonomy
0 walking; 1 approaching_robot; 2 avoiding_robot; 3 requesting_assistance;
4 object_handover; 5 object_pickup; 6 following_robot; 7 stopping;
8 greeting_robot; 9 pointing; 10 sitting_down; 11 unknown_intention.

## Columns and units
Identifiers: sample_id, episode_id, subject_id, split, frame_index, timestamp_s,
intention_id, intention_label.
Robot-relative position and motion:
- human_x_m: simulated lateral offset in metres (NOT world-coordinate X)
- distance_to_robot_m: simulated radial distance in metres
- walking_speed_mps, robot_speed_mps: metres per second
- heading_rad, body_orientation_rad, head_orientation_rad: radians
Behavior scores (0 to 1, higher means stronger): gaze_to_robot_score,
hand_raise_score, arm_extension_score, object_proximity_score, seatedness_score.
- torso_bend_rad: radians
- human_height_m: synthetic subject stature in metres
Pose coordinates: nose_x/y, left_shoulder_x/y, right_shoulder_x/y,
left_wrist_x/y, right_wrist_x/y, left_hip_x/y, right_hip_x/y.
All pose coordinates are normalized 0–1 in a *synthetic person-centered
image frame*, not measured from a camera. Left/right refer to anatomical side
for the simulated subject. No RGB images, depth frames, real skeletons,
or measured gaze vectors are included.

## Generation design and temporal consistency
Each episode has a constant intention label and virtual subject ID. Its
10 frames share subject height, initial position, gait scaling, and heading
baseline. Distance evolves incrementally, not independently per frame.
Class-conditioned tendencies include: approaching decreases robot distance;
avoiding increases distance and lateral offset; following approximately
maintains distance while human and robot speeds are correlated; stopping
decelerates; handover progressively extends the arm toward an object;
pointing and assistance involve hand raising; sitting lowers the body;
pickup increases torso bending. Noise and overlap are introduced so that
features are not perfectly separated by label.

## Modeling protocol
For frame-based classification, exclude all ID, split, and target columns.
For sequence models, group by episode_id, sort by frame_index, and use
the 10x28 numeric feature tensor. Do not use sample_id, episode_id,
subject_id, split, intention_id or intention_label as inputs.
Fit normalization/scalers on train only. Evaluate on test only after
hyperparameter selection on validation.
Because labels are constant across an episode, an episode-level metric
is more informative than treating adjacent frames as independent examples.
Consider macro-F1, per-class recall, confusion matrix, and temporal
early-intention accuracy at frames 1, 3, 5, 10.

## Limitations and research integrity
THIS IS FULLY SYNTHETIC DATA, NOT COLLECTED FROM HUMAN SUBJECTS OR ROBOTS.
Patterns are handcrafted, not learned from real human motion, and the
label is assigned by the simulator. Scores are proxies, not validated
computer-vision detections. The 'unknown' class is a simulated mixed
behavior distribution, not a rigorous out-of-distribution benchmark.
Models may exploit synthetic class signatures and perform unrealistically
well. No real-world performance or generalization claim should be made
without external, human-annotated experimental validation.
This dataset is suitable for prototyping, teaching, pipeline debugging,
and controlled synthetic baseline experiments.
