# 🚗 Multi-Camera Vehicle Intelligence System

A production-grade AI-powered vehicle surveillance dashboard with real-time detection,
tracking, OCR license plate reading, and an interactive map UI.

---

## Architecture

```
Video → YOLOv8 → DeepSORT → EasyOCR → FastAPI → React + Leaflet
```

```
be_project/
├── backend/
│   ├── app/
│   │   ├── main.py                  ← FastAPI entry point
│   │   ├── api/routes/
│   │   │   ├── cameras.py           ← /api/cameras
│   │   │   └── detections.py        ← /api/detections, /api/stats, /ws/live
│   │   ├── models/schemas.py        ← Pydantic schemas
│   │   └── services/
│   │       ├── camera_service.py    ← Pipeline lifecycle
│   │       └── detection_service.py ← DB queries & analytics
│   ├── pipelines/
│   │   ├── ingestion.py             ← Video frame reader (file + RTSP)
│   │   ├── detection.py             ← YOLOv8 vehicle detection
│   │   ├── tracking.py              ← DeepSORT multi-object tracking
│   │   ├── ocr.py                   ← EasyOCR + Indian plate correction
│   │   ├── counting.py              ← Line-crossing counter
│   │   └── pipeline.py              ← Orchestrator (all stages wired)
│   ├── database/
│   │   ├── models.py                ← SQLAlchemy ORM models
│   │   └── session.py               ← Async session factory
│   ├── config/
│   │   ├── settings.py              ← Pydantic settings (from .env)
│   │   └── cameras.json             ← Camera definitions
│   ├── .env                         ← Environment variables
│   └── requirements.txt
├── frontend/
│   └── src/
│       ├── components/
│       │   ├── Map/VehicleMap.jsx   ← Leaflet map
│       │   ├── Search/SearchBar.jsx
│       │   ├── Search/SearchResults.jsx
│       │   ├── Camera/CameraPanel.jsx
│       │   └── Stats/StatsCards.jsx
│       ├── pages/Dashboard.jsx      ← Main page
│       └── services/api.js          ← Axios API client
└── tools/
    └── generate_test_video.py       ← Synthetic test video generator
```

---

## Prerequisites

| Tool | Version |
|---|---|
| Python | 3.10 or 3.11 |
| Node.js | 18+ |
| FFmpeg | any |
| CUDA (optional) | 11.8+ |

---

## Quick Start

### Step 1 — Clone & enter project

```bash
git clone https://github.com/YOUR_USERNAME/be_project.git
cd be_project
```

### Step 2 — Backend setup

```bash
# Make scripts executable
chmod +x setup_backend.sh setup_frontend.sh

# Install Python dependencies
./setup_backend.sh

# Activate virtual environment
cd backend
source venv/bin/activate   # Linux/macOS
# OR
venv\Scripts\activate      # Windows
```

### Step 3 — Generate test videos (if you don't have real footage)

```bash
cd backend
python ../tools/generate_test_video.py
# Creates: backend/videos/sample1.mp4, sample2.mp4, sample3.mp4
```

### Step 4 — Start the backend

```bash
cd backend
source venv/bin/activate
python -m app.main
```

Backend runs at: **http://localhost:8000**
API docs at:     **http://localhost:8000/docs**

### Step 5 — Frontend setup

```bash
# In a new terminal
cd be_project
./setup_frontend.sh

cd frontend
npm run dev
```

Dashboard at: **http://localhost:5173**

---

## Configuration

### Add your cameras — `backend/config/cameras.json`

```json
[
  {
    "id": "cam_001",
    "name": "My Camera",
    "location": "Main Gate, City",
    "lat": 19.9975,
    "lng": 73.7898,
    "source": "videos/my_video.mp4",
    "type": "file",
    "active": true,
    "counting_line": [0.5, 0.0, 0.5, 1.0]
  }
]
```

**For RTSP streams**, set:
```json
"source": "rtsp://admin:password@192.168.1.100:554/stream",
"type": "rtsp"
```

### Tune performance — `backend/.env`

```env
FRAME_SKIP=2              # Process every 2nd frame (1 = all)
YOLO_MODEL=yolov8n.pt     # n=fastest, s/m=more accurate
YOLO_DEVICE=cpu           # "cpu" or "0" for GPU
CONFIDENCE_THRESHOLD=0.45
OCR_GPU=false             # true if NVIDIA GPU available
```

### Use PostgreSQL (production)

```env
DATABASE_URL=postgresql+asyncpg://user:password@localhost:5432/vehicle_intel
```

Then: `pip install asyncpg`

---

## API Reference

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/cameras/` | All cameras + pipeline status |
| GET | `/api/cameras/{id}` | Single camera details |
| POST | `/api/cameras/pipeline` | Start/stop pipeline |
| GET | `/api/detections?plate=MH12AB` | Search by plate → route |
| GET | `/api/camera/{id}/vehicles` | Camera analytics |
| GET | `/api/stats` | Global dashboard stats |
| WS | `/ws/live` | Real-time detection stream |

---

## GPU Acceleration

If you have an NVIDIA GPU:

```bash
# Install CUDA-enabled PyTorch (replace cu118 with your CUDA version)
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu118

# Update .env
YOLO_DEVICE=0
OCR_GPU=true
```

---

## Features

### Phase 1 — Vehicle Tracking
- Search any plate number in the sidebar
- See all detections with timestamps
- Click **"Show Route on Map"** → polyline across cameras

### Phase 2 — Camera Analytics
- Click any camera marker on the map
- See total vehicle count, type breakdown (pie chart)
- Recent plates, time window selector (1h / 6h / 24h / 72h)
- Start/Stop pipeline per camera

### Real-time
- WebSocket live feed updates stats every 2 seconds
- Fallback to 5-second HTTP polling when WS unavailable

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `ModuleNotFoundError: ultralytics` | `pip install ultralytics` in venv |
| YOLO model download fails | Run `from ultralytics import YOLO; YOLO('yolov8n.pt')` once |
| DeepSORT import error | `pip install deep-sort-realtime` |
| EasyOCR slow | Set `OCR_GPU=true` in .env if GPU available |
| Video not found | Ensure `source` path in cameras.json is relative to `backend/` |
| CORS error in browser | Check `CORS_ORIGINS` in .env matches your frontend URL |

---

## License

MIT — Educational / Research use. For production deployments, ensure compliance with local surveillance laws.
