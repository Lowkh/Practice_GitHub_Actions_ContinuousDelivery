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

```powershell
@"
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
"@ | Out-File -Encoding UTF8 src\app.py
```

### 2.3: Create Tests

**Create file: `tests/test_app.py`**

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
"@ | Out-File -Encoding UTF8 tests\test_app.py
```

### 2.4: Create Dependencies File

**Create file: `requirements.txt`**

```powershell
@"
pytest==7.4.3
pytest-cov==4.1.0
"@ | Out-File -Encoding UTF8 requirements.txt
```

### 2.5: Create Setup File

**Create file: `setup.py`**

```powershell
@"
from setuptools import setup, find_packages

setup(
    name="calculator-app",
    version="0.1.0",
    description="Calculator with CI/CD",
    author="Your Name",
    packages=find_packages(),
    python_requires=">=3.9",
)
"@ | Out-File -Encoding UTF8 setup.py
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

**Create: `.github/workflows/1-build-test.yml`**

```powershell
$workflow = @"
name: 1️⃣ Build & Test

on:
  push:
    branches: [develop]

jobs:
  build:
    name: Build and Test
    runs-on: ubuntu-latest
    outputs:
      rc_version: `${{ steps.version.outputs.rc_version }}

    steps:
      # Step 1: Get the code
      - name: 📥 Checkout code
        uses: actions/checkout@v4

      # Step 2: Setup Python
      - name: 🐍 Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      # Step 3: Install dependencies
      - name: 📦 Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      # Step 4: Run tests
      - name: ✅ Run Tests
        run: |
          pytest tests/ -v --cov=src --cov-report=html --cov-report=term
          echo "All tests passed!"

      # Step 5: Generate version
      - name: 🏷️ Generate Version
        id: version
        run: |
          SHORT_SHA=`$(git rev-parse --short HEAD)
          TIMESTAMP=`$(date +%Y%m%d_%H%M%S)
          RC_VERSION="v0.0.0-rc.`${TIMESTAMP}.`${SHORT_SHA}"
          echo "rc_version=`${RC_VERSION}" >> "`$GITHUB_OUTPUT"
          echo "RC Version: `${RC_VERSION}"

      # Step 6: Upload test results
      - name: 📊 Upload Test Report
        uses: actions/upload-artifact@v4
        with:
          name: test-report-`${{ steps.version.outputs.rc_version }}
          path: htmlcov/
          retention-days: 7

      # Step 7: Create draft release with artifacts
      - name: 📤 Create Draft Release
        uses: softprops/action-gh-release@v2
        with:
          tag_name: `${{ steps.version.outputs.rc_version }}
          draft: true
          body: |
            # Release Candidate
            
            **Version:** `${{ steps.version.outputs.rc_version }}`
            **Build Time:** `$(date)`
            **Branch:** develop
            
            ## ✅ What Passed
            - [x] All unit tests passed
            - [x] Code built successfully
            
            ## 👥 Next Step
            Team review is required:
            1. Download test report
            2. Review test coverage
            3. If OK, approve this release
            
            **To Approve:** Click "Edit" → Uncheck "Draft" → "Update release"
        env:
          GITHUB_TOKEN: `${{ secrets.GITHUB_TOKEN }}
"@

$workflow | Out-File -Encoding UTF8 .github\workflows\1-build-test.yml
```

### 3.2: Push Workflow to GitHub

```powershell
# Stage the workflow
git add .github\workflows\1-build-test.yml

# Commit
git commit -m "Add build and test workflow"

# Push to develop
git push origin develop
```

---

## Step 4: Create Second Workflow – Approval & Merge (CD - Delivery)

This workflow **runs when team approves the release**.

### 4.1: Create Merge Workflow

**Create: `.github/workflows/2-approval-merge.yml`**

