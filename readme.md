# Complete CI/CD Pipeline for Beginners – GitHub to Deployment

**Everything inside GitHub. No external tools needed.**

A step-by-step beginner's guide from code creation → testing → team approval → automatic deployment.

---

## 🎯 What You'll Learn

By the end, you'll have a **complete automated pipeline**:

```
1. Write code & push to GitHub
   ↓
2. ✅ Automatic Tests Run
   ↓
3. 📦 Artifacts Created
   ↓
4. 👥 Team Reviews & Approves (Manual approval gate)
   ↓
5. ✅ Auto-Merge to main branch
   ↓
6. 🏷️ Auto-Tag with version (triggers deployment)
   ↓
7. 🚀 Auto-Deploy to staging
   ↓
8. ✅ Health checks pass
   ↓
🎉 LIVE IN STAGING!
```

**Total time:** ~15 minutes from code push to live staging

---

## 📚 Understanding the Pipeline

### What is CI/CD?

| Phase | What It Does | Who | When |
|-------|-------------|-----|------|
| **CI** (Continuous Integration) | Tests your code automatically | GitHub | Every push |
| **CD** (Continuous Delivery) | Prepares to deploy, waits for approval | GitHub | After CI passes |
| **Approval Gate** | Team reviews and approves | You & Team | Manual decision |
| **CD** (Continuous Deployment) | Deploys automatically after approval | GitHub | After approval |

### The Three Branches

```
develop  → Daily work (feature branches merge here)
   ↓
(Build, test, create draft release)
   ↓
team reviews & approves
   ↓
main     → Production-ready code (only approved code)
   ↓
(Tag triggers deployment)
   ↓
staging  → Live test server
```

---

# 🏗️ Part 1: Setup (Create Your Project)

## Step 1: Create Repository on GitHub

### 1.1: Go to GitHub.com

1. **Go to** https://github.com
2. **Sign in** to your account
3. **Click** "+" icon (top right)
4. **Click** "New repository"

### 1.2: Create Repository

Fill in the form:

| Field | Value |
|-------|-------|
| Repository name | `my-calculator` (or your project name) |
| Description | `Calculator with CI/CD pipeline` |
| Public/Private | Choose one |
| Add README | Check this box |

**Click "Create repository"**

### 1.3: Clone to Your Computer

**On Windows (PowerShell):**

```powershell
# Go to Documents
cd ~\Documents

# Clone your repo
git clone https://github.com/yourusername/my-calculator.git

# Navigate into it
cd my-calculator

# Create develop branch
git checkout -b develop

# Push develop to GitHub
git push -u origin develop

# Verify
git branch -a
# Should show: * develop and remotes/origin/main, remotes/origin/develop
```

---

## Step 2: Create Your Application Files

**In your repo folder, create these files:**

### 2.1: Create Folders

```powershell
# Create folders
mkdir src
mkdir tests
mkdir .github\workflows

# Verify
ls
```

### 2.2: Create Application Code

**Create file: `src/app.py`**

Use VS Code or your editor to create this file:

```python
"""Simple calculator application."""

class Calculator:
    @staticmethod
    def add(a, b):
        """Add two numbers."""
        return a + b
    
    @staticmethod
    def subtract(a, b):
        """Subtract two numbers."""
        return a - b
    
    @staticmethod
    def multiply(a, b):
        """Multiply two numbers."""
        return a * b
    
    @staticmethod
    def divide(a, b):
        """Divide two numbers."""
        if b == 0:
            raise ValueError("Cannot divide by zero")
        return a / b

if __name__ == "__main__":
    calc = Calculator()
    print(f"2 + 3 = {calc.add(2, 3)}")
    print(f"10 - 4 = {calc.subtract(10, 4)}")
```

### 2.3: Create Tests

**Create file: `tests/test_app.py`**

```python
"""Tests for calculator."""

import pytest
from src.app import Calculator

class TestCalculator:
    @pytest.fixture
    def calc(self):
        return Calculator()
    
    def test_add(self, calc):
        assert calc.add(2, 3) == 5
        assert calc.add(-1, 1) == 0
    
    def test_subtract(self, calc):
        assert calc.subtract(5, 2) == 3
        assert calc.subtract(0, 5) == -5
    
    def test_multiply(self, calc):
        assert calc.multiply(4, 3) == 12
        assert calc.multiply(0, 100) == 0
    
    def test_divide(self, calc):
        assert calc.divide(10, 2) == 5
        assert calc.divide(7, 2) == 3.5
    
    def test_divide_by_zero(self, calc):
        with pytest.raises(ValueError):
            calc.divide(10, 0)
```

