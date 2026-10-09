# 🚀 Complete Plan: Free + Self-Hosted AI Video Generator

## 1. Final product

Your application should eventually look like this:

```text
                    AI VIDEO STUDIO
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   Text → Video     Image → Video    Video Tools
        │                │                │
        └────────────────┼────────────────┘
                         ↓
                  AI Generation Engine
                         ↓
              ┌─────────────────────┐
              │   Open Models       │
              │                     │
              │     Wan 2.x         │
              │    LTX-Video        │
              │    FLUX / SDXL      │
              │    Whisper          │
              └─────────────────────┘
                         ↓
                    GPU Worker
                         ↓
                  Generated MP4
                         ↓
              Storage + Download
```

---

# 2. Your two main features
## A. Text → Video
User enters:

> A beautiful Indian bride walking through a palace, cinematic lighting, slow camera movement, realistic photography.

Then:

```text
Prompt
  ↓
Prompt processing
  ↓
Video model
  ↓
GPU
  ↓
MP4
```

---

## B. Image → Video

User uploads:

```text
bride.jpg
```

Prompt:

> The bride slowly turns toward the camera while her dupatta moves naturally in the wind.

Then:

```text
Image
 +
Motion prompt
      ↓
Video model
      ↓
GPU
      ↓
MP4
```

This is particularly useful for the **AI reels** you've been creating.

---

# 3. Model strategy

Don't depend on one model forever.

Build a **model abstraction layer**.

```text
                    Video Engine
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
      Wan             LTX-Video       Future Models
        │                │
        └────────────────┴────────────────┘
                         ↓
                    MP4 output
```

### Start with

**Wan 2.x**

Use it for:

- Text → Video
- Image → Video
- cinematic scenes
- realistic motion
- short clips

Then experiment with other open models as they become practical.

This prevents your application from being tied to a single model.

---

# 4. Recommended technology stack

## Frontend

```text
Next.js
TypeScript
Tailwind CSS
shadcn/ui
```

UI:

```text
Mode
├── Text → Video
└── Image → Video

Prompt
Image Upload

Duration
Resolution
Aspect Ratio
Seed
Motion

Generate
```

---

# 5. Backend

Use:

```text
Python
FastAPI
```

Why Python?

Because the AI ecosystem is overwhelmingly Python-based.

Your architecture:

```text
Next.js
   │
   │ REST API
   ↓
FastAPI
   │
   ├── Authentication
   ├── Generation API
   ├── Job management
   ├── File management
   └── Model management
```

---

# 6. AI inference layer

Don't initially write the entire diffusion pipeline yourself.

Use:

## ComfyUI

Think of ComfyUI as your:

> **AI workflow engine**

Architecture:

```text
FastAPI
   ↓
ComfyUI API
   ↓
Workflow
   ↓
Wan
   ↓
GPU
   ↓
Video
```

This makes experimentation dramatically easier.

Once you understand the workflows, you can later replace parts with your own Python inference service.

---

# 7. GPU architecture

This is the most important part.

If you want **unlimited generation**, the GPU must be yours or under your control.

### Option A — Your own PC

```text
Windows/Linux
     ↓
NVIDIA GPU
     ↓
CUDA
     ↓
PyTorch
     ↓
ComfyUI
     ↓
Wan
```

This is the closest to:

> **₹0 per generated video**

You still pay electricity.

---

### Option B — Your own dedicated GPU server

Better for production.

```text
Internet
   ↓
Your Web App
   ↓
API Server
   ↓
Redis Queue
   ↓
GPU Worker
   ↓
Wan
```

You pay for the server/GPU infrastructure, but **not per video/API credit**.

---

### Option C — Free cloud notebooks ✅ (chosen for Phases 1–2)

Free notebook services give you a real NVIDIA GPU in the cloud at no cost.

This is how this project starts, because the laptop GPU is too small (see section 8).

| Service | GPU | Limits (check current) | Use |
|---|---|---|---|
| **Kaggle** (primary) | T4 16 GB (or 2× T4 / P100) | ~30 GPU hours/week, sessions up to ~12 h, ~30 GB RAM | Phase 1–2 experiments |
| **Google Colab** (backup) | T4 16 GB | Session limits vary, ~12 GB RAM, GPU not always available | When Kaggle quota runs out |

Tips:

- Kaggle needs phone verification before you can turn on GPU and internet.
- Sessions are temporary: models download again every session unless you save them as a Kaggle Dataset.
- Download generated MP4s before the session ends.

