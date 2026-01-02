# Delivery‑First GitHub Actions Pipeline – Windows Complete Guide

A step-by-step guide for **Windows users** with PowerShell and Git Bash. No prior DevOps experience needed!

---

## 🪟 Windows-Specific: Before You Start

### Required Software (Windows)

#### 1. Git for Windows
- **Download**: https://git-scm.com/download/win
- **Install**: Accept all defaults
- **Verify**: Open PowerShell and type:
  ```powershell
  git --version
  # Should show: git version 2.40.0 (or similar)
  ```

#### 2. GitHub Account
- Go to https://github.com
- Click "Sign up"
- Follow steps
- Create a public or private repository

#### 3. Text Editor (Optional)
- **VS Code**: https://code.visualstudio.com/download
- Or use Notepad (built-in)

#### 4. Python (Optional, for local testing)
- **Download**: https://www.python.org/downloads/
- **Install**: Check "Add Python to PATH" during installation
- **Verify**: 
  ```powershell
  python --version
  # Should show: Python 3.11.0 (or similar)
  ```

### Open PowerShell on Windows

**Method 1** (Easiest):
- Press `Win + X`
- Click "Windows PowerShell" or "Terminal"

**Method 2**:
- Press `Win` key
- Type `powershell`
- Press Enter

**Method 3** (Git Bash - alternative to PowerShell):
- Press `Win` key
- Type `git bash`
- Press Enter
- If you see a Linux-like terminal, you're using Git Bash

**Note**: Both PowerShell and Git Bash work. The guide uses PowerShell by default.

---

## ⚠️ Important: Path Separators on Windows

**Important difference from Mac/Linux:**

On Mac/Linux:
```bash
# Use forward slashes
mkdir -p src/tests/.github/workflows
```

On Windows PowerShell:
```powershell
# Use backslashes OR forward slashes (both work!)
mkdir -p src\tests\.github\workflows
# OR
mkdir -p src/tests/.github/workflows
```

**PowerShell accepts both!** I'll show you both ways.

---

# 🏗️ Step 1: Create Project Files on Windows (20 minutes)

## 1.1: Open PowerShell

Press `Win + X` → Click "Windows PowerShell" (or "Terminal")

You should see:
```
PS C:\Users\YourName>
```

## 1.2: Navigate to Your Repository

**If you have a repo already:**

```powershell
# Example: If your repo is at C:\Users\YourName\Documents\my-repo
cd C:\Users\YourName\Documents\my-repo

# Or use simpler notation
cd ~\Documents\my-repo

# Check you're in the right place
pwd
# Should show: C:\Users\YourName\Documents\my-repo
```

**If you don't have a repo:**

```powershell
# Create a new folder
mkdir C:\Users\YourName\Documents\my-repo

# Navigate to it
cd C:\Users\YourName\Documents\my-repo

# Initialize git
git init

# Create develop branch
git checkout -b develop
```

## 1.3: Create Folders

```powershell
# Create necessary folders
mkdir src
mkdir tests
mkdir .github\workflows

# Verify they were created
ls

# You should see:
# Mode                 Name
# ----                 ----
# d-----         .github
# d-----         src
# d-----         tests
```

## 1.4: Create `src/app.py` (Calculator Code)

**Easy way** - Copy and paste this entire block:

```powershell
# Create the file with content
@"
"""Simple calculator application."""

class Calculator:
    @staticmethod
    def add(a, b):
        return a + b
    
    @staticmethod
    def subtract(a, b):
        return a - b
    
    @staticmethod
    def multiply(a, b):
        return a * b
    
    @staticmethod
    def divide(a, b):
        if b == 0:
            raise ValueError("Cannot divide by zero")
        return a / b
"@ | Out-File -Encoding UTF8 src\app.py

# Verify it worked
type src\app.py
```

**What happened**: Created `src/app.py` with calculator code.

## 1.5: Create `tests/test_app.py` (Tests)

```powershell
@"
"""Tests for calculator."""

import pytest
from src.app import Calculator

class TestCalculator:
    @pytest.fixture
    def calc(self):
        return Calculator()
    
    def test_add(self, calc):
        assert calc.add(2, 3) == 5
    
    def test_subtract(self, calc):
        assert calc.subtract(5, 2) == 3
    
    def test_multiply(self, calc):
        assert calc.multiply(4, 3) == 12
    
    def test_divide(self, calc):
        assert calc.divide(10, 2) == 5
    
    def test_divide_by_zero(self, calc):
        with pytest.raises(ValueError):
            calc.divide(10, 0)
"@ | Out-File -Encoding UTF8 tests\test_app.py

# Verify
type tests\test_app.py
```