### 2.4: Create Dependencies File

**Create file: `requirements.txt`**

```
pytest==7.4.3
pytest-cov==4.1.0
```

### 2.5: Create Setup File

**Create file: `setup.py`**

```python
from setuptools import setup, find_packages

setup(
    name="calculator-app",
    version="0.1.0",
    description="Calculator with CI/CD",
    author="Your Name",
    packages=find_packages(),
    python_requires=">=3.9",
)
```

### 2.6: Create Empty Init Files

```powershell
# Create __init__.py files
New-Item -Path src\__init__.py -ItemType File -Force
New-Item -Path tests\__init__.py -ItemType File -Force
```

### 2.7: Push to GitHub

```powershell
# Check files
git status

# Add all files
git add .

# Commit
git commit -m "Initial project setup with calculator app"

# Push to develop branch
git push origin develop
```

**Verify on GitHub:**
1. Go to your repo on GitHub.com
2. Click "Code" tab
3. Switch to "develop" branch (dropdown)
4. You should see all your files there ✅

---

# ⚙️ Part 2: Create Workflows (The Automation)

## Step 3: Create First Workflow – Build & Test (CI)

This workflow **automatically runs when you push code**.

### 3.1: Create Workflow File

**Create file: `.github/workflows/1-build-test.yml`**

Copy-paste this entire content directly:

```yaml
name: 1️⃣ Build & Test

on:
  push:
    branches: [develop]

jobs:
  build:
    name: Build and Test
    runs-on: ubuntu-latest
    outputs:
      rc_version: ${{ steps.version.outputs.rc_version }}

    steps:
      - name: 📥 Checkout code
        uses: actions/checkout@v4

      - name: 🐍 Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: 📦 Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: 📦 Install package
        run: |
          pip install -e .

      - name: ✅ Run Tests
        run: |
          pytest tests/ -v --cov=src --cov-report=html --cov-report=term
          echo "All tests passed!"

      - name: 🏷️ Generate Version
        id: version
        run: |
          SHORT_SHA=$(git rev-parse --short HEAD)
          TIMESTAMP=$(date +%Y%m%d_%H%M%S)
          RC_VERSION="v0.0.0-rc.${TIMESTAMP}.${SHORT_SHA}"
          echo "rc_version=${RC_VERSION}" >> $GITHUB_OUTPUT
          echo "RC Version: ${RC_VERSION}"

      - name: 📊 Upload Test Report
        uses: actions/upload-artifact@v4
        with:
          name: test-report-${{ steps.version.outputs.rc_version }}
          path: htmlcov/
          retention-days: 7

      - name: 📤 Create Draft Release
        uses: softprops/action-gh-release@v2
        with:
          tag_name: ${{ steps.version.outputs.rc_version }}
          draft: true
          body: |
            # Release Candidate
            **Version:** ${{ steps.version.outputs.rc_version }}
            **Status:** Waiting for team approval
            **To Approve:** Edit → Uncheck Draft → Update
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 3.2: How to Create the File

**Method 1: VS Code (Easiest)**
1. Open your project in VS Code
2. Create file: `.github/workflows/1-build-test.yml`
3. Copy-paste the YAML above
4. Save (Ctrl+S)

**Method 2: GitHub Web Editor**
1. Go to github.com/yourusername/my-calculator
2. Click "Add file" → "Create new file"
3. Type: `.github/workflows/1-build-test.yml`
4. Paste the YAML content
5. Click "Commit changes"

**Method 3: PowerShell**
```powershell
# Copy the YAML content from above, paste into Notepad
# Save as: .github\workflows\1-build-test.yml (with quotes to preserve filename)
```

### 3.3: Push to GitHub

```powershell
git add .github\workflows\1-build-test.yml
git commit -m "Add build and test workflow"
git push origin develop
```

---

## Step 4: Create Second Workflow – Approval & Merge (CD - Delivery)

**Create file: `.github/workflows/2-approval-merge.yml`**

```yaml
name: 2️⃣ Approval & Merge to Main

