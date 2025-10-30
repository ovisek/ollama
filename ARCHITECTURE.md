# Ollama Architecture Deep Dive

## System Architecture Overview

Ollama is designed as a client-server application with multiple layers handling different aspects of LLM deployment and execution.

## Component Interaction Flow

### 1. User Request Flow

```
User Command (CLI)
    │
    ▼
cmd/cmd.go (Command Parser)
    │
    ├─► serve → Start HTTP Server
    ├─► run → Interactive Session
    ├─► pull → Download Model
    ├─► create → Build Custom Model
    └─► ... other commands
    │
    ▼
HTTP Server (server/routes.go)
    │
    ▼
Request Handler
    │
    ├─► /api/generate → Generate text
    ├─► /api/chat → Chat conversation
    ├─► /api/pull → Fetch model
    └─► /api/create → Create model
    │
    ▼
Model Scheduler (server/sched.go)
    │
    ├─► Load Model (if not loaded)
    ├─► Queue Request
    └─► Manage Concurrency
    │
    ▼
LLM Runtime (llm/server.go)
    │
    ▼
llama.cpp Backend
    │
    ├─► Token Generation
    ├─► GPU Acceleration
    └─► Memory Management
    │
    ▼
Response Stream
    │
    ▼
User Output
```

### 2. Model Loading Sequence

```
1. User Request (e.g., ollama run llama3.2)
   │
   ▼
2. Model Resolution (server/modelpath.go)
   - Parse model name
   - Resolve to registry path
   │
   ▼
3. Check Local Storage (server/images.go)
   - Look for model manifest
   - Verify blob integrity
   │
   ▼
4. Load Model Components
   - Read manifest (metadata)
   - Load config blob (parameters)
   - Load model weights (GGUF file)
   │
   ▼
5. Initialize LLM Runtime (llm/server.go)
   - Allocate memory (llm/memory.go)
   - Detect GPU (discover/gpu.go)
   - Load into llama.cpp
   │
   ▼
6. Ready for Inference
```

### 3. GPU Detection and Initialization

```
Application Start
    │
    ▼
discover/gpu.go
    │
    ├─► discover/gpu_darwin.go (macOS)
    │   └─► Check for Metal
    │
    ├─► discover/gpu_linux.go (Linux)
    │   ├─► Check for NVIDIA CUDA
    │   └─► Check for AMD ROCm
    │
    └─► discover/gpu_windows.go (Windows)
        ├─► Check for NVIDIA CUDA
        └─► Check for AMD ROCm
    │
    ▼
Select Best Available GPU
    │
    ▼
Load GPU-specific Libraries
    │
    ▼
Initialize llama.cpp with GPU support
```

## Core Components Detailed

### Command Line Interface (cmd/)

**Purpose**: Parse user commands and route to appropriate handlers

**Key Files**:
- `cmd.go` - Main command definitions and handlers
- `interactive.go` - Interactive chat mode handling
- `start_*.go` - Platform-specific startup code

**Commands**:
- `serve` - Start the API server
- `run <model>` - Run a model interactively
- `pull <model>` - Download a model
- `push <model>` - Upload a model
- `create <name> -f Modelfile` - Create custom model
- `list` - Show local models
- `show <model>` - Display model info
- `rm <model>` - Delete a model
- `cp <source> <dest>` - Copy a model
- `ps` - List running models
- `stop <model>` - Stop a running model

### HTTP Server (server/)

**Purpose**: RESTful API for programmatic access

**Key Files**:
- `routes.go` - API endpoint definitions (~44KB)
- `sched.go` - Model scheduler (~32KB)
- `images.go` - Model registry management
- `create.go` - Model creation logic
- `download.go` / `upload.go` - Model distribution
- `quantization.go` - Model quantization

**API Categories**:
1. **Generation**: Text generation and chat
2. **Model Management**: Pull, push, create, delete
3. **Information**: List models, show details
4. **Embeddings**: Generate vector embeddings

**Scheduler Responsibilities**:
- Queue incoming requests
- Load/unload models based on demand
- Manage concurrent requests to same model
- Memory management (evict unused models)
- GPU allocation

### LLM Runtime (llm/)

**Purpose**: Interface to the actual model inference engine

**Key Files**:
- `server.go` - Main runtime interface (~31KB)
- `memory.go` - Memory estimation and management
- `llm_*.go` - Platform-specific implementations

**Responsibilities**:
- Start/stop llama.cpp processes
- Manage model loading
- Handle token generation requests
- Stream responses back to server
- Monitor resource usage

### Model Management (model/, server/images.go)

**Model Storage Structure**:
```
~/.ollama/models/
├── manifests/
│   └── registry.ollama.ai/
│       └── library/
│           └── llama3.2/
│               └── latest
├── blobs/
│   ├── sha256-abc123... (model weights)
│   ├── sha256-def456... (config)
│   └── sha256-ghi789... (template)
```

