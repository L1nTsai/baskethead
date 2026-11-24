# Baskethead — Project Plan

_App name Baskethead, chosen November 24, 2026. Planned November 23–24, 2026._

Items marked **Open** still need your answer. Nothing in this plan was assumed; every decision below is one you made or approved.

---

## 1. Goal

A native iOS app that helps a player improve shooting, ball handling and defense using the phone camera and on-device machine learning. An AWS backend adds long-term analytics and coaching.

It's a personal project for a software engineering resume, so the priorities are clean architecture, tests, and real use through TestFlight.

---

## 2. Current milestone: the MVP

Film a player shooting from one spot, detect the ball and the hoop, log makes and misses accurately, and save the session on the device. Nothing else gets built until this works reliably.

The Mac labeling tool (section 5) is part of the MVP. The model has to detect the hoop from the start, and it can't be trained without labeled images.

### 2.1 What the MVP does

| Area | Decision |
|---|---|
| Phone setup | Tripod (fixed position) |
| Camera angle | Behind the shooter. Directly behind vs. behind and 30–45° off to one side gets tested with real clips before choosing (Phase 1). |
| When detection runs | Live, during recording |
| After the session | A stats summary screen (no second detection pass over the video) |
| Saved per session | Results and the full video, on the device only |
| Video retention | Kept until the user deletes it |
| Shooting spot | The user taps the spot on a court diagram (prepares for shot zones) |
| Finding the hoop | The model detects it from the start (no tap-to-mark) |
| Make/miss signal | Always shown on screen, plus a sound effect or a spoken result, which the user picks in Settings |
| Corrections | After the session, from the summary or playback screen |
| Corrected shots | Marked for export, to reuse as training or test data |
| Screens | Live camera with count, session summary, session history, video playback with each shot marked |

### 2.2 What it has to handle

- **Shots that must be called correctly:** airballs, bank shots, rim rattles.
- **Tip-ins and putbacks:** deferred, to decide later.
- **Places:** indoor gym, outdoor court, driveway or home hoop, at night under lights.
- **Who's in the frame:** the shooter, possibly a rebounder, and other players nearby. Other balls in play weren't selected, so they're out of scope.

### 2.3 Accuracy and "done"

- **Accuracy target:** **Open.** First we measure a baseline on hand-labeled clips, then you set the bar.
- **The MVP is done when** the accuracy target is met on a held-out set of labeled clips covering the places above, **and** a real session works end to end on your iPhone from a TestFlight build.

---

## 3. MVP build order

Claude checks in with you before starting each feature. Each feature goes on its own branch and pull request, and Claude merges only after you OK it in chat.

### Phase 0: Setup

1. **You create the Xcode project in `ios/`.** Xcode creates it; we don't generate it. Proposed options, **to confirm**: iOS App template, SwiftUI interface, Swift, unit tests with Swift Testing, no UI test target, and no SwiftData sample storage (we add SwiftData ourselves so there's no template model to delete).
2. Set the minimum iOS version to 17 and turn on Swift 6 language mode.
3. Set up the repo: git, a public GitHub repo, a `.gitignore` for Xcode build output and ML data, a swift-format config, and a GitHub Actions workflow that builds and runs the unit tests with `xcodebuild` on a macOS runner.
4. Add a macOS target for the labeling tool to the same Xcode project (proposed name `LabelTool`).

### Phase 1: Footage and the labeling tool

1. Film first sessions from both candidate angles, in as many of the target places as you can. Store them on your Mac, outside the repo.
2. Build LabelTool v1 (features in section 5), with unit tests for the annotation file it writes.
3. Write labeling guidelines in `ml/`, for example how to box a partly hidden or motion-blurred ball.
4. Find public datasets, and check and record each one's license in `ml/` before using it.
5. Choose the camera angle based on the test clips.

> **Why label consecutive frames:** boxes drawn on frames in a row do double duty. They're training data for the model, and they're also ground truth for testing the make/miss logic before the model exists.

### Phase 2: Model v1

1. Train the ball-and-hoop object detector in Create ML on the public datasets plus your labeled frames.
2. Record the version, training data, settings and metrics in `ml/`.

### Phase 3: Make/miss logic (no camera, no UI)

1. Define protocols for:
   - a **frame source** (camera or a video file),
   - an **object detector** (the Core ML model, or a fake that replays fixtures),
   - a **shot tracker** (pure logic that turns detections over time into shot events).
2. **Fixtures:** small JSON files in git holding the ball and hoop boxes for each frame, plus the hand-labeled outcome of each shot. The source clips stay on your Mac.
3. Write make/miss rules that handle airballs, bank shots and rim rattles.
4. Write unit tests with Swift Testing that run on the fixtures. CI runs them on every push.
5. Build an accuracy harness: on the Mac, run the real model and the logic over all labeled clips and report accuracy for each place. This gives the baseline, and then you set the target.

