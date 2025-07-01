# modctl API Reference

This document provides comprehensive documentation for all public APIs, functions, and components in the modctl project.

## Table of Contents

- [Overview](#overview)
- [Core Interfaces](#core-interfaces)
- [Packages](#packages)
  - [Backend Package](#backend-package)
  - [Archiver Package](#archiver-package)
  - [Modelfile Package](#modelfile-package)
  - [Storage Package](#storage-package)
  - [Codec Package](#codec-package)
  - [Config Package](#config-package)
  - [Source Package](#source-package)
  - [Version Package](#version-package)
- [Command Line Interface](#command-line-interface)
- [Examples](#examples)

## Overview

modctl is a CLI tool for managing OCI model artifacts based on the [Model Spec](https://github.com/CloudNativeAI/model-spec). It provides functionality to build, pull, push, and manage AI model artifacts in OCI-compatible registries.

## Core Interfaces

### Backend Interface

The `Backend` interface is the main entry point for all modctl operations.

```go
type Backend interface {
    // Login logs into a registry
    Login(ctx context.Context, registry, username, password string, cfg *config.Login) error
    
    // Logout logs out from a registry
    Logout(ctx context.Context, registry string) error
    
    // Build builds user materials into a model artifact following Model Spec
    Build(ctx context.Context, modelfilePath, workDir, target string, cfg *config.Build) error
    
    // Pull pulls an artifact from a registry
    Pull(ctx context.Context, target string, cfg *config.Pull) error
    
    // Push pushes an image to the registry
    Push(ctx context.Context, target string, cfg *config.Push) error
    
    // Attach attaches user materials into an existing model artifact
    Attach(ctx context.Context, filepath string, cfg *config.Attach) error
    
    // Upload uploads a file to a model artifact repository in advance
    Upload(ctx context.Context, filepath string, cfg *config.Upload) error
    
    // Fetch fetches partial files to the output
    Fetch(ctx context.Context, target string, cfg *config.Fetch) error
    
    // List lists all model artifacts
    List(ctx context.Context) ([]*ModelArtifact, error)
    
    // Remove deletes a model artifact
    Remove(ctx context.Context, target string) (string, error)
    
    // Prune removes unused blobs and cleans up storage
    Prune(ctx context.Context, dryRun, removeUntagged bool) error
    
    // Inspect inspects a model artifact
    Inspect(ctx context.Context, target string, cfg *config.Inspect) (*InspectedModelArtifact, error)
    
    // Extract extracts a model artifact
    Extract(ctx context.Context, target string, cfg *config.Extract) error
    
    // Tag creates a new tag that refers to the source model artifact
    Tag(ctx context.Context, source, target string) error
    
    // Nydusify converts the model artifact to nydus format
    Nydusify(ctx context.Context, target string) (string, error)
}
```

**Usage Example:**

```go
package main

import (
    "context"
    "github.com/CloudNativeAI/modctl/pkg/backend"
    "github.com/CloudNativeAI/modctl/pkg/config"
)

func main() {
    // Create a new backend instance
    backend, err := backend.New("/path/to/storage")
    if err != nil {
        panic(err)
    }
    
    ctx := context.Background()
    
    // Login to registry
    loginCfg := &config.Login{
        PlainHTTP: false,
        Insecure:  false,
    }
    err = backend.Login(ctx, "registry.example.com", "username", "password", loginCfg)
    if err != nil {
        panic(err)
    }
    
    // Build a model artifact
    buildCfg := &config.Build{
        OutputRemote: false,
        Platform:     "linux/amd64",
    }
    err = backend.Build(ctx, "Modelfile", "/workspace", "registry.example.com/model:v1.0.0", buildCfg)
    if err != nil {
        panic(err)
    }
}
```

### Modelfile Interface

The `Modelfile` interface provides access to modelfile parsing and metadata extraction.

```go
type Modelfile interface {
    // GetConfigs returns configuration file paths
    GetConfigs() []string
    
    // GetModels returns model file paths
    GetModels() []string
    
    // GetCodes returns code file paths
    GetCodes() []string
    
    // GetDatasets returns dataset file paths
    GetDatasets() []string
    
    // GetDocs returns documentation file paths
    GetDocs() []string
    
    // GetName returns the model name
    GetName() string
    
    // GetArch returns the model architecture
    GetArch() string
    
    // GetFamily returns the model family
    GetFamily() string
    
    // GetFormat returns the model format
    GetFormat() string
    
    // GetParamsize returns the parameter size
    GetParamsize() string
    
    // GetPrecision returns the model precision
    GetPrecision() string
    
    // GetQuantization returns the quantization method
    GetQuantization() string
    
    // Content returns the raw modelfile content
    Content() []byte
}
```

## Packages

### Backend Package

**Location:** `pkg/backend/`

The backend package provides the core implementation for all modctl operations.

#### Functions

##### `New(storageDir string) (Backend, error)`

Creates a new backend instance with the specified storage directory.

**Parameters:**
- `storageDir`: Path to the storage directory for caching artifacts

**Returns:**
- `Backend`: A new backend instance
- `error`: Error if storage initialization fails

**Example:**

```go
backend, err := backend.New("/home/user/.modctl")
if err != nil {
    log.Fatal(err)
}
```

#### Types

##### `ModelArtifact`

Represents a model artifact with metadata.

```go
type ModelArtifact struct {
    Repository  string
    Tag         string
    Digest      string
    Size        int64
    CreatedAt   time.Time
    // Additional metadata fields
}
```

##### `InspectedModelArtifact`

Contains detailed information about a model artifact.

```go
type InspectedModelArtifact struct {
    ModelArtifact
    Manifest    ocispec.Manifest
    Config      modelspec.Model
    Layers      []LayerInfo
}
```

### Archiver Package

**Location:** `pkg/archiver/`

The archiver package provides utilities for creating and extracting tar archives.

#### Functions

##### `Tar(srcPath string, workDir string) (io.Reader, error)`

Creates a tar archive of the specified path.

**Parameters:**
- `srcPath`: Path to the file or directory to archive
- `workDir`: Working directory for relative path calculation

**Returns:**
- `io.Reader`: Stream containing the tar archive
- `error`: Error if archiving fails

**Example:**

```go
reader, err := archiver.Tar("/path/to/model", "/workspace")
if err != nil {
    log.Fatal(err)
}
defer reader.Close()
```

##### `Untar(reader io.Reader, destPath string) error`

Extracts a tar archive to the specified destination.

**Parameters:**
- `reader`: Stream containing the tar archive
- `destPath`: Destination directory for extraction

**Returns:**
- `error`: Error if extraction fails

**Example:**

```go
file, err := os.Open("model.tar")
if err != nil {
    log.Fatal(err)
}
defer file.Close()

err = archiver.Untar(file, "/extract/path")
if err != nil {
    log.Fatal(err)
}
```

### Modelfile Package

**Location:** `pkg/modelfile/`

The modelfile package handles parsing and generation of Modelfiles.

#### Functions

##### `NewModelfile(path string) (Modelfile, error)`

Creates a new Modelfile instance by parsing an existing Modelfile.

**Parameters:**
- `path`: Path to the Modelfile

**Returns:**
- `Modelfile`: Parsed modelfile interface
- `error`: Error if parsing fails

**Example:**

```go
mf, err := modelfile.NewModelfile("./Modelfile")
if err != nil {
    log.Fatal(err)
}

fmt.Printf("Model name: %s\n", mf.GetName())
fmt.Printf("Architecture: %s\n", mf.GetArch())
```

##### `NewModelfileByWorkspace(workspace string, config *configmodelfile.GenerateConfig) (Modelfile, error)`

Generates a Modelfile by analyzing a workspace directory.

**Parameters:**
- `workspace`: Path to the workspace directory
- `config`: Configuration for Modelfile generation

**Returns:**
- `Modelfile`: Generated modelfile interface
- `error`: Error if generation fails

**Example:**

```go
genConfig := &configmodelfile.GenerateConfig{
    Name:         "my-model",
    Arch:         "transformer",
    Family:       "llama",
    Format:       "safetensors",
    ParamSize:    "7b",
    Precision:    "fp16",
    Quantization: "none",
}

mf, err := modelfile.NewModelfileByWorkspace("/path/to/model", genConfig)
if err != nil {
    log.Fatal(err)
}

// Save generated Modelfile
content := mf.Content()
err = os.WriteFile("Modelfile", content, 0644)
if err != nil {
    log.Fatal(err)
}
```

#### Constants

The package defines various file type patterns:

```go
// Configuration file patterns
ConfigFilePatterns = []string{"*.json", "*.yaml", "*.yml", "*.toml"}

// Model file patterns  
ModelFilePatterns = []string{"*.bin", "*.safetensors", "*.gguf", "*.pt", "*.pth"}

// Code file patterns
CodeFilePatterns = []string{"*.py", "*.js", "*.ts", "*.go", "*.cpp", "*.c"}

// Documentation patterns
DocFilePatterns = []string{"*.md", "*.txt", "*.rst"}
```

### Storage Package

**Location:** `pkg/storage/`

The storage package provides an abstraction layer for artifact storage.

#### Interface

```go
type Storage interface {
    // Methods for storing and retrieving OCI artifacts
    GetManifest(ctx context.Context, repo, reference string) (*ocispec.Manifest, error)
    PutManifest(ctx context.Context, repo, reference string, manifest *ocispec.Manifest) error
    GetBlob(ctx context.Context, repo, digest string) (io.ReadCloser, error)
    PutBlob(ctx context.Context, repo string, desc ocispec.Descriptor, reader io.Reader) error
    // Additional storage methods...
}
```

#### Functions

##### `New(storageType Type, storageDir string, opts ...Option) (Storage, error)`

Creates a new storage instance.

**Parameters:**
- `storageType`: Type of storage (currently supports local filesystem)
- `storageDir`: Directory for storage
- `opts`: Optional configuration options

**Returns:**
- `Storage`: Storage interface implementation
- `error`: Error if initialization fails

### Codec Package

**Location:** `pkg/codec/`

The codec package provides encoding/decoding capabilities for different file formats.

#### Interface

```go
type Codec interface {
    // Encode encodes a file for storage
    Encode(targetFilePath, workDirPath string) (io.Reader, error)
    
    // Decode decodes a file from storage
    Decode(outputDir, filePath string, reader io.Reader, desc ocispec.Descriptor) error
}
```

#### Available Codecs

- **Raw Codec**: Handles files without compression
- **Tar Codec**: Handles tar-compressed files

### Config Package

**Location:** `pkg/config/`

The config package defines configuration structures for all operations.

#### Key Configuration Types

##### `Build`

Configuration for build operations.

```go
type Build struct {
    OutputRemote bool              // Output directly to remote registry
    Platform     string            // Target platform
    PlainHTTP    bool             // Use plain HTTP
    Insecure     bool             // Allow insecure connections
    Annotations  map[string]string // Custom annotations
}
```

##### `Pull`

Configuration for pull operations.

```go
type Pull struct {
    ExtractDir        string    // Directory to extract to
    ExtractFromRemote bool      // Extract directly from remote
    PlainHTTP         bool      // Use plain HTTP
    Insecure          bool      // Allow insecure connections
    Platform          string    // Target platform
    Hooks             PullHooks // Hooks for pull events
}
```

##### `Push`

Configuration for push operations.

```go
type Push struct {
    PlainHTTP bool   // Use plain HTTP
    Insecure  bool   // Allow insecure connections
    Platform  string // Target platform
}
```

### Source Package

**Location:** `pkg/source/`

The source package provides source control integration.

#### Interface

```go
type Parser interface {
    // Parse extracts source information from workspace
    Parse(workspace string) (*Info, error)
}
```

#### Types

##### `Info`

Contains source control information.

```go
type Info struct {
    Type      string // Source type (git, etc.)
    URL       string // Repository URL
    Commit    string // Commit hash
    Branch    string // Branch name
    Tag       string // Tag name
    Timestamp time.Time // Commit timestamp
}
```

#### Functions

##### `NewParser(typ string) (Parser, error)`

Creates a new source parser for the specified type.

**Parameters:**
- `typ`: Parser type ("git", "zeta")

**Returns:**
- `Parser`: Parser implementation
- `error`: Error if type is unsupported

### Version Package

**Location:** `pkg/version/`

The version package provides version information.

#### Functions

##### `GetVersion() string`

Returns the current modctl version.

##### `GetBuildInfo() BuildInfo`

Returns detailed build information including commit, date, and platform.

## Command Line Interface

The CLI is organized into subcommands, each handling specific operations:

### Available Commands

- `build` - Build model artifacts from Modelfile
- `pull` - Pull model artifacts from registry
- `push` - Push model artifacts to registry
- `attach` - Attach files to existing artifacts
- `upload` - Upload files to repository
- `fetch` - Fetch partial files from artifacts
- `list` - List local model artifacts
- `rm` - Remove model artifacts
- `prune` - Clean up unused storage
- `inspect` - Inspect model artifact details
- `extract` - Extract model artifacts
- `tag` - Create artifact tags
- `login` - Login to registry
- `logout` - Logout from registry
- `version` - Show version information
- `modelfile` - Modelfile operations

### Global Flags

```bash
--storage-dir string    # Storage directory (default: ~/.modctl)
--no-progress          # Disable progress bars
--log-dir string       # Log directory (default: ~/.modctl/logs)
--log-level string     # Log level (default: info)
--pprof                # Enable pprof server
--pprof-addr string    # Pprof server address (default: localhost:6060)
```

## Examples

### Building a Model Artifact

```bash
# Create a Modelfile
cat > Modelfile << EOF
NAME llama2-7b
ARCH transformer
FAMILY llama
FORMAT safetensors
PARAMSIZE 7b
PRECISION fp16

CONFIG config.json
MODEL *.safetensors
CODE *.py
DOC README.md
EOF

# Build the artifact
modctl build -t registry.example.com/models/llama2:v1.0.0 -f Modelfile .
```

### Pulling and Extracting

```bash
# Pull and extract in one step
modctl pull registry.example.com/models/llama2:v1.0.0 \
  --extract-dir ./extracted \
  --extract-from-remote

# Or pull first, then extract
modctl pull registry.example.com/models/llama2:v1.0.0
modctl extract registry.example.com/models/llama2:v1.0.0 --output ./extracted
```

### Programmatic Usage

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

func main() {
    // Initialize backend
    backend, err := backend.New("/tmp/modctl-storage")
    if err != nil {
        log.Fatal(err)
    }
    
    ctx := context.Background()
    
    // Generate Modelfile from workspace
    genConfig := &config.GenerateConfig{
        Name:   "my-model",
        Arch:   "transformer", 
        Family: "gpt",
        Format: "pytorch",
    }
    
    mf, err := modelfile.NewModelfileByWorkspace("/path/to/model", genConfig)
    if err != nil {
        log.Fatal(err)
    }
    
    // Save generated Modelfile
    content := mf.Content()
    err = os.WriteFile("Modelfile", content, 0644)
    if err != nil {
        log.Fatal(err)
    }
    
    // Build the model
    buildConfig := &config.Build{
        OutputRemote: false,
        Platform:     "linux/amd64",
    }
    
    err = backend.Build(ctx, "Modelfile", "/path/to/model", "localhost:5000/my-model:latest", buildConfig)
    if err != nil {
        log.Fatal(err)
    }
    
    // List all artifacts
    artifacts, err := backend.List(ctx)
    if err != nil {
        log.Fatal(err)
    }
    
    for _, artifact := range artifacts {
        fmt.Printf("Repository: %s, Tag: %s, Size: %d bytes\n", 
            artifact.Repository, artifact.Tag, artifact.Size)
    }
}
```

### Advanced Configuration

```go
// Complex build with annotations and remote output
buildConfig := &config.Build{
    OutputRemote: true,
    Platform:     "linux/amd64,linux/arm64",
    PlainHTTP:    false,
    Insecure:     false,
    Annotations: map[string]string{
        "org.opencontainers.image.title":       "My Custom Model",
        "org.opencontainers.image.description": "A fine-tuned language model",
        "org.opencontainers.image.version":     "v1.0.0",
        "org.opencontainers.image.authors":     "AI Team <ai@example.com>",
    },
}

// Pull with custom hooks
pullConfig := &config.Pull{
    ExtractDir:        "/opt/models",
    ExtractFromRemote: true,
    Platform:          "linux/amd64", 
    Hooks: &CustomPullHooks{
        // Implement PullHooks interface
    },
}
```

This comprehensive API reference covers all major components and interfaces in the modctl project. For the most up-to-date information, please refer to the source code and inline documentation.