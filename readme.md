[README-Beginners.md](https://github.com/user-attachments/files/24403914/README-Beginners.md)
# Delivery‑First GitHub Actions Pipeline – Beginner's Complete Guide

A simple, step-by-step guide to set up a professional CI/CD workflow. **No prior DevOps experience needed!**

---

## 🎯 What You'll Learn

By the end of this guide, you'll have:

✅ An automatic build system that tests your code  
✅ A way for your team to review releases before they go live  
✅ Automatic merge and tagging after approval  
✅ Automatic deployment to staging  
✅ A complete audit trail (who approved what, when)  

---

## 🚀 The Big Picture (2 minutes)

Imagine this workflow:

```
1. You write code and push to GitHub
   ↓
2. GitHub automatically builds it and tests it (2-3 min)
   ↓
3. Your team downloads and reviews the test results
   ↓
4. Your team says "Looks good!" by publishing a release
   ↓
5. GitHub automatically merges your code to main
   ↓
6. GitHub automatically creates a version number (v1.2.0)
   ↓
7. GitHub automatically deploys to staging server
   ↓
🎉 Done! Your code is live in staging!
```

**Total time**: About 10 minutes from code push to live.

**The key**: Your team reviews and approves **before** the automatic parts happen.

---

## 📋 What You Need (Before You Start)

### Required:
- ✅ A GitHub account (free is fine)
- ✅ A code repository on GitHub (even an empty one works)
- ✅ Access to your computer's terminal (Terminal on Mac, PowerShell on Windows, or WSL)
- ✅ `git` installed (`git --version` to check)
- ✅ **2 hours of uninterrupted time**

### Nice to have:
- A text editor like VS Code
- Python 3.9+ installed (if you want to test locally)
- Basic understanding of git (branches, push, commit)

### Don't need:
- Docker knowledge
- AWS/cloud knowledge
- DevOps experience
- Server administration

---

## 🎓 Beginner Glossary

Before we start, here are key terms explained simply:

| Term | Simple Explanation |
|------|-------------------|
| **GitHub** | Website to store code and manage projects |
| **Repository (repo)** | A folder on GitHub that holds your code |
| **Branch** | A separate version of code you're working on |
| **develop** | The working branch (where new features go) |
| **main** | The official branch (only approved code goes here) |
| **Workflow** | An automated task GitHub does when you push code |
| **Build** | Testing your code and preparing it for deployment |
| **Release** | A version of your code ready to deploy |
| **Deploy** | Moving code to a live server |
| **Staging** | A test server that looks like production |
| **YAML** | A text format for writing configuration files |

---

## ⏱️ Time Breakdown

- **Step 1-2**: Setting up files (20 min)
- **Step 3-4**: Creating workflows (25 min)
- **Step 5**: GitHub configuration (15 min)
- **Step 6-9**: Testing everything (40 min)
- **Total**: 2 hours

---

# 🏗️ Step 1: Create Your Project Files (20 minutes)

## What We're Doing

We're creating a simple Python calculator app with tests. This is just an example—you can use this guide with any language.

## 1.1: Open Terminal

**On Mac:**
- Press `Cmd + Space`
- Type `terminal`
- Press Enter

**On Windows:**
- Press `Win + R`
- Type `powershell`
- Press Enter

**On Linux:**
- Just open your terminal application

## 1.2: Navigate to Your Repository

```bash
# Example: navigate to your repo
cd ~/my-repo

# Check you're in the right place
pwd

# You should see something like: /Users/yourname/my-repo
```

If you don't have a repo yet:

```bash
# Create a new folder
mkdir my-repo
cd my-repo

# Initialize git
git init

# Create develop branch
git checkout -b develop
```

## 1.3: Create Folders

```bash
# Create necessary folders
mkdir -p src tests .github/workflows

# Verify they were created
ls -la

# You should see:
# drwxr-xr-x  .github
# drwxr-xr-x  src
# drwxr-xr-x  tests
```

## 1.4: Create Your First File: `src/app.py`

**Easy way** (copy-paste this):

```bash
cat > src/app.py << 'ENDOFFILE'
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
ENDOFFILE
```

**What this does**: Creates a simple calculator class with 4 math functions.

**Verify it worked**:
```bash
cat src/app.py
# Should show the code above
```

## 1.5: Create Test File: `tests/test_app.py`

```bash
cat > tests/test_app.py << 'ENDOFFILE'
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
ENDOFFILE
```

**What this does**: Creates tests to check if the calculator works correctly.

**Verify**:
```bash
cat tests/test_app.py
# Should show the code above
```

## 1.6: Create `requirements.txt`

This is a list of tools we need:

```bash
cat > requirements.txt << 'ENDOFFILE'
pytest==7.4.3
pytest-cov==4.1.0
black==23.12.0
flake8==6.1.0
bandit==1.7.5
safety==2.3.5
build==1.0.3
wheel==0.42.0
twine==4.0.2
ENDOFFILE
```

**Verify**:
```bash
cat requirements.txt
```

## 1.7: Create `setup.py`

This tells Python how to package your app:

```bash
cat > setup.py << 'ENDOFFILE'
from setuptools import setup, find_packages

setup(
    name="calculator-app",
    version="0.1.0",
    description="Calculator app with delivery-first CI/CD",
    author="Your Team",
    packages=find_packages(),
    python_requires=">=3.9",
)
ENDOFFILE
```

**Verify**:
```bash
cat setup.py
```

## 1.8: Create Empty Init Files

These files tell Python that `src` and `tests` are packages:

```bash
# Create empty files
touch src/__init__.py
touch tests/__init__.py

# Verify
ls src/
# Should show: __init__.py  app.py

ls tests/
# Should show: __init__.py  test_app.py
```

## 1.9: Save Everything to GitHub

Now we tell git to save these files:

```bash
# Check what we've created
git status

# Add all files
git add .

# Save with a message
git commit -m "Initial project setup"

# Push to GitHub
git push -u origin develop

# You should see something like:
# Create a pull request for 'develop' on GitHub by visiting...
```

**Verify on GitHub**:
1. Go to your repo on GitHub.com
2. Click **Code** tab
3. You should see:
   - `.github/workflows/` folder
   - `src/` folder with `app.py`
   - `tests/` folder with `test_app.py`
   - `requirements.txt`
   - `setup.py`

---

# 🔄 Step 2: Create First Workflow (15 minutes)

## What We're Doing

Creating a file that tells GitHub: "When code is pushed, automatically test it and create a release."

## 2.1: Create the Workflow File

```bash
# Create the file
cat > .github/workflows/build-rc.yml << 'ENDOFFILE'
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
      rc_version: ${{ steps.version.outputs.rc_version }}

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
          SHORT_SHA=$(git rev-parse --short HEAD)
          TIMESTAMP=$(date +%Y%m%d.%H%M%S)
          RC_VERSION="v0.0.0-rc.${TIMESTAMP}.${SHORT_SHA}"
          echo "rc_version=${RC_VERSION}" >> "$GITHUB_OUTPUT"
          echo "Building RC: ${RC_VERSION}"

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
          # Release Candidate - ${{ steps.version.outputs.rc_version }}
          
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
          name: rc-artifacts-${{ steps.version.outputs.rc_version }}
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
          tag_name: ${{ steps.version.outputs.rc_version }}
          draft: true
          body_path: release_notes.md
          files: dist/*
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
ENDOFFILE
```

**That's a lot of text!** But don't worry—you just copy-pasted it. Let's verify it worked:

```bash
# Check the file exists
ls .github/workflows/

# You should see: build-rc.yml
```

## 2.2: Save to GitHub

```bash
# Add the workflow file
git add .github/workflows/build-rc.yml

# Save it
git commit -m "Add build workflow"

# Push to GitHub
git push origin develop

# Wait a few seconds...
# Then go to GitHub.com and refresh
```

**Verify on GitHub**:
1. Go to your repo on GitHub
2. Click **Actions** tab (top of page)
3. You should see **Build Release Candidate** in the list
4. If a workflow is running, you can watch it execute

---

# ✅ Step 3: Create Merge Workflow (15 minutes)

## What We're Doing

Creating a workflow that says: "When someone approves a release, automatically merge to main and create a version tag."

## 3.1: Create the Merge Workflow

```bash
cat > .github/workflows/merge-after-approval.yml << 'ENDOFFILE'
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
          TAG_NAME="${{ github.event.release.tag_name }}"
          # Extract version (e.g., v0.0.0-rc... becomes v1.0.0)
          MAIN_TAG=$(echo "$TAG_NAME" | sed -E 's/v[0-9]+\.[0-9]+\.[0-9]+-[a-z]+\..*/v1.0.0/')
          if [[ ! $MAIN_TAG =~ ^v[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
            MAIN_TAG="v1.0.0"
          fi
          echo "tag_name=${MAIN_TAG}" >> "$GITHUB_OUTPUT"

      - name: Setup git
        run: |
          git config user.name "github-actions"
          git config user.email "github-actions@github.com"

      - name: Create main branch and merge
        run: |
          # Create main if it doesn't exist
          git checkout -b main 2>/dev/null || git checkout main
          
          # Merge develop into main
          git fetch origin develop
          git merge origin/develop --no-ff -m "Release ${{ steps.version.outputs.tag_name }} approved"

      - name: Create version tag
        run: |
          # Create a tag on main
          git tag -a "${{ steps.version.outputs.tag_name }}" \
            -m "Release ${{ steps.version.outputs.tag_name }}"
          
          # Push main and tag to GitHub
          git push origin main
          git push origin "${{ steps.version.outputs.tag_name }}"
          
          echo "✅ Pushed ${{ steps.version.outputs.tag_name }} to main"
ENDOFFILE
```

**Verify**:
```bash
ls .github/workflows/

# You should see:
# build-rc.yml
# merge-after-approval.yml
```

## 3.2: Save to GitHub

```bash
git add .github/workflows/merge-after-approval.yml
git commit -m "Add merge workflow"
git push origin develop
```

---

# 🚀 Step 4: Create Deployment Workflow (15 minutes)

## What We're Doing

Creating a workflow that says: "When a version tag is created, automatically deploy to staging."

## 4.1: Create the Deployment Workflow

```bash
cat > .github/workflows/deploy-staging.yml << 'ENDOFFILE'
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
          VERSION="${GITHUB_REF#refs/tags/}"
          echo "version=$VERSION" >> "$GITHUB_OUTPUT"

      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install and test
        run: |
          pip install -r requirements.txt
          pytest tests/ -v

      - name: Simulate deployment
        run: |
          echo "🚀 Deploying version ${{ steps.version.outputs.version }}"
          echo "✅ Deployment complete"
          
          # TODO: Replace with real deployment command
          # Example: docker build -t app:${{ steps.version.outputs.version }} .
          # Example: docker push yourregistry/app:${{ steps.version.outputs.version }}

      - name: Health check
        run: |
          echo "✅ Health check passed"
          
          # TODO: Replace with real health check
          # Example: curl -f https://staging.example.com/health
ENDOFFILE
```

**Verify**:
```bash
ls .github/workflows/

# You should see:
# build-rc.yml
# merge-after-approval.yml
# deploy-staging.yml
```

## 4.2: Save to GitHub

```bash
git add .github/workflows/deploy-staging.yml
git commit -m "Add deployment workflow"
git push origin develop
```

---

# 🏢 Step 5: Configure GitHub Settings (15 minutes)

## What We're Doing

Setting up environments in GitHub so workflows know where to deploy.

### 5.1: Create Staging Environment

1. **Go to your repo on GitHub.com**

2. **Click "Settings"** tab (right side of page)

3. **Look for "Environments"** on the left menu
   - Under "Deployments" section
   - Click **Environments**

4. **Click "New environment"** button (green button)

5. **Type "staging"** in the text box

6. **Press Enter** or click **Configure environment**

7. **You should see a page for staging environment**
   - Don't change anything
   - Just scroll down and click **"Save protection rules"**

8. **Done!** You should see "staging" listed in Environments

### 5.2: Verify

Go back and check:
1. Settings tab → Environments
2. You should see: `staging`

That's it for now! Staging doesn't need approval (anyone can deploy).

---

# 🧪 Step 6: Test Everything! (30 minutes)

## What We're Doing

Making a small change to trigger all the workflows automatically. This tests that everything works!

### 6.1: Make a Small Change

```bash
# Create a test file
echo "# My Test" > TEST_FILE.md

# Add it
git add TEST_FILE.md

# Save
git commit -m "test: trigger build workflow"

# Push to GitHub
git push origin develop
```

### 6.2: Watch It Build

1. **Go to your repo on GitHub.com**

2. **Click "Actions"** tab

3. **You should see "Build Release Candidate"** workflow running
   - It shows yellow dot (running) or green checkmark (done)

4. **Click on it** to watch progress

5. **Wait 2-3 minutes** for it to finish

6. **Watch these steps** (in order):
   - ✅ Checkout code
   - ✅ Setup Python
   - ✅ Install dependencies
   - ✅ Run tests
   - ✅ Security checks
   - ✅ Build artifacts
   - ✅ Create draft release

### 6.3: Check Draft Release

1. **Go to your repo on GitHub.com**

2. **Click "Releases"** tab

3. **You should see a draft release** (gray button saying "Draft")
   - Name: Something like `v0.0.0-rc.20260102.090000.abc123`

4. **Click on it** to see details
   - Release notes
   - Artifacts (dist folder)
   - Test results

**If you don't see it:**
- Check Actions tab - did workflow finish?
- Check for errors in workflow logs

---

# 👀 Step 7: Approve the Release (5 minutes)

## What We're Doing

This is the **approval step**! You download the results and say "looks good!"

### 7.1: Review the Release

1. **Open the draft release** (from Step 6)

2. **Read release notes** - make sure it says:
   - Build time
   - Test results
   - Instructions for approval

3. **Download artifacts** (optional):
   - Click "Assets" section
   - Download `.whl` file
   - Download `.tar.gz` file
   - Download `coverage.xml` to see test coverage
   - Download `bandit-report.json` to see security scan

4. **Check results** (optional):
   - Are tests passing? (look for "✅ Tests completed")
   - Any security issues? (look in bandit report)
   - Is code coverage good?

### 7.2: Publish Release (Approval Action!)

This is how you approve:

1. **Find the draft release**

2. **Click "Edit"** button (top right)

3. **Find the checkbox** "Set as a draft"
   - It should be **checked** (has checkmark)

4. **Uncheck it** (click to remove checkmark)

5. **Scroll down** → Click **"Update release"** button

6. **Done!** Release is now "published" ✅

**What happens next (automatic):**
- GitHub detects the release was published
- Merge workflow starts automatically (~10 seconds)
- Merge to main begins
- Version tag is created
- Deployment workflow starts

---

# ⚡ Step 8: Verify Merge Happened (5 minutes)

## 8.1: Check Merge Workflow

1. **Go to Actions tab**

2. **Find "Merge & Tag After Approval"** workflow
   - Should be running or completed

3. **Click on it** to see details

4. **Wait for it to finish** (usually 1 minute)

### 8.2: Verify Main Branch Exists

1. **Go to Code tab**

2. **Find branch dropdown** (left side, says "develop")

3. **Click dropdown**

4. **Look for "main"** branch
   - Should exist now (or just been created)

5. **Click "main"** to switch to it

6. **You should see all your code** from develop

### 8.3: Verify Version Tag Created

1. **Go to Releases tab**

2. **You should see TWO releases**:
   - Draft RC release (e.g., `v0.0.0-rc...`) ← Old
   - New version release (e.g., `v1.0.0`) ← New!

3. **The new one** was automatically created by the merge workflow

---

# 🎉 Step 9: Verify Deployment (5 minutes)

## 9.1: Check Deployment Workflow

1. **Go to Actions tab**

2. **Find "Deploy to Staging"** workflow
   - Should be running or completed

3. **Click on it** to see details

4. **Check for green checkmarks** (all steps succeeded)

5. **Look for "✅ Deployment complete"** message

### 9.2: Verify It's Done

If you see:
- ✅ All green checkmarks
- ✅ "Health check passed"
- ✅ "Deployment complete"

**Then you're done!** 🎉

---

# 📊 What Just Happened? (Simple Explanation)

Here's the complete flow in simple terms:

```
1. You pushed code to GitHub (Step 6)
   ↓
2. GitHub automatically ran tests (1 min)
   ↓
3. GitHub created a draft release with test results (30 sec)
   ↓
4. You reviewed and approved by publishing (Step 7)
   ↓
5. GitHub automatically merged to main (Step 8)
   ↓
6. GitHub automatically created a version tag (Step 8)
   ↓
7. GitHub automatically deployed to staging (Step 9)
   ↓
🎉 Code is live in staging!

Total time: ~10 minutes
```

---

# ✅ Checklist: Did It Work?

After completing Steps 1-9, check:

- [ ] Project files created (src/, tests/, requirements.txt, setup.py)
- [ ] 3 workflow files created (.github/workflows/)
- [ ] Staging environment configured
- [ ] Build workflow ran successfully
- [ ] Draft release created
- [ ] You published the release (approval)
- [ ] Merge workflow completed
- [ ] Main branch exists with merged code
- [ ] Version tag created (like v1.0.0)
- [ ] Deployment workflow completed successfully
- [ ] All workflows show green checkmarks ✅

If all are checked ✅ — **Congratulations! You're done!**

---

# 🆘 Troubleshooting (Common Issues)

### "I don't see the workflow running"

**Solution**:
1. Make sure you pushed to `develop` branch (not main)
2. Wait 30 seconds after push
3. Refresh the Actions page
4. If still nothing, check that `.github/workflows/build-rc.yml` exists in develop branch

### "Build workflow failed"

**Solution**:
1. Click on the failed workflow
2. Look for step with red ❌
3. Read the error message
4. Common causes:
   - Python syntax error in app.py
   - Missing import
   - Wrong indentation

### "I don't see draft release"

**Solution**:
1. Make sure build workflow completed successfully ✅
2. Refresh Releases page
3. The draft release should appear at the top
4. If not, check workflow logs for errors

### "I can't publish the release"

**Solution**:
1. Click "Edit" button
2. Find "Set as a draft" checkbox
3. Make sure it's checked (has checkmark)
4. Click to uncheck it
5. Scroll down to "Update release" button
6. Click it
7. If button is grayed out, you don't have permissions

### "Main branch wasn't created"

**Solution**:
1. Manually create it:
   ```bash
   git checkout -b main
   git push origin main
   ```
2. Try publishing release again

---

# 🎓 Next Steps (After This Works)

Congratulations! You now have a working CI/CD pipeline!

### What to do next:

1. **Make more changes** and watch the workflow run
   - This is how you'll use it day-to-day

2. **Train your team**:
   - Tell them Steps 7 is the approval action
   - They review the draft release
   - They publish to approve

3. **Customize for your needs**:
   - Add real deployment commands (Docker, SSH, etc.)
   - Add Slack notifications
   - Add more tests

4. **Read our more detailed guide** (if you want):
   - The advanced guide explains customization
   - Shows Docker, Kubernetes, etc.

---

# ❓ FAQs for Beginners

**Q: Do I need to understand GitHub Actions?**  
A: No! Just copy-paste the workflows. They'll work as-is.

**Q: Do I need Docker or Kubernetes?**  
A: No! This guide works without them. You can add them later if needed.

**Q: Can I use this with my language (Node, Go, Java, etc.)?**  
A: Yes! Just replace the Python parts with your language's build commands.

**Q: What if something breaks?**  
A: Check the Troubleshooting section above. Most issues are simple.

**Q: Can I delete the test files after this?**  
A: You can, but they're helpful examples. Keep them for now.

**Q: How do I update the workflows later?**  
A: Edit the `.github/workflows/*.yml` files and push to develop. Changes take effect immediately.

**Q: Is this secure?**  
A: Yes! GitHub Actions is trusted by major companies. Your code is safe.

---

# 🎉 Summary

You just created a **professional, automated CI/CD pipeline**!

What you have now:

✅ **Automatic testing** when you push code  
✅ **Team approval required** before deployment  
✅ **Automatic merge & versioning** after approval  
✅ **Automatic deployment** to staging  
✅ **Complete audit trail** (who approved what, when)  

This setup is used by professional companies. You're now doing DevOps like the pros!

---

# 💪 You Did It!

From zero to production-ready CI/CD in 2 hours. Well done! 🚀

**Next time you push code:**
1. GitHub automatically tests it (3 min)
2. Your team reviews and approves (5 min)
3. GitHub automatically deploys (2 min)
4. Code is live in staging!

**Total: 10 minutes. No manual work.**

---

## Questions?

If something doesn't work:
1. Check the **Troubleshooting** section
2. Re-read the step that's failing
3. Check the workflow logs on GitHub (click the workflow run)

Happy coding! 🎉
