# modctl Developer Guide

This guide provides comprehensive information for developers working with modctl, including best practices, integration patterns, and advanced use cases.

## Table of Contents

- [Development Setup](#development-setup)
- [Integration Patterns](#integration-patterns)
- [Best Practices](#best-practices)
- [Use Cases](#use-cases)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## Development Setup

### Prerequisites

- Go 1.24.1 or later
- Git
- Docker (for testing with registries)

### Building from Source

```bash
# Clone the repository
git clone https://github.com/CloudNativeAI/modctl.git
cd modctl

# Build the binary
make build

# Or build with Go directly
go build -o modctl .

# Install globally
make install
```

### Development Dependencies

```bash
# Install development tools
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
go install github.com/vektra/mockery/v2@latest

# Run tests
make test

# Run linter
make lint

# Generate mocks
make generate
```

## Integration Patterns

### CI/CD Integration

#### GitHub Actions

```yaml
name: Model Build and Deploy
on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Install modctl
      run: |
        curl -L https://github.com/CloudNativeAI/modctl/releases/latest/download/modctl-linux-amd64.tar.gz | tar xz
        sudo mv modctl /usr/local/bin/
    
    - name: Login to registry
      run: modctl login -u ${{ secrets.REGISTRY_USER }} -p ${{ secrets.REGISTRY_PASS }} ${{ secrets.REGISTRY_URL }}
    
    - name: Build and push model
      run: |
        modctl build -t ${{ secrets.REGISTRY_URL }}/models/my-model:${{ github.sha }} --output-remote .
        modctl tag ${{ secrets.REGISTRY_URL }}/models/my-model:${{ github.sha }} ${{ secrets.REGISTRY_URL }}/models/my-model:latest
        modctl push ${{ secrets.REGISTRY_URL }}/models/my-model:latest
```

#### GitLab CI

```yaml
stages:
  - build
  - deploy

variables:
  MODEL_IMAGE: $CI_REGISTRY_IMAGE/model

build_model:
  stage: build
  image: golang:1.24-alpine
  before_script:
    - apk add --no-cache curl tar
    - curl -L https://github.com/CloudNativeAI/modctl/releases/latest/download/modctl-linux-amd64.tar.gz | tar xz
    - mv modctl /usr/local/bin/
    - modctl login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - modctl build -t $MODEL_IMAGE:$CI_COMMIT_SHA --output-remote .
    - modctl tag $MODEL_IMAGE:$CI_COMMIT_SHA $MODEL_IMAGE:latest
    - modctl push $MODEL_IMAGE:latest
  only:
    - main
```

### Docker Integration

#### Multi-stage Dockerfile

```dockerfile
# Build stage
FROM golang:1.24-alpine AS builder
RUN apk add --no-cache git curl tar
RUN curl -L https://github.com/CloudNativeAI/modctl/releases/latest/download/modctl-linux-amd64.tar.gz | tar xz \
    && mv modctl /usr/local/bin/

# Runtime stage
FROM alpine:latest
RUN apk add --no-cache ca-certificates
COPY --from=builder /usr/local/bin/modctl /usr/local/bin/
COPY Modelfile /workspace/
COPY model/ /workspace/model/
WORKDIR /workspace

# Build and deploy model
ARG REGISTRY_URL
ARG MODEL_NAME
ARG MODEL_TAG
ENV REGISTRY_URL=${REGISTRY_URL}
ENV MODEL_NAME=${MODEL_NAME}
ENV MODEL_TAG=${MODEL_TAG}

RUN modctl build -t ${REGISTRY_URL}/${MODEL_NAME}:${MODEL_TAG} --output-remote .
```

### Kubernetes Integration

#### Job for Model Building

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: model-build-job
spec:
  template:
    spec:
      containers:
      - name: modctl
        image: alpine:latest
        command: ["/bin/sh"]
        args:
        - -c
        - |
          apk add --no-cache curl tar
          curl -L https://github.com/CloudNativeAI/modctl/releases/latest/download/modctl-linux-amd64.tar.gz | tar xz
          mv modctl /usr/local/bin/
          cd /workspace
          modctl login -u $REGISTRY_USER -p $REGISTRY_PASS $REGISTRY_URL
          modctl build -t $REGISTRY_URL/models/$MODEL_NAME:$BUILD_ID --output-remote .
        env:
        - name: REGISTRY_USER
          valueFrom:
            secretKeyRef:
              name: registry-credentials
              key: username
        - name: REGISTRY_PASS
          valueFrom:
            secretKeyRef:
              name: registry-credentials
              key: password
        - name: REGISTRY_URL
          value: "myregistry.com"
        - name: MODEL_NAME
          value: "llama2-7b"
        - name: BUILD_ID
          value: "v1.0.0"
        volumeMounts:
        - name: workspace
          mountPath: /workspace
      volumes:
      - name: workspace
        persistentVolumeClaim:
          claimName: model-workspace-pvc
      restartPolicy: Never
```

## Best Practices

### Modelfile Best Practices

#### Organized Structure

```modelfile
# Model metadata
NAME llama2-7b-chat
ARCH transformer
FAMILY llama
FORMAT safetensors
PARAMSIZE 7b
PRECISION fp16
QUANTIZATION none

# Configuration files (order matters for dependencies)
CONFIG config.json
CONFIG generation_config.json
CONFIG tokenizer_config.json

# Model weights (use specific patterns)
MODEL model-*.safetensors
MODEL pytorch_model.bin

# Code and utilities
CODE tokenizer.py
CODE modeling_*.py
CODE configuration_*.py

# Documentation
DOC README.md
DOC LICENSE
DOC model_card.md
```

#### Environment-specific Modelfiles

```bash
# Development
Modelfile.dev
NAME llama2-7b-dev
CONFIG config.dev.json
MODEL model.bin

# Production
Modelfile.prod
NAME llama2-7b-prod
CONFIG config.prod.json
MODEL model-*.safetensors
```

### Security Best Practices

#### Credential Management

```bash
# Use environment variables
export MODCTL_REGISTRY_USER="username"
export MODCTL_REGISTRY_PASS="password"

# Or use credential helpers
modctl login --password-stdin myregistry.com < ~/.modctl/password

# For CI/CD, use secret management
# GitHub Actions: ${{ secrets.REGISTRY_PASSWORD }}
# GitLab CI: $CI_REGISTRY_PASSWORD
# Kubernetes: secretKeyRef
```

#### Registry Security

```bash
# Always use HTTPS in production
modctl login myregistry.com

# For development with self-signed certificates
modctl login --insecure localhost:5000

# Use specific tags, avoid 'latest' in production
modctl build -t myregistry.com/model:v1.2.3 .
```

### Performance Optimization

#### Large Model Handling

```bash
# Use output-remote for large models to save local space
modctl build -t myregistry.com/large-model:v1.0.0 --output-remote .

# Use attach for incremental updates
modctl attach new-config.json \
  -s myregistry.com/model:v1.0.0 \
  -t myregistry.com/model:v1.0.1 \
  --output-remote

# Pre-upload large files
modctl upload large-weights.bin -t myregistry.com/model:v1.0.0
```

#### Parallel Operations

```bash
# Build multiple platforms in parallel
modctl build -t myregistry.com/model:v1.0.0 \
  --platform linux/amd64,linux/arm64,darwin/amd64 .
```

### Version Management

#### Semantic Versioning

```bash
# Use semantic versioning
modctl build -t myregistry.com/model:v1.0.0 .
modctl build -t myregistry.com/model:v1.0.1 .  # patch
modctl build -t myregistry.com/model:v1.1.0 .  # minor
modctl build -t myregistry.com/model:v2.0.0 .  # major

# Tag releases
modctl tag myregistry.com/model:v1.0.0 myregistry.com/model:stable
modctl tag myregistry.com/model:v2.0.0-beta myregistry.com/model:beta
```

#### Automated Versioning

```bash
#!/bin/bash
# auto-version.sh
VERSION=$(git describe --tags --abbrev=0)
COMMIT=$(git rev-parse --short HEAD)
BUILD_TIME=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

modctl build \
  -t myregistry.com/model:${VERSION} \
  -t myregistry.com/model:${VERSION}-${COMMIT} \
  --annotation org.opencontainers.image.version="${VERSION}" \
  --annotation org.opencontainers.image.revision="${COMMIT}" \
  --annotation org.opencontainers.image.created="${BUILD_TIME}" \
  .
```

## Use Cases

### Machine Learning Workflow

#### Model Development Lifecycle

```bash
# 1. Initial model creation
mkdir my-llm-model
cd my-llm-model

# 2. Generate initial Modelfile
modctl modelfile generate . \
  --name my-llm \
  --arch transformer \
  --family custom \
  --format pytorch

# 3. Build development version
modctl build -t localhost:5000/my-llm:dev .

# 4. Test locally
modctl pull localhost:5000/my-llm:dev --extract-dir ./test-model
python test_model.py ./test-model

# 5. Build and push release
modctl build -t myregistry.com/my-llm:v1.0.0 --output-remote .
```

#### Model Versioning and Rollback

```bash
# Deploy new version
modctl build -t myregistry.com/model:v2.0.0 .
modctl tag myregistry.com/model:v2.0.0 myregistry.com/model:latest

# Rollback if needed
modctl tag myregistry.com/model:v1.9.0 myregistry.com/model:latest
```

### Multi-Environment Deployment

#### Development to Production Pipeline

```bash
#!/bin/bash
# deploy-pipeline.sh

MODEL_NAME="my-model"
VERSION=${1:-"latest"}

# Development
modctl build -t dev-registry.com/${MODEL_NAME}:${VERSION} .
modctl push dev-registry.com/${MODEL_NAME}:${VERSION}

# Staging (with specific configuration)
modctl attach staging-config.json \
  -s dev-registry.com/${MODEL_NAME}:${VERSION} \
  -t staging-registry.com/${MODEL_NAME}:${VERSION} \
  --output-remote

# Production (after validation)
if validate_model staging-registry.com/${MODEL_NAME}:${VERSION}; then
  modctl pull staging-registry.com/${MODEL_NAME}:${VERSION}
  modctl tag staging-registry.com/${MODEL_NAME}:${VERSION} \
         prod-registry.com/${MODEL_NAME}:${VERSION}
  modctl push prod-registry.com/${MODEL_NAME}:${VERSION}
fi
```

### Model Registry Management

#### Cleanup and Maintenance

```bash
#!/bin/bash
# cleanup-models.sh

# Remove old development builds (keep last 5)
modctl list | grep ":dev-" | tail -n +6 | while read line; do
  MODEL=$(echo $line | awk '{print $1":"$2}')
  modctl rm $MODEL
done

# Prune unused blobs
modctl prune --remove-untagged

# Archive old models
ARCHIVE_DATE=$(date -d '30 days ago' +%Y-%m-%d)
modctl list | while read line; do
  CREATED=$(echo $line | awk '{print $5}')
  if [[ "$CREATED" < "$ARCHIVE_DATE" ]]; then
    MODEL=$(echo $line | awk '{print $1":"$2}')
    # Move to archive registry
    modctl pull $MODEL
    modctl tag $MODEL archive-registry.com/$MODEL
    modctl push archive-registry.com/$MODEL
    modctl rm $MODEL
  fi
done
```

### Custom Integration

#### Using modctl as a Library

```go
package main

import (
    "context"
    "fmt"
    "log"
    
    "github.com/CloudNativeAI/modctl/pkg/backend"
    "github.com/CloudNativeAI/modctl/pkg/config"
    "github.com/CloudNativeAI/modctl/pkg/modelfile"
)

type ModelManager struct {
    backend backend.Backend
    ctx     context.Context
}

func NewModelManager(storageDir string) (*ModelManager, error) {
    backend, err := backend.New(storageDir)
    if err != nil {
        return nil, err
    }
    
    return &ModelManager{
        backend: backend,
        ctx:     context.Background(),
    }, nil
}

func (m *ModelManager) BuildModel(workspace, target string, options BuildOptions) error {
    // Generate Modelfile if it doesn't exist
    if !fileExists(filepath.Join(workspace, "Modelfile")) {
        mf, err := modelfile.NewModelfileByWorkspace(workspace, &config.GenerateConfig{
            Name:   options.Name,
            Arch:   options.Arch,
            Family: options.Family,
        })
        if err != nil {
            return err
        }
        
        content := mf.Content()
        err = os.WriteFile(filepath.Join(workspace, "Modelfile"), content, 0644)
        if err != nil {
            return err
        }
    }
    
    // Build the model
    buildConfig := &config.Build{
        OutputRemote: options.OutputRemote,
        Platform:     options.Platform,
        Annotations:  options.Annotations,
    }
    
    return m.backend.Build(m.ctx, "Modelfile", workspace, target, buildConfig)
}

func (m *ModelManager) DeployModel(source, target string) error {
    // Pull source model
    pullConfig := &config.Pull{}
    err := m.backend.Pull(m.ctx, source, pullConfig)
    if err != nil {
        return err
    }
    
    // Tag for new target
    err = m.backend.Tag(m.ctx, source, target)
    if err != nil {
        return err
    }
    
    // Push to target registry
    pushConfig := &config.Push{}
    return m.backend.Push(m.ctx, target, pushConfig)
}

type BuildOptions struct {
    Name         string
    Arch         string
    Family       string
    OutputRemote bool
    Platform     string
    Annotations  map[string]string
}
```

## Testing

### Unit Testing

```go
package main

import (
    "context"
    "testing"
    "os"
    "path/filepath"
    
    "github.com/CloudNativeAI/modctl/pkg/backend"
    "github.com/CloudNativeAI/modctl/pkg/config"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

func TestModelBuild(t *testing.T) {
    // Setup temporary workspace
    tmpDir := t.TempDir()
    
    // Create test files
    err := os.WriteFile(filepath.Join(tmpDir, "model.bin"), []byte("fake model"), 0644)
    require.NoError(t, err)
    
    err = os.WriteFile(filepath.Join(tmpDir, "config.json"), []byte(`{"model_type": "test"}`), 0644)
    require.NoError(t, err)
    
    // Create Modelfile
    modelfile := `NAME test-model
ARCH transformer
MODEL model.bin
CONFIG config.json`
    err = os.WriteFile(filepath.Join(tmpDir, "Modelfile"), []byte(modelfile), 0644)
    require.NoError(t, err)
    
    // Create backend
    storageDir := t.TempDir()
    backend, err := backend.New(storageDir)
    require.NoError(t, err)
    
    // Build model
    ctx := context.Background()
    buildConfig := &config.Build{
        OutputRemote: false,
    }
    
    err = backend.Build(ctx, "Modelfile", tmpDir, "test-registry.com/test-model:v1.0.0", buildConfig)
    require.NoError(t, err)
    
    // Verify model exists
    artifacts, err := backend.List(ctx)
    require.NoError(t, err)
    assert.Len(t, artifacts, 1)
    assert.Equal(t, "test-registry.com/test-model", artifacts[0].Repository)
    assert.Equal(t, "v1.0.0", artifacts[0].Tag)
}
```

### Integration Testing

```bash
#!/bin/bash
# integration-test.sh

set -e

# Setup test registry
docker run -d -p 5000:5000 --name test-registry registry:2

# Wait for registry to start
sleep 5

# Test basic workflow
echo "Testing basic workflow..."

# Create test model
mkdir -p test-model
echo '{"model_type": "test"}' > test-model/config.json
echo "fake model data" > test-model/model.bin

# Generate Modelfile
modctl modelfile generate test-model \
  --name test-model \
  --arch transformer

# Build model
modctl build -t localhost:5000/test-model:v1.0.0 test-model

# Push model
modctl push localhost:5000/test-model:v1.0.0 --insecure

# Pull model
modctl rm localhost:5000/test-model:v1.0.0
modctl pull localhost:5000/test-model:v1.0.0 --insecure

# Extract model
mkdir -p extracted
modctl extract localhost:5000/test-model:v1.0.0 --output extracted

# Verify extracted files
test -f extracted/config.json
test -f extracted/model.bin

echo "Integration test passed!"

# Cleanup
docker stop test-registry
docker rm test-registry
rm -rf test-model extracted
```

## Troubleshooting

### Common Issues

#### Build Failures

```bash
# Check Modelfile syntax
modctl modelfile validate Modelfile

# Verify file patterns
ls -la | grep -E "\.(bin|safetensors|json)$"

# Check workspace size limits
du -sh . | awk '{print $1}'
```

#### Registry Connection Issues

```bash
# Test registry connectivity
curl -k https://myregistry.com/v2/

# Check authentication
modctl login --insecure myregistry.com

# Verify credentials
cat ~/.modctl/config.json
```

#### Storage Issues

```bash
# Check available space
df -h ~/.modctl

# Clean up storage
modctl prune --dry-run
modctl prune

# Reset storage completely
rm -rf ~/.modctl
```

### Debugging

#### Enable Debug Logging

```bash
export MODCTL_LOG_LEVEL=debug
modctl build -t myregistry.com/model:debug .
```

#### Enable Profiling

```bash
modctl --pprof --pprof-addr=localhost:6060 build -t myregistry.com/model:profile . &
go tool pprof http://localhost:6060/debug/pprof/profile
```

### Performance Analysis

```bash
# Monitor resource usage
top -p $(pgrep modctl)

# Check network usage
nethogs

# Analyze build time
time modctl build -t myregistry.com/model:benchmark .
```

## Contributing

### Development Setup

```bash
# Fork and clone the repository
git clone https://github.com/yourusername/modctl.git
cd modctl

# Create a development branch
git checkout -b feature/my-feature

# Install development dependencies
make dev-setup

# Run tests
make test

# Run linter
make lint
```

### Code Style

- Follow Go best practices
- Use meaningful variable names
- Add comments for public functions
- Write unit tests for new features
- Update documentation

### Submitting Changes

1. Ensure all tests pass
2. Update documentation if needed
3. Add changelog entry
4. Submit pull request with clear description

### Building Documentation

```bash
# Generate API documentation
go doc -all > docs/api-generated.md

# Build documentation site
make docs

# Preview documentation
make docs-serve
```

This comprehensive developer guide covers the essential aspects of working with modctl in various development scenarios. For more specific use cases or advanced configurations, refer to the API reference and source code.