# How-To: Adapting the Impresso Cookbook Template

This document provides a comprehensive step-by-step guide to adapt the Impresso Cookbook Template for your specific newspaper processing task, from initial setup to production deployment.

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Initial Setup](#2-initial-setup)
3. [Understanding the Template Structure](#3-understanding-the-template-structure)
4. [Testing the Template](#4-testing-the-template)
5. [Planning Your Adaptation](#5-planning-your-adaptation)
6. [Creating Your Adapted Pipeline](#6-creating-your-adapted-pipeline)
7. [Implementing Your Processing Logic](#7-implementing-your-processing-logic)
8. [Configuring Data Paths and S3](#8-configuring-data-paths-and-s3)
9. [Testing Your Adapted Pipeline](#9-testing-your-adapted-pipeline)
10. [Deployment and Production](#10-deployment-and-production)
11. [Troubleshooting](#11-troubleshooting)

## 1. Prerequisites

### System Requirements

Before starting, ensure your system meets these requirements:

**For Ubuntu/Debian:**

```bash
sudo apt-get update
sudo apt-get install -y make git git-lfs parallel coreutils python3.11 python3.11-pip python3.11-venv
```

**For macOS:**

```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install dependencies
brew install make git git-lfs parallel coreutils python3.11
```

### Required Credentials

Gather the following information before proceeding:

- **S3 Access Credentials**: Access key, secret key, and host URL
- **Processing Requirements**: What type of newspaper processing you need to implement
- **Input/Output Data**: Understanding of your data sources and expected outputs

## 2. Initial Setup

### Step 2.1: Create Your New Repository

**Option A: Using GitHub CLI (Fastest)**

```bash
# Create repository from template using GitHub CLI
gh repo create impresso-myprocessing-pipeline --template impresso/impresso-cookbook-template --public
# Or for private repository:
# gh repo create impresso-myprocessing-pipeline --template impresso/impresso-cookbook-template --private

# Clone your new repository
git clone --recursive https://github.com/YOUR_USERNAME/impresso-myprocessing-pipeline.git
cd impresso-myprocessing-pipeline
```

**Option B: Using GitHub Web Interface**

1. **Navigate to the template repository:**
   Go to https://github.com/impresso/impresso-cookbook-template

2. **Create a new repository from template:**

   - Click the green "Use this template" button
   - Choose "Create a new repository"
   - Name your repository (e.g., `impresso-myprocessing-pipeline`)
   - Set visibility (public/private)
   - Click "Create repository from template"

3. **Clone your new repository:**
   ```bash
   git clone --recursive https://github.com/YOUR_USERNAME/impresso-myprocessing-pipeline.git
   cd impresso-myprocessing-pipeline
   ```

**Option C: Manual Setup**

1. **Create a new repository on GitHub/GitLab**
2. **Download and setup the template:**

   ```bash
   # Download template
   curl -L https://github.com/impresso/impresso-cookbook-template/archive/main.zip -o template.zip
   unzip template.zip
   mv impresso-cookbook-template-main impresso-myprocessing-pipeline
   cd impresso-myprocessing-pipeline

   # Initialize git and add your remote
   rm -rf .git
   git init
   git remote add origin https://github.com/YOUR_USERNAME/impresso-myprocessing-pipeline.git
   git add .
   git commit -m "Initial commit from template"
   git push -u origin main
   ```

### Step 2.2: Environment Configuration

1. **Create your environment file:**

   ```bash
   cp dotenv.sample .env
   ```

2. **Edit `.env` with your credentials:**

   ```bash
   # Required S3 Configuration
   SE_ACCESS_KEY=your_access_key_here
   SE_SECRET_KEY=your_secret_key_here
   SE_HOST_URL=https://os.zhdk.cloud.switch.ch/

   # Optional Configuration
   LOGGING_LEVEL=INFO
   BUILD_DIR=build.d
   ```

### Step 2.3: Python Environment Setup

**Option A: Using pipenv (recommended):**

```bash
pipenv install
pipenv shell
```

**Option B: Using venv:**

```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### Step 2.4: Initialize Template

```bash
make setup
```

## 3. Understanding the Template Structure

### Key Components

- **`Makefile`**: Main build configuration and entry point
- **`lib/cli_TEMPLATE.py`**: Template processing script (to be customized)
- **`cookbook/`**: Build system components (Make recipes and configurations)
- **`.env`**: Environment variables and S3 configuration
- **`build.d/`**: Local build directory (auto-created)

### Important Files to Customize

| File                              | Purpose                           | Customization Level                   |
| --------------------------------- | --------------------------------- | ------------------------------------- |
| `lib/cli_TEMPLATE.py`             | Main processing logic             | **High** - Core implementation        |
| `cookbook/processing_TEMPLATE.mk` | Processing targets and rules      | **High** - Define your pipeline       |
| `cookbook/paths_TEMPLATE.mk`      | Input/output paths and S3 buckets | **Medium** - Configure data sources   |
| `cookbook/setup_TEMPLATE.mk`      | Environment setup                 | **Low** - Usually minimal changes     |
| `cookbook/sync_TEMPLATE.mk`       | Data synchronization              | **Low** - Template handles most cases |

## 4. Testing the Template

### Step 4.1: Verify Installation

```bash
make help
```

You should see available targets and current configuration.

### Step 4.2: Test with Sample Data

```bash
# Test the template with a small newspaper
make newspaper NEWSPAPER=actionfem
```

### Step 4.3: Verify S3 Connectivity

```bash
# Test S3 sync (should list available newspapers)
make config
```

## 5. Planning Your Adaptation

### Step 5.1: Define Your Processing Pipeline

Answer these questions:

1. **What is your processing task?** (e.g., OCR correction, entity recognition, text classification)
2. **What are your inputs?** (newspaper articles, specific file formats, etc.)
3. **What are your outputs?** (processed files, metadata, statistics)
4. **What processing steps are required?** (preprocessing, main processing, post-processing)

### Step 5.2: Choose Your Pipeline Acronym

Select a short, descriptive acronym for your pipeline (3-15 characters):

- Good examples: `ocr`, `ner`, `classification`, `sentiment`
- Avoid spaces, special characters, or very long names

### Step 5.3: Identify Required Dependencies

List any additional Python packages or system tools your processing will need.

## 6. Creating Your Adapted Pipeline

### Step 6.1: Generate Adapted Files

Replace `YOUR_ACRONYM` with your chosen acronym:

```bash
export PROCESSING_ACRONYM=YOUR_ACRONYM
make -f cookbook/template-starter.mk
```

This creates:

- `Makefile.YOUR_ACRONYM`
- `lib/cli_YOUR_ACRONYM.py`
- `cookbook/*_YOUR_ACRONYM.mk` files

### Step 6.2: Update Dependencies (if needed)

If your processing requires additional packages:

1. **Add to Pipfile:**

   ```toml
   [packages]
   # ...existing packages...
   your-new-package = "*"
   another-package = ">=1.0.0"
   ```

2. **Install new dependencies:**
   ```bash
   pipenv install
   ```

### Step 6.3: Verify Adaptation

```bash
# Test with your new Makefile
make -f Makefile.YOUR_ACRONYM help
```

## 7. Implementing Your Processing Logic

### Step 7.1: Understand the CLI Template

Open `lib/cli_YOUR_ACRONYM.py` and examine the structure:

```python
def process_newspaper_year(newspaper: str, year: int, input_dir: Path, output_dir: Path):
    """Main processing function - customize this for your task"""
    pass
```

### Step 7.2: Implement Your Processing Function

Replace the template function with your processing logic:

```python
def process_newspaper_year(newspaper: str, year: int, input_dir: Path, output_dir: Path):
    """
    Process all articles for a given newspaper and year.

    Args:
        newspaper: Newspaper identifier (e.g., 'actionfem')
        year: Year to process (e.g., 1975)
        input_dir: Path to input data
        output_dir: Path where results should be saved
    """
    # Your processing implementation here
    print(f"Processing {newspaper} for year {year}")

    # Example: Process each article file
    for article_file in input_dir.glob("*.json"):
        # Load article data
        with open(article_file, 'r') as f:
            article_data = json.load(f)

        # Your processing logic here
        processed_data = your_processing_function(article_data)

        # Save results
        output_file = output_dir / f"processed_{article_file.name}"
        with open(output_file, 'w') as f:
            json.dump(processed_data, f, indent=2)
```

### Step 7.3: Add Error Handling and Logging

```python
import logging
from pathlib import Path

def process_newspaper_year(newspaper: str, year: int, input_dir: Path, output_dir: Path):
    """Process newspaper year with proper error handling"""

    # Setup logging
    logging.basicConfig(level=logging.INFO)
    logger = logging.getLogger(__name__)

    try:
        logger.info(f"Starting processing for {newspaper} {year}")

        # Ensure output directory exists
        output_dir.mkdir(parents=True, exist_ok=True)

        # Your processing logic here
        # ...

        logger.info(f"Successfully processed {newspaper} {year}")

    except Exception as e:
        logger.error(f"Error processing {newspaper} {year}: {str(e)}")
        raise
```

## 8. Configuring Data Paths and S3

### Step 8.1: Configure Input/Output Buckets

Edit `cookbook/paths_YOUR_ACRONYM.mk`:

```makefile
# Input data bucket (where you read source data from)
S3_BUCKET_INPUT := 22-rebuilt-final

# Output data bucket (where you save results)
S3_BUCKET_OUTPUT := YOUR-PROCESSING-BUCKET

# Processing paths
S3_INPUT_PREFIX := text-reuse/
S3_OUTPUT_PREFIX := YOUR_ACRONYM/
```

### Step 8.2: Configure Local Paths

```makefile
# Local build paths
BUILD_INPUT_DIR := $(BUILD_DIR)/input
BUILD_OUTPUT_DIR := $(BUILD_DIR)/output
BUILD_STAMPS_DIR := $(BUILD_DIR)/stamps
```

### Step 8.3: Define File Patterns

```makefile
# Input file patterns
INPUT_PATTERN := *.jsonl.bz2
OUTPUT_PATTERN := *.processed.json
```

## 9. Testing Your Adapted Pipeline

### Step 9.1: Test with Small Dataset

```bash
# Test your adapted pipeline
make -f Makefile.YOUR_ACRONYM newspaper NEWSPAPER=actionfem
```

### Step 9.2: Debug Processing Issues

```bash
# Enable verbose logging
export LOGGING_LEVEL=DEBUG
make -f Makefile.YOUR_ACRONYM newspaper NEWSPAPER=actionfem
```

### Step 9.3: Validate Output

```bash
# Check generated output files
ls -la build.d/output/actionfem/
```

### Step 9.4: Test Multiple Years

```bash
# Process specific years
make -f Makefile.YOUR_ACRONYM newspaper NEWSPAPER=actionfem YEARS="1975 1976"
```

## 10. Deployment and Production

### Step 10.1: Production Configuration

1. **Update `.env` for production:**

   ```bash
   # Production S3 settings
   S3_BUCKET_OUTPUT=your-production-bucket
   LOGGING_LEVEL=INFO
   PARALLEL_JOBS=8
   ```

2. **Test production settings:**
   ```bash
   make -f Makefile.YOUR_ACRONYM config
   ```

### Step 10.2: Large-Scale Processing

```bash
# Process entire collection
make -f Makefile.YOUR_ACRONYM collection COLLECTION_JOBS=4

# Process with custom parallelization
make -f Makefile.YOUR_ACRONYM collection COLLECTION_JOBS=4 MAX_LOAD=8
```

### Step 10.3: Monitoring and Logging

Set up proper logging for production:

```python
# In your cli_YOUR_ACRONYM.py
import logging
import sys

# Configure logging for production
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('processing.log'),
        logging.StreamHandler(sys.stdout)
    ]
)
```

### Step 10.4: Performance Optimization

Monitor and optimize:

```bash
# Monitor resource usage
htop

# Check processing speed
time make -f Makefile.YOUR_ACRONYM newspaper NEWSPAPER=actionfem
```

## 11. Troubleshooting

### Common Issues

**Issue: "No such file or directory" errors**

- Check S3 credentials in `.env`
- Verify bucket names in `paths_YOUR_ACRONYM.mk`
- Ensure input data exists: `make -f Makefile.YOUR_ACRONYM config`

**Issue: Python import errors**

- Activate virtual environment: `pipenv shell`
- Install dependencies: `pipenv install`
- Check Python path: `which python`

**Issue: Processing fails silently**

- Enable debug logging: `export LOGGING_LEVEL=DEBUG`
- Check stamp files: `ls -la build.d/stamps/`
- Manually test CLI: `python lib/cli_YOUR_ACRONYM.py --help`

**Issue: S3 sync problems**

- Test S3 connection: `aws s3 ls s3://your-bucket/ --endpoint-url=$SE_HOST_URL`
- Check credentials: `make -f Makefile.YOUR_ACRONYM config`
- Verify bucket permissions

### Getting Help

1. **Check the logs:** `tail -f processing.log`
2. **Review the cookbook documentation:** `cookbook/README.md`
3. **Test individual components:** Use Make targets step by step
4. **Contact the Impresso team:** Open an issue on the repository

### Performance Tips

- **Start small:** Always test with small newspapers first
- **Use parallel processing:** Set `COLLECTION_JOBS` and `PARALLEL_JOBS` appropriately
- **Monitor resources:** Use `htop` and `iostat` to monitor system load
- **Optimize I/O:** Consider local SSD storage for intensive processing

---

## Next Steps

After successfully adapting the template:

1. **Document your pipeline:** Update this how-to.md with your specific adaptations
2. **Add tests:** Create test cases for your processing logic
3. **Optimize performance:** Profile and optimize your processing code
4. **Share with the community:** Consider contributing improvements back to the template

For more detailed information about the build system, see [cookbook/README.md](cookbook/README.md).
