# Modelfile Reference

This document provides a comprehensive reference for the Modelfile format used by modctl to describe model artifacts.

## Table of Contents

- [Overview](#overview)
- [Syntax](#syntax)
- [Directives](#directives)
- [File Patterns](#file-patterns)
- [Examples](#examples)
- [Best Practices](#best-practices)
- [Validation](#validation)

## Overview

A Modelfile is a text file that describes how to build a model artifact. It contains directives that specify model metadata, file locations, and build instructions. The format is inspired by Dockerfile but designed specifically for AI/ML model artifacts.

### Basic Structure

```modelfile
# Model metadata
NAME model-name
ARCH architecture
FAMILY model-family
FORMAT file-format
PARAMSIZE parameter-size
PRECISION precision
QUANTIZATION quantization-method

# File specifications
CONFIG config-files
MODEL model-files
CODE code-files
DATASET dataset-files
DOC documentation-files
```

## Syntax

### Comments

```modelfile
# This is a comment
# Comments start with # and continue to the end of the line
```

### Line Structure

- Each directive must be on its own line
- Directives are case-sensitive
- Whitespace before directives is ignored
- Empty lines are ignored

### String Values

```modelfile
# Simple string (no quotes needed)
NAME my-model

# String with spaces (quotes recommended but not required)
NAME "my model with spaces"

# String with special characters
NAME "my-model_v1.0"
```

### File Patterns

File patterns support glob syntax:

```modelfile
# Exact filename
CONFIG config.json

# Wildcard patterns
MODEL *.safetensors
CODE *.py

# Character classes
MODEL model-[0-9]*.bin

# Multiple extensions
DOC *.{md,txt,rst}
```

## Directives

### Metadata Directives

#### NAME

Specifies the model name.

**Syntax:** `NAME <string>`

**Required:** Yes

**Examples:**
```modelfile
NAME llama2-7b
NAME "GPT-4 Turbo"
NAME bert-base-uncased
```

#### ARCH

Specifies the model architecture.

**Syntax:** `ARCH <string>`

**Required:** No

**Common Values:**
- `transformer`
- `cnn`
- `rnn`
- `lstm`
- `gru`
- `vae`
- `gan`
- `diffusion`

**Examples:**
```modelfile
ARCH transformer
ARCH cnn
ARCH "custom-architecture"
```

#### FAMILY

Specifies the model family or type.

**Syntax:** `FAMILY <string>`

**Required:** No

**Common Values:**
- `llama`
- `bert`
- `gpt`
- `t5`
- `resnet`
- `vit`
- `clip`
- `stable-diffusion`

**Examples:**
```modelfile
FAMILY llama
FAMILY bert
FAMILY "custom-family"
```

#### FORMAT

Specifies the primary model file format.

**Syntax:** `FORMAT <string>`

**Required:** No

**Common Values:**
- `safetensors`
- `pytorch`
- `tensorflow`
- `onnx`
- `huggingface`
- `pickle`

**Examples:**
```modelfile
FORMAT safetensors
FORMAT pytorch
FORMAT onnx
```

#### PARAMSIZE

Specifies the number of parameters in the model.

**Syntax:** `PARAMSIZE <string>`

**Required:** No

**Common Values:**
- `7b` (7 billion)
- `13b` (13 billion)
- `70b` (70 billion)
- `175b` (175 billion)
- `1.5b` (1.5 billion)

**Examples:**
```modelfile
PARAMSIZE 7b
PARAMSIZE 13b
PARAMSIZE 175b
```

#### PRECISION

Specifies the model precision/data type.

**Syntax:** `PRECISION <string>`

**Required:** No

**Common Values:**
- `fp32` (32-bit floating point)
- `fp16` (16-bit floating point)
- `bf16` (bfloat16)
- `int8` (8-bit integer)
- `int4` (4-bit integer)

**Examples:**
```modelfile
PRECISION fp16
PRECISION bf16
PRECISION int8
```

#### QUANTIZATION

Specifies the quantization method used.

**Syntax:** `QUANTIZATION <string>`

**Required:** No

**Common Values:**
- `none`
- `dynamic`
- `static`
- `awq`
- `gptq`
- `bnb`
- `ggml`

**Examples:**
```modelfile
QUANTIZATION none
QUANTIZATION awq
QUANTIZATION gptq
```

### File Directives

#### CONFIG

Specifies configuration files.

**Syntax:** `CONFIG <file-pattern>`

**Required:** No (can be used multiple times)

**Description:** Configuration files include model configuration, tokenizer configuration, and other metadata files.

**Examples:**
```modelfile
CONFIG config.json
CONFIG generation_config.json
CONFIG tokenizer_config.json
CONFIG *.json
```

#### MODEL

Specifies model weight/parameter files.

**Syntax:** `MODEL <file-pattern>`

**Required:** Yes (at least one)

**Description:** Model files contain the actual model weights and parameters.

**Examples:**
```modelfile
MODEL model.safetensors
MODEL pytorch_model.bin
MODEL model-*.safetensors
MODEL *.{bin,safetensors}
```

#### CODE

Specifies code files.

**Syntax:** `CODE <file-pattern>`

**Required:** No (can be used multiple times)

**Description:** Code files include model implementation, utilities, and helper scripts.

**Examples:**
```modelfile
CODE modeling_*.py
CODE tokenization_*.py
CODE *.py
CODE src/*.{py,js,ts}
```

#### DATASET

Specifies dataset files.

**Syntax:** `DATASET <file-pattern>`

**Required:** No (can be used multiple times)

**Description:** Dataset files include training data, evaluation data, and related files.

**Examples:**
```modelfile
DATASET train.jsonl
DATASET validation.jsonl
DATASET data/*.json
DATASET *.{jsonl,csv,parquet}
```

#### DOC

Specifies documentation files.

**Syntax:** `DOC <file-pattern>`

**Required:** No (can be used multiple times)

**Description:** Documentation files include README, model cards, licenses, and other documentation.

**Examples:**
```modelfile
DOC README.md
DOC LICENSE
DOC model_card.md
DOC docs/*.{md,rst,txt}
```

## File Patterns

### Glob Patterns

modctl supports standard glob patterns for file matching:

| Pattern | Matches |
|---------|---------|
| `*` | Any sequence of characters (except path separator) |
| `?` | Any single character |
| `[abc]` | Any character in the set |
| `[a-z]` | Any character in the range |
| `[!abc]` | Any character not in the set |
| `**` | Any sequence of directories |
| `{a,b}` | Either a or b |

### Examples

```modelfile
# All JSON files
CONFIG *.json

# Specific numbered files
MODEL model-[0-9]*.safetensors

# Multiple extensions
CODE *.{py,js,ts}

# Exclude specific files
MODEL *.bin
MODEL !test.bin

# Recursive patterns
DOC docs/**/*.md
```

### File Type Detection

modctl automatically categorizes files based on patterns:

#### Configuration Files
- `*.json`
- `*.yaml`, `*.yml`
- `*.toml`
- `*.ini`
- `*config*`

#### Model Files
- `*.safetensors`
- `*.bin`
- `*.pt`, `*.pth`
- `*.onnx`
- `*.h5`
- `*.pickle`, `*.pkl`

#### Code Files
- `*.py`
- `*.js`, `*.ts`
- `*.go`
- `*.cpp`, `*.c`, `*.h`
- `*.java`
- `*.r`, `*.R`

#### Documentation Files
- `*.md`
- `*.txt`
- `*.rst`
- `README*`
- `LICENSE*`
- `CHANGELOG*`

## Examples

### Basic LLaMA Model

```modelfile
# Basic LLaMA 2 7B model
NAME llama2-7b-chat
ARCH transformer
FAMILY llama
FORMAT safetensors
PARAMSIZE 7b
PRECISION fp16

# Configuration
CONFIG config.json
CONFIG generation_config.json
CONFIG tokenizer_config.json

# Model weights
MODEL model-*.safetensors

# Implementation code
CODE modeling_llama.py
CODE tokenization_llama.py

# Documentation
DOC README.md
DOC LICENSE
```

### Computer Vision Model

```modelfile
# ResNet image classifier
NAME resnet50-imagenet
ARCH cnn
FAMILY resnet
FORMAT pytorch
PARAMSIZE 25m
PRECISION fp32

# Model configuration
CONFIG config.json

# Pre-trained weights
MODEL pytorch_model.bin

# Implementation
CODE modeling_resnet.py
CODE image_processing.py

# Training data info
DATASET imagenet_classes.txt

# Documentation
DOC README.md
DOC model_card.md
```

### Multimodal Model

```modelfile
# CLIP vision-language model
NAME clip-vit-base-patch32
ARCH transformer
FAMILY clip
FORMAT safetensors
PARAMSIZE 400m
PRECISION fp16

# Model configuration
CONFIG config.json
CONFIG preprocessor_config.json

# Model components
MODEL vision_model.safetensors
MODEL text_model.safetensors

# Implementation
CODE modeling_clip.py
CODE image_processing_clip.py
CODE tokenization_clip.py

# Documentation
DOC README.md
DOC model_card.md
DOC LICENSE
```

### Quantized Model

```modelfile
# Quantized LLaMA model
NAME llama2-7b-awq
ARCH transformer
FAMILY llama
FORMAT safetensors
PARAMSIZE 7b
PRECISION int4
QUANTIZATION awq

# Configuration includes quantization settings
CONFIG config.json
CONFIG quantization_config.json

# Quantized weights
MODEL model.safetensors

# Quantization code
CODE quantization_utils.py
CODE modeling_llama_awq.py

# Documentation
DOC README.md
DOC quantization_notes.md
```

### Development Model

```modelfile
# Development version with additional tools
NAME my-model-dev
ARCH transformer
FAMILY custom
FORMAT pytorch
PARAMSIZE 1b
PRECISION fp32

# Configuration
CONFIG config.json

# Model files
MODEL model.bin

# Development code
CODE *.py
CODE notebooks/*.ipynb
CODE tests/*.py

# Development data
DATASET sample_data.json
DATASET eval_data.json

# Development documentation
DOC README.md
DOC DEVELOPMENT.md
DOC TODO.md
```

## Best Practices

### Organization

1. **Group related directives together**
   ```modelfile
   # Metadata first
   NAME model-name
   ARCH transformer
   FAMILY llama
   
   # Configuration files
   CONFIG config.json
   CONFIG tokenizer_config.json
   
   # Model files
   MODEL *.safetensors
   
   # Code files
   CODE *.py
   
   # Documentation last
   DOC README.md
   DOC LICENSE
   ```

2. **Use descriptive names**
   ```modelfile
   NAME llama2-7b-chat-finetune
   # Better than: NAME model
   ```

3. **Be specific with file patterns**
   ```modelfile
   MODEL model-*.safetensors
   # Better than: MODEL *
   ```

### Metadata Completeness

1. **Always specify NAME**
   ```modelfile
   NAME my-model  # Required
   ```

2. **Include architecture and family for ML models**
   ```modelfile
   ARCH transformer
   FAMILY llama
   ```

3. **Specify format and precision**
   ```modelfile
   FORMAT safetensors
   PRECISION fp16
   ```

### File Organization

1. **Order files by importance**
   ```modelfile
   # Most important configs first
   CONFIG config.json
   CONFIG generation_config.json
   CONFIG tokenizer_config.json
   
   # Main model files
   MODEL model-*.safetensors
   
   # Helper files
   CODE modeling_*.py
   ```

2. **Use specific patterns**
   ```modelfile
   # Good - specific
   MODEL model-[0-9]*.safetensors
   CODE modeling_*.py
   
   # Avoid - too broad
   MODEL *
   CODE *
   ```

3. **Document special files**
   ```modelfile
   # Include important documentation
   DOC README.md         # Always include
   DOC LICENSE          # Legal requirements
   DOC model_card.md    # Model information
   ```

### Version Management

1. **Include version in name for releases**
   ```modelfile
   NAME llama2-7b-v1.0.0
   ```

2. **Use environment-specific Modelfiles**
   ```bash
   Modelfile.dev    # Development
   Modelfile.prod   # Production
   Modelfile.test   # Testing
   ```

3. **Document changes**
   ```modelfile
   DOC CHANGELOG.md
   DOC VERSION_NOTES.md
   ```

## Validation

### Syntax Validation

modctl provides built-in validation:

```bash
# Validate Modelfile syntax
modctl modelfile validate Modelfile

# Validate with specific workspace
modctl modelfile validate Modelfile --workspace .
```

### Common Validation Errors

1. **Missing required fields**
   ```
   Error: NAME directive is required
   ```

2. **Duplicate directives**
   ```
   Error: duplicate NAME directive on line 5
   ```

3. **Invalid file patterns**
   ```
   Error: file pattern '*.json' matches no files
   ```

4. **File not found**
   ```
   Error: specified file 'config.json' not found in workspace
   ```

### Validation Rules

1. **NAME is required**
2. **At least one MODEL directive is required**
3. **Metadata directives cannot be duplicated**
4. **File patterns must match existing files**
5. **Workspace must not exceed size limits**

### File Limits

- Maximum workspace size: 50GB
- Maximum single file size: 10GB
- Maximum number of files: 10,000

### Manual Validation

```bash
# Check file patterns manually
ls -la *.safetensors
ls -la *.json

# Verify file sizes
du -sh *

# Count files
find . -type f | wc -l
```

## Advanced Features

### Conditional Directives

While not directly supported, you can use multiple Modelfiles:

```bash
# Different configurations for different environments
if [ "$ENV" = "production" ]; then
  modctl build -f Modelfile.prod -t registry.com/model:prod .
else
  modctl build -f Modelfile.dev -t registry.com/model:dev .
fi
```

### Template Modelfiles

Use shell scripts to generate dynamic Modelfiles:

```bash
#!/bin/bash
# generate-modelfile.sh

cat > Modelfile << EOF
NAME ${MODEL_NAME:-my-model}
ARCH ${MODEL_ARCH:-transformer}
FAMILY ${MODEL_FAMILY:-custom}
FORMAT ${MODEL_FORMAT:-safetensors}
PARAMSIZE ${MODEL_PARAMSIZE:-7b}
PRECISION ${MODEL_PRECISION:-fp16}

CONFIG config.json
MODEL *.safetensors
CODE *.py
DOC README.md
EOF
```

### Integration with Build Systems

```makefile
# Makefile integration
build-model:
	@echo "Validating Modelfile..."
	modctl modelfile validate Modelfile
	@echo "Building model..."
	modctl build -t $(REGISTRY)/$(MODEL_NAME):$(VERSION) .

.PHONY: build-model
```

This comprehensive Modelfile reference covers all aspects of creating and managing Modelfiles for AI/ML model artifacts. For the latest features and updates, refer to the modctl documentation and source code.