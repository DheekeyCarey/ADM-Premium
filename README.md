# ADM Premium — Tauri Desktop Build

This wraps your "ADM Premium v4.0 (Final)" web app (the LJK/answer-sheet-scanner
version, since it's a superset of the other file) into a native Windows app
using Tauri. Your original Blogger `<b:...>` template tags were stripped out —
`src/index.html` is now a plain, self-contained HTML/CSS/JS file with no
Blogger-specific markup left in it.

## 1. One-time setup (only needed once per machine)

You need Rust and Node.js installed on the Windows machine you're building on:

1. Install Rust: https://rustup.rs (also installs the MSVC build tools prompt —
   accept it, or install "Desktop development with C++" via Visual Studio
   Build Tools if it asks).
2. Install Node.js LTS: https://nodejs.org
3. Install the Tauri CLI dependency for this project:
   ```
   npm install
   ```

## 2. Set your Google Apps Script URL

Open `src/index.html`, find this line (search for `GAS_URL`):

```js
const GAS_URL = "URL_APPS_SCRIPT_ANDA_DISINI";
```

Replace the placeholder with your deployed Apps Script Web App URL. Make sure
that Apps Script deployment is set to **Execute as: Me** and
**Who has access: Anyone** (or "Anyone with Google account", matching what
your backend expects), otherwise the desktop app's `fetch()` calls will fail
or get blocked by CORS.

## 3. Add app icons (optional but recommended)

Put a square PNG (at least 1024×1024) somewhere, then run:

```
npx tauri icon path/to/your-logo.png
```

This auto-generates everything referenced in `src-tauri/tauri.conf.json`
(`icons/32x32.png`, `128x128.png`, `icon.ico`, `icon.icns`, etc.) directly
into `src-tauri/icons`. If you skip this step, drop placeholder icon files
into `src-tauri/icons` yourself or the build will fail looking for them.

## 4. Run it in dev mode

```
npx tauri dev
```

This opens a native window loading `src/index.html` straight off disk — good
for checking that login, camera access (for the QR/LJK scanner), and the
Apps Script calls all work before you package anything.

## 5. Build the installer

```
npx tauri build
```

Output lands in `src-tauri/target/release/bundle/`:
- `nsis/ADM Premium_4.0.0_x64-setup.exe` — installer
- `msi/ADM Premium_4.0.0_x64_en-US.msi` — alternative MSI installer

Either one is a normal Windows installer you can hand to end users — no
browser, no Blogger hosting required. The app runs fully offline except for
the `fetch()` calls to your Apps Script backend.

## Building via GitHub Actions (no local install needed)

There's a ready-made workflow at `.github/workflows/build.yml` that builds
the Windows installer for you on GitHub's own Windows runner — you don't
need Rust, Node, or Visual Studio Build Tools on your own machine.

1. Create a new (can be private) GitHub repo and push this whole folder to it:
   ```
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/<you>/<repo>.git
   git push -u origin main
   ```
2. That push alone triggers the workflow. Go to the **Actions** tab on your
   repo and watch "Build Windows Installer" run (takes a few minutes — it's
   compiling Rust from scratch the first time).
3. When it finishes, open the finished run and scroll to **Artifacts** at the
   bottom — download `adm-premium-windows-installers.zip`. Inside is the
   `.exe` (NSIS) and `.msi` installer, built exactly as `npx tauri build`
   would produce locally.
4. To rebuild later after editing `src/index.html`, just commit and push
   again, or use the **Run workflow** button on the Actions tab (no push
   needed, thanks to `workflow_dispatch` in the workflow file).

Since your repo will contain your Apps Script URL once you fill it in (step
2 above), keep the GitHub repo **private** unless that URL is meant to be
public.

## Notes / things worth knowing

- **Camera permission**: the LJK scanner uses `getUserMedia`. Tauri's
  underlying WebView2 will show the normal Windows camera-permission prompt
  the first time — nothing extra to configure.
- **CSP is disabled** (`security.csp: null` in `tauri.conf.json`) because the
  app loads Tailwind, SweetAlert2, Chart.js, jsPDF, JsBarcode, jsQR, and
  QRCode.js from public CDNs, plus Google Fonts. If you'd rather lock this
  down, download those libraries and serve them locally from `src/`, then
  re-enable a strict CSP.
- **The plain (non-LJK) file** you uploaded is functionally a subset of this
  one (same app, minus the answer-sheet scanner page) — if you actually want
  that version instead, just swap in `Frontend_guru_kelas.txt`'s cleaned body
  content for the LJK-specific bits in `src/index.html`. Say the word and I
  can produce that variant too.
