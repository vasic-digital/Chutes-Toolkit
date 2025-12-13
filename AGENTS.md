# Chutes AI Provider Development Guide

This document provides comprehensive development guidelines for the Chutes AI Provider implementation.

## Project Structure

```
Toolkit/Chutes/
├── providers/
│   └── chutes/
│       ├── chutes.go       # Main provider implementation
│       ├── builder.go      # Configuration management
│       ├── client.go       # HTTP client and API interactions
│       ├── discovery.go    # Model discovery and capability inference
│       └── chutes_test.go  # Comprehensive test suite
├── Upstreams/
│   └── GitHub.sh          # Upstream repository configuration
├── LICENSE                # MIT license
└── README.md             # Project documentation
```

## Essential Commands

### Building and Testing
```bash
# Build the provider
go build ./providers/chutes

# Run all tests
go test ./providers/chutes/... -v

# Run specific test
go test ./providers/chutes/... -run TestChutesProvider -v

# Test with coverage
go test ./providers/chutes/... -coverprofile=coverage.out
go tool cover -html=coverage.out
```

### Development Tools
```bash
# Format code
gofmt -w providers/chutes/

# Vet code
go vet ./providers/chutes/...

# Check for issues
golint ./providers/chutes/...

# Update dependencies
go mod tidy
```

### Integration Testing
```bash
# Test with main toolkit
cd ../..  # Go to main toolkit directory
go build -o toolkit ./cmd/toolkit
./toolkit list providers
./toolkit config generate provider chutes
./toolkit validate provider chutes provider-chutes-config.json
```

## Code Architecture

### Provider Interface Implementation
The `chutes.go` file implements the complete `toolkit.Provider` interface:

```go
type Provider interface {
    Name() string
    Chat(ctx context.Context, req ChatRequest) (ChatResponse, error)
    Embed(ctx context.Context, req EmbeddingRequest) (EmbeddingResponse, error)
    Rerank(ctx context.Context, req RerankRequest) (RerankResponse, error)
    DiscoverModels(ctx context.Context) ([]ModelInfo, error)
    ValidateConfig(config map[string]interface{}) error
}
```

### Configuration Management
The `builder.go` file provides robust configuration handling:

```go
type ConfigBuilder interface {
    Build(config map[string]interface{}) (interface{}, error)
    Validate(config interface{}) error
    Merge(base, override interface{}) (interface{}, error)
}
```

### HTTP Client Architecture
The `client.go` file implements a dedicated HTTP client:

```go
type Client struct {
    httpClient *http.Client
}

func (c *Client) ChatCompletion(ctx context.Context, req toolkit.ChatRequest) (toolkit.ChatResponse, error)
func (c *Client) CreateEmbeddings(ctx context.Context, req toolkit.EmbeddingRequest) (toolkit.EmbeddingResponse, error)
func (c *Client) CreateRerank(ctx context.Context, req toolkit.RerankRequest) (toolkit.RerankResponse, error)
func (c *Client) GetModels(ctx context.Context) ([]ModelInfo, error)
```

### Model Discovery System
The `discovery.go` file provides intelligent model discovery:

```go
type ChutesCapabilityInferrer struct{}
type ChutesModelFormatter struct{}
type Discovery struct {
    *discovery.BaseDiscovery
    client *Client
}
```

## Development Patterns

### Error Handling
Always use structured error wrapping:
```go
return nil, fmt.Errorf("failed to build config: %w", err)
```

### Logging
Use the toolkit's logging system:
```go
log.Printf("Chutes: Performing chat completion with model %s", req.Model)
```

### Configuration Validation
Validate all required fields:
```go
if c.APIKey == "" {
    return fmt.Errorf("api_key is required")
}
```

### Context Handling
Always respect context cancellation:
```go
select {
case <-ctx.Done():
    return ctx.Err()
default:
}
```

## Testing Guidelines

### Unit Test Structure
Each component should have comprehensive tests:
```go
func TestChutesProvider(t *testing.T) {
    // Test provider creation
    // Test configuration validation
    // Test error cases
    // Test edge cases
}
```

### Mock Testing
Use the toolkit's mock utilities:
```go
mockProvider := toolkit.NewMockProvider("chutes")
mockProvider.SetChatResponse(expectedResponse)
```

