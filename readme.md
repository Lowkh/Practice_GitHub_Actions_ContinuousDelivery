# Complete CI/CD Pipeline for Beginners – GitHub to Deployment

**Everything inside GitHub. No external tools needed.**

A step-by-step beginner's guide from code creation → testing → team approval → automatic deployment. Updated with fixes for Python imports and GitHub Actions permissions.[^1][^2]

***

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

**Total time:** ~15 minutes from code push to live staging.[^1]

***

## 📚 Understanding the Pipeline

### What is CI/CD?

| Phase | What It Does | Who | When |
| :-- | :-- | :-- | :-- |
| **CI** (Continuous Integration) | Tests your code automatically | GitHub | Every push |
| **CD** (Continuous Delivery) | Prepares to deploy, waits for approval | GitHub | After CI passes |
| **Approval Gate** | Team reviews and approves | You \& Team | Manual decision |
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
staging  → Live test server (GitHub Environment)
```


***

# 🏗️ Part 1: Setup (Create Your Project)

## Step 1: Create Repository on GitHub

### 1.1: Go to GitHub.com

1. Go to https://github.com
2. Sign in to your account
3. Click “+” icon (top right)
4. Click “New repository”[^1]

### 1.2: Create Repository

Fill in the form:


| Field | Value |
| :-- | :-- |
| Repository name | `my-calculator` (or your project name) |
| Description | `Calculator with CI/CD pipeline` |
| Public/Private | Choose one |
| Add README | Check this box |

Click **Create repository**.[^1]

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


***

## Step 2: Create Your Application Files

In your repo folder, create these files and folders.[^1]

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

```text
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


### 2.6: Create Empty Init Files (Important)

To avoid `ModuleNotFoundError: No module named 'src'`, ensure both `src` and `tests` are Python packages.[^1]
Create the empty files accordingly to the folders  `src` and `tests`

```powershell
# Create __init__.py files
# This is an empty file that must exist in 'src' and 'tests'
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

Verify on GitHub (Code tab, `develop` branch) that everything is there.[^1]

***

# ⚙️ Part 2: Create Workflows (The Automation)

### Important: Enable Read/Write Permissions for Workflows

For GitHub Actions to create releases and push tags/branches, the `GITHUB_TOKEN` must have **read and write** permissions.[^2][^3]

1. In your repo, go to **Settings → Actions → General**.
2. Under **Workflow permissions**, select **Read and write permissions**, then click **Save**.
3. In your workflow files, ensure you set permissions explicitly, for example:
```yaml
# 1-build-test.yml
permissions:
  contents: write

# 2-approval-merge.yml
permissions:
  contents: write

# 3-deploy-staging.yml
permissions:
  contents: read
```

This avoids `403 Resource not accessible by integration` when creating releases or pushing tags from workflows.[^4][^5]

***

## Step 3: Create First Workflow – Build \& Test (CI)

This workflow runs when you push to `develop`, installs dependencies, installs your package, runs tests with coverage, and creates a draft release.[^2][^1]

### 3.1: Workflow File (with Fixes)

**Create file: `.github/workflows/1-build-test.yml`**

```yaml
name: 1️⃣ Build & Test

on:
  push:
    branches: [develop]

# Allow creating releases with GITHUB_TOKEN
permissions:
  contents: write

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
            **To Approve:** Edit → Publish release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```


***

## Step 4: Create Second Workflow – Approval \& Merge (CD - Delivery)

This workflow runs when you **publish** a draft release (approval gate).[^1]

**Create file: `.github/workflows/2-approval-merge.yml`**

```yaml
name: 2️⃣ Approval & Merge to Main

on:
  release:
    types: [published]

permissions:
  contents: write

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


***

## Step 5: Create Third Workflow – Deploy (CD - Deployment)

This workflow runs when a tag (`v*`) is pushed (from Workflow 2).[^1]

**Create file: `.github/workflows/3-deploy-staging.yml`**

```yaml
name: 3️⃣ Deploy to Staging

on:
  push:
    tags:
      - "v*"

permissions:
  contents: read

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
          pip install -e .

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


***

# 🏢 Part 3: Setup GitHub Environments

## Step 6: Create Staging Environment

On GitHub:

1. Go to your repo
2. Click **Settings**
3. Left menu → **Environments**
4. Click **New environment**
5. Name: `staging`
6. Click **Save protection rules**[^1]

***

# 🧪 Part 4: Test the Complete Pipeline

## Step 7: Trigger the Pipeline

Make a small change:

```powershell
# Add a line to README
Add-Content -Path README.md -Value "`n## CI/CD Status: Active"

