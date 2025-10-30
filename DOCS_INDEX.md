# Ollama Documentation Index

Welcome to the Ollama repository! This document helps you navigate the documentation and understand the project.

## 📚 Documentation Overview

We've created comprehensive documentation to help you understand and contribute to Ollama:

### For All Users

**[REPOSITORY_EXPLAINED.md](./REPOSITORY_EXPLAINED.md)** - Start here!
- What is Ollama and what does it do?
- Repository structure and organization
- Key features and capabilities
- Technology stack
- Use cases and applications
- Community integrations
- ~266 lines of comprehensive overview

### For Developers and Contributors

**[DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md)** - Quick reference for development
- Quick start guide for contributors
- Common development tasks
- Code navigation cheat sheet
- API usage examples (Python, JavaScript)
- Debugging tips and common issues
- Command reference
- Contributing checklist
- ~482 lines of practical guidance

**[ARCHITECTURE.md](./ARCHITECTURE.md)** - Deep technical dive
- System architecture and component interaction
- Data flow diagrams
- Core component descriptions
- Concurrency model
- GPU detection and initialization
- Performance characteristics
- Security considerations
- ~412 lines of technical details

## 📖 Official Documentation (in `/docs`)

The `/docs` directory contains the official documentation:

- **[docs/api.md](./docs/api.md)** - Complete REST API reference
- **[docs/modelfile.md](./docs/modelfile.md)** - Modelfile specification and examples
- **[docs/development.md](./docs/development.md)** - Build and development setup
- **[docs/import.md](./docs/import.md)** - Importing models guide
- **[docs/openai.md](./docs/openai.md)** - OpenAI API compatibility
- **[docs/linux.md](./docs/linux.md)** - Linux installation and setup
- **[docs/windows.md](./docs/windows.md)** - Windows installation and setup
- **[docs/troubleshooting.md](./docs/troubleshooting.md)** - Common issues and solutions

## 🚀 Quick Start by Role

### I want to USE Ollama
1. Read the [README.md](./README.md) - installation and basic usage
2. Check [docs/api.md](./docs/api.md) for API integration
3. Browse the [model library](https://ollama.com/library)

### I want to UNDERSTAND Ollama
1. Start with [REPOSITORY_EXPLAINED.md](./REPOSITORY_EXPLAINED.md)
2. Dive into [ARCHITECTURE.md](./ARCHITECTURE.md) for technical details
3. Review relevant files in `/docs` for specific topics

### I want to CONTRIBUTE to Ollama
1. Read [CONTRIBUTING.md](./CONTRIBUTING.md) - contribution guidelines
2. Follow [DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md) for setup
3. Check [docs/development.md](./docs/development.md) for build instructions
4. Review [ARCHITECTURE.md](./ARCHITECTURE.md) to understand the system

### I want to CREATE custom models
1. Read [docs/modelfile.md](./docs/modelfile.md) - Modelfile syntax
2. Check [docs/import.md](./docs/import.md) - importing models
3. See examples in the README

### I want to INTEGRATE Ollama into my app
1. Review [docs/api.md](./docs/api.md) - API endpoints
2. Check [DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md) - code examples
3. See `/api/examples` for working code
4. Consider [docs/openai.md](./docs/openai.md) for OpenAI compatibility

## 🗺️ Repository Map

```
ollama/
├── README.md                    # Project overview, installation, quickstart
├── REPOSITORY_EXPLAINED.md      # 📘 High-level explanation (START HERE)
├── ARCHITECTURE.md              # 🏗️ Technical architecture deep dive
├── DEVELOPER_GUIDE.md           # 💻 Developer quick reference
├── CONTRIBUTING.md              # 🤝 How to contribute
├── LICENSE                      # MIT License
│
├── main.go                      # Entry point
├── cmd/                         # CLI implementation
├── server/                      # HTTP API server
├── llm/                         # LLM runtime interface
├── api/                         # API types and examples
├── model/                       # Model format handling
├── llama/                       # llama.cpp integration
├── discover/                    # GPU detection
├── openai/                      # OpenAI-compatible API
├── docs/                        # Official documentation
│   ├── api.md                   # API reference
│   ├── modelfile.md             # Modelfile guide
│   ├── development.md           # Build instructions
│   └── ...
└── integration/                 # Integration tests
```

## 🎯 Common Questions

### What is Ollama?
See [REPOSITORY_EXPLAINED.md](./REPOSITORY_EXPLAINED.md#overview)

### How does it work?
See [ARCHITECTURE.md](./ARCHITECTURE.md#system-architecture-overview)

### How do I build it?
See [docs/development.md](./docs/development.md) or [DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md#quick-start-for-contributors)

### How do I use the API?
See [docs/api.md](./docs/api.md) or [DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md#api-usage-examples)

### How do I create a custom model?
See [docs/modelfile.md](./docs/modelfile.md) or [DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md#working-with-models)

### Where are the tests?
- Unit tests: Alongside source files (`*_test.go`)
- Integration tests: `/integration` directory

### How do I contribute?
See [CONTRIBUTING.md](./CONTRIBUTING.md) and [DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md#contributing-checklist)

## 📊 Project Statistics

- **Language**: Go 1.24+
- **License**: MIT
- **Lines of Code**: 100,000+ (estimated)
- **Documentation Files**: 15+
- **Community Integrations**: 500+
- **Supported Models**: 100+

## 🔗 External Resources

- **Website**: https://ollama.com
- **Discord**: https://discord.gg/ollama
- **GitHub**: https://github.com/ollama/ollama
- **Model Library**: https://ollama.com/library
- **Docker Hub**: https://hub.docker.com/r/ollama/ollama

## 🆘 Getting Help

1. **Search existing documentation** - Check this index first
2. **GitHub Issues** - Search for similar issues
3. **Discord Community** - Ask questions in the Discord server
4. **Documentation** - Review the `/docs` directory

## 📝 Documentation Contributions

If you find gaps in the documentation or want to improve it:
1. Check [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines
2. Small documentation fixes are welcome
3. Large documentation changes should be discussed first

---

**Last Updated**: 2025-10-30  
**Documentation Created By**: GitHub Copilot Agent

For the most current information, always check the [official website](https://ollama.com) and the [main README](./README.md).
