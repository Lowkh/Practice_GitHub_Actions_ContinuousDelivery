[README.md](https://github.com/user-attachments/files/24403810/README.md)
# Delivery‑First GitHub Actions Pipeline

A complete CI/CD workflow where **team members review and approve releases before code is merged to main and versioned**, then **all deployments run automatically**.

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

- GitHub account with admin access to repo
- Git installed locally
- Python 3.9+ (adapt for your language)
- Text editor (VS Code recommended)
- 1-2 hours to implement

---

## 🏗️ Step 1: Set Up Project Files

Create the following files in your repository root:

### `src/app.py`
```python
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
```

### `tests/test_app.py`
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
    
    def test_subtract(self, calc):
        assert calc.subtract(5, 2) == 3
    
    def test_multiply(self, calc):
        assert calc.multiply(4, 3) == 12
    
    def test_divide(self, calc):
        assert calc.divide(10, 2) == 5
    
    def test_divide_by_zero(self, calc):
        with pytest.raises(ValueError):
            calc.divide(10, 0)
```

### `requirements.txt`
```
pytest==7.4.3
pytest-cov==4.1.0
black==23.12.0
flake8==6.1.0
bandit==1.7.5
safety==2.3.5
build==1.0.3
wheel==0.42.0
twine==4.0.2
```

### `setup.py`
```python
from setuptools import setup, find_packages

setup(
    name="calculator-app",
    version="0.1.0",
    description="Calculator app with delivery-first CI/CD",
    author="Your Team",
    packages=find_packages(),
    python_requires=">=3.9",
)
```

### `src/__init__.py`
Empty file (just create it).

### `tests/__init__.py`
Empty file (just create it).

---

**Commit these files:**

```bash
git add src/ tests/ requirements.txt setup.py
git commit -m "chore: initial project structure"
git push origin develop
```

---

## 🔄 Step 2: Create Build Release Candidate Workflow

Create `.github/workflows/build-rc.yml`:

```yaml
name: Build Release Candidate

on:
  push:
    branches: [develop]
  workflow_dispatch:
    inputs:
      version:
        description: "Release version (e.g. v1.2.0)"
        required: true
      release_type:
        description: "Release type"
        type: choice
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

      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
          cache: pip

      - name: Determine RC version
        id: version
        run: |
          if [ "${{ github.event_name }}" = "workflow_dispatch" ]; then
            RC_VERSION="${{ github.event.inputs.version }}-${{ github.event.inputs.release_type }}"
          else
            RC_VERSION="v0.$(date +%Y%m%d)-rc.$(git rev-parse --short HEAD)"
          fi
          echo "rc_version=${RC_VERSION}" >> "$GITHUB_OUTPUT"
          echo "Building RC: ${RC_VERSION}"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install build twine bandit safety

      - name: Run tests with coverage
        run: |
          pytest tests/ -v --cov=src --cov-report=xml
          echo "✅ Tests passed"

      - name: Lint and format check
        run: |
          flake8 src tests --max-line-length=100 || true
          black --check src tests || true

      - name: Security checks
        run: |
          bandit -r src/ -f json -o bandit-report.json || true
          safety check --json > safety-report.json || true

      - name: Build artifacts
        run: |
          python -m build
          twine check dist/*
          echo "✅ Artifacts built and validated"

      - name: Generate release notes
        run: |
          cat > release_notes.md << 'EOF'
          # Release Candidate

          Built from `${{ github.ref_name }}` at commit `${{ github.sha }}`.

          ## What to review

          1. Download artifacts (wheel and source distributions)
          2. Review test coverage (coverage.xml)
          3. Review security reports:
             - bandit-report.json (code security)
             - safety-report.json (dependency vulnerabilities)
          4. Optionally install package locally and test

          ## How to approve

          If everything looks good, **publish this release** to approve it.
          This will automatically:
          - Merge develop → main
          - Create version tag (e.g. v1.2.0)
          - Deploy to staging
          EOF

      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: rc-${{ steps.version.outputs.rc_version }}
          path: |
            dist/
            coverage.xml
            bandit-report.json
            safety-report.json
            release_notes.md
          retention-days: 30

      - name: Create draft GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          tag_name: ${{ steps.version.outputs.rc_version }}
          draft: true
          prerelease: true
          files: dist/*
          body_path: release_notes.md
```

---

**Commit this workflow:**

```bash
git add .github/workflows/build-rc.yml
git commit -m "ci: add build release candidate workflow"
git push origin develop
```

---

## ✅ Step 3: Create Merge & Tag After Approval Workflow

Create `.github/workflows/merge-after-approval.yml`:

```yaml
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
        run: |
          TAG_NAME="${{ github.event.release.tag_name }}"
          MAIN_TAG=$(echo "$TAG_NAME" | sed 's/-\(alpha\|beta\|rc\).*//')
          echo "tag_name=$MAIN_TAG" >> "$GITHUB_OUTPUT"
          echo "RC tag: $TAG_NAME"
          echo "Release tag: $MAIN_TAG"

      - name: Configure git
        run: |
          git config user.name  "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"

      - name: Fetch branches
        run: |
          git fetch origin develop
          git fetch origin main || git branch -a

      - name: Merge develop into main
        run: |
          git checkout main 2>/dev/null || git checkout -b main
          git pull origin main 2>/dev/null || true
          git merge origin/develop --no-ff -m "Release ${{ steps.version.outputs.tag_name }} (approved)"

      - name: Create version tag on main
        run: |
          git tag -a "${{ steps.version.outputs.tag_name }}" \
            -m "Release ${{ steps.version.outputs.tag_name }}
          
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
- **Merged**: $(date -u +'%Y-%m-%d %H:%M:%S UTC')
- **Next**: Deploying to staging...`
            });
```

---

**Commit this workflow:**

```bash
git add .github/workflows/merge-after-approval.yml
git commit -m "ci: add merge and tag after approval workflow"
git push origin develop
```

---

## 🚀 Step 4: Create Staging Deployment Workflow

Create `.github/workflows/deploy-staging.yml`:

```yaml
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

      - name: Extract version
        id: version
        run: |
          VERSION="${GITHUB_REF#refs/tags/}"
          VERSION="${VERSION:-$(git describe --tags --always)}"
          echo "version=$VERSION" >> "$GITHUB_OUTPUT"
          echo "🚀 Deploying version: $VERSION"

      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          pip install -r requirements.txt

      - name: Deploy to staging
        run: |
          echo "Deploying ${{ steps.version.outputs.version }} to staging..."
          # TODO: Replace with real deployment (SSH, Docker, Kubernetes, etc.)
          # Example: docker build -t app:${{ steps.version.outputs.version }} .
          # Example: docker push yourregistry/app:${{ steps.version.outputs.version }}
          echo "✅ Staging deployment complete"

      - name: Health check
        run: |
          echo "Running health checks..."
          # TODO: Replace with real health check
          # Example: curl -f https://your-staging-url.example.com/health || exit 1
          python -c "from src.app import Calculator; c = Calculator(); assert c.add(2, 3) == 5; print('✅ Health check passed')"

      - name: Notify completion
        run: |
          echo "🎉 Staging deployment successful!"
          echo "Version: ${{ steps.version.outputs.version }}"
          echo "Time: $(date -u +'%Y-%m-%d %H:%M:%S UTC')"
          # TODO: Optionally send Slack notification here
```

---

**Commit this workflow:**

```bash
git add .github/workflows/deploy-staging.yml
git commit -m "ci: add staging deployment workflow"
git push origin develop
```

---

## 🏢 Step 5: Configure GitHub Environments

1. Go to your repo **Settings** → **Environments**.

2. Create **staging** environment (no approval needed):
   - Click **New environment**.
   - Name: `staging`
   - Click **Configure environment**.
   - No deployment branches or approval rules needed.
   - Click **Save protection rules**.

3. (Optional) Create **production** environment (with approval):
   - Click **New environment**.
   - Name: `production`
   - Click **Configure environment**.
   - Check **Require reviewers**.
   - Add team members as required reviewers.
   - Click **Save protection rules**.

---

## 🧪 Step 6: Test the Build RC Workflow

1. **Make a test change** on develop:

   ```bash
   git checkout develop
   git pull origin develop
   echo "# Calculator App" > README_TEMP.md
   git add README_TEMP.md
   git commit -m "test: trigger RC build"
   git push origin develop
   ```

2. **Watch the build run**:
   - Go to your repo → **Actions** tab.
   - Click **Build Release Candidate**.
   - Watch it:
     - Run tests ✅
     - Run linting ✅
     - Run security checks ✅
     - Build artifacts ✅
     - Create draft release ✅

   **Expected time**: 2-3 minutes.

---

## 👀 Step 7: Test Team Review & Approval

1. **Go to Releases**:
   - Click **Releases** tab.
   - Find the draft release (e.g. `v0.20260102-rc.abc123def`).

2. **Review the RC**:
   - Read the release notes.
   - Download artifacts (dist/).
   - Review coverage.xml.
   - Review security reports.

3. **Publish the release** (this is the approval):
   - Click **Edit**.
   - Click **Publish release**.
   - Confirm.

   **Result**: This triggers the **Merge & Tag** workflow.

---

## ⚡ Step 8: Verify Merge & Tag

1. **Watch the merge workflow**:
   - Go to **Actions** tab.
   - Click **Merge & Tag After Approval**.
   - Watch it merge and tag.

   **Expected time**: 1 minute.

2. **Verify on main branch**:
   - Go to **Code** tab.
   - Click **main** branch.
   - Should see new merge commit: "Release v... (approved)".
   - Should see new tag in **Releases** tab.

---

## 🎉 Step 9: Verify Staging Deployment

1. **Watch the deployment workflow**:
   - Go to **Actions** tab.
   - Click **Deploy to Staging**.
   - Watch it deploy and run health checks.

   **Expected time**: 1-2 minutes.

2. **Result**:
   - Version deployed to staging ✅
   - Health checks passed ✅
   - Ready for manual testing.

---

## 📊 Complete Flow Summary

| Step | Trigger | Action | Time | Status |
|------|---------|--------|------|--------|
| 1 | Push to develop | Build RC | 2-3 min | Automatic ✅ |
| 2 | Team reviews | Approve (publish) | Variable | Manual ⏸️ |
| 3 | Release published | Merge & tag | 1 min | Automatic ✅ |
| 4 | Tag on main | Deploy staging | 1-2 min | Automatic ✅ |
| 5 | - | Manual QA testing | Variable | Manual ⏸️ |
| 6 | (Optional) | Deploy production | Variable | Manual ⏸️ |

**Total time from code to staging**: ~20 minutes (including team review).

---

## 🔐 Branch Protection (Recommended)

1. Go to **Settings** → **Branches**.
2. Click **Add rule**.
3. Fill in:
   - **Branch name pattern**: `main`
   - **Require pull request reviews**: Unchecked (releases approve instead).
   - **Require status checks to pass**: Checked (if you have CI).
   - **Require branches to be up to date**: Checked.
4. Click **Create**.

This ensures only the release automation can push to main.

---

## 📝 Team Process Guide

### For Developers

1. Create feature branch from develop.
2. Make changes + write tests.
3. Create PR, get reviewed, merge to develop.
4. Wait for RC to build (automatic).

### For Release Approvers

1. Get notification of draft release.
2. Download artifacts from release.
3. Review:
   - Test coverage (coverage.xml).
   - Security reports (bandit, safety).
   - Any manual testing needed.
4. If good: click **Publish release**.
   If bad: delete draft and request changes.

### For QA/Testers

1. Monitor staging after deployment.
2. Run manual tests.
3. Approve for production (if needed).

### For DevOps

1. Configure environments in GitHub.
2. Set up real deployment scripts (in workflows).
3. Monitor all deployments.

---

## 🔧 Customization Tips

### Real Deployment (replace TODO sections)

**Deploy to Docker Hub + Server**:
```bash
docker build -t yourregistry/app:${{ steps.version.outputs.version }} .
docker push yourregistry/app:${{ steps.version.outputs.version }}
ssh user@staging-server "docker pull yourregistry/app:${{ steps.version.outputs.version }} && docker run ..."
```

**Deploy to Kubernetes**:
```bash
kubectl set image deployment/app app=yourregistry/app:${{ steps.version.outputs.version }} -n staging
```

**Health Check**:
```bash
curl -f https://your-staging-url.example.com/health || exit 1
```

### Add Slack Notifications

Add to any workflow:
```yaml
      - name: Notify Slack
        if: always()
        uses: slackapi/slack-github-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
          payload: |
            {
              "text": "Release ${{ github.event.release.tag_name }} deployed to staging ✅"
            }
```

Set `SLACK_WEBHOOK` in **Settings** → **Secrets and variables** → **Actions**.

### Optional: Production Deployment Workflow

Create `.github/workflows/deploy-production.yml`:

```yaml
name: Deploy to Production

on:
  workflow_dispatch:
    inputs:
      version:
        description: "Version to deploy (e.g. v1.2.0)"
        required: true

jobs:
  deploy-prod:
    name: Deploy to Production
    runs-on: ubuntu-latest
    environment:
      name: production   # requires approval from Settings

    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.inputs.version }}

      - name: Deploy to production
        run: |
          echo "Deploying ${{ github.event.inputs.version }} to production..."
          # TODO: Real production deployment here
          echo "✅ Production deployment complete"
```

---

## ✅ Success Checklist

- [ ] All 4 workflows created and committed.
- [ ] Staging environment configured.
- [ ] Test RC build triggered (push to develop).
- [ ] Draft release created with artifacts.
- [ ] Team can review and publish release.
- [ ] Merge workflow runs after publish.
- [ ] Main branch updated with merge commit.
- [ ] Version tag created.
- [ ] Staging deployment workflow runs.
- [ ] Artifacts deployed to staging.
- [ ] Health checks pass.
- [ ] Team understands approval process.

---

## 🆘 Troubleshooting

### Build RC doesn't trigger
- Check: Is `develop` branch the correct name?
- Check: Did you push to develop?
- Check: Is `.github/workflows/build-rc.yml` valid YAML?

### Draft release not created
- Check: Do you have write permissions?
- Check: GitHub token (auto-provided, should work).

### Merge fails
- Check: Do both `develop` and `main` branches exist?
- Check: Are there conflicts between develop and main?
- Solution: Merge manually once, then retry automation.

### Staging deployment fails
- Check: Is `staging` environment configured?
- Check: Are deployment commands real (not TODO)?
- Check: Does server/deployment target exist?

---

## 📚 Key Concepts

- **RC (Release Candidate)**: Built from develop, has artifacts + test results.
- **Draft Release**: Private release visible only to maintainers, for review.
- **Published Release**: Public release; triggers automation.
- **Approval Gate**: Publishing the release is the explicit approval.
- **Version Tag**: Semantic tag (vX.Y.Z) on main; triggers deployment.
- **Environment**: GitHub deployment environment; can require approvals.

---

## 🎯 Why This Approach?

✅ **Team approval required** before code becomes a version  
✅ **Reviews actual artifacts**, not just code diffs  
✅ **Audit trail**: Clear record of who approved when  
✅ **Automatic merge**: No manual git commands after approval  
✅ **Automatic deployment**: No manual deployment after merge  
✅ **Safe rollback**: Easy to revert to previous version tag  
✅ **Scales**: Works for teams of any size  

---

## 📖 Next Steps

1. **Implement** the workflows in this repo.
2. **Test** the complete flow once.
3. **Train** your team on the approval process.
4. **Customize** deployment steps for your infrastructure.
5. **(Optional) Add** Slack notifications.
6. **(Optional) Add** production deployment workflow with approval.

---

## 💬 Questions?

Refer to:
- GitHub Actions Docs: https://docs.github.com/en/actions
- Environment Protection: https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment

---

**Ready to get started? Follow Step 1-9 above! 🚀**