# Commit and push
git add README.md
git commit -m "test: trigger CI/CD pipeline"
git push origin develop
```

Wait ~30 seconds for GitHub to detect the push.[^1]

***

## Step 8: Watch Workflow 1 – Build \& Test

1. Go to your repo → **Actions**.
2. Click **“1️⃣ Build \& Test”**.
3. Open the latest run and wait ~2–3 minutes.
4. Confirm:
    - All steps are green.
    - An HTML coverage report artifact was uploaded.
    - A **draft** release was created with a tag like `v0.0.0-rc.20...`.[^1]

***

## Step 9: Approve the Release in GitHub (Manual Gate)

This is your manual approval gate: you decide if the build from `develop` is good enough to go to `main` and staging.[^1]

1. Open your repo on GitHub and click the **Releases** tab.
2. Find the latest **Draft** release created by Workflow 1 (tag like `v0.0.0-rc.2026...`) and click its title.
3. Review:
    - The tag and target branch (should be based on `develop`).
    - The description/body explaining it is a release candidate.
    - Optionally, open **Actions → 1️⃣ Build \& Test** in a new tab to inspect logs and the HTML coverage report artifact.[^1]
4. If something looks wrong (failed tests, wrong commit, etc.):
    - You can delete or ignore this draft and fix your code, then push again to `develop` to generate a new draft release.
5. If everything looks good and you want to approve:
    - Scroll to the bottom of the draft release page.
    - Click **Publish release**.

Publishing the release is the **approval action** and triggers Workflow 2 because it is configured with `on: release: types: [published]`.[^6][^1]

***

## Step 10: Workflow 2 – Approval \& Merge to Main

Workflow 2 automatically merges approved code from `develop` into `main` and creates a version tag.[^1]

1. Go to the **Actions** tab.
2. Click the workflow **“2️⃣ Approval \& Merge to Main”** in the left sidebar.
3. Open the run that started just after you published the release.
4. Confirm the steps:
    - **📥 Checkout code**: fetches the full repository history (`fetch-depth: 0`).
    - **⚙️ Configure Git**: sets the Git user name and email for commits/tags.
    - **🌳 Prepare main branch**:
        - If `main` does not exist, it creates and pushes it.
        - If `main` exists, it checks it out so it can be updated.
    - **🔄 Merge develop → main**: merges `origin/develop` into `main` with a `--no-ff` merge commit like `"Release approved"`.
    - **🏷️ Create Release Tag**: creates an annotated tag (for example `v1.0.0`) pointing at the new `main` commit.
    - **📤 Push to GitHub**: pushes the updated `main` and the new tag to the remote.[^1]
5. When all steps are green:
    - In the **Code** tab, switch to the `main` branch and confirm it now has your latest approved changes.
    - In **Releases**, confirm a release with the new tag (e.g. `v1.0.0`) exists.

Pushing this tag is what triggers Workflow 3, which listens on `push` to tags matching `v*`.[^1]

***

## Step 11: Workflow 3 – Deploy to Staging and Health Check

Workflow 3 runs on the new tag and re-validates the code while simulating a deployment to the `staging` environment.[^1]

1. In **Actions**, click the workflow **“3️⃣ Deploy to Staging”**.
2. Open the run that started after Workflow 2 completed.
3. Verify each step:
    - **📥 Checkout code**: checks out the repository at the tag commit (e.g. `v1.0.0`).
    - **🏷️ Get Version from Tag**: extracts the tag from `GITHUB_REF` and logs `Deploying version: v1.0.0` so you know exactly which version is being deployed.
    - **🐍 Setup Python**: sets up Python 3.11 on the runner.
    - **📦 Install dependencies**: upgrades `pip`, installs `requirements.txt`, and installs your package with `pip install -e .` so `from src.app import Calculator` works reliably.[^1]
    - **✅ Run Tests**: executes `pytest tests/ -v` again on the tagged revision to confirm tests still pass at deploy time.
    - **🚀 Deploy to Staging**: in this tutorial it just echoes deployment messages, but in a real system you would run your deployment script here.
    - **🏥 Health Check**: runs a small Python snippet that:
        - Imports `Calculator` from `src.app`.
        - Asserts that `calc.add(2, 3) == 5`.
        - Prints `✅ Health check passed` if successful.
    - **✨ Deployment Complete**: prints `🎉 Successfully deployed!` at the end.[^1]
4. If any step fails:
    - Click the failing step to see logs and fix the underlying issue (tests, packaging, or workflow config), then start again from pushing to `develop`.
5. When all steps are green and you see `🎉 Successfully deployed!`, the tagged version is considered successfully deployed to the **staging** environment and the full path `develop → approval → main → tag → staging` is working end-to-end.[^2][^1]

***

# 📊 Complete Pipeline Flow Summary

| Step | Trigger | Time | Status |
| :-- | :-- | :-- | :-- |
| Code pushed | You push | 0 min | 👤 Manual |
| Tests run | Push detected | 1 min | ✅ Auto |
| Draft release | Tests pass | 3 min | ✅ Auto |
| Team approves | Publish release | 3 min | 👥 Manual |
| Merge to main | Release published | 5 min | ✅ Auto |
| Deploy to staging | Tag created | 7 min | ✅ Auto |
| Health checks pass | Deploy succeeds | 9 min | ✅ Auto |

Total: ~10 minutes from push to staging live.[^1]

***

# ✅ Verification Checklist

### Workflow 1

- [ ] Workflow ran
- [ ] All tests passed
- [ ] Draft release created


### Workflow 2

- [ ] Draft release published
- [ ] Workflow ran automatically
- [ ] `main` branch updated
- [ ] Release tag created


### Workflow 3

- [ ] Workflow ran automatically
- [ ] Tests ran
- [ ] Health checks passed
- [ ] All green checkmarks


### Overall

- [ ] Three workflows in `.github/workflows/`
- [ ] `staging` environment configured
- [ ] Complete flow works end-to-end[^1]

***

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

1. Get notified of new draft release
2. Review results
3. If OK: **Publish release**
4. Auto-merge to `main` and deploy to `staging` (~10 min total)[^1]

***

# 🆘 Troubleshooting

### Workflow does not run after push

```powershell
git status      # Ensure you're on branch develop
git log --oneline -3
```


### Draft release not appearing

- Refresh Actions page
- Wait 2 minutes
- Check for failed Workflow 1 run and inspect logs.[^1]


### Cannot publish release

- Ensure you opened the **Draft** release.
- Use the **Publish release** button at the bottom (no more “Set as a draft” checkbox in the new UI).[^6]


### Merge workflow does not run

- Confirm the release status is “Published”, not “Draft”.
- Check Actions for **2️⃣ Approval \& Merge to Main**.[^1]


### Deploy does not trigger

- Check that a tag (e.g. `v1.0.0`) exists in **Releases**.
- Wait a couple of minutes.
- Refresh Actions and look for **3️⃣ Deploy to Staging**.[^1]

***

# 📚 Next Steps

### Option 1: Use the Pipeline Daily

- Develop on `develop`.
- Team approves draft releases.
- Auto-deployment to `staging`.[^1]


### Option 2: Add Production Deployment

Create `4-deploy-production.yml` with `workflow_dispatch` and manual version input for production deployments.[^1]

### Option 3: Add Notifications

Add Slack/email notifications for:

- Test failures.
- Release waiting for approval.
- Deployment success/failure.[^1]

***

# 🎉 Summary

You now have a **production-grade CI/CD pipeline**:

- Continuous Integration (automatic testing).
- Continuous Delivery (manual approval gate).
- Continuous Deployment (automatic deployment to staging).
- Team approval workflow.
- Automatic tagging and versioning.
- All inside GitHub, with proper package imports and token permissions configured.[^3][^1]

<div align="center">⁂</div>

[^1]: https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/12895207/ded8abd3-ab1e-4083-8e70-5426f6bc8101/readme.md

[^2]: https://docs.github.com/en/actions/tutorials/authenticate-with-github_token

[^3]: https://github.blog/changelog/2021-04-20-github-actions-control-permissions-for-github_token/

[^4]: https://github.com/softprops/action-gh-release/issues/236

[^5]: https://stackoverflow.com/questions/76362343/creating-a-release-using-github-action-fails-with-http-403/76523728

[^6]: https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository

