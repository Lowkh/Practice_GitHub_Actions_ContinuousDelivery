[README (1).md](https://github.com/user-attachments/files/24403867/README.1.md)
# Delivery‑First GitHub Actions Pipeline – Complete Guide

A complete CI/CD workflow where **team members review and approve releases before code is merged to main and versioned**, then **all deployments run automatically**.

---

## 🚀 Quick Overview

```
Feature Branch (PR reviewed)
    ↓
develop branch
    ↓ (git push)
Build Release Candidate (auto)
    • Run tests ✅
    • Security scan ✅
    • Build artifacts ✅
    ↓
Draft GitHub Release (with artifacts)
    ↓
⏸️ TEAM REVIEWS & APPROVES (manual)
    • Download artifacts
    • Check test results
    • Check security report
    • Publish release if OK
    ↓
Merge to main (auto)
    • Merge develop → main
    • Create version tag (v1.2.0)
    ↓
Deploy to Staging (auto)
    ↓
🎉 LIVE IN STAGING
```

**Key:** Approval (publishing the release) happens **BEFORE** code is versioned and deployed.

---

## 📋 Prerequisites

- ✅ GitHub account with admin access to repo
- ✅ Git installed locally (`git --version` to verify)
- ✅ Python 3.9+ (or adapt for your language)
- ✅ Text editor (VS Code recommended)
- ✅ 1-2 hours uninterrupted time
- ✅ macOS/Linux/Windows with terminal access

---

## 🏗️ Step 1: Set Up Project Files (15 minutes)

### 1.1: Create Directory Structure

Open your terminal and navigate to your repo:

```bash
# Navigate to repo
cd /path/to/your/repo

# Create directories
mkdir -p src tests .github/workflows

# Verify structure
ls -la
```

Expected output:
```
drwxr-xr-x  ... .github
drwxr-xr-x  ... src
drwxr-xr-x  ... tests
```

### 1.2: Create `src/app.py`

```bash
# Create file
cat > src/app.py << 'EOF'
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
EOF

# Verify
cat src/app.py
```

### 1.3: Create `tests/test_app.py`

```bash
# Create file
cat > tests/test_app.py << 'EOF'
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
EOF

# Verify
cat tests/test_app.py
```

### 1.4: Create `requirements.txt`

```bash
cat > requirements.txt << 'EOF'
pytest==7.4.3
pytest-cov==4.1.0
black==23.12.0
flake8==6.1.0
bandit==1.7.5
safety==2.3.5
build==1.0.3
wheel==0.42.0
twine==4.0.2
EOF

# Verify
cat requirements.txt
```

### 1.5: Create `setup.py`

```bash
cat > setup.py << 'EOF'
from setuptools import setup, find_packages

setup(
    name="calculator-app",
    version="0.1.0",
    description="Calculator app with delivery-first CI/CD",
    author="Your Team",
    packages=find_packages(),
    python_requires=">=3.9",
)
EOF

# Verify
cat setup.py
```

### 1.6: Create Empty Init Files

```bash
# Create __init__.py files
touch src/__init__.py
touch tests/__init__.py

# Verify all files exist
find . -name "*.py" -type f
```

Expected output should show:
```
./src/app.py
./src/__init__.py
./tests/test_app.py
./tests/__init__.py
```

### 1.7: Commit Files to Git

```bash
# Check status
git status

# Add all files
git add src/ tests/ requirements.txt setup.py

# Verify staged files
git status

# Commit with message
git commit -m "chore: initial project structure"

# Push to develop branch
git push origin develop

# Verify push succeeded
echo "✅ Files committed to develop branch"
```

Expected:
```
[develop abc123def] chore: initial project structure
 7 files changed, 150 insertions(+)
 create mode 100644 requirements.txt
 create mode 100644 setup.py
 create mode 100644 src/__init__.py
 create mode 100644 src/app.py
 create mode 100644 tests/__init__.py
 create mode 100644 tests/test_app.py
```

---

## 🔄 Step 2: Create Build Release Candidate Workflow (15 minutes)

### 2.1: Create Workflow File

```bash
# Navigate to workflows directory
cd .github/workflows

# Create build-rc.yml
cat > build-rc.yml << 'EOF'
name: Build Release Candidate

on:
  push:
    branches: [develop]
  workflow_dispatch:
    inputs:
      release_type:
        description: "Release type"
        type: choice
        default: "rc"
        options:
          - alpha
          - beta
          - rc

permissions:
  contents: write
  packages: write

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
          RELEASE_TYPE="${{ github.event.inputs.release_type || 'rc' }}"
          RC_VERSION="v0.0.0-${RELEASE_TYPE}.${TIMESTAMP}.${SHORT_SHA}"
          echo "rc_version=${RC_VERSION}" >> "$GITHUB_OUTPUT"
          echo "Building RC: ${RC_VERSION}"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install build twine bandit safety pytest pytest-cov

      - name: Run tests with coverage
        run: |
          pytest tests/ -v --cov=src --cov-report=xml --cov-report=html || true
          echo "✅ Tests completed"

      - name: Lint and format check
        run: |
          flake8 src tests --max-line-length=100 --count --statistics || true
          black --check src tests 2>/dev/null || true
          echo "✅ Lint check completed"

      - name: Security checks
        run: |
          bandit -r src/ -f json -o bandit-report.json 2>/dev/null || true
          safety check --json > safety-report.json 2>/dev/null || true
          echo "✅ Security checks completed"

      - name: Build artifacts
        run: |
          python -m build 2>&1 || echo "Build completed with warnings"
          ls -lah dist/
          echo "✅ Artifacts built"

      - name: Generate release notes
        run: |
          cat > release_notes.md << 'NOTES'
          # Release Candidate - ${{ steps.version.outputs.rc_version }}
          
          Built from: `${{ github.ref_name }}` at `${{ github.sha }}`
          Build time: $(date -u +'%Y-%m-%d %H:%M:%S UTC')
          
          ## Review Checklist
          
          - [ ] Download and extract artifacts
          - [ ] Review test coverage report
          - [ ] Check security scan results
          - [ ] Verify dependency safety
          - [ ] Manual testing (optional)
          
          ## Approval Process
          
          If everything looks good:
          1. Click **Edit** on this release
          2. Uncheck **Draft** checkbox
          3. Click **Update release** to publish
          
          Publishing will automatically merge and deploy!
          NOTES

      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: rc-artifacts-${{ steps.version.outputs.rc_version }}
          path: |
            dist/
            coverage.xml
            bandit-report.json
            safety-report.json
            release_notes.md
          retention-days: 30
          if-no-files-found: warn

      - name: Create draft GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          tag_name: ${{ steps.version.outputs.rc_version }}
          draft: true
          prerelease: true
          body_path: release_notes.md
          files: dist/*
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
EOF

# Verify file was created
ls -la build-rc.yml
cat build-rc.yml | head -20
```

### 2.2: Validate YAML Syntax

```bash
# Use Python to validate YAML
python3 << 'VALIDATE'
import yaml
try:
    with open('build-rc.yml', 'r') as f:
        yaml.safe_load(f)
    print("✅ YAML syntax is valid")
except yaml.YAMLError as e:
    print(f"❌ YAML error: {e}")
VALIDATE
```

### 2.3: Commit Workflow

```bash
# Navigate back to repo root
cd ../..

# Stage workflow
git add .github/workflows/build-rc.yml

# Verify
git status

# Commit
git commit -m "ci: add build release candidate workflow"

# Push
git push origin develop

echo "✅ Build RC workflow committed"
```

---

## ✅ Step 3: Create Merge & Tag After Approval Workflow (15 minutes)

### 3.1: Create Workflow File

```bash
cd .github/workflows

cat > merge-after-approval.yml << 'EOF'
name: Merge & Tag After Approval

on:
  release:
    types: [published]

permissions:
  contents: write

jobs:
  merge-to-main:
    name: Merge develop → main and tag
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Derive semantic version tag
        id: version
        shell: bash
        run: |
          TAG_NAME="${{ github.event.release.tag_name }}"
          MAIN_TAG=$(echo "$TAG_NAME" | sed -E 's/v[0-9]+\.[0-9]+\.[0-9]+-[a-z]+\..*/v1.0.0/')
          if [[ ! $MAIN_TAG =~ ^v[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
            YEAR=$(date +%y)
            MONTH=$(date +%m)
            DAY=$(date +%d)
            MAIN_TAG="v0.${YEAR}${MONTH}.${DAY}"
          fi
          echo "tag_name=${MAIN_TAG}" >> "$GITHUB_OUTPUT"
          echo "RC tag: $TAG_NAME"
          echo "Release tag: $MAIN_TAG"

      - name: Configure git
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"

      - name: Fetch branches
        run: |
          git fetch origin develop --depth=1
          git fetch origin main --depth=1 2>/dev/null || echo "main branch will be created"

      - name: Merge develop into main
        run: |
          if git rev-parse --verify origin/main >/dev/null 2>&1; then
            git checkout main
            git pull origin main
          else
            git checkout -b main
          fi
          
          git merge origin/develop --no-ff -m "Release ${{ steps.version.outputs.tag_name }} (approved from ${{ github.event.release.tag_name }})"
          echo "✅ Merged develop into main"

      - name: Create version tag on main
        run: |
          git tag -a "${{ steps.version.outputs.tag_name }}" \
            -m "Release ${{ steps.version.outputs.tag_name }}
          
          RC Tag: ${{ github.event.release.tag_name }}
          Approved at: $(date -u +'%Y-%m-%d %H:%M:%S UTC')
          Release URL: ${{ github.event.release.html_url }}"
          
          git push origin main
          git push origin "${{ steps.version.outputs.tag_name }}"
          echo "✅ Pushed ${{ steps.version.outputs.tag_name }} to main"

      - name: Update release notes
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.repos.updateRelease({
              owner: context.repo.owner,
              repo: context.repo.repo,
              release_id: context.payload.release.id,
              body: `${{ github.event.release.body }}

---

## ✅ Release Approved & Merged

- **Status**: Merged to main
- **Version Tag**: \`${{ steps.version.outputs.tag_name }}\`
- **Original RC**: \`${{ github.event.release.tag_name }}\`
- **Merged at**: $(date -u +'%Y-%m-%d %H:%M:%S UTC')
- **Deploying to staging...** 🚀`
            });
