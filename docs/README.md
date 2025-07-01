# modctl Documentation

Welcome to the comprehensive documentation for modctl, a user-friendly CLI tool for managing OCI model artifacts based on the [Model Spec](https://github.com/CloudNativeAI/model-spec).

## Overview

modctl provides commands such as `build`, `pull`, `push`, and more, making it easy for users to convert their AI models into OCI artifacts and manage them throughout their lifecycle.

## Documentation Structure

### 📚 Core Documentation

| Document | Description | Audience |
|----------|-------------|----------|
| **[Getting Started](./getting-started.md)** | Quick start guide with installation and basic usage examples | New users |
| **[API Reference](./api-reference.md)** | Complete API documentation for all public interfaces and functions | Developers |
| **[CLI Reference](./cli-reference.md)** | Detailed command-line interface documentation with examples | All users |
| **[Modelfile Reference](./modelfile-reference.md)** | Complete Modelfile syntax reference and examples | Model builders |
| **[Developer Guide](./developer-guide.md)** | Best practices, integration patterns, and advanced use cases | Developers |

### 🚀 Quick Navigation

#### New to modctl?
Start with the **[Getting Started](./getting-started.md)** guide to learn:
- How to install modctl
- Basic workflow (build, pull, push)
- Creating your first Modelfile
- Common use cases

#### Building Models?
Check the **[Modelfile Reference](./modelfile-reference.md)** for:
- Complete syntax documentation
- File pattern examples
- Best practices for organizing model artifacts
- Validation and troubleshooting

#### Using the CLI?
Refer to the **[CLI Reference](./cli-reference.md)** for:
- Complete command documentation
- All available options and flags
- Usage examples for each command
- Environment variables and configuration

#### Developing with modctl?
Explore the **[Developer Guide](./developer-guide.md)** for:
- Integration with CI/CD systems
- Using modctl as a Go library
- Testing strategies
- Performance optimization

#### Need API Details?
Browse the **[API Reference](./api-reference.md)** for:
- All public interfaces and types
- Function signatures and examples
- Package-by-package documentation
- Code examples and usage patterns

## Key Concepts

### Model Artifacts
Model artifacts in modctl are OCI-compatible containers that bundle:
- **Model weights** (safetensors, pytorch, onnx, etc.)
- **Configuration files** (config.json, tokenizer configs)
- **Code** (modeling scripts, utilities)
- **Documentation** (README, model cards, licenses)
- **Metadata** (architecture, family, precision, etc.)

### Modelfile
A Modelfile is a text file that describes how to build a model artifact:
```modelfile
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
```

### Registries
modctl works with OCI-compatible registries to store and distribute model artifacts:
- Docker Hub
- Amazon ECR
- Google Container Registry
- Azure Container Registry
- Harbor
- Local registries

## Common Workflows

### 1. Model Development
```bash
# Generate Modelfile from existing model directory
modctl modelfile generate .

# Build model artifact
modctl build -t myregistry.com/model:v1.0.0 .

# Test locally
modctl pull myregistry.com/model:v1.0.0 --extract-dir ./test
```

### 2. Model Distribution
```bash
# Login to registry
modctl login myregistry.com

# Push model to registry
modctl push myregistry.com/model:v1.0.0

# Create additional tags
modctl tag myregistry.com/model:v1.0.0 myregistry.com/model:latest
```

### 3. Model Deployment
```bash
# Pull model for deployment
modctl pull myregistry.com/model:latest --extract-dir /opt/model

# Fetch specific files only
modctl fetch myregistry.com/model:latest --patterns "*.json" --output ./config
```

### 4. Model Maintenance
```bash
# List local models
modctl list

# Inspect model details
modctl inspect myregistry.com/model:v1.0.0

# Clean up unused storage
modctl prune
```

## Examples by Use Case

### Large Language Models (LLMs)
- [LLaMA 2 Example](./examples/llama2.md)
- [GPT Model Example](./examples/gpt.md)
- [Quantized Models](./examples/quantized.md)

### Computer Vision Models
- [ResNet Example](./examples/resnet.md)
- [Vision Transformer Example](./examples/vit.md)
- [Object Detection Models](./examples/detection.md)

### Multimodal Models
- [CLIP Example](./examples/clip.md)
- [Vision-Language Models](./examples/vl-models.md)

### Specialized Workflows
- [Fine-tuning Workflow](./examples/fine-tuning.md)
- [Model Versioning](./examples/versioning.md)
- [CI/CD Integration](./examples/cicd.md)

## Integration Guides

### Development Tools
- [VS Code Extension](./integrations/vscode.md)
- [Jupyter Notebooks](./integrations/jupyter.md)
- [Python SDK](./integrations/python.md)

### CI/CD Platforms
- [GitHub Actions](./integrations/github-actions.md)
- [GitLab CI](./integrations/gitlab-ci.md)
- [Jenkins](./integrations/jenkins.md)
- [Azure DevOps](./integrations/azure-devops.md)

### Cloud Platforms
- [AWS Integration](./integrations/aws.md)
- [Google Cloud](./integrations/gcp.md)
- [Azure](./integrations/azure.md)
- [Kubernetes](./integrations/kubernetes.md)

### ML Platforms
- [Hugging Face](./integrations/huggingface.md)
- [MLflow](./integrations/mlflow.md)
- [Weights & Biases](./integrations/wandb.md)

## Reference Materials

### Specifications
- [Model Spec](https://github.com/CloudNativeAI/model-spec) - The underlying specification for model artifacts
- [OCI Image Spec](https://github.com/opencontainers/image-spec) - OCI image format specification
- [OCI Distribution Spec](https://github.com/opencontainers/distribution-spec) - OCI registry API specification

### Related Projects
- [ORAS](https://oras.land/) - OCI Registry As Storage
- [Distribution](https://github.com/distribution/distribution) - Docker registry implementation
- [Buildah](https://buildah.io/) - Container building tool

## Getting Help

### Community
- [GitHub Discussions](https://github.com/CloudNativeAI/modctl/discussions) - Ask questions and share ideas
- [GitHub Issues](https://github.com/CloudNativeAI/modctl/issues) - Report bugs and request features
- [Discord Server](#) - Real-time community chat

### Support
- [FAQ](./faq.md) - Frequently asked questions
- [Troubleshooting](./troubleshooting.md) - Common issues and solutions
- [Migration Guide](./migration.md) - Migrating from other tools

### Contributing
- [Contributing Guide](../CONTRIBUTING.md) - How to contribute to modctl
- [Development Setup](./developer-guide.md#development-setup) - Setting up development environment
- [Code Style Guide](./code-style.md) - Coding standards and conventions

## Changelog and Releases

- [Changelog](../CHANGELOG.md) - All notable changes to modctl
- [Releases](https://github.com/CloudNativeAI/modctl/releases) - Download releases and release notes
- [Roadmap](./roadmap.md) - Planned features and improvements

---

*For the most up-to-date information, please refer to the [modctl repository](https://github.com/CloudNativeAI/modctl) on GitHub.*