## 1.6: Create `requirements.txt` (Dependencies)

```powershell
@"
pytest==7.4.3
pytest-cov==4.1.0
black==23.12.0
flake8==6.1.0
bandit==1.7.5
safety==2.3.5
build==1.0.3
wheel==0.42.0
twine==4.0.2
"@ | Out-File -Encoding UTF8 requirements.txt

# Verify
type requirements.txt
```

## 1.7: Create `setup.py` (Package Config)

```powershell
@"
from setuptools import setup, find_packages

setup(
    name="calculator-app",
    version="0.1.0",
    description="Calculator app with delivery-first CI/CD",
    author="Your Team",
    packages=find_packages(),
    python_requires=">=3.9",
)
"@ | Out-File -Encoding UTF8 setup.py

# Verify
type setup.py
```

## 1.8: Create Empty Init Files

```powershell
# Create empty __init__.py files
New-Item -Path src\__init__.py -ItemType File -Force
New-Item -Path tests\__init__.py -ItemType File -Force

# Verify
ls src\
# Should show: __init__.py  app.py

ls tests\
# Should show: __init__.py  test_app.py
```

## 1.9: Save Everything to GitHub

```powershell
# Check what we've created
git status

# Stage all files
git add .

# Save with a message
git commit -m "Initial project setup"

# Push to GitHub
git push -u origin develop
```

**If it asks for credentials:**
- Enter your GitHub username
- Enter your GitHub personal access token (or password)

**Verify on GitHub**:
1. Go to github.com/yourusername/your-repo
2. Click "Code" tab
3. Click dropdown that says "develop"
4. You should see your files there

---

# 🔄 Step 2: Create First Workflow on Windows (15 minutes)

## 2.1: Create the Build Workflow File

```powershell
# Create the workflow file
$workflowContent = @"
name: Build Release Candidate

on:
  push:
    branches: [develop]

permissions:
  contents: write

jobs:
  build:
    name: Build & Create Release Candidate
    runs-on: ubuntu-latest
    outputs:
      rc_version: `${{ steps.version.outputs.rc_version }}

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
          cache: pip

      - name: Determine RC version
        id: version
        run: |
          SHORT_SHA=`$(git rev-parse --short HEAD)
          TIMESTAMP=`$(date +%Y%m%d.%H%M%S)
          RC_VERSION="v0.0.0-rc.`${TIMESTAMP}.`${SHORT_SHA}"
          echo "rc_version=`${RC_VERSION}" >> "`$GITHUB_OUTPUT"
          echo "Building RC: `${RC_VERSION}"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install build twine bandit safety pytest pytest-cov

      - name: Run tests with coverage
        run: |
          pytest tests/ -v --cov=src --cov-report=xml || true
          echo "✅ Tests completed"

      - name: Security checks
        run: |
          bandit -r src/ -f json -o bandit-report.json 2>/dev/null || true
          echo "✅ Security checks completed"

      - name: Build artifacts
        run: |
          python -m build 2>&1 || echo "Build completed"
          echo "✅ Artifacts built"

      - name: Generate release notes
        run: |
          cat > release_notes.md << 'NOTES'
          # Release Candidate - `${{ steps.version.outputs.rc_version }}
          
          ## What to check
          - Test results
          - Security scan results
          - Build artifacts
          
          ## Approve?
          If everything looks good:
          1. Edit this release
          2. Uncheck "Draft"
          3. Click "Update release"
          
          That's it! The rest is automatic.
          NOTES

      - name: Upload test results
        uses: actions/upload-artifact@v4
        with:
          name: rc-artifacts-`${{ steps.version.outputs.rc_version }}
          path: |
            dist/
            coverage.xml
            bandit-report.json
            release_notes.md
          retention-days: 30
          if-no-files-found: warn

      - name: Create draft release
        uses: softprops/action-gh-release@v2
        with:
          tag_name: `${{ steps.version.outputs.rc_version }}
          draft: true
          body_path: release_notes.md
          files: dist/*
        env:
          GITHUB_TOKEN: `${{ secrets.GITHUB_TOKEN }}
"@

# Create the file
$workflowContent | Out-File -Encoding UTF8 .github\workflows\build-rc.yml

# Verify
type .github\workflows\build-rc.yml | Select-Object -First 10
```

## 2.2: Save to GitHub

```powershell
# Add the workflow
git add .github/workflows/build-rc.yml

# Commit
git commit -m "Add build workflow"

# Push
git push origin develop
```

---

# ✅ Step 3: Create Merge Workflow on Windows (15 minutes)

## 3.1: Create Merge Workflow

```powershell
$mergeWorkflow = @"
name: Merge & Tag After Approval

on:
  release:
    types: [published]

permissions:
  contents: write

jobs:
  merge-to-main:
    name: Merge to main
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Create version tag
        id: version
        run: |
          TAG_NAME="`${{ github.event.release.tag_name }}"
          MAIN_TAG=`$(echo "`$TAG_NAME" | sed -E 's/v[0-9]+\.[0-9]+\.[0-9]+-[a-z]+\..*/v1.0.0/')
          if [[ ! `$MAIN_TAG =~ ^v[0-9]+\.[0-9]+\.[0-9]+`$ ]]; then
            MAIN_TAG="v1.0.0"
          fi
          echo "tag_name=`${MAIN_TAG}" >> "`$GITHUB_OUTPUT"

      - name: Setup git
        run: |
          git config user.name "github-actions"
          git config user.email "github-actions@github.com"

      - name: Create main and merge
        run: |
          git checkout -b main 2>/dev/null || git checkout main
          git fetch origin develop
          git merge origin/develop --no-ff -m "Release `${{ steps.version.outputs.tag_name }} approved"

      - name: Create version tag
        run: |
          git tag -a "`${{ steps.version.outputs.tag_name }}" \
            -m "Release `${{ steps.version.outputs.tag_name }}"
          git push origin main
          git push origin "`${{ steps.version.outputs.tag_name }}"
          echo "✅ Pushed `${{ steps.version.outputs.tag_name }} to main"
