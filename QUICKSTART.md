# Quick Start Guide - llama.cpp

Get up and running with llama.cpp in minutes!

## 🚀 Installation

### Option 1: Build from Source (Recommended)

```bash
# Clone the repository
git clone https://github.com/balajirajput96/llama.cpp.git
cd llama.cpp

# Build with CMake (takes 5-10 minutes)
cmake -B build
cmake --build build --config Release -j $(nproc)
```

### Option 2: Using Docker

```bash
# Pull and run CPU version
docker build -t llama-cpp:cpu -f .devops/cpu.Dockerfile .
```

## ✅ Verify Installation

```bash
# Check version
./build/bin/llama-cli --version

# Expected output:
# version: 2 (f5a7357)
# built with cc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0 for x86_64-linux-gnu
```

## 🎯 First Steps

### 1. Get a Model

Download a GGUF model from Hugging Face:

```bash
# Example: Download a small model (optional - for testing)
mkdir -p models
# You can download models from https://huggingface.co/models?library=gguf
```

### 2. Run Text Generation

```bash
# Run with a local model
./build/bin/llama-cli -m models/your-model.gguf -p "Hello, world!" -n 50

# Or download and run directly from Hugging Face
./build/bin/llama-cli -hf ggml-org/gemma-3-1b-it-GGUF
```

### 3. Start API Server

```bash
# Start the OpenAI-compatible API server
./build/bin/llama-server -m models/your-model.gguf --port 8080

# Or with Hugging Face model
./build/bin/llama-server -hf ggml-org/gemma-3-1b-it-GGUF --port 8080
```

### 4. Test the API

```bash
# Health check
curl http://localhost:8080/health

# List models
curl http://localhost:8080/v1/models

# Generate text (OpenAI-compatible)
curl http://localhost:8080/v1/completions \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Once upon a time",
    "max_tokens": 50,
    "temperature": 0.7
  }'
```

## 🛠️ Common Tasks

### Run Tests

```bash
ctest --test-dir build --output-on-failure -j $(nproc)

# Expected: 36/39 tests pass (3 may fail without internet)
```

### Update Code

```bash
git pull
cmake --build build --config Release -j $(nproc)
```

### Format Code (for contributors)

```bash
git clang-format
```

### Clean Build

```bash
rm -rf build
cmake -B build
cmake --build build --config Release -j $(nproc)
```

## 🐳 Docker Quick Start

```bash
# Build CPU image
docker build -t llama-cpp:cpu -f .devops/cpu.Dockerfile .

# Run server
docker run -d \
  --name llama-server \
  -p 8080:8080 \
  -v $(pwd)/models:/models \
  llama-cpp:cpu \
  llama-server -m /models/your-model.gguf --host 0.0.0.0 --port 8080

# View logs
docker logs -f llama-server

# Stop server
docker stop llama-server
```

## 🎮 Interactive Mode

```bash
# Run in interactive mode
./build/bin/llama-cli -m models/your-model.gguf -i

# Type your prompts and press Enter
# Use Ctrl+C to exit
```

## 🔧 Common Options

### llama-cli
```bash
-m, --model <path>      # Model file path
-p, --prompt <text>     # Input prompt
-n, --n-predict <num>   # Number of tokens to generate
-t, --threads <num>     # Number of CPU threads
-c, --ctx-size <num>    # Context size
-ngl, --gpu-layers <n>  # GPU layers to offload
-i, --interactive       # Interactive mode
```

### llama-server
```bash
-m, --model <path>      # Model file path
--host <address>        # Bind address (default: 127.0.0.1)
--port <num>            # Port number (default: 8080)
-t, --threads <num>     # Number of CPU threads
-c, --ctx-size <num>    # Context size
-ngl, --gpu-layers <n>  # GPU layers to offload
```

## 💡 Tips

1. **Use ccache** for faster rebuilds (automatically detected)
2. **Enable GPU** if you have NVIDIA/AMD GPU:
   ```bash
   # CUDA
   cmake -B build -DGGML_CUDA=ON
   
   # ROCm (AMD)
   cmake -B build -DGGML_ROCM=ON
   ```
3. **Optimize for your CPU**:
   ```bash
   cmake -B build -DCMAKE_CXX_FLAGS="-march=native"
   ```
4. **Use smaller models** if you have limited RAM
5. **Check documentation** in `docs/` for advanced features

## 📚 Next Steps

- Read the [Deployment Guide](DEPLOYMENT.md) for production setup
- Explore `examples/` for more use cases
- Check `docs/` for detailed documentation
- Visit the [main repository](https://github.com/ggml-org/llama.cpp) for updates

## 🆘 Troubleshooting

### Build fails
```bash
# Check CMake version (need 3.14+)
cmake --version

# Check compiler version
gcc --version  # or clang --version
```

### Out of memory
```bash
# Use smaller context size
./build/bin/llama-cli -m model.gguf -c 2048

# Or use a smaller model
```

### Port already in use
```bash
# Use a different port
./build/bin/llama-server -m model.gguf --port 8081
```

## 📞 Support

- Issues: https://github.com/balajirajput96/llama.cpp/issues
- Upstream: https://github.com/ggml-org/llama.cpp
- Documentation: `docs/` directory

## 📄 License

MIT License - See LICENSE file for details
