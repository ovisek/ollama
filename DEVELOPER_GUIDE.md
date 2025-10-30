# Ollama Developer Quick Reference

## Quick Start for Contributors

### 1. First Time Setup

```bash
# Clone the repository
git clone https://github.com/ollama/ollama.git
cd ollama

# Install Go 1.24+ (if not installed)
# Visit: https://go.dev/doc/install

# Run without building (development mode)
go run . serve

# In another terminal, test it
go run . run llama3.2
```

### 2. Project Navigation Cheat Sheet

| Directory | Purpose | When to Look Here |
|-----------|---------|-------------------|
| `/cmd` | CLI commands | Adding/modifying CLI commands |
| `/server` | HTTP API | Working on API endpoints |
| `/llm` | LLM runtime | Model execution issues |
| `/api` | API types | Understanding request/response formats |
| `/model` | Model formats | Model parsing/conversion |
| `/discover` | GPU detection | Hardware support issues |
| `/docs` | Documentation | Understanding features, API docs |
| `/integration` | E2E tests | Testing complete workflows |

### 3. Common Development Tasks

#### Add a New CLI Command

**File**: `cmd/cmd.go`

```go
// Add to NewCLI() function
&cobra.Command{
    Use:   "mycommand",
    Short: "Description of my command",
    RunE: func(cmd *cobra.Command, args []string) error {
        // Implementation
        return nil
    },
}
```

#### Add a New API Endpoint

**File**: `server/routes.go`

```go
// In Serve() function, add route
r.POST("/api/myendpoint", func(c *gin.Context) {
    // Parse request
    var req MyRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }
    
    // Process request
    // ...
    
    // Return response
    c.JSON(http.StatusOK, MyResponse{})
})
```

#### Define API Types

**File**: `api/types.go`

```go
type MyRequest struct {
    Model  string `json:"model"`
    Param  string `json:"param"`
}

type MyResponse struct {
    Result string `json:"result"`
}
```

### 4. Building and Testing

#### Run Tests
```bash
# All tests
go test ./...

# Specific package
go test ./server

# With verbose output
go test -v ./server

# Integration tests (requires more setup)
go test -v ./integration
```

#### Build Binary
```bash
# Simple build (CPU only)
go build

# With GPU support (requires CMake)
cmake -B build
cmake --build build
go build
```

#### Run with Debug Logging
```bash
OLLAMA_DEBUG=1 go run . serve
```

#### Test API Manually
```bash
# Start server
go run . serve

# In another terminal - Generate text
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "Why is the sky blue?",
  "stream": false
}'

# Chat
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.2",
  "messages": [
    {"role": "user", "content": "Hello!"}
  ],
  "stream": false
}'

# List models
curl http://localhost:11434/api/tags
```

### 5. Code Style and Conventions

#### Commit Messages
```
Format: <package>: <description>

Good:
  server: add support for custom headers
  llm: improve memory estimation for large models
  api: add embedding endpoint

Bad:
  feat: add feature
  fix: fixed bug
  update code
```

#### Go Code Style
- Follow standard Go conventions
- Use `gofmt` to format code
- Add comments for exported functions
- Keep functions focused and small

#### Testing
- Test behavior, not implementation
- Use table-driven tests where appropriate
- Mock external dependencies
- Include both positive and negative cases

### 6. Understanding the Codebase

#### Key Interfaces

**Model Interface** (`llm/server.go`):
```go
type Server interface {
    Ping(ctx context.Context) error
    Generate(ctx context.Context, req GenerateRequest) error
    Embed(ctx context.Context, prompt string) ([]float64, error)
    Close() error
}
```

**Scheduler Interface** (`server/sched.go`):
- Manages model lifecycle
- Queues inference requests
- Handles concurrent access

#### Configuration

Environment variables (defined in various files):
- `OLLAMA_HOST` - Server bind address (default: 127.0.0.1:11434)
- `OLLAMA_MODELS` - Model storage directory
- `OLLAMA_KEEP_ALIVE` - How long to keep models loaded
- `OLLAMA_NUM_PARALLEL` - Number of parallel requests
- `OLLAMA_MAX_LOADED_MODELS` - Max models in memory
- `OLLAMA_DEBUG` - Enable debug logging
- `OLLAMA_FLASH_ATTENTION` - Enable flash attention

### 7. Debugging Tips

#### Common Issues and Solutions

**Issue**: "model not found"
```bash
# Check available models
go run . list

# Pull the model
go run . pull llama3.2
```

**Issue**: "out of memory"
- Check model size vs available RAM
- Use smaller quantization (Q4_0 instead of Q8_0)
- Reduce `OLLAMA_MAX_LOADED_MODELS`

**Issue**: Build fails with CMake errors
- Ensure CMake is installed and in PATH
- Check for required compilers (gcc/clang)
- On Windows, use Visual Studio 2022

#### Enable Verbose Logging

```bash
# Set debug environment variable
export OLLAMA_DEBUG=1
go run . serve

# Or inline
OLLAMA_DEBUG=1 go run . serve
```

#### Inspect Model Details
```bash
# Show model information
go run . show llama3.2

# Show with Modelfile
go run . show llama3.2 --modelfile
```

### 8. Working with Models

#### Create a Custom Model

