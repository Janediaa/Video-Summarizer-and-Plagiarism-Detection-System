# Video Summarizer and Plagiarism Detection Project

A lightweight multi-component project that transcribes audio, summarizes transcripts (locally or via Gemini), and checks summary similarity/plagiarism using a fine-tuned SBERT model.

---

## Key features
- Audio transcription using Google Cloud Speech (short and long audio handling).
- Local summarization and optional Gemini model summarization.
- SBERT-based similarity / plagiarism check served by a Flask API.
- Static frontend and Node/Express backend that tie components together.

## Repository structure
- `fine_tuned_sbert_scored/` — fine-tuned Sentence-BERT model files (large binaries & configs).
- `minor-final/`
  - `minor-final/backend/` — Node/Express backend (`server.js`), uploads folder, service-account JSON, `.env` expected.
  - `minor-final/frontend/` — static frontend (`index.html`, `dist/output.css`).
  - `minor-final/python/` — Flask SBERT API (`sbert_api.py`) for similarity checks.
- `.venv/` (recommended local Python venv)
- `req.txt` — Python dependencies for the SBERT service and other Python tooling.
- `package.json` / `package-lock.json` — root and subproject metadata.

## Prerequisites
- Node.js (v16+ recommended) and npm
- Python 3.9+ and `pip`
- PowerShell or another shell on Windows
- (Optional) ffmpeg — backend uses `ffmpeg-static`, but system ffmpeg can help
- Google Cloud service account JSON and a GCS bucket if you use long-running speech recognition
- Ensure `fine_tuned_sbert_scored/` model files exist locally before starting the SBERT API

## Local setup and run (Windows / PowerShell)

1) Clone and enter repo
```
git clone <repo-url>
cd new_project
```

2) Create Python venv and install Python deps
```
python -m venv .venv
# Activate (PowerShell)
.venv\Scripts\Activate.ps1
pip install -r req.txt
```

> Note: `req.txt` includes `sentence-transformers`, `torch`, `flask`, etc. Installing these can be large and may require selecting the correct `torch` wheel for your platform.

3) Start the SBERT Flask API
```
cd minor-final\python
# with venv activated
python sbert_api.py
```
- Service listens on `http://127.0.0.1:5001` and is used by the Node backend for plagiarism checks.

4) Configure and run Node backend
- Create a `.env` file in `minor-final/backend/` with these entries:
```
GCS_BUCKET_NAME=your-gcs-bucket
GEMINI_API_KEY=your-gemini-api-key-or-empty
GOOGLE_APPLICATION_CREDENTIALS=./sharp-starlight-472906-f7-07a9ef7d6329.json
PORT=1330
```
- Install and run:
```
cd minor-final\backend
npm install
node server.js
# or use nodemon:
npx nodemon server.js
```
- Backend endpoints:
  - `POST /transcribe` — form upload `file` → returns `{ transcription }`
  - `POST /summarize` — JSON `{ text, mode, bullets }` → returns summary
  - `POST /check-plag` — JSON `{ summary }` → returns SBERT similarity (calls SBERT API)

5) Frontend
- The frontend is static: `minor-final/frontend/index.html`. You can open it directly in a browser or serve it:
```
cd minor-final\frontend
npm install
npx serve .
```
- Ensure backend is running and CORS is allowed.


## Troubleshooting
- Model load errors: confirm `fine_tuned_sbert_scored/` is present and contains model files; check CPU vs GPU `torch` wheel compatibility.
- Google Speech errors: verify `GCS_BUCKET_NAME`, service account permissions, and `GOOGLE_APPLICATION_CREDENTIALS`.
- Large installs: `torch` and some ML deps may require more disk/time.

