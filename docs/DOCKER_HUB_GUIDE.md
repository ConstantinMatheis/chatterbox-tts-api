# 🐳 Docker Hub Usage Guide

This guide covers how to use the official Chatterbox TTS API Docker images from Docker Hub.

## 🚀 Quick Start

### Option 1: Default Setup (Recommended)

```bash
# Pull and run the latest image (CUDA-compatible, works on both GPU and CPU)
docker run -d \
  --name chatterbox-tts \
  -p 5123:5123 \
  -v ./voice-sample.mp3:/app/voice-sample.mp3:ro \
  -v chatterbox-models:/cache \
  -e EXAGGERATION=0.7 \
  travisvn/chatterbox-tts-api:latest

# Test the API
curl -X POST http://localhost:5123/v1/audio/speech \
  -F "input=Hello from Docker Hub!" \
  --output test.wav
```

### Option 2: CPU-Only Setup

```bash
# For CPU-only environments (smaller image)
docker run -d \
  --name chatterbox-tts \
  -p 5123:5123 \
  -v ./voice-sample.mp3:/app/voice-sample.mp3:ro \
  -v chatterbox-models:/cache \
  travisvn/chatterbox-tts-api:cpu
```

### Option 3: GPU Setup

```bash
# For explicit GPU usage
docker run -d \
  --name chatterbox-tts \
  --gpus all \
  -p 5123:5123 \
  -v ./voice-sample.mp3:/app/voice-sample.mp3:ro \
  -v chatterbox-models:/cache \
  -e DEVICE=cuda \
  travisvn/chatterbox-tts-api:gpu
```

## 📦 Available Images

| Image Tag | Size   | Description                          | Use Case                       |
| --------- | ------ | ------------------------------------ | ------------------------------ |
| `latest`  | ~4.5GB | CUDA-compatible, works on CPU or GPU | **Recommended for most users** |
| `cpu`     | ~2.8GB | CPU-only, smaller image              | CPU-only environments          |
| `gpu`     | ~4.5GB | Same as latest, clearer GPU intent   | Explicit GPU usage             |
| `v1.0.0`  | ~4.5GB | Specific version tag                 | Production deployments         |

### Which Image Should I Use?

- **🎯 Most users**: Use `latest` - it automatically detects your hardware
- **💻 CPU-only systems**: Use `cpu` - smaller download, CPU-optimized
- **🎮 GPU systems**: Use `gpu` or `latest` - both work the same
- **🏢 Production**: Use version tags like `v1.0.0` for stability

## ⚙️ Configuration

### Environment Variables

All images support the same environment variables:

```bash
docker run -d \
  --name chatterbox-tts \
  -p 5123:5123 \
  -e PORT=5123 \
  -e EXAGGERATION=0.7 \
  -e CFG_WEIGHT=0.4 \
  -e TEMPERATURE=0.9 \
  -e DEVICE=auto \
  -e MAX_CHUNK_LENGTH=300 \
  -v ./voice-sample.mp3:/app/voice-sample.mp3:ro \
  -v chatterbox-models:/cache \
  travisvn/chatterbox-tts-api:latest
```

### Key Environment Variables

| Variable           | Default | Description                    |
| ------------------ | ------- | ------------------------------ |
| `PORT`             | `5123`  | API server port                |
| `EXAGGERATION`     | `0.5`   | Emotion intensity (0.25-2.0)   |
| `CFG_WEIGHT`       | `0.5`   | Speech pace control (0.0-1.0)  |
| `TEMPERATURE`      | `0.8`   | Sampling randomness (0.05-5.0) |
| `DEVICE`           | `auto`  | Device: auto/cuda/cpu          |
| `MAX_CHUNK_LENGTH` | `280`   | Max characters per chunk       |

## 📁 Volume Mounting

### Required Volumes

```bash
# Model cache (persistent storage for downloaded models)
-v chatterbox-models:/cache

# Voice sample (your voice for cloning)
-v ./your-voice.mp3:/app/voice-sample.mp3:ro
```

### Optional Volumes

