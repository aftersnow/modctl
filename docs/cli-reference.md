# modctl CLI Reference

This document provides detailed information about all modctl command-line interface commands.

## Table of Contents

- [Global Options](#global-options)
- [Commands](#commands)
  - [build](#build)
  - [pull](#pull)
  - [push](#push)
  - [attach](#attach)
  - [upload](#upload)
  - [fetch](#fetch)
  - [list](#list)
  - [rm](#rm)
  - [prune](#prune)
  - [inspect](#inspect)
  - [extract](#extract)
  - [tag](#tag)
  - [login](#login)
  - [logout](#logout)
  - [version](#version)
  - [modelfile](#modelfile)
- [Environment Variables](#environment-variables)
- [Configuration Files](#configuration-files)

## Global Options

These options are available for all commands:

```bash
--storage-dir string    Storage directory for modctl (default: ~/.modctl)
--no-progress          Disable progress bar display
--log-dir string       Log directory (default: ~/.modctl/logs)
--log-level string     Log level: debug, info, warn, error (default: info)
--pprof                Enable pprof server for debugging
--pprof-addr string    Pprof server address (default: localhost:6060)
-h, --help             Show help information
```

## Commands

### build

Build model artifacts from a Modelfile.

#### Synopsis

```bash
modctl build [OPTIONS] CONTEXT
```

#### Description

Builds a model artifact from the specified context directory using a Modelfile. The Modelfile describes the structure and metadata of the model artifact.

#### Options

```bash
-t, --tag string           Name and optionally tag in 'name:tag' format
-f, --file string          Name of the Modelfile (default: "Modelfile")
    --output-remote        Build and output directly to remote registry
    --platform strings     Set target platform for build (e.g., linux/amd64,linux/arm64)
    --plain-http           Use plain HTTP for registry communication
    --insecure             Allow insecure registry connections
    --annotation strings   Add custom annotations (key=value)
```

#### Examples

```bash
# Basic build
modctl build -t myregistry.com/model:v1.0.0 .

# Build with custom Modelfile
modctl build -t myregistry.com/model:v1.0.0 -f MyModelfile .

# Build and push directly to remote registry
modctl build -t myregistry.com/model:v1.0.0 --output-remote .

# Multi-platform build
modctl build -t myregistry.com/model:v1.0.0 --platform linux/amd64,linux/arm64 .

# Build with annotations
modctl build -t myregistry.com/model:v1.0.0 \
  --annotation org.opencontainers.image.title="My Model" \
  --annotation org.opencontainers.image.version="1.0.0" .
```

### pull

Pull model artifacts from a registry.

#### Synopsis

```bash
modctl pull [OPTIONS] NAME[:TAG]
```

#### Description

Pulls a model artifact from a remote registry to local storage. Optionally extracts the artifact directly to a specified directory.

#### Options

```bash
    --extract-dir string      Directory to extract the artifact to
    --extract-from-remote     Extract directly from remote without storing locally
    --platform string         Pull specific platform variant
    --plain-http              Use plain HTTP for registry communication
    --insecure                Allow insecure registry connections
```

#### Examples

```bash
# Basic pull
modctl pull myregistry.com/model:v1.0.0

# Pull and extract in one step
modctl pull myregistry.com/model:v1.0.0 --extract-dir ./my-model

# Pull and extract directly from remote (saves local storage)
modctl pull myregistry.com/model:v1.0.0 \
  --extract-dir ./my-model \
  --extract-from-remote

# Pull specific platform
modctl pull myregistry.com/model:v1.0.0 --platform linux/arm64
```

### push

Push model artifacts to a registry.

#### Synopsis

```bash
modctl push [OPTIONS] NAME[:TAG]
```

#### Description

Pushes a locally stored model artifact to a remote registry.

#### Options

```bash
    --platform string    Push specific platform variant
    --plain-http         Use plain HTTP for registry communication
    --insecure           Allow insecure registry connections
```

#### Examples

```bash
# Basic push
modctl push myregistry.com/model:v1.0.0

# Push specific platform
modctl push myregistry.com/model:v1.0.0 --platform linux/amd64

# Push to insecure registry
modctl push localhost:5000/model:v1.0.0 --insecure
```

### attach

Attach additional files to an existing model artifact.

#### Synopsis

```bash
modctl attach [OPTIONS] FILE
```

#### Description

Attaches a file to an existing model artifact, creating a new version without rebuilding the entire artifact.

#### Options

```bash
-s, --source string        Source artifact reference
-t, --target string        Target artifact reference
    --output-remote        Attach and output directly to remote registry
    --plain-http           Use plain HTTP for registry communication
    --insecure             Allow insecure registry connections
```

#### Examples

```bash
# Attach a file locally
modctl attach README.md \
  -s myregistry.com/model:v1.0.0 \
  -t myregistry.com/model:v1.0.1

# Attach and push directly to remote
modctl attach config.yaml \
  -s myregistry.com/model:v1.0.0 \
  -t myregistry.com/model:v1.0.1 \
  --output-remote
```

### upload

Upload files to a model artifact repository.

#### Synopsis

```bash
modctl upload [OPTIONS] FILE
```

#### Description

Uploads a file to a model artifact repository without creating a complete artifact. Useful for pre-uploading large files.

#### Options

```bash
-t, --target string       Target repository reference
    --plain-http          Use plain HTTP for registry communication
    --insecure            Allow insecure registry connections
```

#### Examples

```bash
# Upload a large model file
modctl upload model.bin -t myregistry.com/model:v1.0.0

# Upload to insecure registry
modctl upload weights.safetensors \
  -t localhost:5000/model:latest \
  --insecure
```

### fetch

Fetch specific files from a model artifact.

#### Synopsis

```bash
modctl fetch [OPTIONS] NAME[:TAG]
```

#### Description

Fetches specific files from a model artifact using glob patterns, without downloading the entire artifact.

#### Options

```bash
-o, --output string         Output directory for fetched files
-p, --patterns strings      File patterns to fetch (glob format)
    --plain-http            Use plain HTTP for registry communication
    --insecure              Allow insecure registry connections
```

#### Examples

```bash
# Fetch configuration files
modctl fetch myregistry.com/model:v1.0.0 \
  --output ./configs \
  --patterns "*.json"

# Fetch multiple file types
modctl fetch myregistry.com/model:v1.0.0 \
  --output ./files \
  --patterns "*.json,*.yaml,*.md"

# Fetch specific files
modctl fetch myregistry.com/model:v1.0.0 \
  --output ./docs \
  --patterns "README.md,LICENSE"
```

### list

List local model artifacts.

#### Synopsis

```bash
modctl list [OPTIONS]
```

#### Aliases

```bash
modctl ls
```

#### Description

Lists all model artifacts stored locally, showing repository, tag, size, and creation time.

#### Examples

```bash
# List all artifacts
modctl list

# Alternative command
modctl ls
```

#### Output Format

```
REPOSITORY                 TAG       DIGEST      SIZE        CREATED
myregistry.com/model      v1.0.0    sha256:...  1.2GB       2 hours ago
myregistry.com/model      v1.0.1    sha256:...  1.2GB       1 hour ago
localhost:5000/test       latest    sha256:...  500MB       30 minutes ago
```

### rm

Remove model artifacts from local storage.

#### Synopsis

```bash
modctl rm [OPTIONS] NAME[:TAG]...
```

#### Description

Removes one or more model artifacts from local storage.

#### Options

```bash
-f, --force    Force removal without confirmation
```

#### Examples

```bash
# Remove specific artifact
modctl rm myregistry.com/model:v1.0.0

# Remove multiple artifacts
modctl rm myregistry.com/model:v1.0.0 myregistry.com/model:v1.0.1

# Force removal without confirmation
modctl rm -f myregistry.com/model:v1.0.0
```

### prune

Clean up unused storage space.

#### Synopsis

```bash
modctl prune [OPTIONS]
```

#### Description

Removes unused blobs and temporary files to free up storage space.

#### Options

```bash
    --dry-run             Show what would be removed without actually removing
    --remove-untagged     Also remove untagged artifacts
```

#### Examples

```bash
# Show what would be pruned
modctl prune --dry-run

# Prune unused data
modctl prune

# Prune including untagged artifacts
modctl prune --remove-untagged
```

### inspect

Inspect model artifact details.

#### Synopsis

```bash
modctl inspect [OPTIONS] NAME[:TAG]
```

#### Description

Displays detailed information about a model artifact, including manifest, configuration, and layer information.

#### Options

```bash
    --format string       Output format: json, yaml (default: yaml)
    --plain-http          Use plain HTTP for registry communication
    --insecure            Allow insecure registry connections
```

#### Examples

```bash
# Inspect artifact in YAML format
modctl inspect myregistry.com/model:v1.0.0

# Inspect in JSON format
modctl inspect myregistry.com/model:v1.0.0 --format json

# Inspect from insecure registry
modctl inspect localhost:5000/model:latest --insecure
```

### extract

Extract model artifacts to a directory.

#### Synopsis

```bash
modctl extract [OPTIONS] NAME[:TAG]
```

#### Description

Extracts a model artifact to the specified output directory.

#### Options

```bash
-o, --output string       Output directory for extraction
    --plain-http          Use plain HTTP for registry communication
    --insecure            Allow insecure registry connections
```

#### Examples

```bash
# Extract to current directory
modctl extract myregistry.com/model:v1.0.0

# Extract to specific directory
modctl extract myregistry.com/model:v1.0.0 --output ./my-model

# Extract from insecure registry
modctl extract localhost:5000/model:latest \
  --output ./local-model \
  --insecure
```

### tag

Create tags for model artifacts.

#### Synopsis

```bash
modctl tag SOURCE_NAME[:TAG] TARGET_NAME[:TAG]
```

#### Description

Creates a new tag that refers to an existing model artifact.

#### Examples

```bash
# Create a new tag
modctl tag myregistry.com/model:v1.0.0 myregistry.com/model:latest

# Tag with different repository
modctl tag myregistry.com/model:v1.0.0 myregistry.com/model-prod:v1.0.0
```

### login

Login to a registry.

#### Synopsis

```bash
modctl login [OPTIONS] SERVER
```

#### Description

Logs into a registry server for authentication.

#### Options

```bash
-u, --username string     Username for registry
-p, --password string     Password for registry
    --password-stdin      Read password from stdin
    --plain-http          Use plain HTTP for registry communication
    --insecure            Allow insecure registry connections
```

#### Examples

```bash
# Login with username and password
modctl login -u myuser -p mypass myregistry.com

# Login with password from stdin
echo "mypass" | modctl login -u myuser --password-stdin myregistry.com

# Login to insecure registry
modctl login -u myuser -p mypass localhost:5000 --insecure
```

### logout

Logout from a registry.

#### Synopsis

```bash
modctl logout [SERVER]
```

#### Description

Logs out from a registry server, removing stored credentials.

#### Examples

```bash
# Logout from specific registry
modctl logout myregistry.com

# Logout from all registries
modctl logout
```

### version

Show version information.

#### Synopsis

```bash
modctl version [OPTIONS]
```

#### Description

Displays version and build information for modctl.

#### Options

```bash
    --format string    Output format: text, json, yaml (default: text)
```

#### Examples

```bash
# Show version
modctl version

# Show version in JSON format
modctl version --format json
```

### modelfile

Modelfile operations.

#### Synopsis

```bash
modctl modelfile COMMAND [OPTIONS]
```

#### Available Commands

- `generate` - Generate a Modelfile from workspace

#### Examples

```bash
# Generate Modelfile from current directory
modctl modelfile generate .

# Generate with custom configuration
modctl modelfile generate . \
  --name my-model \
  --arch transformer \
  --family llama \
  --format safetensors
```

## Environment Variables

modctl respects the following environment variables:

- `MODCTL_STORAGE_DIR` - Default storage directory
- `MODCTL_LOG_LEVEL` - Default log level
- `MODCTL_LOG_DIR` - Default log directory
- `MODCTL_NO_PROGRESS` - Disable progress bars (set to "true")
- `MODCTL_REGISTRY_CONFIG` - Path to registry configuration file

## Configuration Files

### Registry Configuration

modctl uses Docker-compatible registry configuration stored in `~/.modctl/config.json`:

```json
{
  "auths": {
    "myregistry.com": {
      "auth": "base64-encoded-credentials"
    }
  }
}
```

### Storage Layout

Local storage is organized as follows:

```
~/.modctl/
├── blobs/
│   └── sha256/
│       └── [digest files]
├── repositories/
│   └── [registry]/
│       └── [repository]/
│           ├── _manifests/
│           └── _uploads/
├── config.json
└── logs/
    └── modctl.log
```

## Exit Codes

- `0` - Success
- `1` - General error
- `2` - Command line parsing error
- `3` - Configuration error
- `4` - Network error
- `5` - Authentication error

## Shell Completion

Generate shell completion scripts:

```bash
# Bash
modctl completion bash > /etc/bash_completion.d/modctl

# Zsh
modctl completion zsh > "${fpath[1]}/_modctl"

# Fish
modctl completion fish > ~/.config/fish/completions/modctl.fish

# PowerShell
modctl completion powershell > modctl.ps1
```