```powershell
$workflow = @"
name: 2️⃣ Approval & Merge to Main

on:
  release:
    types: [published]

jobs:
  merge:
    name: Merge to main and create release tag
    runs-on: ubuntu-latest

    steps:
      # Step 1: Get code
      - name: 📥 Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # Step 2: Setup git
      - name: ⚙️ Configure Git
        run: |
          git config user.name "GitHub Actions"
          git config user.email "github-actions@github.com"

      # Step 3: Create main branch if needed
      - name: 🌳 Prepare main branch
        run: |
          git fetch origin develop
          if ! git rev-parse --verify origin/main >/dev/null 2>&1; then
            git checkout -b main
            git push origin main
          else
            git checkout main
          fi

      # Step 4: Merge develop into main
      - name: 🔄 Merge develop → main
        run: |
          git fetch origin develop
          git merge origin/develop --no-ff -m "Approved Release: `${{ github.event.release.tag_name }}"
          echo "Merged develop into main"

      # Step 5: Create semantic version tag
      - name: 🏷️ Create Release Tag
        id: tag
        run: |
          # Extract version from RC tag
          RC_TAG="`${{ github.event.release.tag_name }}"
          echo "RC Tag: `$RC_TAG"
          
          # Create release tag (v1.0.0, v1.0.1, etc)
          RELEASE_TAG="v1.0.0"
          echo "release_tag=`${RELEASE_TAG}" >> "`$GITHUB_OUTPUT"
          
          git tag -a "`${RELEASE_TAG}" -m "Release `${RELEASE_TAG}"
          echo "Created tag: `${RELEASE_TAG}"

      # Step 6: Push main and tag
      - name: 📤 Push to GitHub
        run: |
          git push origin main
          git push origin `${{ steps.tag.outputs.release_tag }}
          echo "✅ Pushed main branch and tag"

      # Step 7: Update release notes
      - name: 📝 Update Release Notes
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.repos.updateRelease({
              owner: context.repo.owner,
              repo: context.repo.repo,
              release_id: context.payload.release.id,
              body: `${{ github.event.release.body }}

---

## ✅ APPROVED & MERGED

- **Status**: Merged to main ✅
- **Release Tag**: \`${{ steps.tag.outputs.release_tag }}\`
- **Approved at**: \`$(date)\`
- **Next**: Automatic deployment starting...`
            });
"@

$workflow | Out-File -Encoding UTF8 .github\workflows\2-approval-merge.yml
```

### 4.2: Push to GitHub

```powershell
git add .github\workflows\2-approval-merge.yml
git commit -m "Add approval and merge workflow"
git push origin develop
```

---

## Step 5: Create Third Workflow – Deploy (CD - Deployment)

This workflow **automatically runs when a release tag is created**.

### 5.1: Create Deployment Workflow

**Create: `.github/workflows/3-deploy-staging.yml`**

```powershell
$workflow = @"
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
      # Step 1: Get code
      - name: 📥 Checkout code
        uses: actions/checkout@v4

      # Step 2: Extract version
      - name: 🏷️ Get Version from Tag
        id: version
        run: |
          VERSION="`${GITHUB_REF#refs/tags/}"
          echo "version=`${VERSION}" >> "`$GITHUB_OUTPUT"
          echo "Deploying version: `${VERSION}"

      # Step 3: Setup Python
      - name: 🐍 Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      # Step 4: Install dependencies
      - name: 📦 Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      # Step 5: Run tests (ensure quality before deploy)
      - name: ✅ Run Tests
        run: |
          pytest tests/ -v
          echo "All tests passed before deployment"

      # Step 6: Simulate deployment
      - name: 🚀 Deploy to Staging
        run: |
          echo "═══════════════════════════════"
          echo "🚀 DEPLOYING TO STAGING"
          echo "═══════════════════════════════"
          echo "Version: `${{ steps.version.outputs.version }}"
          echo "Timestamp: `$(date)"
          echo ""
          echo "Deployment steps:"
          echo "1. ✅ Code pulled from main"
          echo "2. ✅ Dependencies installed"
          echo "3. ✅ Tests ran successfully"
          echo "4. ✅ Application started"
          echo ""
          echo "Staging URL: https://staging.example.com"
          echo "═══════════════════════════════"

      # Step 7: Health check
      - name: 🏥 Health Check
        run: |
          echo "Running health checks..."
          python -c "from src.app import Calculator; c = Calculator(); assert c.add(2, 3) == 5; print('✅ Calculator health check passed')"
          echo "✅ All health checks passed"

      # Step 8: Deployment success notification
      - name: ✨ Deployment Complete
        run: |
          echo "🎉 Successfully deployed version `${{ steps.version.outputs.version }} to staging!"
          echo ""
          echo "Next steps:"
          echo "- QA can test the application"
          echo "- When ready, promote to production"
"@

$workflow | Out-File -Encoding UTF8 .github\workflows\3-deploy-staging.yml
```

### 5.2: Push to GitHub

```powershell
git add .github\workflows\3-deploy-staging.yml
git commit -m "Add deployment workflow"
git push origin develop
```

---

# 🏢 Part 3: Setup GitHub Environments

## Step 6: Create Staging Environment

This tells GitHub where to deploy.

### 6.1: Create Environment

1. **Go to your repo on GitHub.com**
2. **Click "Settings"** tab (top right)
3. **Left menu** → **"Environments"** (scroll down under "Deployments")
4. **Click "New environment"** (green button)
5. **Type name:** `staging`
6. **Press Enter**
7. **Scroll down** → **"Save protection rules"**

**Done!** Environment is created ✅

---

# 🧪 Part 4: Test the Complete Pipeline

## Step 7: Trigger the Pipeline with a Code Change

### 7.1: Make a Code Change

```powershell
# Create a small change
Add-Content -Path README.md -Value "`n## CI/CD Pipeline Status: ✅ Active"

