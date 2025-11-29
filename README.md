# 🏠 Interior Design Video Generator API

A FastAPI-based REST API for generating cinematic videos from interior design images using Stable Video Diffusion.

## 🚀 Features

- **RESTful API** with async support
- **Room-specific presets** (living room, bedroom, kitchen, etc.)
- **Multiple motion styles** (subtle, moderate, dynamic, showcase)
- **Image enhancement** (lighting, contrast)
- **Background processing** with job queue
- **Progress tracking** for video generation
- **Automatic cleanup** of old files
- **CORS enabled** for web applications
- **Health check** endpoint
- **Docker support**

## 📋 Requirements

- Python 3.10+
- CUDA-compatible GPU (recommended)
- NVIDIA CUDA 11.8+
- 8GB+ VRAM for optimal performance

## 🔧 Installation

### Method 1: Local Installation

```bash
# Clone or create project directory
mkdir interior-video-api
cd interior-video-api

# Install dependencies
pip install -r requirements.txt

# Run the server
python main.py
```

### Method 2: Docker

```bash
# Build Docker image
docker build -t interior-video-api .

# Run container with GPU support
docker run --gpus all -p 8000:8000 interior-video-api
```

## 🎯 API Endpoints

### Base URL
```
http://localhost:8000
```

### 1. **Health Check**
```http
GET /health
```

**Response:**
```json
{
  "status": "healthy",
  "device": "cuda",
  "model_loaded": true,
  "timestamp": "2024-11-29T10:30:00"
}
```

---

### 2. **Generate Video**
```http
POST /api/v1/generate
```

**Parameters:**
- `file` (required): Image file (JPG, JPEG, PNG)
- `room_type` (optional): Room type
  - Options: `living_room`, `bedroom`, `kitchen`, `bathroom`, `office`, `dining_room`, `exterior`
  - Default: `living_room`
- `motion_style` (optional): Animation style
  - Options: `subtle`, `moderate`, `dynamic`, `showcase`
  - Default: `moderate`
- `enhance_lighting` (optional): Enhance lighting (boolean, default: true)
- `adjust_contrast` (optional): Adjust contrast (boolean, default: true)

**Example (cURL):**
```bash
curl -X POST "http://localhost:8000/api/v1/generate" \
  -F "file=@interior.jpg" \
  -F "room_type=living_room" \
  -F "motion_style=moderate"
```

**Example (Python):**
```python
import requests

with open("interior.jpg", "rb") as f:
    response = requests.post(
        "http://localhost:8000/api/v1/generate",
        files={"file": f},
        data={
            "room_type": "living_room",
            "motion_style": "moderate"
        }
    )

job_id = response.json()["job_id"]
```

**Response:**
```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "queued",
  "message": "Video generation started. Use job_id to check status."
}
```

---

### 3. **Check Job Status**
```http
GET /api/v1/status/{job_id}
```

**Response:**
```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "completed",
  "progress": 100,
  "message": "Video generated successfully!",
  "video_url": "/api/v1/download/550e8400-e29b-41d4-a716-446655440000",
  "created_at": "2024-11-29T10:30:00",
  "completed_at": "2024-11-29T10:31:30",
  "duration": 2.3,
  "frames": 16,
  "file_size_mb": 1.5
}
```

**Status Values:**
- `queued`: Job is waiting to be processed
- `processing`: Video is being generated
- `completed`: Video is ready for download
- `failed`: An error occurred

---

### 4. **Download Video**
```http
GET /api/v1/download/{job_id}
```

**Example:**
```bash
curl -O "http://localhost:8000/api/v1/download/550e8400-e29b-41d4-a716-446655440000"
```

**Response:**
- Content-Type: `video/mp4`
- File download

---

### 5. **List All Jobs**
```http
GET /api/v1/jobs
```

**Response:**
```json
{
  "jobs": [
    {
      "job_id": "550e8400-e29b-41d4-a716-446655440000",
      "status": "completed",
      "progress": 100,
      "created_at": "2024-11-29T10:30:00"
    }
  ]
}
```

---

### 6. **Delete Job**
```http
DELETE /api/v1/jobs/{job_id}
```