But I **wouldn't design the final platform around free notebook sessions** because sessions can have:

- time limits
- GPU availability limits
- disconnects
- storage limits
- changing policies

Use them for learning/testing (Phases 1–2), not as the foundation of an "unlimited" service.

---

# 8. Hardware recommendation

### Current hardware (checked 2026-10-08)

| | This laptop | Needed for Wan |
|---|---|---|
| GPU | GTX 1650, 4 GB VRAM | 8 GB minimum, 16–24 GB for good results |
| RAM | 20 GB usable | OK |
| CPU | Ryzen 5 5600H | OK |
| Disk | C: is tight, D: has ~143 GB free | Keep models and projects on D: |

The laptop GPU is too small for video generation (and a laptop GPU can't be upgraded), so:

```text
Phases 1–2  (AI model testing)              → Kaggle / Colab free GPU
Phases 3–7  (API, queue, website, DB, tools) → this laptop, with a fake generator
Real GPU for the finished app               → decide after Phase 5
```

For the finished app, choose between:

- renting a cloud GPU by the hour (RunPod, Vast.ai, E2E Networks)
- a serverless GPU that charges only while a video is generating (Modal, RunPod Serverless)
- buying a desktop GPU with 16–24 GB VRAM

For your first prototype, don't immediately buy an expensive GPU.

Start with the smallest model/workflow that your available GPU can handle.

For example:

```text
Prototype
↓
Wan smaller model
↓
Lower resolution
↓
Short clips
↓
8GB+ VRAM target
```

Then move toward:

```text
16GB VRAM
      ↓
Better resolution
      ↓
Longer videos
      ↓
More generation options
```

And eventually:

```text
24GB+ VRAM
      ↓
Production-quality workloads
```

The exact GPU choice should be based on the model version and resolution you settle on, because VRAM requirements vary considerably.

---

# 9. Backend architecture

Don't make your API wait until a video finishes.

This is a common beginner mistake.

Instead:

```text
POST /generate
       ↓
Create Job
       ↓
Return job_id
       ↓
Queue
       ↓
GPU Worker
       ↓
Generate
       ↓
Save MP4
       ↓
Update Job
```

Example:

```json
{
  "job_id": "abc123",
  "status": "queued"
}
```

Frontend then checks:

```text
GET /jobs/abc123
```

Response:

```json
{
  "status": "processing",
  "progress": 64
}
```

Eventually:

```json
{
  "status": "completed",
  "video_url": "/videos/abc123.mp4"
}
```

---

# 10. Use Redis

For unlimited/self-hosted generation, a queue is extremely important.

Use:

```text
Redis
+
Celery / RQ / equivalent worker system
```

Architecture:

```text
               FastAPI
                  │
                  ↓
                Redis
                  │
          ┌───────┴───────┐
          ↓               ↓
      GPU Worker 1    GPU Worker 2
          │               │
        Wan             Wan
          │               │
          └───────┬───────┘
                  ↓
                MP4
```

If you have one GPU:

```text
Job 1
Job 2
Job 3
Job 4
```

They wait in the queue.

If later you have 4 GPUs:

```text
           Redis
             │
      ┌──────┼──────┬──────┐
      ↓      ↓      ↓      ↓
    GPU 1  GPU 2  GPU 3  GPU 4
```

Now you can generate multiple videos simultaneously.

---

# 11. Database

Use PostgreSQL.

Your database could contain:

### users

```text
id
name
email
password_hash
created_at
```

### generations

```text
id
user_id
type
prompt
negative_prompt
model
status
progress
seed
duration
resolution
created_at
completed_at
```

### generated_files

```text
id
generation_id
file_path
file_size
duration
created_at
```

### models

```text
id
name
version
type
enabled
```

---

# 12. Storage

For development:

```text
/mnt/videos
/mnt/images
/mnt/models
```

For production:

```text
Object Storage
     ↓
videos/
images/
thumbnails/
```

You could later use S3-compatible storage.

---

# 13. Video processing

Install:

## FFmpeg

You'll need it for:

- MP4 conversion
- resizing
- FPS changes
- extracting frames
- thumbnails
- merging clips
- audio
- subtitles
- video compression

Example:

```text
AI generated frames
        ↓
      FFmpeg
        ↓
       MP4
```

---

# 14. Your API design

Build these APIs.

### Authentication

```text
POST /auth/register
POST /auth/login
POST /auth/logout
```

### Generation

```text
POST /generate/text
POST /generate/image
```

### Jobs

```text
GET /jobs
GET /jobs/{id}
POST /jobs/{id}/cancel
```

### Videos

```text
GET /videos
GET /videos/{id}
DELETE /videos/{id}
```

### Models

```text
GET /models
GET /models/{id}
```

---

# 15. Text-to-video pipeline

Detailed pipeline:

```text
User prompt
     ↓
Prompt validation
     ↓
Prompt enhancement
     ↓
Model selection
     ↓
Workflow creation
     ↓
ComfyUI
     ↓
Wan
     ↓
GPU
     ↓
Frames
     ↓
VAE decode
     ↓
FFmpeg
     ↓
MP4
     ↓
Thumbnail
     ↓
Storage
     ↓
Frontend
```

---

# 16. Image-to-video pipeline

```text
Upload image
      ↓
Validate image
      ↓
Resize/preprocess
      ↓
User motion prompt
      ↓
Wan image-to-video workflow
      ↓
GPU
      ↓
Frames
      ↓
FFmpeg
      ↓
MP4
      ↓
Storage
```

---

# 17. Add prompt enhancement

This can become another AI component.

User writes:

> girl walking in rain

Your prompt enhancer converts it into something like:

```text
A cinematic young woman walking slowly through a rain-soaked
city street at night, realistic wet clothing, soft reflections
on pavement, shallow depth of field, subtle camera tracking,
natural human movement, volumetric lighting, cinematic
composition, realistic photography.
```

You can use a **local open-source LLM** for this.

That means:

```text
No paid OpenAI API
No paid Claude API
```

Everything stays self-hosted.

---

# 18. Add image generation later

Eventually:

```text
AI Video Studio
│
├── Text → Video
├── Image → Video
├── Text → Image
├── Image → Image
└── Video → Video
```

For image generation, you could integrate suitable open models such as FLUX-family or SDXL-family models depending on hardware and licensing.

---

# 19. Add AI voice

Later:

```text
Text
 ↓
Local TTS model
 ↓
Voice
 ↓
MP3/WAV
```

Potential open-source options include:

- Piper
- Kokoro
- other suitable local TTS models

Then:

```text
Video
 +
Voice
 ↓
FFmpeg
 ↓
Final video
```

---

# 20. Add subtitles

Pipeline:

```text
Audio
 ↓
Whisper
 ↓
Transcript
 ↓
SRT
 ↓
FFmpeg
 ↓
Burned subtitles
```

Then users can choose:

```text
☑ Auto subtitles
```

---

# 21. Build an AI Reel Generator

This would be a **very good advanced version for you**.

User enters:

> Create a 30-second Instagram reel about chiropractic therapy.

System:

```text
Topic
 ↓
Local LLM
 ↓
Script
 ↓
Scene breakdown
 ↓
Scene 1 → Video
Scene 2 → Video
Scene 3 → Video
Scene 4 → Video
 ↓
TTS
 ↓
Music
 ↓
Subtitles
 ↓
FFmpeg
 ↓
9:16 Reel
```

Output:

```text
30-second Instagram Reel
1080 × 1920
```

This is much more impressive than simply cloning a text-to-video interface.

---

# 22. Frontend pages

Build these pages.

```text
/
├── Landing Page
│
├── /login
├── /register
│
├── /studio
│
│   ├── Text → Video
│   ├── Image → Video
│   └── Video History
│
├── /library
│
├── /settings
│
└── /admin
```

---

# 23. Studio UI

I'd make the main studio look approximately like:

```text
┌──────────────────────────────────────────────┐
│ AI VIDEO STUDIO                    Profile   │
├──────────────────────────────────────────────┤
│                                              │
│  CREATE                                      │
│                                              │
│  [ Text → Video ] [ Image → Video ]          │
│                                              │
│  Prompt                                      │
│  ┌────────────────────────────────────────┐  │
│  │ Describe your video...                 │  │
│  │                                        │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  Upload Image                                │
│  ┌────────────────────────────────────────┐  │
│  │      Drag image here                   │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  Model       Wan                             │
│  Duration    5 sec                           │
│  Resolution  480p                            │
│  Ratio       9:16                            │
│                                              │
│           [ Generate Video ]                 │
│                                              │
├──────────────────────────────────────────────┤
│  GENERATION                                  │
│                                              │
│  ████████████████░░░░░░ 72%                  │
│                                              │
│  Your video is being generated...            │
└──────────────────────────────────────────────┘
```

---

# 24. Project folder

I'd use:

```text
ai-video-studio/
│
├── frontend/
│   ├── app/
│   ├── components/
│   ├── hooks/
│   ├── lib/
│   └── types/
│
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── api/
│   │   ├── models/
│   │   ├── schemas/
│   │   ├── services/
│   │   ├── workers/
│   │   └── utils/
│   │
│   └── requirements.txt
│
├── ai/
│   ├── models/
│   ├── workflows/
│   │   ├── text_to_video/
│   │   └── image_to_video/
│   ├── inference/
│   └── preprocessing/
│
├── storage/
│   ├── images/
│   ├── videos/
│   └── thumbnails/
│
├── docker/
│
├── docker-compose.yml
│
└── README.md
```

---

# 25. Docker architecture

Eventually:

```text
docker-compose
│
├── frontend
│
├── backend
│
├── postgres
│
├── redis
│
├── comfyui
│
├── gpu-worker
│
└── nginx
```

Development:

```text
Next.js
   ↓
FastAPI
   ↓
Redis
   ↓
ComfyUI
   ↓
GPU
```

---

# 26. MLOps architecture

Since you're learning MLOps, make this project teach you real MLOps.

Eventually:

```text
                    GitHub
                       │
                       ↓
                    CI/CD
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
          Backend             Frontend
             │
             ↓
        Docker Image
             │
             ↓
       GPU Server
             │
             ↓
        Model Version
             │
             ↓
       Generation Queue
             │
             ↓
          Metrics
```

Track:

```text
GPU utilization
VRAM usage
generation time
queue length
failed generations
model version
video resolution
storage usage
```

---

# 27. Important: "Unlimited" architecture

Suppose users generate:

```text
User A → 10 videos
User B → 50 videos
User C → 100 videos
```

You don't need to limit them artificially.

Your queue handles it:

```text
                 Redis
                   │
       ┌───────────┼────────────┐
       ↓           ↓            ↓
     Job 1       Job 2        Job 3
       ↓           ↓            ↓
              GPU Worker
                   │
              Generate
                   │
                  MP4
```

If one video takes 5 minutes, the next waits.

That's still unlimited.

The limitation becomes:

> **How many videos can your hardware generate per hour/day?**

not:

> **How many credits did the API provider give me?**

---

# 28. Phase-by-phase development

Don't try to build everything at once.

## 🟢 Phase 1 — AI model testing

Learn:

```text
Python
PyTorch
CUDA
Diffusion
ComfyUI
Wan
FFmpeg
```

Goal:

```text
Prompt
 ↓
Video
```

**No frontend yet.**

Where: **Kaggle free GPU** (Colab as backup). Start with the small Wan text-to-video model (Wan 2.1 T2V 1.3B, 480p), which fits easily in 16 GB.

---

# 🟢 Phase 2 — Image → Video

Build:

```text
Image
 +
Prompt
 ↓
Wan
 ↓
MP4
```

Goal:

Generate a stable 5-second clip from an image.

Where: **Kaggle free GPU**, with **Wan 2.2 TI2V 5B**: one model for both text→video and image→video, 10 GB, so it fits a 16 GB T4 at full quality. The 14B image-to-video models are 14–33 GB and far too slow on a T4. Notebook: `ai/notebooks/phase2_image_to_video_kaggle.ipynb`.

---

# 🟢 Phase 3 — FastAPI

Create:

```text
POST /generate/text
POST /generate/image
GET /jobs/{id}
```

Goal:

Control your AI engine through HTTP.

Where: **this laptop**, from here through Phase 7. Use a fake generator that returns a sample MP4, so the API, queue, website and database can be built without a GPU. Test real generation on a notebook or rented GPU.

---

# 🟢 Phase 4 — Queue

Add:

```text
Redis
+
Worker
```

Goal:

Multiple generation requests can wait safely.

---

# 🟢 Phase 5 — Next.js

Build:

```text
Studio
Upload
Prompt
Generate
Progress
Preview
Download
```

Goal:

A complete usable application.

Then: choose the GPU for real use (see section 8): rent by the hour, serverless, or buy a 16–24 GB GPU.

---

# 🟢 Phase 6 — Database

Add:

```text
PostgreSQL
```

Track:

```text
users
jobs
videos
models
settings
```

---

# 🟢 Phase 7 — Video tools

Add:

```text
Resize
Crop
FPS
Duration
Aspect ratio
Thumbnail
Video compression
```

---

# 🟢 Phase 8 — AI enhancements

Add:

```text
Prompt enhancer
Negative prompt generator
Auto scene generation
Local LLM
```

---

# 🟢 Phase 9 — AI Reel Generator

Add:

```text
Topic
 ↓
Script
 ↓
Scenes
 ↓
Video clips
 ↓
Voice
 ↓
Subtitles
 ↓
Music
 ↓
Final Reel
```

This would be your **major differentiating feature**.

---

# 🟢 Phase 10 — MLOps

Add:

```text
Docker
GitHub Actions
Model versioning
GPU monitoring
Logging
Queue monitoring
Error tracking
```

---

# 29. What you should learn

Because you're already moving toward AI + cloud + MLOps, I'd learn in this order:

### AI fundamentals

```text
PyTorch
Tensors
GPU
CUDA
Diffusion models
VAE
UNet / transformer-based video architectures
Latent space
Schedulers
Attention
Quantization
```

### Generative AI

```text
Text → Image
Image → Image
Text → Video
Image → Video
Diffusion
Video diffusion
Control mechanisms
LoRA
Quantization
Model inference
```

### Backend

```text
FastAPI
REST API
WebSockets
Background jobs
Redis
Celery/RQ
PostgreSQL
```

### Frontend

```text
Next.js
TypeScript
Tailwind
shadcn
React state
File upload
Progress tracking
```

### MLOps

```text
Docker
NVIDIA Container Toolkit
GPU monitoring
Model serving
Queues
CI/CD
Logging
Metrics
AWS
```

---

# 30. Final technology stack

I'd target this:

| Layer | Technology |
|---|---|
| Frontend | Next.js |
| Language | TypeScript |
| UI | Tailwind + shadcn |
| Backend | FastAPI |
| AI | PyTorch |
| Video model | Wan |
| Workflow | ComfyUI |
| Queue | Redis |
| Workers | Python |
| Database | PostgreSQL |
| Video processing | FFmpeg |
| Image processing | Pillow |
| Speech-to-text | Whisper |
| TTS | Local open-source TTS |
| Containers | Docker |
| GPU | NVIDIA CUDA (Kaggle/Colab for learning → rented or own GPU for production) |
| Storage | Local → S3-compatible |
| CI/CD | GitHub Actions |
| Monitoring | Prometheus/Grafana later |

---

# 31. What NOT to do

Don't start with:

```text
Next.js
 ↓
API
 ↓
Kling API
```

or:

```text
Next.js
 ↓
Runway API
```

because that doesn't satisfy your actual objective.

You want:

```text
Next.js
     ↓
YOUR API
     ↓
YOUR GPU
     ↓
YOUR MODEL
     ↓
YOUR VIDEO
```

That is what gives you **self-hosted, effectively unlimited generation**.

---
# 32. The ultimate version

Eventually your project could become:

```text
╔══════════════════════════════════════════════╗
║              AI VIDEO STUDIO                 ║
╠══════════════════════════════════════════════╣
║                                              ║
║  ✨ Text → Video                            ║
║  🖼️ Image → Video                           ║
║  🎬 Video → Video                           ║
║  🎨 Text → Image                            ║
║  🎙️ AI Voice                                ║
║  📝 Auto Subtitles                          ║
║  🎵 Background Music                        ║
║  🎞️ AI Reel Generator                       ║
║                                              ║
║  ──────────────────────────────────────────  ║
║                                              ║
║       SELF-HOSTED OPEN-SOURCE AI             ║
║                                              ║
║              NO VIDEO CREDITS                ║
║              NO API LIMITS                   ║
║              NO PER-VIDEO FEES               ║
║                                              ║
╚══════════════════════════════════════════════╝
```
### The most important first milestone

Don't start with the website.
Start with:

**`Wan → ComfyUI → free Kaggle GPU → generate one 5-second video.`**

Once that works, everything else—FastAPI, Redis, Next.js, PostgreSQL, Docker, MLOps—is built around that working inference engine.

The GPU setup is decided (see sections 7 and 8): the laptop's 4 GB GPU is too small, so model testing happens on free Kaggle/Colab notebooks and the rest of the app is built on the laptop.

Next step: a Kaggle notebook that installs ComfyUI, downloads Wan 2.1 T2V 1.3B and generates the first clip.