# Stage it
git add README.md

# Commit
git commit -m "test: trigger CI/CD pipeline"

# Push to develop
git push origin develop
```

**Wait 30 seconds for GitHub to detect the push...**

---

## Step 8: Watch Workflow 1 (Build & Test)

### 8.1: Go to Actions Tab

1. **Go to your repo on GitHub.com**
2. **Click "Actions"** tab
3. **Find "1️⃣ Build & Test"** workflow running
4. **Click on it** to watch progress

### 8.2: Watch the Steps

You should see:

```
✅ 📥 Checkout code
✅ 🐍 Setup Python
✅ 📦 Install dependencies
✅ ✅ Run Tests
✅ 🏷️ Generate Version
✅ 📊 Upload Test Report
✅ 📤 Create Draft Release
```

**Wait for all steps to complete (2-3 minutes)**

**Expected Result:** Green checkmarks on all steps ✅

---

## Step 9: Team Approval – Publish the Draft Release

### 9.1: Go to Releases

1. **Go to your repo on GitHub.com**
2. **Click "Releases"** tab
3. **Find the draft release** (gray "Draft" button)
   - Name looks like: `v0.0.0-rc.20260102_150000.abc123`

### 9.2: Review the Release

1. **Click on the draft release**
2. **Read the release notes:**
   - ✅ Tests passed
   - ✅ Build successful
3. **Download test report** (click "Assets" if available)
4. **Review** test coverage

### 9.3: Approve by Publishing

**This is the team approval step!**

1. **Click "Edit"** button (top right)
2. **Find checkbox:** "Set as a draft" (should be ✓ checked)
3. **Click the checkbox** to uncheck it ☐
4. **Scroll down**
5. **Click "Update release"** button

**Wait a few seconds...**

---

## Step 10: Watch Workflow 2 (Approval & Merge)

### 10.1: Check Actions

1. **Go to "Actions"** tab
2. **Find "2️⃣ Approval & Merge to Main"** workflow
3. **Click on it** to watch progress

### 10.2: Watch the Steps

```
✅ 📥 Checkout code
✅ ⚙️ Configure Git
✅ 🌳 Prepare main branch
✅ 🔄 Merge develop → main
✅ 🏷️ Create Release Tag
✅ 📤 Push to GitHub
✅ 📝 Update Release Notes
```

**Wait 1 minute for completion**

**Expected Result:** Green checkmarks ✅

---

## Step 11: Verify Merge & Tag Creation

### 11.1: Check Main Branch

1. **Go to "Code"** tab
2. **Find branch dropdown** (left side)
3. **Click dropdown**
4. **You should see "main"** branch now!
5. **Click "main"** to verify your code is there

### 11.2: Check Version Tag

1. **Go to "Releases"** tab
2. **You should see TWO releases:**
   - Old RC release (draft, gray) ← From Step 8
   - New v1.0.0 release ← **NEW!** Just created!
3. **Click on v1.0.0** to view it

---

## Step 12: Watch Workflow 3 (Deployment)

### 12.1: Check Deployment

1. **Go to "Actions"** tab
2. **Find "3️⃣ Deploy to Staging"** workflow
3. **Click on it** to watch progress

### 12.2: Watch the Steps

```
✅ 📥 Checkout code
✅ 🏷️ Get Version from Tag
✅ 🐍 Setup Python
✅ 📦 Install dependencies
✅ ✅ Run Tests
✅ 🚀 Deploy to Staging
✅ 🏥 Health Check
✅ ✨ Deployment Complete
```

**Wait 1-2 minutes for completion**

**Expected Result:** All green checkmarks ✅

---

## Step 13: Verify Deployment Succeeded

### 13.1: Check Workflow Output

1. **Open "3️⃣ Deploy to Staging"** workflow
2. **Find "Deploy to Staging"** job
3. **Scroll to "🚀 Deploy to Staging"** step
4. **You should see:**
   ```
   ═══════════════════════════════
   🚀 DEPLOYING TO STAGING
   ═══════════════════════════════
   Version: v1.0.0
   Timestamp: [date]
   ...
   ✅ Application started
   Staging URL: https://staging.example.com
   ═══════════════════════════════
   ```

### 13.2: Check Health Check

1. **Find "🏥 Health Check"** step
2. **You should see:**
   ```
   ✅ Calculator health check passed
   ✅ All health checks passed
   ```

---

# 📊 Complete Pipeline Flow Summary

Here's what happened in order:

| Step | Trigger | Action | Time | Status |
|------|---------|--------|------|--------|
| 1 | You pushed to develop | Workflow 1 starts | 0 min | Automatic ✅ |
| 2 | Tests run | Draft release created | 3 min | Automatic ✅ |
| 3 | You click "Edit" + Publish | Workflow 2 starts | 3 min | **Manual** 👥 |
| 4 | Release published | Merge to main + create tag | 5 min | Automatic ✅ |
| 5 | Tag created | Workflow 3 starts | 5 min | Automatic ✅ |
| 6 | Tests pass | Deploy to staging | 7 min | Automatic ✅ |
| 7 | Health checks pass | **Staging is LIVE!** 🎉 | 9 min | Done! ✅ |

**Total time:** ~10 minutes from code push to live staging

---

# 🔄 Day-to-Day Usage

Now that it's set up, here's how you use it:

## Developer's Daily Workflow

```powershell
# Make changes to your code
# Edit src/app.py or tests/test_app.py