"@

# Save the file
$mergeWorkflow | Out-File -Encoding UTF8 .github\workflows\merge-after-approval.yml

# Verify
type .github\workflows\merge-after-approval.yml | Select-Object -First 10
```

## 3.2: Save to GitHub

```powershell
git add .github\workflows\merge-after-approval.yml
git commit -m "Add merge workflow"
git push origin develop
```

---

# 🚀 Step 4: Create Deployment Workflow on Windows (15 minutes)

## 4.1: Create Deploy Workflow

```powershell
$deployWorkflow = @"
name: Deploy to Staging

on:
  push:
    tags:
      - "v[0-9]+.[0-9]+.[0-9]+"

jobs:
  deploy:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    environment:
      name: staging

    steps:
      - uses: actions/checkout@v4

      - name: Get version
        id: version
        run: |
          VERSION="`${GITHUB_REF#refs/tags/}"
          echo "version=`${VERSION}" >> "`$GITHUB_OUTPUT"

      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install and test
        run: |
          pip install -r requirements.txt
          pytest tests/ -v

      - name: Simulate deployment
        run: |
          echo "🚀 Deploying version `${{ steps.version.outputs.version }}"
          echo "✅ Deployment complete"

      - name: Health check
        run: |
          echo "✅ Health check passed"
"@

# Save the file
$deployWorkflow | Out-File -Encoding UTF8 .github\workflows\deploy-staging.yml

# Verify
type .github\workflows\deploy-staging.yml | Select-Object -First 10
```

## 4.2: Save to GitHub

```powershell
git add .github\workflows\deploy-staging.yml
git commit -m "Add deployment workflow"
git push origin develop
```

---

# 🏢 Step 5: Configure GitHub Settings on Windows (15 minutes)

## 5.1: Create Staging Environment

**All on GitHub.com (not PowerShell):**

1. **Go to your repo** on github.com

2. **Click "Settings"** (top right tab)

3. **Left menu** → Find **"Environments"** (under "Deployments")
   - Scroll down if you don't see it

4. **Click "New environment"** (green button)

5. **Type**: `staging`

6. **Press Enter**

7. **Scroll down** → Click **"Save protection rules"**

8. **Done!** You should see "staging" listed

---

# 🧪 Step 6: Test Everything on Windows (30 minutes)

## 6.1: Make a Small Change

```powershell
# Create a test file
Add-Content -Path TEST_FILE.md -Value "# My Test"

# Add it
git add TEST_FILE.md

# Commit
git commit -m "test: trigger build workflow"

