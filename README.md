# Journal

Journal is a local-first journaling project with a Flutter mobile app, an Electron AI companion, and a React landing page. Write diary entries, keep notes, and capture passing thoughts in Flow. Your writing is stored locally in ObjectBox; Google Drive sync and local AI are optional.

**Website:** [journal.lewiseman.com](https://journal.lewiseman.com/)

**Demo video:** [Watch Journal in action](https://youtu.be/Ndbo28jEzL4)

**Voice Demo video:** [Watch Journal in action](https://www.youtube.com/watch?v=wmvlNUro4z4)

## Features

| Area | What is implemented |
| --- | --- |
| Diary | Text entries, calendar and search, saved photo stories with captions and reflections, and optional AI questions, day details, and answers about past entries |
| Notes | Local notes and attachments, with optional AI summaries, writing help, image questions, page scanning, presentation, and generated backgrounds |
| Flow | Save and read short thoughts offline; optionally ask for a reflection or ask questions about a thought |
| Local AI | Run supported writing and background-generation tasks on the phone, or pair a computer for additional skills, including voice, vision, scanning, and search by meaning |
| People | Android peer-to-peer friend invitations and temporary encrypted notes; both apps must be open |
| Appearance | Themes, fonts, and a saved animation preference |
| Sync | Optional Google Drive app-data sync for journal content and attachment metadata, with primary-device controls and recovery |

AI output is presented for review; saving or applying it remains an explicit action. Photo, audio, scan, and generated-image files stay on the device that holds them and are **not backed up by Drive sync**. People notes are temporary and are not saved to the journal or Drive.

## Folders

| Folder | App | Built with |
| --- | --- | --- |
| [`bookflip/`](bookflip/) | Web landing page with an interactive book, app previews, and download links | React, Vite, Tailwind CSS, GSAP, Lenis |
| [`desktop/`](desktop/) | Journal Companion: runs local AI models and connects to the phone over Wi-Fi | Electron, Node.js, QVAC |
| [`journal/`](journal/) | Mobile app for Android and iOS | Flutter, Dart, Riverpod, ObjectBox |
| [`journal/packages/sync_wise/`](journal/packages/sync_wise/) | Shared sync package used by the mobile app | Dart, Google Drive app-data |
| [`journal/native/qvac/`](journal/native/qvac/) | Native build tools for phone AI and the Android People worker | QVAC SDK, BareKit, HyperDHT |

The three apps run separately and have no root-level install or start command. You can work on the website without Flutter, and use the mobile app without the desktop companion or a Google account.

## Requirements and supported platforms

| Component | Platform requirements |
| --- | --- |
| Android app with native AI and People | Android 10+ / API 29+, ARM64 (`arm64-v8a`) |
| iOS app with native AI | iOS 15+, physical arm64 device or Apple Silicon arm64 simulator; People is unavailable on iOS |
| Desktop companion | Apple Silicon Mac with macOS 14+, or Windows x64; build Windows packages on Windows |
| Landing page | A browser and the Node.js toolchain below |

- Use **Node.js 24.9+** and **npm 10.9+** for a common setup across the JavaScript folders.
- For mobile development, use a Flutter SDK that includes **Dart 3.13.4 or a compatible later Dart 3 version**, matching `journal/pubspec.yaml` (`^3.13.4`). Run `flutter doctor` after installing Flutter and the platform tools.
- For Android, install Android Studio, the Android SDK and Build Tools. The current Gradle daemon configuration requests **JDK 25**.
- For iOS, use an Apple Silicon Mac with Xcode, CocoaPods, and the Ruby `xcodeproj` gem available to the native build.
- Use **Python 3** for the Android release script.
- Initial dependency installation and model downloads need internet access. Model weights are downloaded separately from the apps and can take several GB.

Desktop model memory guidance is based on an **M5 MacBook Pro with 24 GB of unified memory**; larger models can exceed that budget. On-device AI has been checked on physical Android and the arm64 iOS simulator. Physical iPhone AI validation is still pending.

On macOS, if Java is not on your terminal's path, use Android Studio's bundled JDK:

```sh
export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home"
export PATH="$JAVA_HOME/bin:$PATH"
```

The commands below start from the folder containing this README unless stated otherwise. Replace `<device-id>` with an ID from `flutter devices`.

## Web — `bookflip/`

This is the public landing page, including the animated book, screenshots, and Android download links.

```sh
cd bookflip
npm ci
npm run dev
```

Open the local URL printed by Vite, normally **http://localhost:5173**.

To check and build the site, run these commands from `bookflip/`:

```sh
npm run lint
npm run build
npm run preview
```

The static output is written to `bookflip/dist/`. Shared links live in `src/site.js`, and Android release details come from `src/release.json`.

See the [web guide](bookflip/README.md) for page structure, artwork, and screenshot updates.

## Mobile — `journal/`

The Flutter app's main tabs are **Diary**, **Notes**, **Flow**, and **More**. More contains People, appearance settings, account/sync controls, and Local QVAC setup. The Flutter folder is the mobile app; `bookflip/` is the web landing page.

### Android

Use an **ARM64 phone running Android 10 or later** for the complete app with the bundled QVAC runtime. ARM32 and x86_64 devices are unsupported by that runtime.

Install the Flutter dependencies, then build the native QVAC/BareKit resources:

```sh
cd journal
flutter pub get
cd native/qvac
npm ci --ignore-scripts
npm run build -- --platform=android
cd ../..
flutter devices
flutter run -d <device-id>
```

Enable USB debugging on a connected Android phone and approve the computer when prompted. Start with a physical ARM64 phone for on-device AI checks. The native build downloads runtime files, not AI model weights; its generated output is excluded from source control and must be rebuilt on a fresh setup.

### iOS

On an Apple Silicon Mac, run the full native build to generate both Android and iOS resources. From `journal/`:

```sh
flutter pub get
cd native/qvac
npm ci --ignore-scripts
npm run build
cd ../..
cd ios
pod install
cd ..
flutter devices
flutter run -d <device-id>
```

The current iOS target is **iOS 15+**. Use an arm64 simulator; Intel simulators are unsupported by the AI runtime. For a physical iPhone, open `journal/ios/Runner.xcworkspace` in Xcode and configure your development team and signing before running.

If the native build cannot find the Ruby `xcodeproj` gem, install it with `gem install xcodeproj`.

### Optional Google Drive sync

Local journaling works without signing in. To enable Google sign-in and Drive sync in your own build:

1. Configure your Google Cloud OAuth clients and enable Google Drive API access with the app-data scope.
2. Match the native app ID (`com.lewiseman.journal`) and your signing configuration. Android OAuth registration must include your signing certificate's SHA-1. For iOS, configure the iOS client and its reversed-client-ID URL scheme in the Runner target.
3. Create an ignored `journal/env.json` with your own client IDs:

   ```json
   {
     "GOOGLE_SERVER_CLIENT_ID": "YOUR_WEB_OAUTH_CLIENT_ID.apps.googleusercontent.com",
     "GOOGLE_CLIENT_ID": "YOUR_PLATFORM_OAUTH_CLIENT_ID.apps.googleusercontent.com"
   }
   ```

   `GOOGLE_CLIENT_ID` is optional where native configuration supplies it. Do not put OAuth client secrets or Hugging Face tokens in this file; Dart defines are compiled into the app.

4. From `journal/`, run:

   ```sh
   flutter run -d <device-id> --dart-define-from-file=env.json
   ```

5. In the app, open **More → Connect Account**.

Sync transfers journal content and attachment metadata. **Photo, audio, and other attachment files are not transferred.** The first registered device becomes primary. Secondary devices can add notes and thoughts and append to today's diary; existing notes and older diary days are read-only. Read the [mobile guide](journal/README.md) for sync, restore, and primary-device recovery.

### Optional AI on the phone

Open **More → Local QVAC**, choose **Phone**, and use its model setup controls to download and assign a compatible writing or image model. Start with a small writing model and keep the app in the foreground while a task runs. Phone AI runs one model/job at a time; leaving the foreground interrupts the task. Background generation uses Stable Diffusion 2.1 and needs a separate image-model download.

**Computer** uses the paired desktop companion. **Auto** prefers a connected companion and otherwise uses a compatible assigned phone model for supported tasks. A task keeps the provider chosen when it starts; it does not switch machines after failure. Voice capture, scans, vision, website generation, and search by meaning currently need the companion.

### People on Android

Open **More → People** to invite a friend by QR/code and exchange temporary notes. People uses a separate HyperDHT/BareKit worker included in the Android native build. It does not require a Google account or the desktop companion.

Both apps must remain open. Notes are kept in memory and disappear after the reading timer, closing the reader, or leaving the foreground. There is no offline inbox, message history, or automatic retry. Peer discovery uses the public HyperDHT network, and some firewalls or mobile networks prevent connections. Two-phone and carrier-network validation remains a release requirement.

## Desktop — `desktop/`

Journal Companion runs AI models locally and makes its configured skills available to a paired phone. It supports **Apple Silicon Macs with macOS 14+** and **Windows x64**. Build Windows packages on Windows with a fresh dependency installation.

```sh
cd desktop
npm ci
npm start
```

`npm start` builds the QVAC worker before launching Electron. In the app, open **Skills → Writing**, choose a model, then select **Download & use**. Wait until the model is ready before requesting writing help from the phone. Other skills have their own model requirements; nothing downloads automatically.

Writing, speech recognition, vision, embeddings, and background generation are separate capabilities. Android voice uses Whisper plus Silero VAD; diary conversation also needs a writing model. Vision must pass its image self-test before photo questions or scanning are ready.

For authenticated Hugging Face downloads, optionally copy `desktop/.env.example` to `desktop/.env`, set your own `HF_TOKEN`, and restart the companion. Keep that file private. Public model downloads do not require adding a token to the mobile app.

### Pair your phone

1. Keep the phone and computer on the same private Wi-Fi network, with the companion open.
2. On the companion, open **Phones → Connect a phone** to display a fresh pairing QR/code.
3. On the phone, open **More → Local QVAC**, choose **Computer**, and scan the QR or use **Use a code instead**.
4. Check the computer identity, compare the verification code on both devices, and approve the phone on the desktop.
5. Load the model needed for the skill you want to use. A connected computer and a ready model are separate states.

Allow local-network access when prompted. The companion uses IPv4 port **47921** and Bonjour discovery. Guest Wi-Fi, VPNs, or client isolation can prevent a connection; entering the QR/code manually bypasses discovery but still needs a reachable local connection.

AI inference runs on the selected phone or computer. Google Drive sync is a separate, optional service. See the [desktop guide](desktop/README.md) for connection management, model storage, and task recovery.

### Package the companion

From `desktop/`, on macOS:

```sh
npm run make
```

The app is written to `out/Journal Companion-darwin-arm64/Journal Companion.app`, with a ZIP under `out/make/zip/darwin/arm64/`.

On Windows x64:

```sh
npm run make:win
```

The executable is written to `out/Journal Companion-win32-x64/Journal Companion.exe`, with a ZIP under `out/make/zip/win32/x64/`. These packaging commands produce unsigned local builds.

## Checks and development

Run the relevant commands from the app folder you changed:

| Folder | Commands |
| --- | --- |
| `bookflip/` | `npm run lint` and `npm run build` |
| `desktop/` | `npm test` |
| `journal/` | `flutter analyze`, `flutter test`, and `flutter test packages/sync_wise/test` |
| `journal/native/qvac/` | `npm run check` and `npm run test:people` |

People's additional Bare-runtime smoke check is `npm run test:people:bare` from `journal/native/qvac/`, with the `bare` command installed. Network tests use an isolated local DHT; they do not replace physical-device checks.

Host-side Flutter tests that open ObjectBox need a matching native ObjectBox library. This Mac setup uses the ignored `journal/lib/libobjectbox.dylib`; mobile builds get the runtime through `objectbox_flutter_libs`. See the [mobile testing notes](journal/README.md#running-and-checking).

If you change the shared AI prompts, schemas, or model catalog in `desktop/`, regenerate the mobile assets from that folder:

```sh
npm run export:operations
npm run verify:operations
```

`npm test` also checks that those generated assets are current. Keep `journal/lib/objectbox/objectbox-model.json` when regenerating ObjectBox bindings so existing database IDs are preserved:

```sh
# From journal/
dart run build_runner build
```

For device integration tests on an installed Journal app, use `--no-uninstall` to retain local app data and pairing credentials.

## Android release builds

From the repository root, with your own OAuth configuration and release signing configured:

```sh
python3 scripts/release_android.py
```

The script runs tests, rebuilds the Android native runtime, builds and verifies a signed ARM64 APK, and writes release artifacts under `journal/build/releases/`. It also refreshes `bookflip/src/release.json` when that sibling folder is present.

Adding `--publish` uploads a GitHub release and requires clean, committed, pushed source. Keep `env.json`, `android/key.properties`, signing keys, and tokens out of source control.

