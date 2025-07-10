# Impresso Make-Based Processing Template

This repository provides a template for creating new processing pipelines within the Impresso project ecosystem. It demonstrates best practices for building scalable, distributed newspaper processing workflows using Make, Python, and S3 storage.

## Table of Contents

- [Overview](#overview)
- [Template Structure](#template-structure)
- [Quick Start](#quick-start)
- [Setup Instructions](#setup-instructions)
- [Processing a Single Newspaper](#processing-a-single-newspaper)
- [Template Usage](#template-usage)
- [Configuration](#configuration)
- [Build System](#build-system)
- [Contributing](#contributing)
- [About Impresso](#about-impresso)

## Overview

This template provides a complete framework for building newspaper processing pipelines that:

- **Scale Horizontally**: Process data across multiple machines without conflicts
- **Handle Large Datasets**: Efficiently process large collections using S3 and local stamp files
- **Maintain Consistency**: Ensure reproducible results with proper dependency management
- **Support Parallel Processing**: Utilize multi-core systems and distributed computing
- **Integrate with S3**: Seamlessly work with both local files and S3 storage

## Template Structure

```
├── README.md                    # This file
├── Makefile                     # Main build configuration
├── .env                         # Environment variables (create from dotenv.sample)
├── dotenv.sample               # Sample environment configuration
├── Pipfile                     # Python dependencies
├── lib/
│   └── cli_TEMPLATE.py         # Template CLI script
├── cookbook/                   # Build system components
│   ├── README.md              # Detailed cookbook documentation
│   ├── setup_TEMPLATE.mk      # Template-specific setup
│   ├── paths_TEMPLATE.mk      # Path definitions
│   ├── sync_TEMPLATE.mk       # Data synchronization
│   ├── processing_TEMPLATE.mk # Processing targets
│   └── ...                    # Other cookbook components
└── build.d/                   # Local build directory (auto-created)
```

## Quick Start

1. **Clone and set up the template:**

   ```bash
   git clone <your-template-repo>
   cd impresso-cookbook-template
   cp dotenv.sample .env
   # Edit .env with your S3 credentials
   ```

2. **Install dependencies:**

   ```bash
   make setup
   ```

3. **Process a single newspaper:**
   ```bash
   make newspaper NEWSPAPER=actionfem
   ```

## Setup Instructions

### Prerequisites

- Python 3.11+
- Make (GNU Make recommended)
- Git with git-lfs
- AWS CLI (optional, for direct S3 access)

### System Dependencies

**Ubuntu/Debian:**

```bash
sudo apt-get install -y make git-lfs parallel coreutils python3 python3-pip
```

**macOS:**

```bash
brew install make git-lfs parallel coreutils python3
```

### Environment Setup

1. **Create environment file:**

   ```bash
   cp dotenv.sample .env
   ```

2. **Configure S3 credentials in `.env`:**

   ```bash
   SE_ACCESS_KEY=your_s3_access_key
   SE_SECRET_KEY=your_s3_secret_key
   SE_HOST_URL=https://os.zhdk.cloud.switch.ch/
   ```

3. **Install Python dependencies:**

   ```bash
   # Using pipenv (recommended)
   pipenv install

   # Or using pip directly
   python3 -m pip install -r requirements.txt
   ```

4. **Run initial setup:**
   ```bash
   make setup
   ```

### Verify Installation

Test your setup with:

```bash
make test-aws                    # Test S3 connectivity
make newspaper-list-target       # Generate newspaper list
make help                        # Show available targets
```

## Processing a Single Newspaper

### Basic Processing

Process a single newspaper with default settings:

```bash
make newspaper NEWSPAPER=gazette-de-lausanne
```

### Step-by-Step Processing

You can also run individual steps:

1. **Sync input data:**

   ```bash
   make sync-input NEWSPAPER=gazette-de-lausanne
   ```

2. **Run processing:**

   ```bash
   make TEMPLATE-target NEWSPAPER=gazette-de-lausanne
   ```

3. **Sync output data:**
   ```bash
   make sync-output NEWSPAPER=gazette-de-lausanne
   ```

### Advanced Options

```bash
# Process with custom parallel settings
make newspaper NEWSPAPER=journal-de-geneve PARALLEL_JOBS=4

# Process with debug logging
make newspaper NEWSPAPER=actionfem LOGGING_LEVEL=DEBUG

# Process with dry-run (no S3 uploads)
make newspaper NEWSPAPER=gazette-de-lausanne PROCESSING_S3_OUTPUT_DRY_RUN=--s3-output-dry-run

# Process with custom build directory
make newspaper NEWSPAPER=actionfem BUILD_DIR=custom-build
```

## Template Usage

### Adapting the Template

1. **Copy the template files:**

   - `lib/cli_TEMPLATE.py` → `lib/cli_yourprocessing.py`
   - `cookbook/setup_TEMPLATE.mk` → `cookbook/setup_yourprocessing.mk`
   - `cookbook/paths_TEMPLATE.mk` → `cookbook/paths_yourprocessing.mk`
   - `cookbook/sync_TEMPLATE.mk` → `cookbook/sync_yourprocessing.mk`
   - `cookbook/processing_TEMPLATE.mk` → `cookbook/processing_yourprocessing.mk`

2. **Update the Makefile:**
   Replace `TEMPLATE` references with your processing name in [Makefile](Makefile)

3. **Implement your processing logic:**
   Modify the `TemplateProcessor` class in your CLI script to implement your specific processing requirements.

### CLI Script Template

The template CLI script ([lib/cli_TEMPLATE.py](lib/cli_TEMPLATE.py)) provides:

- **Smart I/O**: Seamless handling of local files and S3 URIs
- **Logging Integration**: Consistent logging with configurable levels
- **S3 Integration**: Automatic S3 client configuration
- **Error Handling**: Robust error handling with proper logging
- **Argument Parsing**: Standard command-line interface patterns

### Key Template Features

- **Separation of Concerns**: Processing logic separated from CLI infrastructure
- **Fail-Fast Principle**: Early configuration validation and error detection
- **Unified I/O Model**: Same code handles local files and S3 objects
- **Observability**: Comprehensive logging built-in
- **Environment-Aware**: S3 credentials from environment variables

## Configuration

### Environment Variables

Set these in your `.env` file:

- `SE_ACCESS_KEY`: S3 access key
- `SE_SECRET_KEY`: S3 secret key
- `SE_HOST_URL`: S3 endpoint URL
- `LOGGING_LEVEL`: Logging level (DEBUG, INFO, WARNING, ERROR)

### Processing Variables

Key configurable variables:

- `NEWSPAPER`: Target newspaper to process
- `BUILD_DIR`: Local build directory (default: `build.d`)
- `PARALLEL_JOBS`: Number of parallel jobs (auto-detected)
- `COLLECTION_JOBS`: Number of parallel newspaper collections
- `NEWSPAPER_YEAR_SORTING`: Processing order (`shuf` for random, `cat` for chronological)

### S3 Bucket Configuration

Configure S3 buckets in your paths file:

- `S3_BUCKET_REBUILT`: Input data bucket (default: `22-rebuilt-final`)
- `S3_BUCKET_TEMPLATE`: Output data bucket (default: `140-processed-data-sandbox`)

## Build System

### Core Targets

- `make help`: Show available targets
- `make setup`: Initialize environment
- `make newspaper`: Process single newspaper
- `make collection`: Process multiple newspapers in parallel
- `make all`: Complete processing pipeline with data sync

### Data Management

- `make sync`: Sync input and output data
- `make sync-input`: Download input data from S3
- `make sync-output`: Upload results to S3
- `make clean-build`: Remove build directory

### Parallel Processing

The system automatically detects CPU cores and configures parallel processing:

```bash
# Process collection with custom parallelization
make collection COLLECTION_JOBS=4 MAX_LOAD=8
```

### Build System Architecture

The build system uses:

- **Stamp Files**: Track processing state without downloading full datasets
- **S3 Integration**: Direct processing from/to S3 storage
- **Distributed Processing**: Multiple machines can work independently
- **Dependency Management**: Automatic dependency resolution via Make

For detailed build system documentation, see [cookbook/README.md](cookbook/README.md).

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test with `make newspaper NEWSPAPER=actionfem`
5. Submit a pull request

## About Impresso

### Impresso Project

[Impresso - Media Monitoring of the Past](https://impresso-project.ch) is an interdisciplinary research project that aims to develop and consolidate tools for processing and exploring large collections of media archives across modalities, time, languages and national borders.

The project is funded by:

- Swiss National Science Foundation (grants [CRSII5_173719](http://p3.snf.ch/project-173719) and [CRSII5_213585](https://data.snf.ch/grants/grant/213585))
- Luxembourg National Research Fund (grant 17498891)

### Copyright

Copyright (C) 2024 The Impresso team.

### License

This program is provided as open source under the [GNU Affero General Public License](https://github.com/impresso/impresso-pyindexation/blob/master/LICENSE) v3 or later.

---

<p align="center">
  <img src="https://github.com/impresso/impresso.github.io/blob/master/assets/images/3x1--Yellow-Impresso-Black-on-White--transparent.png?raw=true" width="350" alt="Impresso Project Logo"/>
</p>