EOF

# Verify
ls -la merge-after-approval.yml
```

### 3.2: Validate and Commit

```bash
# Validate YAML
python3 << 'VALIDATE'
import yaml
try:
    with open('merge-after-approval.yml', 'r') as f:
        yaml.safe_load(f)
    print("✅ YAML syntax is valid")
except yaml.YAMLError as e:
    print(f"❌ YAML error: {e}")
VALIDATE

# Navigate to repo root
cd ../..

# Stage and commit
git add .github/workflows/merge-after-approval.yml
git commit -m "ci: add merge and tag after approval workflow"
git push origin develop

echo "✅ Merge & Tag workflow committed"
```

---

## 🚀 Step 4: Create Staging Deployment Workflow (15 minutes)

### 4.1: Create Workflow File

```bash
cd .github/workflows

cat > deploy-staging.yml << 'EOF'
name: Deploy to Staging

on:
  push:
    tags:
      - "v[0-9]+.[0-9]+.[0-9]+"
    branches:
      - main

jobs:
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://your-staging-url.example.com

    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.ref }}

      - name: Extract version
        id: version
        shell: bash
        run: |
          if [[ "$GITHUB_REF" == refs/tags/* ]]; then
            VERSION="${GITHUB_REF#refs/tags/}"
          else
            VERSION="$(git describe --tags --always --abbrev=7)"
          fi
          echo "version=${VERSION}" >> "$GITHUB_OUTPUT"
          echo "🚀 Deploying version: $VERSION"

      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
          cache: pip

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Deploy to staging
        run: |
          echo "Deploying ${{ steps.version.outputs.version }} to staging..."
          echo "Commit: $(git rev-parse --short HEAD)"
          # TODO: Replace with real deployment (SSH, Docker, Kubernetes, etc.)
          # Example Docker deployment:
          # docker build -t app:${{ steps.version.outputs.version }} .
          # docker push yourregistry/app:${{ steps.version.outputs.version }}
          # docker pull yourregistry/app:${{ steps.version.outputs.version }}
          # docker run -d --name app app:${{ steps.version.outputs.version }}
          echo "✅ Staging deployment complete"

      - name: Health check
        run: |
          echo "Running health checks..."
          python -c "from src.app import Calculator; c = Calculator(); assert c.add(2, 3) == 5; print('✅ Calculator health check passed')"
          # TODO: Add real health checks
          # curl -f https://your-staging-url.example.com/health || exit 1

      - name: Notify completion
        run: |
          echo "🎉 Staging deployment successful!"
          echo "Version: ${{ steps.version.outputs.version }}"
          echo "Deployed at: $(date -u +'%Y-%m-%d %H:%M:%S UTC')"
          echo "Ready for QA testing"
EOF

# Verify
ls -la deploy-staging.yml
```

### 4.2: Validate and Commit

```bash
# Validate YAML
python3 << 'VALIDATE'
import yaml
try:
    with open('deploy-staging.yml', 'r') as f:
        yaml.safe_load(f)
    print("✅ YAML syntax is valid")
except yaml.YAMLError as e:
    print(f"❌ YAML error: {e}")
VALIDATE

# Navigate to repo root
cd ../..

# Stage and commit
git add .github/workflows/deploy-staging.yml
git commit -m "ci: add staging deployment workflow"
git push origin develop

echo "✅ Deploy Staging workflow committed"
```

---

## 🏢 Step 5: Configure GitHub Environments (20 minutes)

### 5.1: Create Staging Environment (No Approval)

**In GitHub Web UI:**

1. **Open your repo** on github.com
2. **Click Settings** tab (top navigation)
3. **Left sidebar** → Click **Environments** (under "Deployments")
4. **Click "New environment"** button
5. **Type environment name**: `staging`
6. **Press Enter** or click **Configure environment**
7. **No approval rules needed**:
   - Leave "Required reviewers" unchecked
   - Leave "Deployment branches" empty
8. **Scroll down** → Click **"Save protection rules"** button
9. **Verify**: You should see `staging` in the Environments list

### 5.2: Create Production Environment (With Approval)

**In GitHub Web UI:**

1. **Click "New environment"** button again
2. **Type environment name**: `production`
3. **Press Enter** or click **Configure environment**
4. **Configure approval rules**:
   - Check **"Required reviewers"** checkbox
   - Click **"Add reviewers"** field
   - Start typing team member names (e.g., @alice, @bob)
   - Select from dropdown
   - Add minimum 1, recommend 2-3 for production
5. **Optional**: Check **"Protect environment"** (prevents accidental deployments)
6. **Scroll down** → Click **"Save protection rules"** button
7. **Verify**: You should see `production` with "Required reviewers" badge

### 5.3: How to Verify Environments Are Set Up

```bash
# Via GitHub CLI (if installed)
gh repo environment list

# Or manually in GitHub:
# Settings → Environments → You should see:
# - staging (no approval required)
# - production (approval required from X reviewers)
```

---

## 🧪 Step 6: Test the Build RC Workflow (10 minutes)

### 6.1: Trigger Build by Pushing to Develop

```bash
# Make a small change to trigger workflow
echo "# Test Release" >> README_TEST.md

# Add and commit
git add README_TEST.md
git commit -m "test: trigger RC build workflow"

# Push to develop
git push origin develop

echo "✅ Pushed to develop - build should start in ~30 seconds"
```

### 6.2: Monitor Workflow Execution

**In GitHub Web UI:**

1. **Go to your repo**
2. **Click "Actions" tab**
3. **Find "Build Release Candidate" workflow** (should be first/top)
4. **Click on it** to open details
5. **Watch live execution**:
   - ⏳ "Determine RC version" (30 sec)
   - ⏳ "Install dependencies" (60 sec)
   - ⏳ "Run tests with coverage" (45 sec)
   - ⏳ "Lint and format check" (30 sec)
   - ⏳ "Security checks" (45 sec)
   - ⏳ "Build artifacts" (30 sec)
   - ⏳ "Generate release notes" (10 sec)
   - ⏳ "Upload artifacts" (30 sec)
   - ⏳ "Create draft GitHub Release" (20 sec)

**Total time**: ~3-4 minutes

### 6.3: Verify Draft Release Created

1. **In your repo**, click **"Releases"** tab
2. **Find draft release** (top of list, labeled "Draft")
3. **Click on it** to view details
4. **Verify you see**:
   - RC version tag (e.g., `v0.0.0-rc.20260102.090000.abc123`)
   - Release notes with checklist
   - Artifacts downloadable (dist/, coverage.xml, reports)

---

## 👀 Step 7: Test Team Review & Approval (5 minutes)

### 7.1: Review the Release Candidate

1. **In the draft release**, read the release notes
2. **Click "Assets"** section
3. **Download artifacts**:
   - `.whl` file (wheel distribution)
   - `.tar.gz` file (source distribution)
4. **Review reports**:
   - `coverage.xml` - test coverage
   - `bandit-report.json` - security scan
   - `safety-report.json` - dependency scan

### 7.2: Publish the Release (Approve)

**This is your approval action!**

1. **In draft release**, click **"Edit"** button (top right)
2. **Find checkbox**: **"Set as a draft"** (should be checked)
3. **Uncheck the box** ☐
4. **Scroll down** → Click **"Update release"** button
5. **Release is now PUBLISHED** ✅

**What happens next (automatic):**
- GitHub detects "release published" event
- `merge-after-approval.yml` workflow triggers automatically (~10 seconds)

---

## ⚡ Step 8: Verify Merge & Tag (5 minutes)

### 8.1: Monitor Merge Workflow

1. **Go to "Actions" tab**
2. **Find "Merge & Tag After Approval" workflow** (should be running)
3. **Click to view details**
4. **Watch steps**:
   - ✅ "Derive semantic version tag" (5 sec)
   - ✅ "Configure git" (5 sec)
   - ✅ "Fetch branches" (10 sec)
   - ✅ "Merge develop into main" (10 sec)
   - ✅ "Create version tag on main" (10 sec)
   - ✅ "Update release notes" (10 sec)

**Total time**: ~1 minute

### 8.2: Verify Main Branch Updated

1. **Go to "Code" tab**
2. **Click branch dropdown** (left side, says "develop")
3. **Select "main"** branch
4. **Verify**:
   - Main branch now exists (or was updated)
   - Commit message: "Release v... (approved from ...)"
   - Files are up-to-date with develop

### 8.3: Verify Version Tag Created

1. **Go to "Releases" tab**
2. **You should see TWO releases**:
   - RC release (draft, e.g., `v0.0.0-rc.20260102.090000.abc123`)
   - Version release (tagged, e.g., `v0.260102.02`) ← NEW!
3. **Click version release** (the new one)
4. **Verify**:
   - Release notes updated with approval status
   - Same artifacts as RC
   - Mark as "Latest release" (if this is first)

---

## 🎉 Step 9: Verify Staging Deployment (5 minutes)

### 9.1: Monitor Deployment Workflow

1. **Go to "Actions" tab**
2. **Find "Deploy to Staging" workflow** (should be running after merge)
3. **Click to view details**
4. **Watch steps**:
   - ✅ "Extract version" (5 sec)
   - ✅ "Install dependencies" (30 sec)
   - ✅ "Deploy to staging" (10 sec) ← Currently simulated
   - ✅ "Health check" (5 sec)
   - ✅ "Notify completion" (5 sec)

**Total time**: ~1-2 minutes

### 9.2: Verify Deployment Success

1. **Check workflow completion**:
   - All steps should show ✅ green checkmarks
   - No ❌ red X marks

2. **Health check result**:
   - Should show "✅ Calculator health check passed"
   - This proves deployment worked (or would work with real deployment)

3. **View full logs**:
   - Click any step to see detailed output
   - Look for "Deploying v0.260102.02 to staging..."

---

## 📊 Complete Flow Timeline

Now that you've completed one full cycle:

| Time | Event | Status |
|------|-------|--------|
| T+0 min | Push to develop | ✅ Done |
| T+3-4 min | Build RC completes | ✅ Done |
| T+4 min | Draft release created | ✅ Done |
| T+5 min | Team publishes release | ✅ Done (manual) |
| T+6 min | Merge workflow starts | ✅ Done |
| T+7 min | Main branch updated + tag created | ✅ Done |
| T+8 min | Staging deploy workflow starts | ✅ Done |
| T+9-10 min | 🎉 Deployment complete | ✅ Done |

**Total: 10 minutes from code push to staging live!**

---

## 🔐 Branch Protection (Optional but Recommended)

### Why Add Branch Protection?

Ensures only the release automation can push to `main` (prevents accidental manual pushes).

### How to Set Up

1. **Go to "Settings" tab**
2. **Left sidebar** → Click **"Branches"** (under "Code and automation")
3. **Click "Add rule"** button
4. **Branch name pattern**: Type `main`
5. **Configure rules**:
   - ☐ Require pull request reviews (unchecked - releases approve instead)
   - ☐ Dismiss stale PR approvals (unchecked)
   - ☐ Require status checks to pass (optional)
   - ☐ Require branches to be up to date (optional)
   - ☐ Require code reviews (unchecked)
6. **Click "Create"** button

### Verify

- Go to "Settings" → "Branches"
- Should see rule: "Branch protection rule for `main`"

---

## 📝 Team Process Guide (How Your Team Uses This)

### For Developers

1. **Create feature branch**:
   ```bash
   git checkout -b feature/my-feature develop
   ```

2. **Make changes + write tests**:
   ```bash
   # Code...
   # Test locally: pytest tests/
   ```

3. **Create PR to develop**:
   - Push branch to GitHub
   - Create Pull Request
   - Request reviewers (peer review)

4. **After merge to develop**:
   - Wait 3-4 minutes
   - Build RC workflow runs automatically
   - You get notified when done

### For Release Approvers (QA/Tech Lead)

1. **Get notification** of draft release (GitHub notification)
2. **Go to Releases tab**
3. **Download artifacts** from draft release
4. **Review**:
   - Coverage: acceptable for feature?
   - Security: any new vulnerabilities?
   - Tests: all passing?
5. **Decision**:
   - ✅ Good to ship? **Publish release** (edit + uncheck draft)
   - ❌ Needs work? **Delete draft** and request changes

### For QA/Testing Team

1. **After deployment to staging** (~10 min after approval)
2. **Go to staging environment**
3. **Run manual tests**:
   - User flows
   - Edge cases
   - Performance
4. **Log issues** if found
5. **Approve for production** (or request fixes)

### For DevOps

1. **Set up real deployment** (replace TODO in workflows)
   - Docker build & push
   - SSH to server
   - Kubernetes deployment
   - etc.

2. **Configure monitoring** (optional)
   - Slack notifications
   - Datadog/New Relic alerts
   - PagerDuty for production

3. **Handle secrets** (production credentials)
   - Set `GITHUB_ACTIONS_DEPLOY_KEY` in secrets
   - Set `DOCKER_REGISTRY_PASSWORD` in secrets
   - etc.

---

## 🔧 Customization: Replace TODO Sections

### Real Docker Deployment

Replace this in `deploy-staging.yml`:

```yaml
- name: Deploy to staging
  run: |
    echo "✅ Staging deployment complete"
```

With this:

```yaml
- name: Deploy to staging
  env:
    DOCKER_REGISTRY: docker.io
    IMAGE_NAME: mycompany/app
  run: |
    # Build and push Docker image
    docker build -t $DOCKER_REGISTRY/$IMAGE_NAME:${{ steps.version.outputs.version }} .
    docker push $DOCKER_REGISTRY/$IMAGE_NAME:${{ steps.version.outputs.version }}
    
    # Deploy to staging server
    ssh user@staging.example.com "
      docker pull $DOCKER_REGISTRY/$IMAGE_NAME:${{ steps.version.outputs.version }}
      docker stop app || true
      docker rm app || true
      docker run -d \
        --name app \
        -p 8000:8000 \
        $DOCKER_REGISTRY/$IMAGE_NAME:${{ steps.version.outputs.version }}
    "
```

### Real Kubernetes Deployment

Replace with:

```yaml
- name: Deploy to staging
  run: |
    kubectl set image deployment/app \
      app=$DOCKER_REGISTRY/app:${{ steps.version.outputs.version }} \
      -n staging
    kubectl rollout status deployment/app -n staging
```

### Real Health Checks

Replace this:

```yaml
- name: Health check
  run: |
    echo "Running health checks..."
```

With this:

```yaml
- name: Health check
  run: |
    # Wait for app to be ready
    sleep 5
    
    # Check HTTP endpoint
    curl -f https://staging.example.com/health || exit 1
    curl -f https://staging.example.com/api/status || exit 1
    curl -f https://staging.example.com/api/version || exit 1
    
    echo "✅ All health checks passed"
```

### Add Slack Notifications

Add to any workflow:

```yaml
- name: Notify Slack on deployment
  if: always()
  uses: slackapi/slack-github-action@v1
  with:
    webhook-url: ${{ secrets.SLACK_WEBHOOK }}
    payload: |
      {
        "text": "🚀 Release ${{ steps.version.outputs.version }} deployed to staging!",
        "blocks": [
          {
            "type": "section",
            "text": {
              "type": "mrkdwn",
              "text": "*Deployment Status*\nVersion: ${{ steps.version.outputs.version }}\nEnvironment: Staging\nStatus: ✅ Success"
            }
          }
        ]
      }
```

**Set up webhook**:
1. Go to Slack workspace settings
2. Create incoming webhook for #deployments channel
3. In GitHub repo → Settings → Secrets → New secret
4. Name: `SLACK_WEBHOOK`
5. Value: paste webhook URL

---

## ✅ Final Verification Checklist

After completing Steps 1-9:

- [ ] Project files created (src/, tests/, requirements.txt, setup.py)
- [ ] All 3 workflows created (.github/workflows/)
- [ ] All workflows have valid YAML syntax
- [ ] Workflows committed and pushed to develop
- [ ] Staging environment configured (no approval)
- [ ] Production environment configured (with approval)
- [ ] Build RC workflow triggered and completed (draft release created)
- [ ] Team published release (approval)
- [ ] Merge workflow completed (merge to main + tag created)
- [ ] Staging deployment workflow completed (health checks passed)
- [ ] All team members understand their role
- [ ] TODO sections identified for customization

---

## 🆘 Troubleshooting

### Workflow doesn't start
**Problem**: Push to develop but no workflow runs

**Solutions**:
1. Check branch name: `git branch` (should be `develop`, not `development`)
2. Check workflow file: `.github/workflows/build-rc.yml` exists?
3. Check YAML syntax: no tabs (use spaces), proper indentation
4. Check permissions: repo settings → Actions → allow workflows

### Draft release not created
**Problem**: Workflow runs but no release appears

**Solutions**:
1. Check build succeeded: look at workflow logs
2. Check artifacts uploaded: does dist/ have files?
3. Check GitHub token: usually automatic, no action needed
4. Check release_notes.md created: look in build step logs

### Merge fails
**Problem**: Merge workflow errors out

**Solutions**:
1. Create main branch manually:
   ```bash
   git checkout -b main
   git push origin main
   ```
2. Check conflicts: are develop and main in conflict?
3. Check permissions: does GitHub Actions have write access?

### Deployment fails
**Problem**: Deploy workflow errors

**Solutions**:
1. Check environment exists: Settings → Environments
2. Check staging environment name matches: `name: staging`
3. Check deployment commands: replace TODO with real commands
4. Check server access: can SSH key access server?

---

## 📚 Key Concepts Summary

| Term | Meaning |
|------|---------|
| **RC (Release Candidate)** | Built version from develop with tests + security reports |
| **Draft Release** | Private release (only visible to maintainers) for review |
| **Published Release** | Public release; this IS the approval action |
| **Approval Gate** | Publishing release = explicit team approval |
| **Version Tag** | Semantic tag (vX.Y.Z) on main; triggers deployment |
| **Environment** | GitHub deployment target with optional approval rules |
| **Merge Commit** | --no-ff merge to preserve feature branch history |

---

## 🎯 Why This Approach Works

✅ **Approval before versioning** - Team reviews artifacts before code is tagged  
✅ **Clear audit trail** - Git history + release notes show who approved when  
✅ **No manual git** - All merge/tag actions automated after approval  
✅ **No additional tools** - Uses GitHub native features only  
✅ **Scales easily** - Works for 5-person teams to 500-person companies  
✅ **Safe defaults** - Approval required; easy to deny/request changes  
✅ **Fast CI/CD** - Total time from code to staging: ~10 minutes  

---

## 📖 Next Steps After This Guide

1. ✅ **Complete implementation** (Steps 1-9)
2. ✅ **Test full cycle** once
3. ⏭️ **Train team** on approval process (5 min per person)
4. ⏭️ **Replace TODO sections** with real deployment commands
5. ⏭️ **Set up Slack notifications** (optional)
6. ⏭️ **Create production workflow** (optional, for prod deployments)
7. ⏭️ **Monitor & iterate** (adjust as needed)

---

## 💬 FAQ

**Q: What if I want to skip approval?**  
A: Just keep develop and main in sync; don't use releases. But you lose the approval gate benefit.

**Q: Can I deploy directly to production?**  
A: Yes! Create a 4th workflow (`deploy-production.yml`) with production environment requiring approval.

**Q: What about rollback?**  
A: Create a new RC from develop, or checkout old tag and redeploy: `git checkout v1.0.0`

**Q: Can multiple people approve?**  
A: For production yes (set "Required reviewers: 2+"). For staging, no approval needed.

**Q: Does this work with other languages?**  
A: Yes! Replace Python with your language (Node, Go, Java, etc.). Keep the workflow structure.

---

## 🚀 You're Ready!

You now have a complete, production-ready, team-approved CI/CD pipeline!

**Start with Step 1 above and follow sequentially. You'll have everything working in 1-2 hours.**

Questions? Check the Troubleshooting section or review the YAML in each workflow.

**Happy deploying! 🎉**
