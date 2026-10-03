├── README.md                          # Complete project & interview documentation
├── pubspec.yaml                       # Flutter project configuration & CV dependencies
├── test/
│   └── toe_tap_detector_test.dart     # Unit test suite for State Machine & Hysteresis
│
├── lib/
│   ├── main.dart                      # Orientation lock, system chrome & bootstrap
│   ├── app.dart                       # MaterialApp root & M3 dark theme configuration
│   │
│   ├── camera/
│   │   ├── camera_service.dart        # Camera lifecycle, YUV420 stream, permission gating
│   │   └── camera_frame_throttler.dart# Mutex dropping frames while inference is in flight
│   │
│   ├── ml/
│   │   ├── yolo_pose_service.dart     # Model loader, forward pass, tensor parser & CV test generator
│   │   ├── coordinate_translator.dart # Translates normalized model space [0..1] to screen aspect
│   │   └── models/
│   │       ├── bounding_box.dart      # Normalized AABB [L, T, R, B], center & estimated radius
│   │       ├── pose_keypoint.dart     # COCO 17 keypoint indices (Ankles 15/16) & confidence
│   │       └── detection.dart         # Detection wrapper (Person, Football, keypoint lists)
│   │
│   ├── detection/
│   │   ├── football_detector.dart     # Ball candidate selector & temporal holding buffer
│   │   ├── foot_tracker.dart          # Ankle filter & shoe sole / toe contact projection
│   │   ├── tap_state.dart             # State enum (IDLE, APPROACHING, CONTACT, RELEASED) & TapMetrics
│   │   └── toe_tap_detector.dart      # Core FSM with dual-threshold hysteresis & cooldown
│   │
│   ├── screens/
│   │   └── home_screen.dart           # Coordinator connecting Camera, Inference, and Overlays
│   │
│   ├── widgets/
│   │   ├── camera_preview.dart        # Aspect-fitted CameraPreview wrapper
│   │   ├── detection_overlay.dart     # CustomPainter for boxes, skeleton lines & contact pulse
│   │   └── tap_counter.dart           # HUD card showing tap count, state badge, FPS, and latency
│   │
│   └── utils/
│       ├── geometry.dart              # Euclidean distance, circle surface distance, IoU
│       └── fps_counter.dart           # Rolling-window FPS and inference latency calculator
│
└── app/src/main/                      # Native Android integration & live preview app
    ├── AndroidManifest.xml            # Camera permissions & hardware feature flags
    └── java/com/example/MainActivity.kt # Interactive CV simulator runnable in the live browser preview


Project Overview:
Dual-Threshold Hysteresis State Machine (lib/detection/toe_tap_detector.dart):
: Foot touches the ball boundary 
 transition to CONTACT increments counter exactly once.
Debouncing: Remaining on the ball across subsequent frames keeps the state in CONTACT without triggering duplicate counts.
 (
): Foot must pull away past the larger boundary to transition into RELEASED, eliminating jitter and frame-to-frame noise.
In-Flight Mutex Throttler (lib/camera/camera_frame_throttler.dart):
Prevents camera frames (30 FPS) from queuing up in memory while the mobile ML model runs (~15–20 FPS). Dropped frames are immediately discarded without lag.
Temporal Occlusion Buffer (lib/detection/football_detector.dart):
Holds the last known ball center for up to 4 consecutive frames when the foot partially covers the football, preventing state drops during contact.
100% Deterministic Testing (test/toe_tap_detector_test.dart):
The state machine and geometry utilities are decoupled from device hardware, allowing instant local testing without an attached camera or weights file.
Live Interactive Preview Ready:
The live emulator environment has been compiled with compile_applet and displays an interactive real-time CV visualizer and state machine simulation right in your streaming browser view!