on:
  release:
    types: [published]

jobs:
  merge:
    name: Merge to main
    runs-on: ubuntu-latest

    steps:
      - name: 📥 Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: ⚙️ Configure Git
        run: |
          git config user.name "GitHub Actions"
          git config user.email "github-actions@github.com"

      - name: 🌳 Prepare main branch
        run: |
          git fetch origin develop
          if ! git rev-parse --verify origin/main >/dev/null 2>&1; then
            git checkout -b main
            git push origin main
          else
            git checkout main
          fi

      - name: 🔄 Merge develop → main
        run: |
          git fetch origin develop
          git merge origin/develop --no-ff -m "Release approved"

      - name: 🏷️ Create Release Tag
        id: tag
        run: |
          RELEASE_TAG="v1.0.0"
          git tag -a "${RELEASE_TAG}" -m "Release ${RELEASE_TAG}"
          echo "release_tag=${RELEASE_TAG}" >> $GITHUB_OUTPUT

      - name: 📤 Push to GitHub
        run: |
          git push origin main
          git push origin ${{ steps.tag.outputs.release_tag }}
          echo "✅ Merged and tagged!"
```

### Push to GitHub

```powershell
git add .github\workflows\2-approval-merge.yml
git commit -m "Add approval and merge workflow"
git push origin develop
```

---

## Step 5: Create Third Workflow – Deploy (CD - Deployment)

**Create file: `.github/workflows/3-deploy-staging.yml`**

```yaml
name: 3️⃣ Deploy to Staging

on:
  push:
    tags:
      - "v*"

jobs:
  deploy:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    environment:
      name: staging

    steps:
      - name: 📥 Checkout code
        uses: actions/checkout@v4

      - name: 🏷️ Get Version from Tag
        id: version
        run: |
          VERSION="${GITHUB_REF#refs/tags/}"
          echo "version=${VERSION}" >> $GITHUB_OUTPUT
          echo "Deploying version: ${VERSION}"

      - name: 🐍 Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: 📦 Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: ✅ Run Tests
        run: pytest tests/ -v

      - name: 🚀 Deploy to Staging
        run: |
          echo "🚀 Deploying version ${{ steps.version.outputs.version }}"
          echo "✅ Application started"

      - name: 🏥 Health Check
        run: |
          python -c "from src.app import Calculator; c = Calculator(); assert c.add(2, 3) == 5; print('✅ Health check passed')"

      - name: ✨ Deployment Complete
        run: echo "🎉 Successfully deployed!"
```

### Push to GitHub

```powershell
git add .github\workflows\3-deploy-staging.yml
git commit -m "Add deployment workflow"
git push origin develop
```

---

# 🏢 Part 3: Setup GitHub Environments

## Step 6: Create Staging Environment

**All on GitHub.com (NOT in PowerShell):**

1. **Go to your repo** on github.com
2. **Click "Settings"** tab (top right)
3. **Left menu** → **"Environments"** (scroll down)
4. **Click "New environment"** (green button)
5. **Type name:** `staging`
6. **Press Enter**
7. **Click "Save protection rules"**

**Done!** ✅

---

# 🧪 Part 4: Test the Complete Pipeline

## Step 7: Trigger the Pipeline

### Make a Code Change

```powershell
# Add a line to README
Add-Content -Path README.md -Value "`n## CI/CD Status: Active"

