# Video Link → Local Storage → YouTube Upload

> **Goal**: Build a platform where a user pastes a video URL (YouTube / Instagram / Facebook / direct links), the system **extracts the video**, **saves it locally (or temp storage)**, and then **uploads it to YouTube via APIs**.

> **Scope**: This document is **purely technical**. It focuses on system architecture, components, data flow, and implementation details.

---

## 1. High-Level Architecture

```
[ Client (Web UI) ]
        |
        v
[ API Gateway / Backend ]
        |
        v
[ Job Queue ]  <------------------------------+
        |                                      |
        v                                      |
[ Extraction Worker ]                          |
        |                                      |
        v                                      |
[ Local / Temp Storage ]                       |
        |                                      |
        v                                      |
[ Upload Worker (YouTube API) ] ---------------+
        |
        v
[ YouTube ]
```

---

## 2. Core Components

### 2.1 Client (Frontend)

* Input field for **video URL**
* Optional metadata:

  * title
  * description
  * tags
  * privacy
* Trigger button: `Extract & Upload`

**Responsibilities**:

* Validate URL format (basic)
* Send request to backend
* Show job status (queued / downloading / uploading / done / failed)

---

### 2.2 API Gateway / Backend

**Tech**:

* Node.js (Express / NestJS) or Python (FastAPI)

**Responsibilities**:

* Accept URL + metadata
* Create a **job record**
* Push job to queue
* Return `jobId`

**Endpoints**:

```
POST /jobs
GET  /jobs/:jobId/status
```

---

### 2.3 Job Queue

**Purpose**: Async, reliable processing

**Options**:

* BullMQ (Redis)
* RabbitMQ
* Kafka (overkill for MVP)

**Why needed**:

* Video extraction is slow
* Uploads can take minutes
* Retry handling

---

## 3. URL → Video Extraction (Deep Technical)

This is the **most critical part**.

### 3.1 URL Classification

When a URL comes in, first step is **classification**:

```
Input URL
   ↓
URL Classifier
   ↓
[ Direct Media | Streaming Playlist | Platform Link ]
```

---

## 4. Extraction Case 1: Direct Media URL

### Example

```
https://cdn.example.com/videos/sample.mp4
```

### Characteristics

* Ends with `.mp4`, `.webm`, `.mov`
* Single HTTP GET stream

### Extraction Strategy

* Stream download
* Pipe directly to file

### Pseudocode

```js
const response = await fetch(url);
const stream = fs.createWriteStream(localPath);
response.body.pipe(stream);
```

### Output

* Local file saved

---

## 5. Extraction Case 2: Streaming URLs (HLS / DASH)

### Examples

* `.m3u8` (HLS)
* `.mpd` (DASH)

### Characteristics

* Video split into segments
* Adaptive bitrate

### Extraction Strategy

* Parse playlist
* Download segments
* Merge using FFmpeg

### Tooling

* `ffmpeg`

### Command Example

```bash
ffmpeg -i input.m3u8 -c copy output.mp4
```

### Output

* Single `.mp4` file

---

## 6. Extraction Case 3: Platform URLs (YouTube / Instagram / Facebook)

### Examples

```
https://youtube.com/watch?v=...
https://instagram.com/reel/...
https://facebook.com/video/...
```

### Technical Reality

* These platforms **do not expose raw media URLs directly**
* Video streams are dynamically generated

### Extraction Strategy

**Dedicated Extractor Layer** using:

* `yt-dlp`
* `youtube-dl` (legacy)

These tools:

* Fetch page
* Extract player config
* Resolve media streams
* Download best-quality video

### Why yt-dlp?

* Actively maintained
* Supports YouTube, Instagram, Facebook
* Handles HLS/DASH internally

### Execution Model

```bash
yt-dlp -f best -o /tmp/{jobId}.mp4 <URL>
```

### Output

* Final merged `.mp4` saved locally

---

## 7. Local / Temporary Storage

### Storage Options

* Local disk (`/tmp` or `/data`)
* Mounted volume (Docker)
* Object storage (S3 / GCS) as intermediate

### Recommended Structure

```
/tmp/video-jobs/
  └── {jobId}/
      ├── source.mp4
      ├── logs.txt
      └── metadata.json
```

### Why Local First?

* YouTube API uploads from file/stream
* Retry-friendly
* Resume upload supported

---

## 8. Upload Worker (YouTube)

### API Used

* **YouTube Data API v3**
* Endpoint: `videos.insert`

### Authentication (Multi‑Channel Support)

This system is designed to support **multiple YouTube channels from a single web app**.

#### Key Concept

* **Each YouTube channel = separate OAuth consent**
* Tokens are stored **per user per channel**

```
User
 ├── YouTube Channel A (tokens)
 ├── YouTube Channel B (tokens)
 └── YouTube Channel C (tokens)
```

### OAuth Flow (Per Channel)

1. User clicks **“Add YouTube Channel”**
2. Redirect to Google OAuth consent screen
3. User selects a specific YouTube account/channel
4. App receives:

   * Access Token
   * Refresh Token
5. Tokens stored with:

   * `userId`
   * `channelId`

### Token Storage Model

```
User
 └── Channels[]
      ├── channelId
      ├── channelTitle
      ├── accessToken
      ├── refreshToken
      └── tokenExpiry
```

### Upload Strategy

* Each upload job is linked to a **specific channelId**
* Upload worker:

  * Loads correct OAuth tokens
  * Refreshes token if expired
  * Uploads video to that channel

### Upload Flow

```
Job
 ├── jobId
 ├── userId
 ├── channelId   ← selected by user
 ├── localVideoPath
 └── metadata
```

---

## 9. Job State Machine

```
PENDING
  ↓
DOWNLOADING
  ↓
DOWNLOADED
  ↓
UPLOADING
  ↓
COMPLETED

FAILED (from any state)
RETRYING
```

---

## 10. Error Handling & Retries

### Extraction Errors

* Network timeout
* Broken playlist
* Partial download

### Upload Errors

* Token expired → refresh
* Upload interrupted → resume

### Retry Policy

* Max retries: configurable
* Backoff: exponential

---

## 11. Scalability Considerations

### Horizontal Scaling

* Multiple extraction workers
* Multiple upload workers

### Resource Isolation

* Separate queues for:

  * extraction
  * upload

### Disk Cleanup

* TTL-based cleanup job

---

## 12. Security / Isolation (Technical)

* Run extractors in sandboxed containers
* Limit CPU & RAM per job
* Validate URL input

---

## 13. Final End-to-End Flow (Multi‑Channel)

```
User pastes URL
   ↓
User selects YouTube Channel
   ↓
Backend creates job (linked to channelId)
   ↓
Extractor worker downloads video
   ↓
Video saved locally
   ↓
Upload worker loads channel‑specific tokens
   ↓
Video uploaded to selected YouTube channel
   ↓
Job marked completed
```

---

## 14. Summary

* Single web app can manage **multiple YouTube channels**
* Each channel is connected via **independent OAuth consent**
* Upload jobs are **explicitly bound to a channelId**
* Video extraction pipeline remains **channel‑agnostic**
* System scales cleanly for creators, agencies, and multi‑brand setups

This architecture supports:

* One user → many channels
* One job → one channel
* Many parallel uploads across channels

---

If you want next:

* OAuth DB schema (SQL)
* Google OAuth consent screen setup
* Channel switcher UI design
* Node.js OAuth + upload code
* Agency‑level (team + permissions) extension
