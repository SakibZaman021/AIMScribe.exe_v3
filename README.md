# AIMScribe.exe — the clinic recorder

The Windows program that records a doctor–patient consultation, one per
consulting room, for the AIMScribe v3 study. It runs on the doctor's own PC
beside the CMED browser page, and it is the only part of the system that ever
touches audio.

Built and maintained by **AIMS LAB, United International University**.

---

## What it does

A consultation is recorded continuously, in 30–90 second pieces, and each piece
is uploaded as soon as it is sealed. The program is driven by two signals from
the CMED page over a loopback connection — *a patient has been opened* and *the
prescription has been built* — and it decides for itself where one consultation
ends and the next begins.

| | |
|---|---|
| **Audio** | WAV, 44.1 kHz, mono, 16-bit — a fixed 5.02 MB per minute |
| **Segmentation** | 30–90 seconds, cut in a 1-second silence so no word is split |
| **Local connection** | `ws://127.0.0.1:5050/ws` — loopback only, never the network |
| **If the server is unreachable** | keeps recording and holds about 13 hours on disk |
| **Integrity** | every session is an Ed25519-signed hash chain |

Audio is only removed from the PC once the server has acknowledged it, and a
recording is only deleted anywhere once a signed receipt proves an archive copy
exists.

---

## Layout

```
main.py                the program entry point and single-instance guard
config.py              every setting, read from .env
api/
  protocol.py          the message contract and every reply code
  websocket_server.py  the loopback connection the CMED page talks to
core/
  session_controller.py   one consultation, start to close
  simple_splitter.py      the 30–90 s segmenter and silence detection
  uploader.py             talks to the AIMS LAB server
  recorder.py             the microphone
ui/
  overlay.py           the small on-screen control the doctor sees
keys/                  AIMS LAB public keys, for verifying grants and receipts
tests/                 the test suite
scripts/               enrolment and maintenance helpers
installer.iss          the Inno Setup installer
BUILD.bat              builds the .exe with PyInstaller
```

---

## Running from source

```bat
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
copy .env.example .env
```

Fill in `.env` — at minimum `AIMS_BACKEND_URL` and `AIMS_ALLOWED_ORIGINS`, which
must list the exact web address of the CMED page — then:

```bat
python main.py
```

Tests:

```bat
pytest
```

## Building the installer

```bat
pip install -r requirements-build.txt
BUILD.bat
```

`BUILD.bat` preserves an existing `dist\AIMScribe_Agent\.env` across the build,
so a configured machine does not lose its settings when the program is rebuilt.

---

## Configuration

Every setting lives in `.env`; `.env.example` is the annotated template. The
real `.env` is never committed — it holds the local API key and identifies the
machine.

Two settings matter more than the rest:

- **`AIMS_ALLOWED_ORIGINS`** — the exact web addresses allowed to drive the
  recorder. The page is refused if its address is not listed, and `www.` and
  non-`www` are different addresses, as are `http` and `https`.
- **`AIMS_CLINIC`** — set at enrolment. A recording claiming a different clinic
  is refused outright.

---

## A note on scope

This repository is the recorder only. The server it talks to, the archive
worker, the databases and the dashboard are a separate system.

Recordings and patient data are held by AIMS LAB. Nothing in this repository
contains either, and no credential in it is live.

---

© AIMS LAB, United International University. All rights reserved.