### Phase 4: Live session

1. Capture camera frames with AVFoundation and send them to the model through Vision, recording video at the same time.
2. Show the live count on screen, and play a sound effect or spoken result for each shot, depending on the setting.
3. Add the court diagram for picking the spot before a session.
4. Measure frame rate, battery use and heat on your iPhone.

### Phase 5: Storage and screens

1. Store sessions and shots with SwiftData. Videos are saved as files in the app's storage and referenced from their session.
2. Build the session summary, history, and video playback with shot markers.
3. Add after-session corrections and export of corrected shots.
4. Let the user delete a session together with its video.

### Phase 6: Accuracy gate and TestFlight

1. Enroll in the Apple Developer Program if you haven't yet. TestFlight requires it.
2. Create the Baskethead record in App Store Connect. App Store Connect confirms the name is available, and the bundle ID can't be changed after the first build is uploaded.
3. Improve the model and the logic until the accuracy target is met.
4. Ship an internal TestFlight build, tested by you.

---

## 4. App architecture

- **MVVM with protocols.** Each screen has a view model, an `@Observable` class that holds the screen's state and logic, and the SwiftUI view just draws it. This is similar to a React component paired with a custom hook.
- **Testability.** The camera, detector and storage sit behind protocols, so view models and make/miss logic are tested with fakes and JSON fixtures instead of a live camera.
- **Folders inside the app:** `Camera/`, `Detection/`, `Features/`, `Models/`, `Persistence/`.
- **Dependencies:** no third-party dependencies are planned in the app. Ask before adding any.

---

## 5. ML and data

| Topic | Decision |
|---|---|
| Model | Custom Core ML object detector for two classes, `ball` and `hoop`, trained in Create ML |
| Training data | Public datasets (license checked first) plus your own footage |
| Data you have today | None yet |
| Where data lives | Your Mac's internal drive, never in git |
| Labeling tool | Our own Mac app: a macOS target in the same Xcode project |
| LabelTool v1 | Draw and edit boxes and save them in Create ML's `annotation.json` format; pull frames out of session videos; copy boxes forward to the next frame |
| LabelTool v2 | Import the corrected shots exported from the app |
| `ml/` folder | Training notes, labeling guidelines, dataset sources and licenses, model versions and metrics |

---

## 6. Engineering conventions

| Topic | Decision |
|---|---|
| Mac | Apple silicon |
| Minimum iOS | 17 (the lowest version SwiftData supports) |
| Swift | Swift 6 language mode, so the compiler catches threading bugs |
| Tests | Swift Testing for logic. No UI test target; you test the screens by hand. |
| Test data | Detection fixtures (JSON) in git; clips stay on the Mac |
| Formatting | Apple's swift-format |
| CI | GitHub Actions on a macOS runner: build and run unit tests on every push |
| Local build check | `xcodebuild` from the command line |
| Repo | Public on GitHub from the start |
| Branching | One feature branch per feature, merged through a pull request |
| Commit messages | Conventional Commits (`feat:`, `fix:`, `test:` …), kept small |
| Git roles | Claude commits, pushes and opens PRs, and merges only after your OK in chat |
| Who writes code | Claude writes; you review every PR |
| Explanations | Short notes in chat and comments in the code |
| Check-ins | Before starting each feature |
| Deadline | None |
| TestFlight testers | Just you at first |

---

## 7. Roadmap after the MVP (revised)

1. **Shot zones.** Track makes and misses from several spots, using the court diagram from the MVP.
2. **Shooting form feedback from body pose.**
   - Checks: elbow angle, release height, follow-through, stutters or hitches, one-motion vs. two-motion shot, and whether power comes mostly from the arms or the legs.
   - The app judges each check (good or needs work) and suggests drills.
   - What counts as good: your default rules, which coaches can adjust per player.
   - Camera angle: test behind the shooter vs. the side, then choose.
3. **Backend sync, trends and drill recommendations.**
   - Sign-in, and syncing session results (videos stay on the phone).
   - Trend analysis with pandas.
   - Recommendations target each player's weak points: rules first, ML later.
   - Drills come from your built-in drill library.
4. **Coaches and teams (new).**
   - Coaches use the same iOS app with a coach role, and each account has exactly one role.
   - There are teams only, no separate clubs. A player can be on several teams, and a team can have several coaches.
   - Coaches invite players with a join code, or in the app by username or email.
   - The player can leave a team, and the coach can remove a player.
   - Coaches see stats only (no videos), for all of a player's sessions, including ones from before they joined.
   - Coaches assign drills from the built-in library to one player or a whole team, with an optional note.
   - The drill shows in the player's drill list, the player gets a push notification, and the coach can see when it's done.
