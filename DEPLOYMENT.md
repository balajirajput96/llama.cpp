# Deployment Guide for llama.cpp

This guide provides instructions for deploying llama.cpp in various environments.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Quick Deployment](#quick-deployment)
- [Docker Deployment](#docker-deployment)
- [Production Deployment](#production-deployment)
- [Monitoring and Maintenance](#monitoring-and-maintenance)

## Prerequisites

### System Requirements
- **OS**: Linux (Ubuntu 22.04+ recommended), macOS, or Windows with WSL2
- **RAM**: Minimum 8GB (16GB+ recommended for larger models)
- **Storage**: Minimum 10GB free space for build artifacts
- **CPU**: x86_64 with AVX2 support (or ARM64 with NEON)

### Build Dependencies
- CMake 3.14 or higher
- C++17 compatible compiler (GCC 13.3+, Clang, or MSVC)
- Git (for version control)
- Optional: ccache (for faster rebuilds)

## Quick Deployment

### 1. Clone and Build

```bash
# Clone the repository
git clone https://github.com/balajirajput96/llama.cpp.git
cd llama.cpp

# Configure the build
cmake -B build

# Build (use all available CPU cores)
cmake --build build --config Release -j $(nproc)
```

**Build time**: Approximately 5-10 minutes on a 4-core system with ccache.

### 2. Verify Installation

```bash
# Check llama-cli version
./build/bin/llama-cli --version

# Check llama-server version
./build/bin/llama-server --version
```

### 3. Run Tests

```bash
# Run the test suite
ctest --test-dir build --output-on-failure -j $(nproc)
```

Expected: 36/39 tests pass (3 may fail if network is unavailable for model downloads).

## Docker Deployment

### Available Docker Images

The project provides Dockerfiles for multiple backends:

- **CPU**: `.devops/cpu.Dockerfile`
- **CUDA** (NVIDIA GPUs): `.devops/cuda.Dockerfile`
- **Vulkan**: `.devops/vulkan.Dockerfile`
- **Intel**: `.devops/intel.Dockerfile`
- **MUSA** (Moore Threads GPUs): `.devops/musa.Dockerfile`
- **ROCm** (AMD GPUs): `.devops/rocm.Dockerfile`

### Build Docker Image (CPU)

```bash
# Build the CPU-optimized image
docker build -t llama-cpp:cpu -f .devops/cpu.Dockerfile .
```

### Run with Docker

```bash
# Run llama-server in a container
docker run -d \
  --name llama-server \
  -p 8080:8080 \
  -v /path/to/models:/models \
  llama-cpp:cpu \
  llama-server -m /models/your-model.gguf --host 0.0.0.0 --port 8080
```

### Docker Compose (Optional)

Create a `docker-compose.yml`:

```yaml
version: '3.8'
services:
  llama-server:
    image: llama-cpp:cpu
    ports:
      - "8080:8080"
    volumes:
      - ./models:/models
    command: llama-server -m /models/model.gguf --host 0.0.0.0 --port 8080
    restart: unless-stopped
```

Run with:
```bash
docker-compose up -d
```

## Production Deployment

### 1. Server Deployment

#### Start the Server

```bash
# Start llama-server on port 8080
./build/bin/llama-server \
  -m /path/to/your/model.gguf \
  --host 0.0.0.0 \
  --port 8080 \
  --threads $(nproc) \
  --ctx-size 4096
```

#### Server Options

- `--host`: Bind address (use `0.0.0.0` for public access)
- `--port`: Port number (default: 8080)
- `--threads`: Number of CPU threads to use
- `--ctx-size`: Context size (token limit)
- `--gpu-layers`: Number of layers to offload to GPU (if available)

### 2. Reverse Proxy with Nginx

Create `/etc/nginx/sites-available/llama-cpp`:

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable and restart:
```bash
sudo ln -s /etc/nginx/sites-available/llama-cpp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### 3. Systemd Service

Create `/etc/systemd/system/llama-server.service`:

```ini
[Unit]
Description=llama.cpp Server
After=network.target

[Service]
Type=simple
User=llama
WorkingDirectory=/opt/llama.cpp
ExecStart=/opt/llama.cpp/build/bin/llama-server -m /opt/models/model.gguf --host 127.0.0.1 --port 8080
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Enable and start:
```bash
sudo systemctl daemon-reload
sudo systemctl enable llama-server
sudo systemctl start llama-server
```

### 4. SSL/TLS with Certbot (Optional)

```bash
# Install certbot
sudo apt-get install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d your-domain.com

# Auto-renewal is configured automatically
```

## Monitoring and Maintenance

### Health Check

```bash
# Check server health
curl http://localhost:8080/health

# Check server version
curl http://localhost:8080/v1/models
```

### Logs

```bash
# View systemd logs
sudo journalctl -u llama-server -f

# View Docker logs
docker logs -f llama-server
```

### Resource Monitoring

```bash
# Monitor CPU and memory usage
htop

# Monitor GPU usage (if using CUDA)
nvidia-smi -l 1
```

### Backup and Updates

```bash
# Backup models and configuration
tar -czf llama-backup-$(date +%Y%m%d).tar.gz /opt/llama.cpp/models /opt/llama.cpp/build/bin

# Update llama.cpp
cd /opt/llama.cpp
git pull
cmake --build build --config Release -j $(nproc)
sudo systemctl restart llama-server
```

## Performance Tuning

### CPU Optimization

- Use `-march=native` for CPU-specific optimizations (done by default)
- Set `--threads` to match your CPU core count
- Enable ccache for faster rebuilds

### GPU Acceleration

For NVIDIA GPUs:
```bash
# Build with CUDA support
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Release -j $(nproc)

# Run with GPU layers
./build/bin/llama-server -m model.gguf --gpu-layers 32
```

For AMD GPUs:
```bash
# Build with ROCm support
cmake -B build -DGGML_ROCM=ON
cmake --build build --config Release -j $(nproc)
```

## Troubleshooting

### Build Issues

**Problem**: CMake configuration fails
**Solution**: Ensure CMake 3.14+ is installed: `cmake --version`

**Problem**: Compilation errors
**Solution**: Verify compiler version: `gcc --version` or `clang --version`

### Runtime Issues

**Problem**: Server doesn't start
**Solution**: Check port availability: `sudo netstat -tulpn | grep 8080`

**Problem**: Out of memory errors
**Solution**: Reduce context size with `--ctx-size 2048` or use a smaller model

**Problem**: Slow inference
**Solution**: Increase CPU threads with `--threads` or enable GPU acceleration

## Security Considerations

1. **Firewall**: Restrict access to the server port
   ```bash
   sudo ufw allow from trusted_ip to any port 8080
   ```

2. **Authentication**: Implement API authentication in your reverse proxy

3. **Rate Limiting**: Configure Nginx rate limiting
   ```nginx
   limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
   ```

4. **HTTPS**: Always use SSL/TLS for production deployments

5. **Regular Updates**: Keep llama.cpp and dependencies updated

## Support

- GitHub Repository: https://github.com/balajirajput96/llama.cpp
- Upstream Project: https://github.com/ggml-org/llama.cpp
- Documentation: See `docs/` directory for detailed guides

## License

This project is licensed under the MIT License. See LICENSE file for details.
