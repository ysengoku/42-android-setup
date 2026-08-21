# 42 Android setup

Setup script for Android Studio on 42 Lyon cluster (Ubuntu).

## Context

- `~` is on a small local disk (check with `df -h ~`)
- `/goinfre` is large but machine-local and wiped periodically

So: Studio in `~/opt`, SDK and Gradle cache in `/goinfre`.

## Usage

1. Download the Linux `.tar.gz` from https://developer.android.com/studio
2. Put it in `/goinfre/$USER/downloads/`
3. Run `./setup-android.sh`
4. `source ~/.zshrc && studio.sh`

In the setup wizard, choose **Custom** and set the SDK location to `/goinfre/$USER/android/sdk`.

## After a goinfre wipe

Rerun the script. Studio itself survives in `~/opt`, but the SDK must be
re-downloaded from the SDK Manager (~10 min).

## Android Virtual Device (AVD)

Not created by the script. After a wipe:

```bash
avdmanager create avd -n pixel6_api37 \
  -k "system-images;android-37.0;google_apis;x86_64"
```

```bash
emulator -avd pixel6_api37 &
adb devices
```

## Workflow

At the start of each session, on any machine:

```bash
cd ~/Documents/42-android-setup && ./setup-android.sh
source ~/.zshrc
```

The script is idempotent: a few seconds if goinfre survived, ~10 minutes if it was wiped.

Then:
```bash
android --no-metrics emulator start medium_phone &
studio.sh
```

## Notes

- KVM access is granted via ACL (`getfacl /dev/kvm`), not group membership
- Projects live in `~/Documents`, always pushed to git
- `CLT_URL` may 404 when Google updates it - get the current one with:
  `curl -s https://developer.android.com/studio | grep -o 'commandlinetools-linux-[0-9]*_latest.zip' | head -1`
- Package versions (`android-37.0`, `36.0.0`) are hardcoded in the script
- `sdkmanager` is deprecated; the script uses the newer `android sdk` CLI

- `android emulator create` ignores `$ANDROID_AVD_HOME` and writes to
  `~/.android/avd`, so the script symlinks that path to goinfre
- it also picks its own system image (API 36 playstore as of 2026-08);
  the version can't be pinned
- the installed CLI takes the profile as a positional arg, not `--profile=`
  (the online docs say otherwise)
  