# Commit and push
git add README.md
git commit -m "test: trigger CI/CD pipeline"
git push origin develop
```

**Wait 30 seconds for GitHub to detect the push...**

---

## Step 8: Watch Workflow 1 (Build & Test)

1. Go to your repo on **github.com**
2. Click **"Actions"** tab
3. Find **"1️⃣ Build & Test"** workflow (yellow = running)
4. Wait 2-3 minutes for completion
5. All steps should have green checkmarks ✅

---

## Step 9: Team Approval – Publish the Draft Release

1. Go to **"Releases"** tab
2. Find the **draft release** (gray "Draft" button)
3. Click on it
4. Review the test results
5. Click **"Edit"** button (top right)
6. Find checkbox: **"Set as a draft"** (check it ✓)
7. Click to uncheck it ☐
8. Scroll down → Click **"Update release"**

**This is the APPROVAL step!** 👥

---

## Step 10: Watch Workflow 2 (Merge)

1. Go to **"Actions"** tab
2. Find **"2️⃣ Approval & Merge to Main"** (starts automatically)
3. Wait 1 minute for completion
4. Green checkmarks ✅

---

## Step 11: Watch Workflow 3 (Deploy)

1. Go to **"Actions"** tab
2. Find **"3️⃣ Deploy to Staging"** (starts automatically)
3. Wait 1-2 minutes
4. Green checkmarks ✅

**🎉 DONE! Your pipeline is working!**

---

# 📊 Complete Pipeline Flow Summary

| Step | Trigger | Time | Status |
|------|---------|------|--------|
| Code pushed | You push | 0 min | 👤 Manual |
| Tests run | Push detected | 1 min | ✅ Auto |
| Draft release | Tests pass | 3 min | ✅ Auto |
| Team approves | Publish release | 3 min | 👥 Manual |
| Merge to main | Release published | 5 min | ✅ Auto |
| Deploy to staging | Tag created | 7 min | ✅ Auto |
| Health checks pass | Deploy succeeds | 9 min | ✅ Auto |

**Total: ~10 minutes from push to staging live**

---

# ✅ Verification Checklist

### Workflow 1 ✅
- [ ] Workflow ran
- [ ] All tests passed
- [ ] Draft release created

### Workflow 2 ✅
- [ ] You published the release
- [ ] Workflow ran automatically
- [ ] Main branch was created
- [ ] Release tag created

### Workflow 3 ✅
- [ ] Workflow ran automatically
- [ ] Tests ran
- [ ] Health checks passed
- [ ] All green checkmarks ✅

### Overall ✅
- [ ] Three workflows in `.github/workflows/`
- [ ] Staging environment configured
- [ ] Complete flow works end-to-end

---

# 🔄 Day-to-Day Usage

**Developer:**
```powershell
# Make changes
# Commit and push
git push origin develop

# → Automatic tests run
# → Draft release created
# → Workflow 1 completes in ~3 min
```

**Team:**
```
1. Get notified of new draft release
2. Review results
3. If OK: Edit → Uncheck Draft → Update release

↓ Automatic merge & deployment (~10 min total)
```

---

# 🆘 Troubleshooting

### Workflow doesn't run after push

**Fix:**
```powershell
# Make sure you're on develop
git status
# Should show: On branch develop

# Verify files were pushed
git log --oneline -3
```

### Draft release not appearing

**Fix:**
1. Refresh Actions page
2. Wait 2 minutes
3. Check if workflow has errors (red X)
4. Click failed step to see error message

### Can't publish release

**Fix:**
1. Click "Edit" button
2. Look for "Set as a draft" checkbox
3. Make sure it's checked (✓)
4. Click to uncheck it (☐)
5. Scroll down
6. Click "Update release"

### Merge workflow doesn't run

**Fix:**
1. Go to Releases
2. Click the release
3. Make sure it shows "Published" (NOT "Draft")
4. Go to Actions → look for Workflow 2

### Deploy doesn't trigger

**Fix:**
1. Check that tag was created (Releases tab)
2. Wait 2-3 minutes for tag to propagate
3. Go to Actions
4. Refresh the page
5. Look for "3️⃣ Deploy to Staging"

---

# 📚 Next Steps

After you verify everything works:

### Option 1: Use the Pipeline Daily
- Make changes to code
- Push to develop
- Team approves releases
- Auto-deployment to staging

### Option 2: Add Production Deployment
Create a `4-deploy-production.yml` workflow:
```yaml
on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to deploy to production'
        required: true

jobs:
  deploy:
    # Deploy to production server
```

### Option 3: Add Notifications
Send Slack/email notifications when:
- Tests fail
- Release is waiting approval
- Deployment succeeds/fails

---

# 🎉 Summary

You've created a **production-grade CI/CD pipeline**:

✅ Continuous Integration (automatic testing)
✅ Continuous Delivery (approval gate)
✅ Continuous Deployment (automatic deployment)
✅ Team approval workflow
✅ Automatic tagging and versioning
✅ All in GitHub (no external tools)

**You're now a CI/CD expert! Congratulations!** 🚀