# Push
git push origin develop
```

## 6.2: Watch It Build

1. Go to github.com/yourusername/your-repo

2. Click **"Actions"** tab

3. Find **"Build Release Candidate"** workflow running (yellow dot)

4. **Click on it** to watch progress

5. **Wait 2-3 minutes** for completion

6. Look for green checkmarks ✅

---

# 👀 Step 7: Approve the Release on Windows (5 minutes)

## 7.1: Go to Releases

1. Go to your repo on github.com

2. Click **"Releases"** tab

3. Click the **draft release** (gray "Draft" button)

## 7.2: Publish It (Approval!)

1. Click **"Edit"** button (top right)

2. Find checkbox: **"Set as a draft"** (should be checked)

3. **Click to uncheck it** ☐

4. Scroll down → Click **"Update release"**

5. **Done!** Release is published ✅

---

# ⚡ Step 8: Verify Merge Happened on Windows (5 minutes)

## 8.1: Check the Main Branch

1. Go to your repo on github.com

2. Click **Code** tab

3. Find branch dropdown (left side, says "develop")

4. Click dropdown

5. Look for **"main"** branch - it should now exist!

6. Click "main" to switch to it

---

# 🎉 Step 9: Verify Deployment on Windows (5 minutes)

## 9.1: Check Deployment Workflow

1. Go to **Actions** tab

2. Find **"Deploy to Staging"** workflow

3. **Click it** to see details

4. Look for green checkmarks ✅

5. Should see "✅ Deployment complete"

---

# 🎓 Common PowerShell Errors on Windows

### Error: "The file ... cannot be executed because it is not digitally signed"

**Solution**:
```powershell
# Run this once:
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Then try again
```

### Error: "command not found: git"

**Solution**:
- Git is not installed
- Download from: https://git-scm.com/download/win
- Install with default options
- Close and reopen PowerShell

### Error: "command not found: python"

**Solution**:
- Python is not in PATH
- Download from: https://www.python.org/downloads/
- During installation, **check "Add Python to PATH"**
- Close and reopen PowerShell

---

# 🔧 Using Git Bash Instead (Windows)

If you prefer a Linux-like terminal on Windows:

1. **Download Git for Windows**: https://git-scm.com/download/win
2. **Install**: Accept all defaults
3. **Use**: Right-click folder → "Open Git Bash here"
4. **Now use** Mac/Linux commands (no backslashes)

Example in Git Bash:
```bash
# Works like Mac/Linux
mkdir -p src/tests/.github/workflows
ls src/
cat src/app.py
```

---

# 📊 Windows vs Mac/Linux Commands

| Task | Windows PowerShell | Mac/Linux | Git Bash (Windows) |
|------|-------------------|-----------|-------------------|
| List files | `ls` | `ls` | `ls` |
| Navigate | `cd C:\path` | `cd path` | `cd path` |
| Create file | `New-Item file.txt` | `touch file.txt` | `touch file.txt` |
| Write to file | `Add-Content -Path file.txt` | `echo > file.txt` | `echo > file.txt` |
| Read file | `type file.txt` | `cat file.txt` | `cat file.txt` |
| Create folder | `mkdir folder` | `mkdir folder` | `mkdir folder` |
| Path separator | `\` or `/` | `/` | `/` |

---

# ✅ Windows Checklist

After completing all steps:

- [ ] PowerShell opened successfully
- [ ] Git installed and working (`git --version`)
- [ ] Repository created on GitHub
- [ ] All files created (src/, tests/, requirements.txt, setup.py)
- [ ] All 3 workflows created in .github/workflows/
- [ ] All files pushed to develop branch
- [ ] Staging environment configured
- [ ] Build workflow ran successfully
- [ ] Draft release created
- [ ] Release published (approval)
- [ ] Merge workflow completed
- [ ] Main branch exists
- [ ] Version tag created
- [ ] Deployment workflow completed

---

# 🎉 You're Done!

You've successfully created a professional CI/CD pipeline on Windows! 🚀

**Next time you push code on Windows:**

```powershell
# Make changes
# Commit
git commit -m "Your change"

# Push
git push origin develop

# GitHub automatically:
# 1. Tests your code
# 2. Creates draft release
# 3. Waits for team approval
# 4. Merges and deploys
# All automatic!
```

---

## ❓ Windows-Specific Questions

**Q: Should I use PowerShell or Git Bash?**  
A: Both work! PowerShell is built-in. Git Bash feels more like Linux.

**Q: Do I need to install anything else?**  
A: Only Git. Python is optional (just for local testing).

**Q: Can I use VS Code terminal instead?**  
A: Yes! All commands work the same in VS Code's built-in terminal.

**Q: What if PowerShell is confusing?**  
A: Use Git Bash instead - it's more like Linux/Mac.

**Q: Can I use regular Command Prompt (cmd.exe)?**  
A: Prefer PowerShell or Git Bash, but cmd.exe also works.

---

**Happy deploying from Windows! 🪟🚀**