### Integration Testing
Test with real API calls when possible:
```go
func TestChutesIntegration(t *testing.T) {
    if os.Getenv("CHUTES_API_KEY") == "" {
        t.Skip("CHUTES_API_KEY not set")
    }
    // Test real API integration
}
```

## Model Capability Inference

### Chat Model Detection
```go
chatTypeIndicators := []string{"chat", "text", "completion", "instruction", "instruct"}
chatIDIndicators := []string{"instruct", "chat", "qwen", "deepseek", "glm", "kimi"}
```

### Specialized Model Detection
```go
audioKeywords := []string{"tts", "audio", "speech", "voice"}
videoKeywords := []string{"t2v", "video", "i2v", "flux"}
visionKeywords := []string{"vl", "vision", "visual", "multimodal"}
```

### Context Window Inference
```go
if strings.Contains(modelLower, "deepseek") {
    if strings.Contains(modelLower, "r1") {
        return 131072
    }
    return 131072 // V3 series
}
```

## API Endpoints

### Base URL
- Production: `https://api.chutes.ai/v1`
- Configurable via `base_url` parameter

### Endpoints
- Chat Completions: `POST /chat/completions`
- Embeddings: `POST /embeddings`
- Rerank: `POST /rerank`
- Models: `GET /models`

### Authentication
```go
httpClient.SetAuth("Authorization", "Bearer "+apiKey)
```

## Configuration Options

### Required Parameters
- `api_key`: Chutes API key

### Optional Parameters
- `base_url`: API base URL (default: "https://api.chutes.ai/v1")
- `timeout`: Request timeout in milliseconds (default: 30000)
- `retries`: Number of retry attempts (default: 3)
- `rate_limit`: Rate limit in requests per minute (default: 60)

### Environment Variables
- `CHUTES_API_KEY`: API key for authentication

## Code Style Guidelines

### Naming Conventions
- Use camelCase for variables and functions
- Use PascalCase for exported types and functions
- Use descriptive names that explain purpose

### Import Organization
```go
import (
    // Standard library
    "context"
    "fmt"
    "log"
    
    // Third-party packages
    "github.com/some/package"
    
    // Internal packages
    "github.com/superagent/toolkit/pkg/toolkit"
    "github.com/superagent/toolkit/pkg/toolkit/common/http"
)
```

### Documentation
- Document all exported functions and types
- Use complete sentences in documentation
- Include examples where helpful
- Keep documentation up to date

## Common Development Tasks

### Adding New Model Support
1. Update capability inference in `discovery.go`
2. Add model-specific logic for context windows
3. Update tests with new model examples
4. Document the new models in README.md

### Updating API Endpoints
1. Modify endpoints in `client.go`
2. Update request/response structures if needed
3. Test with real API calls
4. Update documentation

### Enhancing Configuration
1. Update `Config` struct in `builder.go`
2. Add validation logic
3. Update tests
4. Document new options

## Debugging

### Enable Verbose Logging
Set environment variables for detailed logging:
```bash
export TOOLKIT_VERBOSE=1
```

### Common Issues
1. **Authentication Errors**: Check API key and base URL
2. **Model Not Found**: Verify model ID and availability
3. **Timeout Issues**: Increase timeout configuration
4. **Rate Limiting**: Check rate limit settings

### Testing with Real API
```bash
export CHUTES_API_KEY="your-api-key"
go test ./providers/chutes/... -v -run TestIntegration
```

## Performance Optimization

### Connection Pooling
The HTTP client uses connection pooling for efficiency.

### Retry Logic
Exponential backoff with jitter for retry attempts.

### Timeout Management
Configurable timeouts for different operation types.

### Memory Management
Proper cleanup and resource management.

## Security Considerations

### API Key Management
- Never hardcode API keys
- Use environment variables or secure configuration
- Mask API keys in logs and output

### Request Validation
- Validate all input parameters
- Sanitize user inputs
- Implement proper error handling

### Network Security
- Use HTTPS for all API calls
- Implement proper certificate validation
- Handle network timeouts appropriately

## Release Process

1. Update version numbers
2. Run full test suite
3. Update documentation
4. Create release notes
5. Tag the release
6. Update dependencies in main toolkit

## Support and Contributing

For support and contributions:
- Review existing issues and PRs
- Follow the coding standards
- Add tests for new features
- Update documentation
- Submit clear, focused pull requests