```bash
# Multiple voice samples
-v ./voice-samples:/app/voice-samples:ro

# Custom configuration
-v ./custom.env:/app/.env:ro
```

## 🚀 Docker Compose Examples

### Basic Setup

```yaml
version: '3.8'
services:
  chatterbox-tts:
    image: travisvn/chatterbox-tts-api:latest
    container_name: chatterbox-tts
    ports:
      - '5123:5123'
    environment:
      - EXAGGERATION=0.7
      - CFG_WEIGHT=0.4
    volumes:
      - ./voice-sample.mp3:/app/voice-sample.mp3:ro
      - chatterbox-models:/cache
    restart: unless-stopped

volumes:
  chatterbox-models:
```

### GPU-Enabled Setup

```yaml
version: '3.8'
services:
  chatterbox-tts:
    image: travisvn/chatterbox-tts-api:latest
    container_name: chatterbox-tts
    ports:
      - '5123:5123'
    environment:
      - DEVICE=cuda
    volumes:
      - ./voice-sample.mp3:/app/voice-sample.mp3:ro
      - chatterbox-models:/cache
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
    restart: unless-stopped

volumes:
  chatterbox-models:
```

### Multi-Instance Setup (Load Balancing)

```yaml
version: '3.8'
services:
  chatterbox-tts-1:
    image: travisvn/chatterbox-tts-api:latest
    ports:
      - '5123:5123'
    volumes:
      - ./voice-sample.mp3:/app/voice-sample.mp3:ro
      - chatterbox-models-1:/cache

  chatterbox-tts-2:
    image: travisvn/chatterbox-tts-api:latest
    ports:
      - '5124:5123'
    volumes:
      - ./voice-sample.mp3:/app/voice-sample.mp3:ro
      - chatterbox-models-2:/cache

  nginx:
    image: nginx:alpine
    ports:
      - '80:80'
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - chatterbox-tts-1
      - chatterbox-tts-2

volumes:
  chatterbox-models-1:
  chatterbox-models-2:
```

## 🎤 Voice Sample Setup

### Default Voice Sample

The image includes a default voice sample, but you should replace it with your own:

```bash
# Replace with your voice
docker run -d \
  --name chatterbox-tts \
  -p 5123:5123 \
  -v /path/to/your/voice.mp3:/app/voice-sample.mp3:ro \
  -v chatterbox-models:/cache \
  travisvn/chatterbox-tts-api:latest
```

### Multiple Voice Samples

```bash
# Mount a directory of voice samples
docker run -d \
  --name chatterbox-tts \
  -p 5123:5123 \
  -v ./voice-samples:/app/voice-samples:ro \
  -v chatterbox-models:/cache \
  -e VOICE_SAMPLE_PATH=/app/voice-samples/my-voice.mp3 \
  travisvn/chatterbox-tts-api:latest
```

### Voice Sample Requirements

- **Formats**: MP3, WAV, FLAC, M4A, OGG
- **Duration**: 10-30 seconds of clear speech
- **Quality**: Higher quality = better voice cloning
- **Content**: Avoid background noise

## 🔧 Usage Examples

### Basic API Usage

```bash
# Simple text-to-speech
curl -X POST http://localhost:5123/v1/audio/speech \
  -F "input=Hello world!" \
  --output hello.wav

# With custom parameters
curl -X POST http://localhost:5123/v1/audio/speech \
  -F "input=This is more expressive!" \
  -F "exaggeration=0.8" \
  -F "temperature=1.0" \
  --output expressive.wav

# With custom voice upload
curl -X POST http://localhost:5123/v1/audio/speech \
  -F "input=Using my uploaded voice!" \
  -F "voice_file=@my-voice.mp3" \
  --output custom.wav
```

### Python Client

```python
import requests

# Basic usage
response = requests.post(
    "http://localhost:5123/v1/audio/speech",
    data={"input": "Hello from Python!"}
)
with open("output.wav", "wb") as f:
    f.write(response.content)

# With custom voice
with open("my-voice.mp3", "rb") as voice_file:
    response = requests.post(
        "http://localhost:5123/v1/audio/speech",
        data={
            "input": "Custom voice example!",
            "exaggeration": 0.8
        },
        files={"voice_file": voice_file}
    )
```

