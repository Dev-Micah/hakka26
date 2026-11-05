# Repository guidance

## What this repository does

This repository runs a mobile OTP-screen hackathon. `templates/` contains starter apps for Android Compose, Compose Multiplatform, Flutter, bare React Native, Expo, and SwiftUI. A team copies one starter into `submissions/<team>/app`; the submission also needs `PROMPT.md`, `TEAM.md`, `screenshot.png`, and a demo. `README.md` and `RULES.md` define submission rules and scoring; `grader/rubric.md` is the grading source of truth.

The Build workflow validates changed paths and required files, detects the app stack with `scripts/detect_stack.sh`, then builds and tests the submitted app. The Grade workflow runs only repository-controlled grader code from `main`, revalidates the submission through GitHub's API, and publishes scores to the leaderboard. Keep this trust boundary intact: Build executes untrusted PR code and has no secrets; Grade must not check out or execute PR code.

`submissions/` and `dry-run/` contain example or test submissions. Avoid treating them as shared app code or changing them while working on repository infrastructure.

## Build, test, and lint

Run mobile-app commands from the relevant app root (`templates/<stack>` or `submissions/<team>/app`). The Build workflow is authoritative for CI and has a 15-minute job limit.

| Stack | Build | Tests | One test |
|---|---|---|---|
| Android Compose | `./gradlew assembleDebug` | `./gradlew testDebugUnitTest` | `./gradlew :app:testDebugUnitTest --tests 'ke.droidcon.hakka.AppInfoTest'` |
| Compose Multiplatform | `./gradlew assembleDebug` | `./gradlew testDebugUnitTest` | `./gradlew :composeApp:testDebugUnitTest --tests '<fully.qualified.TestClass>'` |
| Flutter | `flutter build apk --debug` | `flutter test` | `flutter test test/widget_test.dart --plain-name 'shows the app name'` |
| React Native / Expo | `npm ci`, then `npx expo prebuild --platform android --no-install` if `android/` is absent, then `(cd android && ./gradlew assembleDebug)` | `npx jest --ci` | `npx jest __tests__/App.test.tsx --runInBand` |
| SwiftUI | `xcodebuild build -scheme App -destination 'platform=iOS Simulator,name=<available iPhone>' CODE_SIGNING_ALLOWED=NO` | `xcodebuild test -scheme App -destination 'platform=iOS Simulator,name=<available iPhone>' CODE_SIGNING_ALLOWED=NO` | `xcodebuild test -scheme App -destination 'platform=iOS Simulator,name=<available iPhone>' -only-testing:AppTests/AppInfoTests/testAppNameIsHakkahakka26 CODE_SIGNING_ALLOWED=NO` |

For Android and multiplatform builds, CI uses Java 21. CI runs iOS jobs on macOS and requires an available iPhone simulator. React Native requires Node 22 in CI. The bare React Native template defines `npm run lint` (ESLint); the repository has no root-level lint command and CI does not run a lint job.

To validate submission metadata locally, provide a file containing one changed path per line:

```sh
python -I scripts/validate_submission.py --changed-files <path-to-changed-files>
```

## Repository-specific conventions

- CI infers the stack from files in the app root: `pubspec.yaml` means Flutter; a `package.json` with `react-native` means React Native/Expo; an Xcode project or `Package.swift` without Gradle means iOS; Kotlin Multiplatform Gradle configuration means KMP; other Gradle apps are Android. Keep the starter's recognizable build files and wrapper intact.
- Submission PRs must change files only under one `submissions/<team-name>/` folder. Names are lowercase `a-z`, digits, and hyphens, 3–30 characters; required files and the three-person limit are enforced by `scripts/validate_submission.py`.
- Keep shared visual assets (`colours.json` and Thamani logos) consistent with `assets/`; each starter includes copies in its app-specific asset locations.
- The Expo starter has scoped guidance in `templates/expo/AGENTS.md`. When changing that starter, check its installed Expo major version and matching Expo documentation before using Expo APIs; do not hand-edit generated `ios/` or `android/` directories.
- Grading code reads submission source, screenshots, and build logs as untrusted input. Preserve the rubric's boundaries and do not let participant-controlled data become grader instructions.
