# AI Exercise Assistant
AI Exercise Assistant is a webcam-based computer vision project that analyzes exercise form in real time. The assistant uses MediaPipe Pose to extract body landmarks and OpenCV to compute joint angles, flag form errors, and overlay guidance with on-screen cues. It also includes configurable angle thresholds, repetition counting, and per-exercise rules for plank, lunges, and shoulder press.


## Features
- **Real-time pose tracking:** MediaPipe Pose and OpenCV video pipeline at webcam framerates (RGB ↔ BGR conversions, landmark drawing).
- **Angle-based form checks:** Compute joint angles from three landmark points and a threshold per exercise. (e.g., plank "straight line" logic)
- **Repetition counting & stages:** Simple state machine per movement (e.g., lunge down→up; shoulder press up→down).
- **On-screen cues:** Live text overlays for current STAGE, TIME/REPS, and numeric joint angles.


## Quickstart
1. Install required packages
  ```
  !pip install opencv-python
  !pip install mediapipe
  ```
2. Run the remaining cells in order
   a. **Imports & Pose setup** (cv2, mediapipe, numpy, time; create pose = mp_pose.Pose(...)).
   b. **Realtime Pose Estimation** block with the webcam check (press ```q``` to quit).
   c. **Exercises blocks:** run one at a time — Planks, Lunges, Shoulder Press. Execution is self-contained; therefore, just execute the cell for the exercise you want.


## Usage
- This repository is a single notebook/script with multiple sections.
- To use an exercise coach, run the section's cell (Planks, Lunges, or Shoulder Press).
- Press ```q``` in the video window to exit each section and move to the next.


## How it works
### Angles from landmarks
The code computes the angle at a joint using three points around it.
```python
def find_angle_body_parts(first, second, third):
    first = np.array(first)
    second = np.array(second)
    third = np.array(third)

    radians = np.arctan2(third[1]-second[1], third[0]-second[0]) \
              - np.arctan2(first[1]-second[1], first[0]-second[0])
    angle = np.abs(radians*180.0/np.pi)

    return 360 - angle if angle > 180.0 else angle
```


### Exercise rules
- **Plank:** two angles (shoulder–hip–knee and hip–knee–ankle) both in [160°, 200°] ⇒ "Good form"; otherwise "Correct your form." Includes a simple timer of "good form" seconds.
- **Lunge:** left/right knee angles within ~70–110° while feet maintain separation; stage down → up increments reps.
- **Shoulder Press:** checks elbow/shoulder extension; stage up ↔ down transitions count reps with elbow/shoulder angle thresholds.


### Configuration
There are no global config variables; thresholds live inline in each exercise block. To adjust their behaviors, change the numeric literals shown below inside the corresponding cell.

#### Visibility checks
``` python
# Example (Planks)
if landmarks[...].visibility < 0.5 or ...:
    stage = 'Show full body'
```
- Change ```0.5``` to adjust minimum landmark visibility.

#### Planks:
``` python
# Good form when both angles are within:
if shoulder_hip_knee > 160 and shoulder_hip_knee < 200 and \
   hip_knee_ankle   > 160 and hip_knee_ankle   < 200:
    stage = "Good form"
```
- Edit ```160``` and ```200``` to widen or narrow acceptable plank angles.

#### Lunges
``` python
# Down position
if left_angle > 70 and left_angle < 110 and \
   right_angle > 70 and right_angle < 110 and \
   (left_ankle[0]-right_ankle[0])**2 + (left_ankle[1]-right_ankle[1])**2 > 0.1:
    stage = "down"
# Up transition (counts a rep)
elif left_angle > 110 and right_angle > 110 and stage == 'down':
    stage = "up"
    counter += 1
```
- Edit ```70``` and ```110``` for knee-angle thresholds.
- Adjust ```0.1``` for ankle-separation requirement.

#### Shoulder Press
``` python
# Up position (full extension)
if left_elbow_angle > 150 and right_elbow_angle > 150 and \
   left_shoulder_angle > 150 and right_shoulder_angle > 150:
    stage = "up"
# Down transition (counts a rep)
elif left_shoulder_angle < 90 and right_shoulder_angle < 90 and stage == 'up':
    stage = "down"
    counter += 1
```
- Edit ```150``` (extension) and ```90``` (down threshold) to match the subjective movement criteria.


### Troubleshooting
- **Black/blank window:** Ensure the correct camera index (cv2.VideoCapture(0) → try 1, 2).
- **Laggy video:** Reduce window size, ensure CPU mode (no GPU needed), and close other camera apps.
- **Landmarks missing:** Step back so the head–ankle region is visible; increase lighting; raise min_detection_confidence.
- **Angles look wrong:** Remember landmarks are normalized coords—don't cast to ints before computing angles; only scale for drawing text.
