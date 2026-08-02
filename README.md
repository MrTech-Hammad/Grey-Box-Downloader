# GreyBox Downloader

Multi-platform video/audio downloader — TikTok, Instagram, Facebook, X, Pinterest, LinkedIn, Reddit.
Built with Flask + yt-dlp, same base pattern as StreamDrop.

How to Use It:
- Download and run the `.exe` installer.
- Follow the installation wizard.
- Select a custom installation location if desired.
- Complete the installation.
- Done! Enjoy using the Grey Box Downloader by simply double click on the icon.

## Run locally (web app)

```
pip install -r requirements.txt
python app.py
```

Open http://127.0.0.1:5000

## How it works

- `/api/fetch` — reads the link with yt-dlp (no download), returns title/thumbnail/duration or playlist count.
- `/api/start-download` — kicks off a background download thread, returns a `job_id`.
- `/api/progress/<job_id>` — polled every second by the frontend for live percentage.
- `/api/file/<job_id>` — once status is `done`, this serves the finished file directly, so the Save button
  saves straight to disk instead of needing a second click.
- Playlists are zipped automatically once every item finishes; the Save button then downloads the zip.

## Known platform limits (carried over from earlier research — verify before shipping)

- **TikTok** needs browser impersonation (`extractor_args` set in `app.py`) or requests get blocked.
- **Instagram** needs the `Referer` header set (already handled) or you'll get 403s. Private posts need cookies.
- **Reddit** blocks datacenter IPs at the network level — if you deploy this to a cloud VPS, Reddit links
  may fail even though the code is correct. Works fine when run locally on a home connection.
- **LinkedIn** — public video posts work; yt-dlp recently dropped LinkedIn login support, so private/authenticated
  content won't extract.
- **Spotify** is DRM-protected and intentionally blocked in the code (`UNSUPPORTED_HOSTS` in `app.py`) —
  yt-dlp cannot legally extract Spotify audio.

## Converting to a desktop app (like StreamDrop)

Same pattern as StreamDrop: wrap this Flask app in a native window instead of rewriting it.

1. `pip install pywebview`
2. Add a small launcher that starts Flask in a background thread and opens a `pywebview` window pointed at
   `http://127.0.0.1:5000` instead of a browser tab.
3. Package with PyInstaller: `pyinstaller --onefile --add-data "templates;templates" --add-data "static;static" launcher.py`
4. Build the installer with Inno Setup, same script pattern as StreamDrop.

Say the word when you're ready for step 2 (the launcher script) — it's a ~30 line addition once the web
version is confirmed working the way you want.
