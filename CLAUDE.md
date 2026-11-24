# Baskethead

## What this is

A native iOS app that helps a player improve shooting, ball handling and defense using the phone camera and on-device machine learning. An AWS backend adds long-term analytics and coaching.

It's a personal project meant to show strong engineering on a software engineering resume, so prioritize clean architecture, tests, and getting it into real users' hands through TestFlight.

The full plan, decisions and open items are in `PLAN.md` (its final location in the repo is still open). If something isn't decided here or in `PLAN.md`, ask; don't assume.

## Current milestone (MVP)

- Film a player shooting from one spot, with the phone on a tripod behind the shooter.
- Detect the ball and hoop with a custom Core ML model.
- Log makes and misses accurately, including airballs, bank shots and rim rattles.
- Save the session (results and full video) on the device.
- The MVP includes the Mac labeling tool needed to train the model.

Build nothing else until this works reliably.

## Roadmap after the MVP

1. Shot zones: track makes and misses from multiple spots on the floor.
2. Shooting form feedback from body pose: elbow angle, release height, follow-through, stutters, one-motion vs. two-motion shot, arms vs. legs. Judge each check and suggest drills.
3. Sync session results to the backend; trend analysis; drill recommendations based on weak points (rules first, ML later).
4. Coaches and teams: coach role in the same app, team invites, drill assignments with notes, push notifications, completion tracking.
5. Ball-handling drills: dribble counting and move classification, with moves from a developer-maintained database. No Apple Watch.
6. Defensive reaction drills: on-screen and sound direction cues, time to reach a spot, stance check.
7. Compare the user's zone percentages with benchmark stats the user provides later. No NBA data.

## Stack

### iOS app (`ios/`)

- Swift (Swift 6 language mode) and SwiftUI; minimum iOS 17
- AVFoundation for camera capture
- Vision for body pose detection and for running Core ML models
- Core ML custom object detection model (ball and hoop), trained in Create ML
- SwiftData for local storage; videos stored as files and referenced from sessions
- Swift Testing for unit tests; no UI test target (the user tests screens by hand)
- Apple's swift-format
- Target: iPhone. Camera features need a physical device; the simulator has no camera.

### Labeling tool

- A macOS target (`LabelTool`) in the same Xcode project
- v1: draw and edit boxes and save Create ML `annotation.json`; pull frames from videos; copy boxes forward
- v2: import corrected shots exported from the app

### Backend (`backend/`): not needed until roadmap step 3

- Python on AWS Lambda behind **Lambda function URLs** (not API Gateway), so it stays on always-free services
- DynamoDB with provisioned capacity
- Amazon Cognito: Sign in with Apple, plus optional email and password
- pandas for session aggregation and trend analysis
- AWS CDK (Python) for all infrastructure
- pytest for tests
- Region us-west-2
- Push notifications sent from Lambda directly to Apple's push service
- AWS Free plan account, created at step 3. Budget: free tier only.

### ML (`ml/`)

- Training notes, labeling guidelines, dataset sources and licenses, model versions and metrics
- Large datasets and raw video stay on the Mac's internal drive, never in git

## Repo layout

```
/
├── CLAUDE.md
├── PLAN.md               (final location open)
├── .github/workflows/    CI: build and unit tests
├── ios/                  Xcode project (created in Xcode, not generated):
│                         iOS app + LabelTool macOS target
├── backend/              Lambda functions and the CDK app
└── ml/                   Training notes and model versions
```

## Conventions

- Keep the camera and vision pipeline separate from the UI. App folders: `Camera/`, `Detection/`, `Features/`, `Models/`, `Persistence/`.
- MVVM: one `@Observable` view model per screen. The camera, detector and storage sit behind protocols.
- Test make/miss logic with Swift Testing against JSON detection fixtures (per-frame ball and hoop boxes, plus the labeled outcome of each shot) committed to git. Source clips stay on the Mac.
- Verify changes compile with `xcodebuild` from the command line. CI runs the build and unit tests on GitHub Actions (macOS runner).
- The Xcode project uses folder-synced groups, so new files in `ios/` folders are picked up automatically. Avoid hand-editing `project.pbxproj`.
- Don't add third-party dependencies without asking first. Approved so far: pytest (backend).
- Format Swift code with swift-format.
- Git: one feature branch and pull request per feature, small commits, Conventional Commits (`feat:`, `fix:`, `test:` …). Claude commits, pushes and opens the PR, and merges only after the user OKs it in chat.

## Working with me

- I've built a full-stack web app before, but I've never worked with Swift. Briefly explain iOS and Swift decisions in chat and in code comments.
- Claude writes the code; I review every PR.
- Check in with me before starting each feature, and before changing the milestone, the stack or the roadmap.
- Don't assume anything I haven't said. Ask instead.