**Manifest Format**:
- Model metadata (architecture, parameters)
- References to blob layers
- Parent model information
- Quantization settings

**Blob Types**:
- **Model**: GGUF quantized weights
- **Adapter**: LoRA adapters
- **Config**: Model hyperparameters
- **Template**: Prompt template
- **System**: System message
- **License**: Model license

### GPU Support (discover/)

**Detection Process**:
1. Scan for GPU libraries on system
2. Query GPU capabilities
3. Estimate available VRAM
4. Select appropriate backend

**Supported Platforms**:

| Platform | GPU Type | Backend | Library |
|----------|----------|---------|---------|
| macOS | Apple Silicon | Metal | Built-in |
| Linux | NVIDIA | CUDA | libcuda.so |
| Linux | AMD | ROCm | libamdhip64.so |
| Windows | NVIDIA | CUDA | cuda.dll |
| Windows | AMD | ROCm | amdhip64.dll |

### API Layer (api/, openai/)

**Standard API** (`api/`):
- Native Ollama API format
- Streaming support
- Model-specific options

**OpenAI Compatible API** (`openai/`):
- Drop-in replacement for OpenAI API
- Compatible with OpenAI SDKs
- Translates between formats

## Data Flow Examples

### Example 1: Chat Request

```
1. HTTP POST /api/chat
   Body: {
     "model": "llama3.2",
     "messages": [{"role": "user", "content": "Hello"}]
   }

2. routes.go → ChatHandler()
   - Validate request
   - Parse model name

3. sched.go → Schedule request
   - Check if model loaded
   - If not: load model
   - Queue request

4. llm/server.go → Generate()
   - Build prompt from messages
   - Call llama.cpp completion

5. Stream tokens back
   - Each token sent as JSON line
   - {"message": {"content": "Hi"}}
   - {"done": true}

6. Client receives response
```

### Example 2: Model Pull

```
1. HTTP POST /api/pull
   Body: {"name": "llama3.2"}

2. routes.go → PullHandler()

3. modelpath.go → Parse name
   - Registry: registry.ollama.ai
   - Namespace: library
   - Model: llama3.2
   - Tag: latest

4. download.go → Download
   - Fetch manifest from registry
   - Check local blobs
   - Download missing blobs
   - Progress updates via stream

5. Store locally
   - Save manifest
   - Save blobs
   - Verify integrity

6. Response: {"status": "success"}
```

## Concurrency Model

### Request Handling

**Scheduler Design**:
- Single scheduler per Ollama instance
- Manages multiple models
- Queues requests per model
- Parallel generation for same model

**Model States**:
1. **Unloaded** - On disk, not in memory
2. **Loading** - Being loaded into memory
3. **Loaded** - Ready for inference
4. **Busy** - Processing request
5. **Idle** - Loaded but inactive
6. **Unloading** - Being removed from memory

### Memory Management

**Strategy**:
1. Estimate model memory requirements
2. Check available RAM/VRAM
3. Evict idle models if needed (LRU)
4. Load requested model
5. Monitor memory usage

## Integration Points

### For Developers

**Embedding Ollama**:
1. Use REST API (any language)
2. Use official SDKs (Python, JavaScript)
3. Use OpenAI-compatible API
4. Direct library integration

**Extension Points**:
1. Custom model formats (convert/)
2. Custom quantization schemes
3. Additional API endpoints
4. Platform-specific optimizations

## Performance Characteristics

### Model Loading Time
- Depends on: Model size, storage speed, available memory
- Typically: 1-10 seconds for 7B models

### Inference Speed
- Depends on: Hardware, quantization, context length
- CPU: 5-20 tokens/sec (7B model)
- GPU: 30-100+ tokens/sec (7B model)

### Memory Usage
- Varies by quantization:
  - Q4_0: ~4GB for 7B model
  - Q5_1: ~5GB for 7B model
  - Q8_0: ~8GB for 7B model

## Security Considerations

### Input Validation
- Model names sanitized
- File paths validated
- Request sizes limited

### Model Isolation
- Each model runs in separate process (llama.cpp)
- Resource limits enforced
- Clean shutdown handling

### Network Security
- Default: localhost only (127.0.0.1)
- Configurable via OLLAMA_HOST
- No authentication by default (local use)

## Monitoring and Debugging

### Logging
- Structured logging (logutil/)
- Debug mode available
- Request/response logging
- Error tracking

### Metrics
- Model load/unload events
- Request queue length
- Generation speed
- Memory usage
- GPU utilization

### Debug Tools
- `ollama ps` - Show running models
- `ollama list` - Show local models
- Log files in ~/.ollama/logs
- Verbose mode: `OLLAMA_DEBUG=1`

## Future Architecture Considerations

**Scalability**:
- Distributed model serving
- Load balancing across GPUs
- Model caching strategies

**Features**:
- Multi-modal support enhancements
- Fine-tuning capabilities
- Advanced scheduling algorithms
- Enhanced quantization methods
