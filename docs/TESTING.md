# GimbalBar — local build and hardware validation

This repository is a public MIT-licensed fork of https://github.com/amirmxt/gimbalbar. Development preserves upstream credits and history.

## Hardware and tooling

- Apple Silicon Mac, macOS 14 or later
- Xcode Command Line Tools
- DJI Osmo Pocket 3 via USB-C, with **Webcam** mode selected
- Camera access permission granted to the app

## Test the 1.1 candidate without replacing the installed 1.0

```bash
cd ~/Projects/gimbalbar
git fetch origin
git switch --track origin/feat/v1.1-presets-smooth-motion
./build.sh && open ./GimbalBar.app
```

If the development branch is already on disk, switch with `git switch feat/v1.1-presets-smooth-motion` and run `git pull --ff-only` before building.

**Keep the existing /Applications/GimbalBar.app unchanged** until tests are complete. The script removes and recreates only the project's own `build/` and `GimbalBar.app/`, then ad-hoc signs the local build. It does not install the application system-wide. Do not consume GitHub Actions minutes for this local hardware validation.

## GimbalBar 1.1 — test matrix

1. **Baseline:** Open and confirm live preview, connected indicator, normal drag pad, arrow nudge and Home.
2. **Save:** With the Pocket 3 connected and still, set a low-risk framing and select **Save** for P1. Move to a second framing and save P2. Optionally save P3. The coordinates shown beside each saved position should update.
3. **Recall:** Move away from P1 and select **Go** for P1. The camera should move through successive small steps toward the saved position rather than receiving a single far target. Test P2.
4. **Motion style:** Select Gentle, Normal, and Quick. Verify that the software step size changes. This is *not* hardware motor-speed control and cannot guarantee a precise physical duration.
5. **Cancel:** While returning to a distant saved position, press **Cancel move**. Future software steps should stop. An already-sent USB command may continue until completed by the camera. This is not an emergency motor stop.
6. **Manual override:** Start a preset recall, then drag the pad or use an arrow. The preset trajectory should cancel immediately and the manual movement should take precedence.
7. **Persistence:** Quit and relaunch the development app. P1/P2/P3 and selected motion style should persist.
8. **Disconnect:** During a move, unplug USB. Automatic trajectory generation should stop. After reconnecting in Webcam mode, the preview/control should return and saved positions remain.
9. **No camera:** With USB unplugged, no preset recall should start, nor should Save accept a new camera position.
10. **Regression:** Check invert pan, saved center, recenter, and global shortcuts Control-Option + arrows / C.

For physical safety, use **Gentle** first, keep the gimbal clear of obstacles, and avoid unattended operation. The device's physical motion may lag software commands. Stop testing if it behaves unpredictably.

## SwiftUI build troubleshooting

Earlier MacBook tests reported `SwiftUIMacros.StateMacro` missing from the Command Line Tools toolchain. The code already uses `@StoredState` as an alias for `SwiftUI.State`, avoiding direct macro expansion for the drag property. If compilation fails again, capture:

```bash
xcode-select -p
xcrun --find swiftc
swiftc --version
```

A successful build explicitly prints `Built .../GimbalBar.app`. If it does not, do not run the incomplete app. Related precedent: https://github.com/drumih/turbo-fieldfare/issues/185

## Validation status

- 1.0 reliability fixes: **compiled and tested successfully** on the user's Apple Silicon MacBook on October 21, 2026. USB webcam operation, installation, and unplug/replug recovery confirmed.
- 1.1 candidate: **source changes committed, no Mac compilation or physical test yet**. Keep the 1.1 PR unmerged until the test matrix passes.
- CI / GitHub Actions: **not run**.

## Release gate

Only after build and physical tests pass: merge the 1.1 PR, sync local `main`, rebuild, quit existing GimbalBar and install the tested build into `/Applications`. Update the app version after validation and before release. Document failures and local toolchain versions in the PR.
