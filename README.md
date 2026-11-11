# Chat PDF Studio — Android local-processing project

This project embeds the original `index.html` interface in a WebView and runs the adapted `Build.py` pipeline locally on the phone through Chaquopy. `server.py` is retained as a reference and is not used in local Android mode.

## Project layout

- `Build.py`: adapted PDF-processing pipeline at project root.
- `server.py`: retained server source/reference.
- `index.html`: original interface at project root.
- `app/src/main/python/Build.py`: Python source packaged into the app.
- `app/src/main/python/backend.py`: Android-facing adapter.
- `app/src/main/assets/index.html`: interface loaded by the app.
- `app/build.gradle.kts`: Android and Python dependencies.

## Required before building

1. Android Studio with JDK 17 selected for Gradle, Android SDK Platform 35, and Gradle 8.13 wrapper support.
2. Python 3.13 available to Chaquopy on Windows (`py -3.13 --version` should succeed).
3. Copy the real font files below into `app/src/main/assets/fonts/`:
   - `NotoSansDevanagari-Regular.ttf`
   - `NotoSansDevanagari-Bold.ttf`
   The project only contains a placeholder, not the font binaries.
4. Sync Gradle and build a new APK. Do not reuse an APK built from the previous project.

## Pillow compatibility change

`app/build.gradle.kts` now explicitly pins `Pillow==11.0.0`, pins `fpdf2==2.8.9`, and configures `extractPackages("PIL")`. Pillow 11.0.0 has Chaquopy Android wheels for Python 3.13 on `arm64-v8a` and `x86_64`. This targets the reported `_imagingcms` import problem. A full Android Studio build and on-device test are still required to confirm the dependency is packaged correctly in your environment.

## Build and test

1. Open the inner `chat_pdf_android` folder in Android Studio.
2. Wait for Gradle Sync to complete.
3. Choose **Build > Clean Project**.
4. Choose **Build > Build APK(s)**.
5. Install the newly generated `app-debug.apk` on the phone.
6. Test a small ZIP archive first, including an archive containing an image.

The expected APK path is usually `app/build/outputs/apk/debug/app-debug.apk`.

## Important

The source archive has not been compiled or run on your specific Android device from this environment, so no one can honestly guarantee a zero-error build before Gradle and on-device testing. If the build fails, capture the first Gradle error. If the app runs but generation fails, capture the full displayed error and Android Studio Logcat traceback.


## Pillow Android CMS compatibility and retry behavior

The Android adapter installs an optional `PIL.ImageCms` compatibility shim only if Pillow's native `_imagingcms` extension is missing. It does not replace Pillow's core `Image` decoder. In that case, ICC color-profile conversion is skipped while normal image decoding remains enabled. The build pins Pillow 11.0.0 and fpdf2 2.8.9 and no longer forces extraction of the entire `PIL` package.

On Android, processed-name tracking uses a temporary per-run state file so a failed PDF generation does not poison the next attempt with “No matching entries found or data already processed.” Desktop/default `Build.main` behavior keeps persistent tracking enabled.

These source changes must be rebuilt into a fresh APK. The project has not been compiled or run on a physical Android device in this environment; validate PDF generation with a ZIP containing both text and images.


## Output bundling and duplicate prevention

- If one PDF is generated, the app offers that PDF for saving. If more than one PDF is generated, the Android backend bundles all PDFs into one ZIP archive and presents a single save action.
- Processed-name registries are stored under the app-private `chat_pdf_studio/processed_names` directory and keyed by the selected archive's original filename. Only biodata blocks that have an image and are actually included in successfully generated PDFs are committed to the registry.
- Biodata blocks with `NO_IMAGE` are excluded from PDF chunks. New names are committed only after every PDF chunk succeeds, so a failed generation does not mark entries as processed.
- If you change the contents of a ZIP but keep exactly the same filename, the registry intentionally treats it as the same archive reference. To process a different archive independently, give it a different filename.