# Test locally (optional)
pytest tests/

# Commit changes
git add .
git commit -m "feature: add new calculator function"

# Push to develop
git push origin develop

# ← GitHub Actions automatically runs tests
# ← Draft release is created
# ← Workflow 1 completes in ~3 min
```

## Team Approval Workflow

```
1. Get notified of new draft release
2. Go to Releases tab
3. Download test report
4. Review test results
5. If OK:
   - Click "Edit"
   - Uncheck "Draft"
   - Click "Update release"
6. ← GitHub Actions automatically merges & deploys
7. ← QA can test in staging ~10 min after push
```

---

# 🎯 Key Concepts

### Branch Strategy

| Branch | Purpose | Who | When |
|--------|---------|-----|------|
| **develop** | Daily development | Developers | Daily |
| **main** | Production-ready | Approved code only | After approval |

### Workflow Triggers

| Workflow | Triggers On | Time | Next Step |
|----------|------------|------|-----------|
| **Build & Test** | Push to develop | 2-3 min | Draft release created |
| **Approval & Merge** | Release published | 1 min | main branch merged |
| **Deploy** | Tag created | 1-2 min | Staging is live |

### Approval Gate

**The draft release is your approval gate!**

```
Build passes → Draft release created
                        ↓
                    Team reviews
                        ↓
                  Team approves (publishes release)
                        ↓
              Auto-merge & auto-deploy
```

---

# ✅ Verification Checklist

After completing all steps, verify everything works:

### Workflow 1 ✅
- [ ] Workflow "1️⃣ Build & Test" ran
- [ ] All tests passed ✅
- [ ] Draft release created

### Workflow 2 ✅
- [ ] You published the draft release
- [ ] Workflow "2️⃣ Approval & Merge to Main" ran
- [ ] main branch was created
- [ ] Release tag v1.0.0 was created

### Workflow 3 ✅
- [ ] Workflow "3️⃣ Deploy to Staging" ran
- [ ] Tests ran again during deployment
- [ ] Health checks passed
- [ ] All steps have green checkmarks ✅

### Overall ✅
- [ ] Three workflows exist in `.github/workflows/`
- [ ] Staging environment configured
- [ ] Complete flow works end-to-end
- [ ] Total time: ~10 minutes from push to staging live

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

You've now created a **complete CI/CD pipeline**:

✅ **Continuous Integration:** Code is tested automatically  
✅ **Continuous Delivery:** Code is prepared for deployment  
✅ **Approval Gate:** Team reviews before deployment  
✅ **Continuous Deployment:** Code automatically deploys after approval  
✅ **Tagging:** Version tags trigger deployment  
✅ **All in GitHub:** No external tools needed  

**This is production-grade automation!** 🚀

---

## 💡 Key Takeaways

1. **Develop branch** = where you work daily
2. **Build & Test workflow** = automatic on every push
3. **Draft release** = waiting for team approval
4. **Team publishes release** = approval happens here
5. **Main branch** = gets merged automatically
6. **Version tag** = triggers deployment automatically
7. **Staging environment** = where code runs after approval

**The entire flow is automated except for step 4 (team approval).**

---

**You're now a CI/CD expert! Congratulations! 🎉**
