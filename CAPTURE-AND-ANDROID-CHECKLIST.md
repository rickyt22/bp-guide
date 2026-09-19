# BP guide — iOS capture spec + Android checklist

Source of truth for iOS: **1.0.50 (build 53), branch `fix/bp-auto-reconnect` @ `ecc6b79`**
in `rpm-ios-app`. That branch is commit `54adc13` plus three docs-only commits
(`0db67b5`, `ee7fe8f`, `ecc6b79`); the app code is byte-identical to `54adc13`.
Android: `22-rpm-android-app` `origin/main` (1.0.39). Both read-only; nothing was modified.

Draft page: `index.html` on branch `draft/bp-guide-ios-1.0.50` (this branch). It replaces
the live guide, so `git diff main -- index.html` shows exactly what would change. It uses
the live page's markup and classes, plus three new classes (`.platform`, `.platform-tag`,
`.platform-note`). Everything yellow or red is draft-only. **Do not merge this branch to
`main` as-is:** GitHub Pages publishes `main`, so a merge would put the draft boxes in
front of patients.

---

## 1. iOS screens to capture

### The existing six (for reference)
All six are iOS screenshots with the status bar cropped off, scaled to **600 px wide**:
`img1-login` 600×1170, `img2`–`img6` 600×1234. The page sets every `.shot` to the same
width (`flex: 0 1 10.5rem`, `img { width: 100% }`), so on the page **width is what
scales**. Height only affects how neatly paired images line up.

I **could not tie them to a device.** They were taken on an older build: the old Home
screen, and layout that no longer matches the 1.0.50 styles. So UI-scale measurements
against current code don't hold. **Recapture all of them in the same session** so every
image shares one scale, rather than mixing new shots into the old set. If the raw
originals or the phone they came from still exist, use that phone instead and adjust the
crop below.

Status of the six against 1.0.50:

| File | Status in 1.0.50 |
|---|---|
| img1-login | Still accurate. Recapture for consistency. |
| img2-otp | Screen still accurate, but the **alt text is wrong** ("six empty boxes"; the app has one field). Recapture. |
| img3-home | **Stale.** Old Home with "Quick Access"; 1.0.50 opens the new home screen. Replace. |
| img4-connect | Still reachable. Recapture. |
| img5-paired | **Stale.** The "Connected to BP2A ✓ / Close" pop-up no longer appears; the list closes itself on connect. Replace. |
| img6-result | Still exists, but a "Reading sent" pop-up now appears on top first. Recapture. |

### Device and output spec
- **A physical iPhone, not the Simulator.** The Simulator has no Bluetooth, so the
  permission prompt and every pairing screen can't be produced there.