## 📊 Monitoring & Health Checks

### Health Check

```bash
# Check if the API is ready
curl http://localhost:5123/health

# Get configuration
curl http://localhost:5123/config

# View API documentation
curl http://localhost:5123/docs
```

### Container Monitoring

```bash
# View logs
docker logs chatterbox-tts -f

# Monitor resource usage
docker stats chatterbox-tts

# Check container health
docker inspect chatterbox-tts --format='{{.State.Health.Status}}'
```

## 🚨 Troubleshooting

### Common Issues

**Model download takes too long:**

```bash
# Pre-download models by starting container and waiting
docker run --name temp-download \
  -v chatterbox-models:/cache \
  travisvn/chatterbox-tts-api:latest \
  python -c "from app.core.tts_model import initialize_model; import asyncio; asyncio.run(initialize_model())"
```

**Out of memory:**

```bash
# Use CPU version or reduce chunk size
docker run -d \
  -e DEVICE=cpu \
  -e MAX_CHUNK_LENGTH=200 \
  travisvn/chatterbox-tts-api:cpu
```

**Voice sample not found:**

```bash
# Check file exists and permissions
ls -la ./voice-sample.mp3

# Test with absolute path
docker run -v /absolute/path/to/voice.mp3:/app/voice-sample.mp3:ro ...
```

**GPU not detected:**

```bash
# Verify NVIDIA Docker runtime
docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu20.04 nvidia-smi

# Check container GPU access
docker exec chatterbox-tts python -c "import torch; print(torch.cuda.is_available())"
```

## 🔒 Security Best Practices

### Production Security

```bash
# Run with limited resources
docker run -d \
  --memory=4g \
  --cpus=2 \
  --user 1000:1000 \
  travisvn/chatterbox-tts-api:latest

# Use Docker secrets for sensitive data
docker swarm init
echo "sensitive-voice-data" | docker secret create voice_sample -
```

### Network Security

```bash
# Bind to localhost only
docker run -d \
  -p 127.0.0.1:5123:5123 \
  travisvn/chatterbox-tts-api:latest

# Use custom network
docker network create chatterbox-net
docker run -d \
  --network chatterbox-net \
  travisvn/chatterbox-tts-api:latest
```

## 📈 Performance Optimization

### For High Volume

```bash
# Scale horizontally
docker service create \
  --name chatterbox-tts \
  --replicas 3 \
  --publish 5123:5123 \
  travisvn/chatterbox-tts-api:latest

# Use faster settings
docker run -d \
  -e CFG_WEIGHT=0.3 \
  -e TEMPERATURE=0.5 \
  -e MAX_CHUNK_LENGTH=400 \
  travisvn/chatterbox-tts-api:latest
```

### Resource Limits

```bash
# Set memory and CPU limits
docker run -d \
  --memory=8g \
  --memory-swap=8g \
  --cpus=4 \
  travisvn/chatterbox-tts-api:latest
```

## 🔄 Updates and Versioning

### Staying Updated

```bash
# Check for new versions
docker pull travisvn/chatterbox-tts-api:latest

# Update running container
docker stop chatterbox-tts
docker rm chatterbox-tts
docker run -d --name chatterbox-tts ... travisvn/chatterbox-tts-api:latest
```

### Version Pinning

```bash
# Use specific version for production
docker run -d travisvn/chatterbox-tts-api:v1.0.0

# Check available tags
curl -s https://registry.hub.docker.com/v2/repositories/travisvn/chatterbox-tts-api/tags/ | jq .
```

## 🆘 Support

- **Documentation**: [GitHub Repository](https://github.com/travisvn/chatterbox-tts-api)
- **Issues**: [GitHub Issues](https://github.com/travisvn/chatterbox-tts-api/issues)
- **Discord**: [Join our Discord](http://chatterboxtts.com/discord)
- **API Docs**: Available at `http://localhost:5123/docs` when running

---

**Happy TTS generation! 🎙️✨**