**Create Modelfile**:
```dockerfile
FROM llama3.2

# Set parameters
PARAMETER temperature 0.8
PARAMETER top_p 0.9

# Set system message
SYSTEM """
You are a helpful coding assistant.
"""

# Add custom prompt template (optional)
TEMPLATE """{{ if .System }}<|system|>
{{ .System }}<|end|>
{{ end }}{{ if .Prompt }}<|user|>
{{ .Prompt }}<|end|>
{{ end }}<|assistant|>
"""
```

**Build the model**:
```bash
go run . create my-coder -f ./Modelfile
go run . run my-coder
```

#### Import GGUF Model
```dockerfile
# Modelfile
FROM ./path/to/model.gguf

PARAMETER temperature 1.0
```

```bash
go run . create custom-model -f ./Modelfile
```

### 9. API Usage Examples

#### Python
```python
import requests

# Generate
response = requests.post('http://localhost:11434/api/generate', 
    json={
        'model': 'llama3.2',
        'prompt': 'Why is the sky blue?',
        'stream': False
    })
print(response.json())

# Chat
response = requests.post('http://localhost:11434/api/chat',
    json={
        'model': 'llama3.2',
        'messages': [
            {'role': 'user', 'content': 'Hello!'}
        ],
        'stream': False
    })
print(response.json())
```

#### JavaScript
```javascript
// Generate
const response = await fetch('http://localhost:11434/api/generate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        model: 'llama3.2',
        prompt: 'Why is the sky blue?',
        stream: false
    })
});
const data = await response.json();
console.log(data);

// Chat
const chatResponse = await fetch('http://localhost:11434/api/chat', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        model: 'llama3.2',
        messages: [
            { role: 'user', content: 'Hello!' }
        ],
        stream: false
    })
});
const chatData = await chatResponse.json();
console.log(chatData);
```

### 10. Useful Code Patterns

#### Streaming Response
```go
// Server-side (in routes.go)
c.Stream(func(w io.Writer) bool {
    // Generate chunk
    chunk := generateNextChunk()
    
    // Write JSON line
    json.NewEncoder(w).Encode(chunk)
    
    // Continue streaming
    return !done
})
```

#### Error Handling
```go
// Consistent error responses
if err != nil {
    c.JSON(http.StatusInternalServerError, gin.H{
        "error": err.Error(),
    })
    return
}
```

#### Context Handling
```go
// Respect context cancellation
func process(ctx context.Context) error {
    for {
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
            // Do work
        }
    }
}
```

### 11. Performance Optimization

#### Model Loading
- Keep frequently used models loaded (`OLLAMA_KEEP_ALIVE`)
- Use quantized models (Q4_0, Q5_1) for faster loading
- Preload models at startup if known

#### Inference Speed
- Use GPU acceleration when available
- Enable flash attention if supported
- Batch similar requests
- Reduce context length when possible

#### Memory Management
- Set appropriate `OLLAMA_MAX_LOADED_MODELS`
- Monitor with `ollama ps`
- Use smaller quantization for memory-constrained systems

### 12. Contributing Checklist

Before submitting a PR:
- [ ] Run `go test ./...` - all tests pass
- [ ] Run `gofmt -w .` - code is formatted
- [ ] Add tests for new functionality
- [ ] Update documentation if needed
- [ ] Follow commit message conventions
- [ ] Check that changes are minimal and focused
- [ ] Discuss large changes in an issue first

### 13. Getting Help

- **Documentation**: Check `/docs` directory
- **Discord**: https://discord.gg/ollama
- **GitHub Issues**: Search existing issues
- **Code Comments**: Many complex parts have detailed comments
- **Examples**: Check `/api/examples` for code samples

### 14. Resources

**Essential Files to Understand**:
1. `README.md` - Project overview
2. `docs/api.md` - Complete API reference
3. `docs/modelfile.md` - Modelfile specification
4. `docs/development.md` - Build instructions
5. `cmd/cmd.go` - CLI command implementation
6. `server/routes.go` - API endpoints
7. `llm/server.go` - LLM runtime interface

**External Resources**:
- llama.cpp: https://github.com/ggerganov/llama.cpp
- GGUF format: https://github.com/ggerganov/ggml/blob/master/docs/gguf.md
- Go documentation: https://go.dev/doc

### 15. Common Gotchas

1. **Models not found after pull**: Check `OLLAMA_MODELS` environment variable
2. **Port already in use**: Another Ollama instance might be running
3. **GPU not detected**: Ensure GPU drivers and libraries are installed
4. **Build fails**: Make sure all prerequisites (Go, CMake, compilers) are installed
5. **Tests fail intermittently**: Some tests may require specific models or resources
6. **Changes not reflected**: Restart the server after code changes

## Quick Command Reference

```bash
# Development
go run . serve              # Start server
go test ./...               # Run all tests
go build                    # Build binary

# Model operations  
go run . pull <model>       # Download model
go run . list               # List local models
go run . rm <model>         # Remove model
go run . show <model>       # Show model info

# Running models
go run . run <model>        # Interactive chat
go run . run <model> "prompt"  # Single prompt

# Custom models
go run . create <name> -f Modelfile  # Create model
go run . cp <src> <dst>     # Copy model

# Server management
go run . ps                 # List running models
go run . stop <model>       # Stop model
```