- **iPhone 15 or 16 (6.1", 393×852 pt, screenshots 1179×2556 px).**
- Phone settings: Display Zoom **Standard**, Text Size **default** (middle notch), Bold
  Text off, Light appearance, a Focus mode on (no notification banners), battery above
  50%, Wi-Fi on.
- App: 1.0.50 from the **App Store** (not a dev build). Use a **test patient** enrolled
  for BP, with a neutral first name ("Hello ___" shows on Home) and no real PHI.
- Cuff: a BP2A, charged.
- Post-processing, the same for every image:
  1. Crop the **top 177 px** (59 pt, the full top safe area, so no status-bar remnants).
  2. Resize to **600 px wide**, giving **600×1211**.
  3. Export JPEG (quality about 82, sRGB, strip metadata).
  4. Use `width="600" height="1211"` on every `<img>`. That's within about 2% of the old
     1234 aspect, which makes no visible difference at the rendered size.
- **Filenames:** new or replaced shots use **new names** (`ios-*.jpg`). The Spanish page
  (`/bp-guide/es/`) loads the same `../assets/` files, so overwriting `img3` or `img5`
  would put new screenshots next to old Spanish text. Overwrite `img1`, `img2`, `img4` and
  `img6` only (same content, same text). Point the ES page at the new names when its text
  is translated.

### Capture order (a single run on a fresh install)
**Prep:** delete the app, then reinstall from the App Store. On first launch, if the login
screen shows "Looking for Face ID…", Face ID credentials survived in the Keychain (Keychain
items outlive an app delete). Tap **Use Password Instead**, then **Remove Saved Face ID**,
force-quit, and relaunch before starting. Deleting the app does clear the saved cuff and
resets the Bluetooth permission.

| # | Screen | How to reach it | File | Guide step |
|---|---|---|---|---|
| 1 | Bluetooth prompt, "“22 RPM” Would Like to Use Bluetooth" (over the login screen) | First launch after reinstall. Appears on its own. | `ios-bluetooth.jpg` (new) | 2, iPhone note |
| 2 | Login screen, empty fields | Tap Allow on #1 | `img1-login.jpg` (recapture) | 2 |
| 3 | "OTP Verification" pop-up, empty | Enter credentials, tap Login | `img2-otp.jpg` (recapture) | 2 |
| 4 | "Enable Face ID?" alert | Enter code, tap Verify | `ios-faceid.jpg` (new) | 2, iPhone note |
| 5 | Home: "Hello ___", reminder card, Blood Pressure card, bottom bar | Tap Enable | `ios-home.jpg` (new, replaces img3) | 3 |
| 6 | BP screen before connecting: "Disconnected", **Connect** button, no banner | **Cuff OFF.** Open Blood Pressure, wait about 15 s for the list to pop up, tap **Close**, and wait for the "Couldn't connect" message to fade. Capture. | `img4-connect.jpg` (recapture) | 4 |
| 7 | "Connect to BP Device" list with the BP2A row | Turn the cuff **on**, tap **Connect**. Capture once the BP2A row shows. | `ios-device-list.jpg` (new) | 4 |
| 8 | BP screen with the green "Connected. Press Start on the BP machine…" bar | Tap the BP2A row. The list closes itself. | `ios-connected.jpg` (new, replaces img5) | 4 |
| 9 | "Reading sent" pop-up | Press Start on the cuff, wait for the result | `ios-reading-sent.jpg` (new) | 6 |
| 10 | Result screen: FINAL RESULT plus history | Tap **Done** | `img6-result.jpg` (recapture) | 6 |

Optional (not used in the draft; only if the FAQ gets pictures): **11** the "Couldn't
connect to the cuff" message with the list open (wait about 15 s after the cuff powers
off); **12** "Reading saved" (airplane mode, take a reading). Turn airplane mode off
afterwards so the saved reading sends.