5. **Ball handling.**
   - Count dribbles and classify moves.
   - Moves come from a database that only you, the developer, maintain. Each move has a name, a description, and labeled example clips the classifier is trained on.
   - No Apple Watch support.
6. **Defensive reaction drills.**
   - Direction cues on screen with sound.
   - Reaction measured as time to reach a spot (for example, a cone).
   - Stance check.
7. **Zone comparison against benchmark stats you provide later.** This replaces the NBA comparison; see section 10 for why.

---

## 8. Backend (from roadmap step 3)

| Topic | Decision |
|---|---|
| Language | Python on AWS Lambda |
| API layer | Lambda function URLs instead of API Gateway, to stay within AWS's always-free services. Sign-in tokens are checked in code. |
| Database | DynamoDB, using provisioned capacity, because the always-free allowance covers provisioned mode, not on-demand |
| Sign-in | Amazon Cognito: Sign in with Apple, plus optional email and password |
| What syncs | Session results only; videos stay on the phone |
| Analytics | pandas |
| Infrastructure | AWS CDK (Python) |
| Tests | pytest (approved third-party dependency) |
| Region | us-west-2 (Oregon) |
| Push notifications | Lambda calls Apple's push service directly |
| AWS account | Free plan, created when step 3 starts |
| Budget | Free tier only |

**Watch out:** the AWS Free plan closes the account after 6 months or when the credits run out, whichever comes first. You then have 90 days to upgrade to a paid plan before AWS deletes the account. Decide what to do before that point (section 9).

---

## 9. Open items

**Needed soon:**
1. Your iPhone model and its iOS version
2. Your Xcode and macOS versions
3. Bundle ID (the app name is decided: Baskethead)
4. Your GitHub username and the repo name
5. ~~Where this plan lives in the repo~~ Decided: `docs/PLAN.md`
6. The Xcode project options proposed in Phase 0

**Decided by testing:**

7. Camera angle for the MVP (Phase 1)
8. Accuracy target (after the Phase 3 baseline)
9. Where the trained model file lives if it's too big for git (Phase 2)

**Later:**

10. Python version for Lambda (roadmap step 3)
11. AWS Free plan expiry: upgrade to a paid plan or not (6 months after the account is created)
12. That the Free plan allows every service we need (check when step 3 starts)
13. Username rules, such as whether usernames are unique and can be changed (roadmap step 4)
14. Where to store Apple's push notification signing key at no cost (roadmap step 4)
15. Tip-ins and putbacks
16. Camera angle for form feedback (roadmap step 2)
17. The benchmark stats you'll provide (roadmap step 7)

---

## 10. Research notes

- **AWS Free plan.** Accounts opened since July 15, 2025 choose a Free plan or a Paid plan. The Free plan ends after 6 months or when the credits are used up, and Free plan accounts only get the always-free offers. ([AWS Billing docs](https://docs.aws.amazon.com/en_us/awsaccountbilling/latest/aboutv2/free-tier.html), [AWS Builder Center](https://builder.aws.com/content/3Kkt4kNiDSUnO3n6db2weSNubzE/the-dollar0-aws-account-in-2026-free-plan-credits-guardrails))
- **Always free.** Lambda includes 1M requests a month; DynamoDB includes 25 GB of storage. ([AWS Builder Center](https://builder.aws.com/content/3BlrxG9mxIb0diPdJF4q15isWJX/whats-next-on-aws-free-tier)) DynamoDB's free capacity applies to provisioned mode, not on-demand. ([DEV Community guide](https://dev.to/kanywst/aws-free-hands-on-build-s3-lambda-dynamodb-on-the-always-free-tier-plus-iam-cloudwatch-and-34le))
- **API Gateway isn't always free.** Its free calls last 12 months, for new customers only. ([API Gateway pricing](https://aws.amazon.com/api-gateway/pricing/))
- **Cognito.** The Lite and Essentials plans include 10,000 monthly active users for free, with no end date. ([Cognito pricing](https://aws.amazon.com/cognito/pricing/))
- **Name check.** Searches found no app or brand called Baskethead. Two things to know: in American English, "basket" on its own usually means a shopping basket, and the name is close to "basket case" (there's a streetwear brand called Basketcase). These names were rejected because they're taken or risky: HomeCoach (too close to HomeCourt), BallMe, BasketCoach, Ball99, MVPCoach, HoopCoach, Hoopify, Swishlab, Arcwise. Runners-up that also looked open: Hoophead, RimRead, Hoopwise, Road to 99.
- **Why no NBA stats.** NBA.com's terms forbid reusing its materials for public or commercial purposes, and its stats FAQ says the stats aren't available for download, even for personal use. nba_api pulls from NBA.com, so the same terms apply. ([NBA.com Terms of Use](https://nba.com/termsofuse), [NBA Stats FAQ](https://www.nba.com/stats/help/faq), [nba_api](https://github.com/swar/nba_api))
