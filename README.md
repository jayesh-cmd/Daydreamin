# 🎵 Bitsongs

A fully functional iOS music player that streams real music from the internet.

> SwiftUI frontend + Python FastAPI backend — search any song, stream it, control from lock screen.

## 📁 Project Structure

```
Bitsongs/
├── Bitsongs/                    # iOS App (SwiftUI)
│   ├── BitsongApp.swift         # App entry point
│   ├── Info.plist               # App configuration
│   ├── Models/
│   │   └── Song.swift           # Song data model
│   ├── ViewModels/
│   │   └── MusicPlayerViewModel.swift  # Player logic & state
│   ├── Views/
│   │   ├── MusicPlayerView.swift       # Main player screen
│   │   └── Components/
│   │       ├── AlbumArtView.swift      # Album artwork (remote)
│   │       ├── PlaybackControlsView.swift  # Play/pause/seek
│   │       ├── SearchBarView.swift     # Search + results list
│   │       ├── UpNextView.swift        # Queue (collapsible)
│   │       └── DynamicBackgroundView.swift  # Animated gradient BG
│   ├── Services/
│   │   └── NetworkService.swift        # API client for server
│   ├── Utilities/
│   │   ├── ColorExtractor.swift        # Extract colors from artwork
│   │   ├── HapticManager.swift         # Haptic feedback
│   │   └── ToneGenerator.swift         # (legacy) Demo tones
│   ├── Assets.xcassets/
│   └── Preview Content/
│
├── Bitsongs.xcodeproj/          # Xcode project
│
├── Server/                      # Backend (Python FastAPI)
│   ├── app.py                   # Main server (API + streaming)
│   ├── requirements.txt         # Python dependencies
│   ├── recommendation/          # Lightweight recommendation engine
│   ├── data/                    # Song catalog + transition tally storage
│   ├── tests/                   # Recommendation tests
│   └── song_cache/              # Cached audio files
│
├── .gitignore
└── README.md
```

## ⚡ Features

- 🔍 **Search** any song or artist (iTunes API)
- 📈 **Trending** charts on launch
- 🎧 **Real audio streaming** via YouTube
- ▶️ **Full controls** — play, pause, next, previous, seek
- 🖼️ **Album artwork** with dynamic color theming
- 🔒 **Background playback** — works with screen off
- 📱 **Lock screen controls** — play/pause/skip from lock screen
- 📋 **Queue** — see all upcoming songs
- 📝 **Lyrics** support (via LRCLIB)
- 🫨 **Haptic feedback** on controls

## 🚀 Setup

### Prerequisites

- **Xcode 15+** (macOS)
- **Python 3.9+**
- **ffmpeg** — `brew install ffmpeg`

### 1. Setup Server

```bash
cd Server
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python app.py
```

You can also start it with:

```bash
uvicorn app:app --host 0.0.0.0 --port 499
```

Server runs on `http://0.0.0.0:499`

Quick health check:

```bash
curl http://127.0.0.1:499/api/mobile/health
```

### 2. Configure iOS App

Edit `Bitsongs/Services/NetworkService.swift`:

```swift
// For Simulator:
@Published var baseURL: String = "http://127.0.0.1:499"

// For physical iPhone (use your Mac's IP):
@Published var baseURL: String = "http://192.168.x.x:499"
```

Notes:
- `127.0.0.1` works only for the iOS Simulator
- a real iPhone must use your Mac's LAN IP
- `Jay.local` may work on some networks, but LAN IP is more reliable

Find your Mac's IP:

```bash
ifconfig | grep "inet " | grep -v 127.0.0.1
```

### 3. Build & Run

1. Open `Bitsongs.xcodeproj` in Xcode
2. Select target device (simulator or iPhone via USB)
3. **⌘R** to build and run
4. First time on iPhone: **Settings → General → VPN & Device Management → Trust**

## 🏗️ Architecture

```
iPhone App ──HTTP──▶ FastAPI Server ──▶ iTunes API (search/metadata)
    │                    │
    │                    └──▶ yt-dlp (audio stream from YouTube)
    │                    │
    │                    └──▶ LRCLIB (lyrics)
    │                    │
    │                    └──▶ songs.json + tally_counter.json (recommendations)
    │                    │
    │                    └──▶ song_cache/ (cached audio)
    │
    └── AVPlayer (streams audio)
    └── MPNowPlayingInfoCenter (lock screen)
    └── MPRemoteCommandCenter (lock screen controls)
```

## 📄 API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/mobile/health` | GET | Server health check |
| `/api/mobile/search?q=` | GET | Search songs |
| `/api/mobile/chart` | GET | Trending songs |
| `/api/mobile/play?id=&artist=&title=&previous_song_id=` | GET | Get stream URL and optionally track transitions |
| `/api/mobile/stream_proxy?url=` | GET | Proxy audio stream |
| `/api/mobile/lyrics?artist=&title=` | GET | Get lyrics |
| `/api/mobile/recommend?song_id=` | GET | Grouped recommendations |
| `/api/mobile/up_next?song_id=&limit=` | GET | Up Next queue suggestions |
| `/api/mobile/cache_song` | POST | Ask the server to cache a song file |

## 🧠 Recommendation System

The backend recommendation engine is lightweight and file-based.

- Behavior-based recommendations use `Server/data/tally_counter.json`
- Content-based recommendations use metadata from `Server/data/songs.json`
- Up Next combines both strategies
- Tally data is auto-cleaned:
  - max 50 songs stored
  - max 3 next-song transitions per source song
  - 7-day 50% decay
  - zero-count transitions removed

Server audio cache is separate from recommendations:

- cached audio files are stored in `Server/song_cache/`
- when cache size goes above `600 MB`, the server clears the full audio cache

## 📦 Scope

This repo is now focused on:

- the iOS app
- the FastAPI mobile API

The old browser-based UI, templates, and static web assets are no longer part of the project.

## ✅ Backend Checks

Useful commands:

```bash
cd Server
source venv/bin/activate
python -m unittest tests.test_recommendation
python -m py_compile app.py recommendation/*.py
```

## 📝 License

Personal project — for personal use only.
