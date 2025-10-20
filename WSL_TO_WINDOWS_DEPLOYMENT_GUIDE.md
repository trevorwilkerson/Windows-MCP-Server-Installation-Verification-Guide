# WSL-to-Windows Development & Deployment Guide

**Complete workflow guide for Python projects developed on WSL and deployed to Windows 11 Enterprise**

**Version**: 2.0
**Last Updated**: 2025-10-20
**Audience**: Developers, New Hires, DevOps

---

## Table of Contents

1. [Overview](#overview)
2. [Environment Setup](#environment-setup)
3. [Project Repository Structure](#project-repository-structure)
4. [Creating a New Repository](#creating-a-new-repository)
5. [Virtual Environment Best Practices](#virtual-environment-best-practices)
6. [Development Workflow](#development-workflow)
7. [Deployment to Windows 11](#deployment-to-windows-11)
8. [Troubleshooting](#troubleshooting)
9. [New Hire Onboarding Checklist](#new-hire-onboarding-checklist)
10. [Template Scripts](#template-scripts)

---

## Overview

### The Challenge

We develop Python applications on **WSL (Windows Subsystem for Linux)** but deploy them to **Windows 11 Enterprise** machines where:
- Users typically don't have admin privileges
- Internet connectivity may be limited or restricted
- Virtual environments must be portable and self-contained

### The Solution

We maintain **dual virtual environments** in the same repository:
- `{project_name}_wsl_env/` - For WSL/Linux development
- `{project_name}_windows_env/` - For Windows production deployment

Both environments coexist peacefully and are gitignored.

### Key Principles

1. **Platform Detection**: Scripts automatically detect and adapt to the platform
2. **Timeout Protection**: Package installations use extended timeouts (300-600 sec)
3. **Batch Installation**: Large packages installed separately to prevent failures
4. **Version Constraints**: Explicit version ranges in requirements.txt
5. **Non-Interactive Mode**: Support `--force-reinstall` flag for automation
6. **Verification**: Always verify package installations with test imports

---

## Environment Setup

### Development Machine (WSL)

**Prerequisites:**
```bash
# Check WSL version
wsl --version

# Check Python version (3.8+ required)
python3 --version

# Install Git if not present
sudo apt update
sudo apt install git
```

**Recommended Tools:**
```bash
# VS Code with WSL extension
# Git Bash (for testing cross-platform scripts)
# Windows Terminal (for better WSL experience)
```

### Production Machine (Windows 11 Enterprise)

**Prerequisites:**
- Python 3.8+ installed and in PATH
- Git installed (or Git Bash)
- No admin privileges required

**Verify Setup:**
```cmd
python --version
git --version
```

---

## Project Repository Structure

### Standard Layout

```
my_project/
├── .gitignore                          # Ignore venvs, outputs, cache
├── README.md                           # User-facing quick start
├── CLAUDE.md                           # Technical documentation for Claude Code
├── requirements.txt                    # Python dependencies with versions
│
├── activate_{project}_env.sh           # WSL/Linux setup script
├── setup_windows_venv.bat              # Windows setup script
├── activate_quick.sh                   # Auto-generated quick activation (WSL)
├── activate_quick.bat                  # Auto-generated quick activation (Windows)
│
├── {project_name}_wsl_env/             # WSL virtual environment (gitignored)
├── {project_name}_windows_env/         # Windows virtual environment (gitignored)
│
├── src/                                # Source code
│   ├── __init__.py
│   └── main.py
│
├── tests/                              # Test suite
│   └── test_main.py
│
└── docs/                               # Additional documentation
    └── architecture.md
```

---

## Creating a New Repository

### Step 1: Initialize Repository

```bash
# Create project directory
mkdir my_new_project
cd my_new_project

# Initialize git
git init

# Create basic structure
mkdir src tests docs
touch README.md CLAUDE.md requirements.txt
```

### Step 2: Copy Template Scripts

Use the template scripts from the [Template Scripts](#template-scripts) section below, or copy from an existing repo:

```bash
# Copy from existing repo (e.g., AirTable_remove_dup_records)
cp /path/to/reference_repo/activate_*_env.sh ./activate_my_project_env.sh
cp /path/to/reference_repo/setup_windows_venv.bat ./

# Edit the scripts to update project name
# Change VENV_NAME variables:
#   VENV_NAME_WSL="my_project_wsl_env"
#   VENV_NAME_WIN="my_project_windows_env"
```

### Step 3: Create requirements.txt

```txt
# My Project Requirements
# Python 3.8+
#
# Installation: Use activate_my_project_env.sh or setup_windows_venv.bat
# Do not use pip install -r requirements.txt directly (missing timeout protection)

# Core dependencies
pandas>=1.3.0,<3.0

# Add your dependencies with version constraints
# package_name>=min_version,<max_version
```

### Step 4: Create .gitignore

```gitignore
# Virtual Environments
*_wsl_env/
*_windows_env/
venv/
env/
*.env

# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
dist/
*.egg-info/

# IDE
.vscode/
.idea/
*.swp

# OS
.DS_Store
Thumbs.db

# Project-specific outputs
output/
*.log
*.csv  # or whatever your outputs are
```

### Step 4b: Create .gitattributes (IMPORTANT!)

This prevents line ending issues when working across WSL and Windows:

```gitattributes
# Git attributes for proper line ending handling

# Shell scripts must use LF (Unix) line endings
*.sh text eol=lf

# Batch files must use CRLF (Windows) line endings
*.bat text eol=crlf

# Python files should use LF
*.py text eol=lf

# Markdown and text files
*.md text eol=lf
*.txt text eol=lf

# Data files
*.csv text eol=lf
*.json text eol=lf
*.yaml text eol=lf
*.yml text eol=lf
```

### Step 5: Create Documentation

**README.md** - User-facing quick start:
```markdown
# Project Name

Brief description of what this project does.

## Quick Start

### WSL/Linux Setup
\`\`\`bash
source ./activate_my_project_env.sh
\`\`\`

### Windows Setup
\`\`\`cmd
setup_windows_venv.bat
\`\`\`

### Run the Application
\`\`\`bash
python src/main.py
\`\`\`

## Requirements
- Python 3.8+
- 2GB RAM
- Internet for initial setup

## Troubleshooting
See CLAUDE.md for detailed documentation.
```

**CLAUDE.md** - Technical documentation for Claude Code:
```markdown
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
[Detailed description of architecture, data flow, etc.]

## Environment Setup
[Cross-platform setup instructions]

## Development Notes
[Important constraints, conventions, gotchas]
```

---

## Virtual Environment Best Practices

### Naming Convention

Always use the pattern:
```
{project_name}_wsl_env/       # WSL/Linux development
{project_name}_windows_env/   # Windows production
```

**Why?**
- Clear platform identification
- No conflicts when switching between WSL and Windows
- Consistent across all projects
- Easy to script and automate

### Installation Best Practices

**DO:**
- ✅ Use extended timeouts: `--timeout 300` or `--timeout 600`
- ✅ Install large packages separately (torch, tensorflow, etc.)
- ✅ Verify installations with test imports
- ✅ Use version constraints in requirements.txt
- ✅ Create quick activation scripts

**DON'T:**
- ❌ Run `pip install -r requirements.txt` without timeout protection
- ❌ Install all packages in one command (causes timeouts)
- ❌ Use `pip install package` without version constraints
- ❌ Commit virtual environments to git
- ❌ Mix WSL and Windows virtual environments

### Package Version Constraints

Always specify version ranges:
```txt
# Good
pandas>=1.3.0,<3.0
numpy>=1.21.0,<2.0
pydantic>=1.10,<2.0

# Bad
pandas  # Could break with major version update
pandas==1.3.5  # Too strict, prevents minor updates
```

**Special Cases:**
```txt
# Critical compatibility requirement (e.g., pydantic v1.x for spacy)
pydantic>=1.10,<2.0  # Must stay on v1.x

# Bleeding edge required
torch>=2.0.0  # Need specific feature from v2+

# Known breaking change
matplotlib>=3.8.0,<4.0  # v4 changes API
```

---

## Development Workflow

### Daily Workflow (WSL)

```bash
# 1. Navigate to project
cd /mnt/c/path/to/project

# 2. Activate virtual environment
source ./activate_quick.sh

# 3. Pull latest changes
git pull

# 4. Make changes, test, commit
# ... your work ...

# 5. Deactivate when done
deactivate
```

### Testing on Windows (Optional)

Before deployment, test on Windows if available:

```cmd
REM 1. Navigate to project (from Windows side)
cd C:\path\to\project

REM 2. Activate Windows environment
activate_quick.bat

REM 3. Test your code
python src/main.py

REM 4. Deactivate
deactivate
```

### Code Quality Checks

Before committing:

```bash
# Format code (if using black)
black src/ tests/

# Lint (if using flake8)
flake8 src/ tests/

# Run tests
pytest tests/

# Type check (if using mypy)
mypy src/
```

---

## Deployment to Windows 11

### Deployment Checklist

- [ ] All code committed and pushed to git
- [ ] requirements.txt is up to date
- [ ] Version constraints are specified
- [ ] Setup scripts tested on WSL
- [ ] Documentation is complete
- [ ] No hardcoded WSL paths in code

### Deployment Process

#### Step 1: Transfer Repository

**Method A: Git Clone (Recommended)**
```cmd
REM On Windows 11 machine
cd C:\Projects
git clone https://github.com/your/repo.git
cd repo
```

**Method B: File Copy**
```cmd
REM Copy from network share or USB
xcopy \\network\share\repo C:\Projects\repo /E /I
cd C:\Projects\repo
```

#### Step 2: Setup Virtual Environment

```cmd
REM Run the Windows setup script
setup_windows_venv.bat

REM Or non-interactive mode
setup_windows_venv.bat --force-reinstall
```

**What happens:**
1. Creates `{project}_windows_env/` directory
2. Installs Python packages with timeout protection
3. Downloads any models (e.g., spaCy)
4. Verifies all installations
5. Creates `activate_quick.bat` for easy activation

#### Step 3: Verify Installation

```cmd
REM Activate environment
activate_quick.bat

REM Verify Python packages
python -c "import pandas, numpy; print('OK: All packages available')"

REM Run application test
python src/main.py --help

REM Or run tests
pytest tests/
```

#### Step 4: Create Desktop Shortcut (Optional)

Create `run_app.bat`:
```bat
@echo off
cd /d "%~dp0"
call activate_quick.bat
python src/main.py
pause
```

Then create a Windows shortcut to `run_app.bat`.

### Handling Restricted Environments

If the Windows 11 machine has **no internet access**:

#### Option 1: Offline Wheel Files

On WSL (with internet):
```bash
# Download all packages as wheels
pip download -r requirements.txt -d wheels/

# Copy wheels/ directory to Windows machine
# Then on Windows:
cd C:\Projects\repo
python -m venv project_windows_env
project_windows_env\Scripts\activate
pip install --no-index --find-links=wheels -r requirements.txt
```

#### Option 2: Pre-Built Virtual Environment

On WSL:
```bash
# Create and populate Windows-compatible venv
python3 -m venv project_windows_env
source project_windows_env/Scripts/activate  # Note: Scripts not bin
pip install -r requirements.txt

# Copy entire project (with venv) to Windows
# On Windows, just activate it:
project_windows_env\Scripts\activate
```

**Warning**: This only works if Python versions match exactly.

---

## Troubleshooting

### Common Issues and Solutions

#### Issue: "Python is not recognized"

**Cause**: Python not in PATH
**Solution**:
```cmd
REM Find Python installation
where python

REM Add to PATH (User variable, no admin needed)
set PATH=%PATH%;C:\Users\YourName\AppData\Local\Programs\Python\Python39
```

#### Issue: pip install times out

**Cause**: Slow network or large packages
**Solution**: Our scripts handle this with `--timeout` flags. If manual install needed:
```bash
pip install --timeout 600 package-name
```

#### Issue: "Permission denied" when creating venv

**Cause**: Antivirus or Group Policy
**Solution**:
1. Try creating venv in user directory: `C:\Users\YourName\venvs\`
2. Contact IT to whitelist Python virtual environments
3. Use portable Python (Anaconda/Miniconda)

#### Issue: Import errors after deployment

**Cause**: Missing dependencies or version mismatch
**Solution**:
```bash
# Regenerate requirements.txt on WSL
pip freeze > requirements.txt

# On Windows, reinstall
pip install -r requirements.txt --force-reinstall
```

#### Issue: Script fails with "bad interpreter" or "$'\r': command not found"

**Cause**: Line endings (CRLF vs LF) - Scripts created on Windows have Windows line endings
**Solution**:
```bash
# Fix immediately (on WSL)
dos2unix activate_*.sh

# Or if dos2unix not installed
sed -i 's/\r$//' activate_*.sh

# Prevent in future: Create .gitattributes
cat > .gitattributes << 'EOF'
*.sh text eol=lf
*.bat text eol=crlf
*.py text eol=lf
EOF

# Then normalize all files
git add --renormalize .
git commit -m "Normalize line endings"
```

**Prevention**: Always create `.gitattributes` file in new repos (see Step 4b in "Creating a New Repository")

#### Issue: spaCy model not found

**Cause**: Model not downloaded
**Solution**:
```bash
python -m spacy download en_core_web_sm --timeout 300
```

#### Issue: pydantic v2 compatibility errors

**Cause**: pydantic v2 breaks some packages (spacy, stanza)
**Solution**:
```bash
pip install "pydantic<2"
```

---

## New Hire Onboarding Checklist

### Day 1: Environment Setup

#### Windows Machine Setup

- [ ] Install Python 3.8+ from python.org
- [ ] Verify: `python --version` in Command Prompt
- [ ] Install Git from git-scm.com
- [ ] Verify: `git --version`
- [ ] Install VS Code (optional but recommended)
- [ ] Install Windows Terminal (optional but recommended)

#### WSL Setup (If Development Role)

- [ ] Enable WSL in Windows Features
- [ ] Install Ubuntu from Microsoft Store
- [ ] Update Ubuntu: `sudo apt update && sudo apt upgrade`
- [ ] Install Python: `sudo apt install python3 python3-pip python3-venv`
- [ ] Install Git: `sudo apt install git`
- [ ] Configure Git:
  ```bash
  git config --global user.name "Your Name"
  git config --global user.email "your.email@company.com"
  ```

### Day 2: Repository Access

- [ ] Get access to company GitHub/GitLab
- [ ] Clone starter repository:
  ```bash
  git clone https://github.com/company/starter-project.git
  cd starter-project
  ```
- [ ] Run setup script:
  ```bash
  source ./activate_project_env.sh
  ```
- [ ] Verify environment:
  ```bash
  python --version
  pip list
  ```

### Day 3: First Project

- [ ] Review this guide: `WSL_TO_WINDOWS_DEPLOYMENT_GUIDE.md`
- [ ] Review CLAUDE.md in starter project
- [ ] Run test script: `python src/main.py --test`
- [ ] Make a small change and commit:
  ```bash
  git checkout -b test-branch
  # make changes
  git add .
  git commit -m "Test commit"
  git push origin test-branch
  ```

### Ongoing

- [ ] Bookmark this guide
- [ ] Join team Slack/Teams channel
- [ ] Ask questions when stuck (no question is dumb!)
- [ ] Read CLAUDE.md for each new project

---

## Template Scripts

### Template: activate_{project}_env.sh

Save this as `activate_{your_project}_env.sh` and customize:

```bash
#!/bin/bash
# Virtual Environment Setup Script for {PROJECT_NAME}
# Compatible with Linux/WSL and Git Bash on Windows
#
# Usage: source ./activate_{project}_env.sh
#        or: bash activate_{project}_env.sh --force-reinstall

set -e  # Exit on any error

# ============================================
# CONFIGURATION - UPDATE THESE
# ============================================
VENV_NAME_WSL="{project}_wsl_env"
VENV_NAME_WIN="{project}_windows_env"
PROJECT_NAME="{PROJECT_NAME}"
MAIN_SCRIPT="src/main.py"  # Update to your main script

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

echo "=== ${PROJECT_NAME} Virtual Environment Setup ==="
echo "Script directory: ${SCRIPT_DIR}"
echo

# Function to detect platform
detect_platform() {
    if [[ "$OSTYPE" == "msys" ]] || [[ "$OSTYPE" == "win32" ]] || [[ -n "$MSYSTEM" ]]; then
        echo "windows"
    elif [[ "$OSTYPE" == "linux-gnu"* ]]; then
        if grep -q Microsoft /proc/version 2>/dev/null; then
            echo "wsl"
        else
            echo "linux"
        fi
    else
        echo "unknown"
    fi
}

PLATFORM=$(detect_platform)
echo "Detected platform: $PLATFORM"

# Select virtual environment name based on platform
if [ "$PLATFORM" == "wsl" ]; then
    VENV_NAME="$VENV_NAME_WSL"
elif [ "$PLATFORM" == "windows" ]; then
    VENV_NAME="$VENV_NAME_WIN"
else
    VENV_NAME="$VENV_NAME_WSL"
    echo "WARNING: Unknown platform, defaulting to: $VENV_NAME"
fi

VENV_PATH="${SCRIPT_DIR}/${VENV_NAME}"
echo "Virtual environment path: ${VENV_PATH}"
echo

# Check if virtual environment already exists
if [ -d "$VENV_PATH" ]; then
    echo "Virtual environment already exists at: $VENV_PATH"

    if [[ "$1" == "--force-reinstall" ]]; then
        echo "INFO: --force-reinstall flag detected"
        echo "Removing existing virtual environment..."
        rm -rf "$VENV_PATH"
    else
        read -p "Do you want to recreate it? (y/n): " -r
        if [[ $REPLY =~ ^[Yy]$ ]]; then
            echo "Removing existing virtual environment..."
            rm -rf "$VENV_PATH"
        else
            echo "Using existing virtual environment."
            if [ "$PLATFORM" == "windows" ]; then
                echo "  source \"${VENV_PATH}/Scripts/activate\""
            else
                echo "  source \"${VENV_PATH}/bin/activate\""
            fi
            exit 0
        fi
    fi
fi

# Create virtual environment
echo "Creating virtual environment..."
python3 -m venv "$VENV_PATH"

# Activate virtual environment
echo "Activating virtual environment..."
if [ "$PLATFORM" == "windows" ]; then
    source "${VENV_PATH}/Scripts/activate"
    ACTIVATE_SCRIPT="${VENV_PATH}/Scripts/activate"
else
    source "${VENV_PATH}/bin/activate"
    ACTIVATE_SCRIPT="${VENV_PATH}/bin/activate"
fi

# Verify activation
if [ -z "$VIRTUAL_ENV" ]; then
    echo "ERROR: Failed to activate virtual environment"
    exit 1
fi

echo "Virtual environment activated: $VIRTUAL_ENV"

# Upgrade pip
echo "Upgrading pip..."
python -m pip install --upgrade pip --timeout 300

# ============================================
# INSTALL DEPENDENCIES - CUSTOMIZE THIS SECTION
# ============================================
echo
echo "Installing required packages..."
echo

# Example: Basic packages
python -m pip install --timeout 300 pandas numpy

# Example: Large packages (install separately)
# python -m pip install --timeout 600 torch torchvision

# Or install from requirements.txt in batches
# python -m pip install --timeout 300 -r requirements.txt

echo "INFO: All packages installed successfully!"

# ============================================
# PACKAGE VERIFICATION - CUSTOMIZE THIS SECTION
# ============================================
echo
echo "=== Package Verification ==="
python -c "
import sys
print(f'Python version: {sys.version}')

try:
    import pandas as pd
    print(f'OK: Pandas version: {pd.__version__}')
except ImportError as e:
    print(f'ERROR: Pandas import error: {e}')

try:
    import numpy as np
    print(f'OK: NumPy version: {np.__version__}')
except ImportError as e:
    print(f'ERROR: NumPy import error: {e}')

# Add more package verifications here
"

echo
echo "=== SUCCESS ==="
echo "Virtual environment created and configured successfully!"
echo
echo "To activate this environment in the future:"
if [ "$PLATFORM" == "windows" ]; then
    echo "  source \"${VENV_PATH}/Scripts/activate\""
else
    echo "  source \"${VENV_PATH}/bin/activate\""
fi
echo
echo "To deactivate:"
echo "  deactivate"
echo

# Verify main script exists
if [ -f "${SCRIPT_DIR}/${MAIN_SCRIPT}" ]; then
    echo "OK: Main script found: ${MAIN_SCRIPT}"
    echo "To run: python ${MAIN_SCRIPT}"
else
    echo "WARNING: Main script not found: ${MAIN_SCRIPT}"
fi

# Create convenience activation script
QUICK_ACTIVATE="${SCRIPT_DIR}/activate_quick.sh"
cat > "$QUICK_ACTIVATE" << EOF
#!/bin/bash
# Quick activation script
cd "$(dirname "\${BASH_SOURCE[0]}")"
source "${ACTIVATE_SCRIPT}"
echo "${PROJECT_NAME} environment activated!"
echo "Python: \$(which python)"
EOF

chmod +x "$QUICK_ACTIVATE"
echo
echo "Created quick activation script: activate_quick.sh"
echo "Usage: source ./activate_quick.sh"
echo
echo "=== Environment Ready ==="
```

### Template: setup_windows_venv.bat

Save this as `setup_windows_venv.bat` and customize:

```bat
@echo off
REM {PROJECT_NAME} - Windows Virtual Environment Setup
REM For Windows 11 Enterprise (non-admin users)
REM
REM Usage: setup_windows_venv.bat
REM        setup_windows_venv.bat --force-reinstall

setlocal enabledelayedexpansion

echo ========================================
echo {PROJECT_NAME} Setup
echo Windows Virtual Environment
echo ========================================
echo.

REM ============================================
REM CONFIGURATION - UPDATE THESE
REM ============================================
set VENV_NAME={project}_windows_env
set PROJECT_NAME={PROJECT_NAME}
set MAIN_SCRIPT=src\main.py

set SCRIPT_DIR=%~dp0
set VENV_PATH=%SCRIPT_DIR%%VENV_NAME%

echo Script directory: %SCRIPT_DIR%
echo Virtual environment: %VENV_NAME%
echo.

REM Check Python
python --version >nul 2>&1
if %errorlevel% neq 0 (
    echo ERROR: Python is not installed or not in PATH
    pause
    exit /b 1
)

echo INFO: Python version:
python --version
echo.

REM Check for existing venv
if exist "%VENV_PATH%" (
    echo INFO: Virtual environment already exists
    echo.

    if "%1"=="--force-reinstall" (
        echo INFO: --force-reinstall flag detected
        echo Removing existing virtual environment...
        rmdir /s /q "%VENV_PATH%"
        goto :CREATE_VENV
    )

    set /p recreate="Do you want to recreate it? (y/N): "
    if /i "!recreate!"=="y" (
        echo Removing existing virtual environment...
        rmdir /s /q "%VENV_PATH%"
        goto :CREATE_VENV
    ) else (
        echo Using existing virtual environment.
        echo To activate: %VENV_PATH%\Scripts\activate.bat
        pause
        exit /b 0
    )
)

:CREATE_VENV
echo Creating virtual environment...
python -m venv "%VENV_PATH%"
if %errorlevel% neq 0 (
    echo ERROR: Failed to create virtual environment
    pause
    exit /b 1
)

echo Activating virtual environment...
call "%VENV_PATH%\Scripts\activate.bat"

echo Upgrading pip...
python -m pip install --upgrade pip --timeout 300

REM ============================================
REM INSTALL DEPENDENCIES - CUSTOMIZE THIS SECTION
REM ============================================
echo.
echo ========================================
echo Installing Required Packages
echo ========================================
echo.

echo Installing core dependencies...
python -m pip install --timeout 300 pandas numpy
if %errorlevel% neq 0 (
    echo ERROR: Failed to install dependencies
    pause
    exit /b 1
)

REM Add more package installations here
REM python -m pip install --timeout 600 torch torchvision

echo.
echo INFO: All packages installed successfully!

REM ============================================
REM PACKAGE VERIFICATION - CUSTOMIZE THIS
REM ============================================
echo.
echo ========================================
echo Package Verification
echo ========================================
echo.

python -c "import pandas as pd; print(f'OK: Pandas {pd.__version__}')" 2>nul
if %errorlevel% neq 0 echo ERROR: Pandas not available

python -c "import numpy as np; print(f'OK: NumPy {np.__version__}')" 2>nul
if %errorlevel% neq 0 echo ERROR: NumPy not available

REM Add more verifications here

echo.
echo ========================================
echo SUCCESS
echo ========================================
echo.
echo To activate: %VENV_PATH%\Scripts\activate.bat
echo To deactivate: deactivate
echo.

if exist "%SCRIPT_DIR%%MAIN_SCRIPT%" (
    echo OK: Main script found
    echo To run: python %MAIN_SCRIPT%
) else (
    echo WARNING: Main script not found: %MAIN_SCRIPT%
)

REM Create quick activation script
set QUICK_ACTIVATE=%SCRIPT_DIR%activate_quick.bat
echo @echo off > "%QUICK_ACTIVATE%"
echo cd /d "%%~dp0" >> "%QUICK_ACTIVATE%"
echo call "%VENV_PATH%\Scripts\activate.bat" >> "%QUICK_ACTIVATE%"
echo echo %PROJECT_NAME% environment activated! >> "%QUICK_ACTIVATE%"
echo python --version >> "%QUICK_ACTIVATE%"

echo.
echo Created quick activation script: activate_quick.bat
echo.

pause
endlocal
```

---

## Quick Reference Cards

### For Developers

```
┌─────────────────────────────────────────────┐
│ Daily WSL Development Workflow              │
├─────────────────────────────────────────────┤
│ 1. cd /mnt/c/path/to/project                │
│ 2. source ./activate_quick.sh               │
│ 3. git pull                                  │
│ 4. [do work]                                 │
│ 5. git add . && git commit -m "msg"         │
│ 6. git push                                  │
│ 7. deactivate                                │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│ Creating New Repository                     │
├─────────────────────────────────────────────┤
│ 1. Copy template scripts                    │
│ 2. Update VENV_NAME variables               │
│ 3. Create requirements.txt                  │
│ 4. Create .gitignore                        │
│ 5. source ./activate_project_env.sh         │
│ 6. Test and commit                          │
└─────────────────────────────────────────────┘
```

### For Deployment

```
┌─────────────────────────────────────────────┐
│ Windows 11 Deployment                       │
├─────────────────────────────────────────────┤
│ 1. git clone [repo] or copy files           │
│ 2. cd project                                │
│ 3. setup_windows_venv.bat                   │
│ 4. activate_quick.bat                        │
│ 5. python src/main.py --test                │
│ 6. Create desktop shortcut (optional)       │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│ Troubleshooting                             │
├─────────────────────────────────────────────┤
│ Timeout: Use --timeout 600                  │
│ Import Error: pip list, reinstall           │
│ Path Error: Check PYTHONPATH                │
│ Line Endings: dos2unix script.sh            │
│ Permission: Use user directory              │
└─────────────────────────────────────────────┘
```

---

## Best Practices Summary

### ✅ DO

- Use template scripts for new projects
- Name venvs with `{project}_wsl_env` / `{project}_windows_env` pattern
- Add version constraints to requirements.txt
- Test on Windows before deployment
- Create quick activation scripts
- Document project-specific setup in CLAUDE.md
- Use `--timeout` flags for pip install
- Verify package installations

### ❌ DON'T

- Commit virtual environments to git
- Hardcode absolute paths (especially WSL paths like `/mnt/c/`)
- Install packages without version constraints
- Use `pip install -r requirements.txt` directly (use setup scripts)
- Mix WSL and Windows virtual environments
- Skip package verification
- Deploy without testing
- Ignore error messages

---

## Changelog

### Version 2.0 (2025-10-20)
- Complete rewrite with template scripts
- Added new hire onboarding section
- Added troubleshooting for offline deployments
- Added quick reference cards

### Version 1.0
- Initial version based on pdf_filler and H0x_summary patterns

---

## Support and Questions

- Check this guide first
- Review project-specific CLAUDE.md
- Search existing issues in project repos
- Ask team lead or senior developer
- Document solutions for next person!

---

**End of Guide** - Save this file in a central location accessible to all team members.
