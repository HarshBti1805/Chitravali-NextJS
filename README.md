## Chitravali — Next.js Social Video Streaming Platform

Chitravali is a modern social media platform for video creators and communities. Users can upload and store videos in a cloud-backed hub, watch with a customizable player, share to a global chat, and host synchronized watch parties (Discord-style watch-alongs) with friends—including live streaming.

### Key Features
- **Video uploads**: Reliable, resumable uploads with server-side validation and thumbnail generation.
- **Cloud hub**: Durable storage with CDN delivery and signed URLs.
- **Custom player**: Theater mode, hotkeys, variable speed, quality selection, captions, chapters, picture-in-picture, and mini-player.
- **Live streaming**: Go live via RTMP/WebRTC; HLS/DASH playback with adaptive bitrate.
- **Watch parties**: Synchronized playback rooms with chat, reactions, and host controls.
- **Global chat**: Share videos and clips to a real-time global feed with moderation.
- **Social graph**: Profiles, follows, likes, comments, playlists, and shares.
- **Search & discovery**: Full-text search, tags, categories, trending.
- **Privacy & access**: Public, unlisted, private; link and role-based room access.
- **Mobile-first**: Responsive UI, accessible controls, light/dark themes.

## Tech Stack
- **Frontend**: Next.js 14+ (App Router), React, TypeScript, TailwindCSS
- **State/RTC**: Zustand/Redux (optional), WebRTC, WebSockets (Socket.IO)
- **Video**: HLS/DASH, `hls.js`/`dash.js`, Media Source Extensions, WebCodecs (optional)
- **Backend**: Node.js (Next.js Route Handlers or Express), REST + WebSocket
- **Database**: PostgreSQL/Prisma (or MongoDB/Mongoose as provided in `models/`)
- **Storage/CDN**: S3-compatible (AWS S3, R2, MinIO) + CDN of choice
- **Transcoding**: FFmpeg (self-host) or cloud encoder (Mux/Cloudflare Stream)
- **Auth**: NextAuth.js / OAuth providers / JWT
- **Infra**: Docker, Nginx (optional), Vercel/Render/Fly.io/AWS

## Repository Layout
```
Chitravali-NextJS/
  client/       # Next.js application (UI, routes, player, chat)
  server/       # API/websocket, upload webhooks, RTMP ingest, FFmpeg jobs
  models/       # Database schemas/models (e.g., Prisma/Mongoose)
  README.md
```

## Getting Started

### Prerequisites
- Node.js 18+ and pnpm/npm/yarn
- FFmpeg (for local thumbnails/transcoding)
- Docker (optional, for DB/storage/RTMP)

### Environment Variables
Create a `.env` (or `.env.local` in `client/`) with at least:

```
# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development

# Auth
AUTH_SECRET=replace-with-strong-secret
GITHUB_ID=...
GITHUB_SECRET=...

# Database (choose one)
DATABASE_URL=postgresql://user:pass@localhost:5432/chitravali
# or
MONGODB_URI=mongodb://localhost:27017/chitravali

# Object Storage
STORAGE_ENDPOINT=http://localhost:9000
STORAGE_BUCKET=chitravali
STORAGE_REGION=auto
STORAGE_ACCESS_KEY=...
STORAGE_SECRET_KEY=...

# CDN (optional)
CDN_BASE_URL=

# WebSockets
WS_URL=ws://localhost:4000

# Live/RTMP (optional)
RTMP_INGEST_URL=rtmp://localhost/live
HLS_BASE_URL=http://localhost:8080/hls
```

### Install
```bash
# frontend
cd client
pnpm install

# backend
cd ../server
pnpm install
```

### Database Setup
If using Prisma + Postgres:
```bash
pnpm prisma migrate dev
pnpm prisma generate
```

### Development
Run backend and frontend in parallel.

```bash
# Terminal 1 — server
cd server
pnpm dev

# Terminal 2 — client
cd client
pnpm dev
```

Open the app at `http://localhost:3000`.

## Core Concepts

### Upload Flow
1. Client requests a signed upload URL from the server.
2. Client uploads video directly to storage (resumable, chunked).
3. Server receives upload-complete webhook → enqueues FFmpeg job.
4. FFmpeg generates HLS/DASH renditions, thumbnails, and poster.
5. Player loads manifest via CDN/signed URL.

### Custom Player
- Hotkeys: space (play/pause), `f` (fullscreen), `i` (PiP), `↑/↓` volume, `←/→` seek
- Controls: timeline with thumbnails, speed, quality, captions, theater mode
- Edge cases: auto-recover on errors, ABR for network changes, caption styling

### Watch Parties (Sync Rooms)
- WebSocket room per party; host is source of truth
- Events: play/pause/seek/speed/change-quality
- Late joiners: server provides state snapshot on join
- Anti-desync: time drift correction with small nudges

### Live Streaming
- RTMP ingest → FFmpeg → HLS segments → CDN
- Low-latency HLS/DASH or WebRTC for near-real-time

## Scripts (suggested)
In `client/package.json` and `server/package.json`:

```json
{
  "scripts": {
    "dev": "next dev",           // client
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  }
}
```

```json
{
  "scripts": {
    "dev": "ts-node-dev src/index.ts",   // server
    "build": "tsc -p .",
    "start": "node dist/index.js"
  }
}
```

## API Surface (high-level)
- `POST /api/upload/sign` — get signed URL for storage
- `POST /api/upload/webhook` — notify server upload completed
- `GET /api/videos/:id` — fetch video metadata & manifests
- `GET /api/search` — search by title/tags
- `POST /api/rooms` — create watch party
- `POST /api/rooms/:id/join` — join a room
- `WS /ws` — realtime chat and sync events

## Production Notes
- Use a CDN in front of HLS manifests and segments
- Use signed URLs and short TTLs for private content
- Run FFmpeg workers separately from API instances
- Prefer object storage lifecycle rules for old segments/thumbnails
- Enable HTTP/2, compression, and caching headers

## Security & Compliance
- Validate uploads (MIME/size), scan for malware if required
- Rate limit API and WS joins, protect room IDs
- Sanitize chat, implement moderation and report flows
- Store only necessary PII, encrypt at rest and in transit

## Performance
- ABR streaming with multiple renditions
- Preload poster and first segment for fast start
- Memoized player state, virtualization in lists
- Edge cache manifests aggressively with revalidation

## Roadmap
- Clip editor and highlights
- Web Subtitles Editor (WebVTT authoring)
- Server-side ad insertion hooks
- Native apps (React Native + ExoPlayer/AVPlayer)

## Contributing
PRs welcome! Please open an issue to discuss large changes first.

## License
MIT


