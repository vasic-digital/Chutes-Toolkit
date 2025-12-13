# Chutes AI Provider for AI Toolkit

A comprehensive provider implementation for the [Chutes AI](https://chutes.ai) platform, designed for the AI Toolkit ecosystem.

## Overview

The Chutes AI Provider implements the complete AI Toolkit Provider interface, offering seamless integration with Chutes AI's model hosting platform. This provider supports chat completions, embeddings, reranking, and comprehensive model discovery with intelligent capability inference.

## Features

### 🚀 Core Capabilities
- **Chat Completions**: Text generation with advanced parameters
- **Embeddings**: High-quality text-to-vector conversion
- **Reranking**: Document relevance scoring and ranking
- **Model Discovery**: Automatic model detection with capability inference
- **Multi-Model Support**: Qwen, DeepSeek, GLM, Kimi, and more

### 🔧 Technical Features
- **Complete Provider Interface**: Full implementation of toolkit.Provider
- **Configuration Management**: Robust config validation and merging
- **HTTP Client**: Optimized API client with retry logic
- **Capability Inference**: Intelligent model capability detection
- **Error Handling**: Comprehensive error management
- **Auto-Registration**: Seamless toolkit integration

## Installation

### As Part of AI Toolkit
The Chutes provider is included in the AI Toolkit by default. Simply import the toolkit and the provider will be automatically available.

### Standalone Usage
```bash
go get github.com/HelixDevelopment/HelixAgent-Chutes
```

## Quick Start

### Environment Setup
```bash
export CHUTES_API_KEY="your-chutes-api-key"
```

### Configuration
```json
{
  "name": "chutes",
  "api_key": "your-chutes-api-key",
  "base_url": "https://api.chutes.ai/v1",
  "timeout": 30000,
  "retries": 3,
  "rate_limit": 60
}
```

### Basic Usage
```go
import (
    "context"
    "github.com/superagent/toolkit/pkg/toolkit"
    _ "github.com/HelixDevelopment/HelixAgent-Chutes/providers/chutes"
)

// Create provider
config := map[string]interface{}{
    "api_key": "your-api-key",
}

provider, err := toolkit.NewProvider("chutes", config)
if err != nil {
    log.Fatal(err)
}

// Chat completion
response, err := provider.Chat(context.Background(), toolkit.ChatRequest{
    Model: "qwen2.5-7b-instruct",
    Messages: []toolkit.ChatMessage{
        {Role: "user", Content: "Hello, world!"},
    },
})
```

## Supported Models

### Chat Models
- **Qwen Series**: Qwen2.5, Qwen3 (various sizes)
- **DeepSeek Series**: DeepSeek-V3, DeepSeek-R1
- **GLM Series**: GLM-4, GLM-4.6
- **Kimi Models**: Various Kimi instruction models

### Embedding Models
- Text embedding models hosted on Chutes
- Various dimension sizes and capabilities

### Rerank Models
- Document reranking models
- Query-document relevance scoring

### Specialized Models
- **Vision Models**: Multimodal vision-language models
- **Audio Models**: Text-to-speech and audio processing
- **Video Models**: Video generation and processing

## Model Capabilities

The provider automatically infers model capabilities:

- **Chat Support**: Based on model type and naming patterns
- **Embedding Support**: Models with embedding-specific architecture
- **Rerank Support**: Models designed for document ranking
- **Vision Support**: Multimodal vision-language capabilities
- **Function Calling**: Advanced function calling support for compatible models
- **Context Windows**: Automatic context window inference (4K-131K tokens)

## Architecture

### Directory Structure
```
providers/chutes/
├── chutes.go       # Main provider implementation
├── builder.go      # Configuration management
├── client.go       # HTTP client and API calls
├── discovery.go    # Model discovery and inference
└── chutes_test.go  # Comprehensive test suite
```

### Key Components

#### Provider (`chutes.go`)
Implements the complete Provider interface with all required methods:
- `Name()` - Provider identification
- `Chat()` - Chat completion requests
- `Embed()` - Embedding generation
- `Rerank()` - Document reranking
- `DiscoverModels()` - Model discovery and listing
- `ValidateConfig()` - Configuration validation

#### Configuration Builder (`builder.go`)
Robust configuration management:
- `Build()` - Configuration construction from maps
- `Validate()` - Configuration validation
- `Merge()` - Configuration merging
- Type-safe configuration extraction

#### HTTP Client (`client.go`)
Optimized API client implementation:
- `ChatCompletion()` - Chat completion requests
- `CreateEmbeddings()` - Embedding generation
- `CreateRerank()` - Document reranking
- `GetModels()` - Model discovery
- Proper authentication and error handling

#### Discovery (`discovery.go`)
Intelligent model discovery system:
- `ChutesCapabilityInferrer` - Capability inference engine
- `ChutesModelFormatter` - Human-readable model names
- Model-specific logic for Qwen, DeepSeek, GLM, Kimi
- Context window and max tokens inference

## Configuration Options

### Required Parameters
- `api_key`: Your Chutes API key

### Optional Parameters
- `base_url`: API base URL (default: "https://api.chutes.ai/v1")
- `timeout`: Request timeout in milliseconds (default: 30000)
- `retries`: Number of retry attempts (default: 3)
- `rate_limit`: Rate limit in requests per minute (default: 60)

### Environment Variables
- `CHUTES_API_KEY`: API key for authentication

## Development

### Building
```bash
go build ./providers/chutes
```

### Testing
```bash
go test ./providers/chutes/... -v
```

### Code Style
Follow the AI Toolkit coding standards:
- Interface-driven design
- Comprehensive error handling
- Extensive documentation
- Consistent naming conventions

## Integration with AI Toolkit

The Chutes provider integrates seamlessly with the AI Toolkit:

### CLI Usage
```bash
# List providers
./toolkit list providers

# Generate configuration
./toolkit config generate provider chutes

# Validate configuration
./toolkit validate provider chutes config.json

# Discover models
./toolkit discover chutes

# Execute tasks
./toolkit execute generic "Hello world" --provider chutes
```

### Programmatic Usage
```go
// Create toolkit with Chutes provider
tk := toolkit.NewToolkit()

// Register Chutes provider
chutes.Register(tk.GetProviderFactoryRegistry())

// Create and use provider
provider, _ := tk.CreateProvider("chutes", config)
response, _ := provider.Chat(ctx, request)
```

## Model Discovery

The provider includes sophisticated model discovery with capability inference:

```go
models, err := provider.DiscoverModels(ctx)
for _, model := range models {
    fmt.Printf("Model: %s\n", model.Name)
    fmt.Printf("Chat: %v, Embedding: %v, Rerank: %v\n",
        model.Capabilities.SupportsChat,
        model.Capabilities.SupportsEmbedding,
        model.Capabilities.SupportsRerank)
}
```

## Error Handling

Comprehensive error handling throughout:
- Configuration validation errors
- API request/response errors
- Network timeout handling
- Rate limit management
- Authentication errors

## Performance

- **Connection Pooling**: Efficient HTTP connection management
- **Retry Logic**: Intelligent retry with exponential backoff
- **Timeout Management**: Configurable timeouts for all operations
- **Rate Limiting**: Built-in rate limit respect and management

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests for new functionality
5. Ensure all tests pass
6. Submit a pull request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

For support and questions:
- Create an issue in the GitHub repository
- Check the documentation in the `docs/` directory
- Review the test files for usage examples

## Changelog

### v1.0.0
- Initial release with full provider implementation
- Complete AI Toolkit integration
- Comprehensive model discovery
- Full test coverage
- Documentation complete