Timing notes, from the code (confirm while capturing):
- The "Connecting..." gray button (#6/#7): if the cuff is **on** when Blood Pressure
  opens and the app has never paired, the button grays out as "Connecting..." for about
  15 s, then the list opens by itself. That's why #6 is taken with the cuff off.
- The green bar (#8) stays until the next measurement starts. There's no rush.
- The "Reading sent" pop-up (#9) waits for **Done**. There's no rush.
- The app speaks ("Device connected", "Measurement started"). Turn the volume down if
  that's distracting during capture.

---

## 2. Android checklist: what must land before the Android section can be written

Written against `22-rpm-android-app` `origin/main` (1.0.39). **B1–B8 change what the
steps say.** Writing before they land means rewriting. B9–B10 are about writing and
publishing it.

- [ ] **B1. Launch crash fixed and merged.** `App.js:34` on `main` registers
  `component={Lowgin}` (undefined), so the app crashes before Login. The fix is
  `cd59317` on `feature/android-outbox`, unmerged. Needed before any Android screenshot.
- [ ] **B2. Identity and distribution final.** Play listing live (org account, D-U-N-S),
  final app name and icon. The launcher label is "22-RPM" vs iOS "22 RPM", and the
  package is still `com.infuzamed`. Step 1 names the store listing and the icon label.
  Until then, decide what Step 1 says about Android.
- [ ] **B3. Biometric login settled.**
  - Port the iOS lockout fix (`f0efab8`). Android `Login.js` has the same stale-state
    retry loop, so "tap Use Password" may not work on Android (read from code, not
    confirmed on a phone).
  - Land the Keystore move (launch plan #4). It may change the prompts.
  - Decide the wording. Today Android shows "Enable Biometrics?" / "Login with
    Biometrics" / "Remove Saved Biometrics". The "Fingerprint" wording in the code never
    appears on Android.
- [ ] **B4. Home screen final.** Remove the made-up vitals (launch plan #3), and decide
  whether Android gets the new home screen iOS has. Step 3 ("open Blood Pressure") and its
  screenshot depend on this. Today it's the old grid with a "Blood Pressure" tile.
- [ ] **B5. Bluetooth permissions actually work on Android 12+.** `bridging/Bp2Module.js`
  requests `ACCESS_FINE_LOCATION` **without** `ACCESS_COARSE_LOCATION`. On some Android
  12+ versions that request is ignored, which would fail every time with "Failed to
  initialize blood pressure service". Also: a denied permission leaves no route to
  Settings. Verify on a phone and fix before documenting the prompts.
- [ ] **B6. Location switch and Bluetooth-state behavior decided.** Either add
  `neverForLocation` to `BLUETOOTH_SCAN` (drops the location requirement on 12+) or
  document "turn on Location". Scanning on Android 11 and older finds nothing with
  Location off, and shows no error. Also: the BP screen checks Bluetooth only once when
  it opens, so turning Bluetooth on afterwards requires leaving and reopening the screen.
  Fix this or document it.
- [ ] **B7. Durable outbox merged** (`feature/android-outbox`, `dece499`). On `main`, a
  reading that fails to upload is dropped after an alert. The FAQ promise ("stored and
  sends automatically") and a "Reading sent/saved" equivalent both depend on this.
- [ ] **B8. Pairing model decided.** Android has no saved-device auto-reconnect today:
  every visit means Connect Device → "Connect to BP2 Device" → Scan for Devices → tap the
  cuff (the row shows its MAC address), and leaving the screen disconnects. Either build
  iOS-style auto-reconnect or write "scan every time". This decides the shape of the whole
  Android Step 4, so settle it first.
- [ ] **B9. Guide delivery on Android.** There's no Learn/Education tab, so there's no
  in-app link to `/bp-guide/`. Also, the in-app help text "Open the Readings tab…" is
  iOS-only.
- [ ] **B10. Device test and capture.** At least one Android 12+ phone (Pixel or Samsung)
  and one Android 11-or-older phone, with the BP2A, running the build that has B1–B8:
  permission prompts, the Location switch, pairing, reading, offline reading. Capture at
  a matching spec: portrait, status bar cropped, 600 px wide, default font and display
  size.

Before publishing (both tracks):
- [ ] Kinza clinical review of the full page. Steps 5 and 6 and the numbers table are
  unchanged from live, but the page as a whole is patient-facing.
- [ ] Spanish (`/bp-guide/es/`) updated to match, and pointed at the new image names.
- [ ] Remove every `.draft-*` / `.stub` box and the draft-only styles before this reaches
  `main`.

---

## iPhone UX issues found while writing (not guide changes — candidates for 1.0.51)
The guide works around these. Fixing them would shorten Step 4 and the Step 6 notes.
1. **First-time pairing waits 15 s.** With no saved cuff, opening Blood Pressure still
   shows "Reconnecting to your device…". If the cuff is on, the Connect button grays to a
   disabled "Connecting...". After 15 s: "Couldn't connect to the cuff. Check that it's
   turned on." and the list opens. Suggested fix: don't start the reconnect window when
   there is no saved device.
2. **After every reading**, when the cuff powers off: the app says "Device disconnected"
   aloud, then 15 s later shows "Couldn't connect to the cuff" and opens the list, even
   though nothing is wrong.
3. **The OTP pop-up always says "sent to your email"**, even when the backend sent it by
   text (phone-number logins go to SMS).
4. **The login screens always say "Face ID"**, including on Touch ID iPhones.