**Response:**
```json
{
  "message": "Job deleted successfully"
}
```

## 📊 Room Types & Settings

| Room Type | Motion Intensity | Frames | FPS | Best For |
|-----------|------------------|--------|-----|----------|
| living_room | 85 | 16 | 7 | General showcases |
| bedroom | 70 | 14 | 6 | Calm presentations |
| kitchen | 95 | 18 | 8 | Dynamic tours |
| bathroom | 75 | 14 | 6 | Elegant reveals |
| office | 80 | 16 | 7 | Professional videos |
| dining_room | 90 | 16 | 7 | Showcase videos |
| exterior | 100 | 20 | 8 | Wide shots |

## 🎨 Motion Styles

| Style | Description | Adjustment | Best Use Case |
|-------|-------------|------------|---------------|
| subtle | Gentle pan | -15 | Client presentations |
| moderate | Balanced | 0 | General use |
| dynamic | Pronounced | +20 | Social media |
| showcase | Virtual tour | +35 | Portfolio pieces |

## 🧪 Testing

Use the included test client:

```bash
# Test with a single image
python test_client.py interior.jpg

# Test with custom settings
python test_client.py bedroom.jpg --room-type bedroom --motion-style subtle
```

## 📝 Complete Workflow Example

```python
import requests
import time

# 1. Upload and start generation
with open("living_room.jpg", "rb") as f:
    response = requests.post(
        "http://localhost:8000/api/v1/generate",
        files={"file": f},
        data={"room_type": "living_room", "motion_style": "showcase"}
    )

job_id = response.json()["job_id"]
print(f"Job ID: {job_id}")

# 2. Poll for completion
while True:
    status = requests.get(f"http://localhost:8000/api/v1/status/{job_id}").json()
    print(f"Status: {status['status']} - {status['progress']}%")
    
    if status['status'] == 'completed':
        break
    elif status['status'] == 'failed':
        print(f"Error: {status['message']}")
        exit(1)
    
    time.sleep(5)

# 3. Download video
video = requests.get(f"http://localhost:8000/api/v1/download/{job_id}")
with open("output.mp4", "wb") as f:
    f.write(video.content)

print("Video saved to output.mp4")
```

## 🌐 Interactive API Documentation

FastAPI provides automatic interactive documentation:

- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc

## ⚙️ Configuration

Edit `main.py` to customize:

```python
class Config:
    UPLOAD_DIR = "uploads"
    OUTPUT_DIR = "outputs"
    MAX_FILE_SIZE = 10 * 1024 * 1024  # 10MB
    ALLOWED_EXTENSIONS = {".jpg", ".jpeg", ".png"}
    HUGGINGFACE_TOKEN = None  # Add your token if needed
```

## 🐛 Troubleshooting

### Out of Memory Error
- Reduce image size
- Use fewer frames
- Lower `decode_chunk_size` in generator

### Slow Generation
- Ensure GPU is being used (check `/health` endpoint)
- Close other GPU-intensive applications
- Use smaller room types (fewer frames)

### Model Download Issues
- Set `HUGGINGFACE_TOKEN` in Config
- Check internet connection
- Verify disk space (~5GB needed)

## 📊 Performance

| GPU | Resolution | Processing Time | VRAM Usage |
|-----|------------|-----------------|------------|
| RTX 3060 | 384x384 | ~30-40s | ~6GB |
| RTX 3080 | 384x384 | ~20-30s | ~7GB |
| RTX 4090 | 512x512 | ~15-20s | ~8GB |

## 🔒 Security Notes

- Add authentication for production use
- Implement rate limiting
- Validate file types strictly
- Set up proper CORS policies
- Use environment variables for secrets

## 📄 License

This project uses Stable Video Diffusion from Stability AI.
Check their license terms before commercial use.

## 🤝 Support

For issues or questions:
1. Check the `/health` endpoint
2. Review logs in terminal
3. Test with the provided test client
4. Check GPU memory with `nvidia-smi`

## 🎉 Credits

Built with:
- FastAPI
- Stable Video Diffusion (Stability AI)
- PyTorch
- Diffusers (Hugging Face)
