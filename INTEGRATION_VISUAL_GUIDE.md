# Git Submodule Integration - Visual Guide

## Before Integration
```
Web-Scrcpy-Docker/                  web-scrcpy/
├── app.py (copy)                   ├── app.py (original)
├── scrcpy.py (copy)       ←→       ├── scrcpy.py (original)
├── templates/ (copy)               ├── templates/ (original)
├── static/ (copy)                  ├── static/ (original)
├── scrcpy-server (copy)            ├── scrcpy-server (original)
├── adb_manager.py (unique)         └── ...
├── adb/ (unique)
└── Dockerfile
```
**Problem:** Files duplicated, hard to keep in sync, scattered development

## After Integration
```
Web-Scrcpy-Docker/
├── web-scrcpy/ (git submodule → points to web-scrcpy repo)
│   ├── app.py
│   ├── scrcpy.py
│   ├── templates/
│   ├── static/
│   ├── scrcpy-server
│   └── ...
├── adb_manager.py (unique to Docker)
├── adb/ (unique to Docker)
├── Dockerfile (updated to copy from web-scrcpy/)
├── .gitmodules (new - tracks submodule)
└── requirements.txt
```
**Benefit:** Single source of truth, clean separation, easy sync

## Docker Build Process

```
┌─────────────────────────────────────────┐
│ Step 1: COPY . /app                     │
│ Copies Web-Scrcpy-Docker repo files      │
│ Including .git and .gitmodules          │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│ Step 2: git submodule update --init     │
│ Initializes web-scrcpy submodule        │
│ Downloads all submodule files            │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│ Step 3: COPY --from=builder             │
│ /app/web-scrcpy/app.py → /app/app.py   │
│ /app/web-scrcpy/scrcpy.py → /app/...   │
│ /app/web-scrcpy/templates → /app/...   │
│ /app/web-scrcpy/static → /app/...      │
│ /app/web-scrcpy/scrcpy-server → /app/..│
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│ Step 4: Final Container                 │
│ /app/app.py (from submodule)            │
│ /app/scrcpy.py (from submodule)         │
│ /app/adb_manager.py (from main repo)    │
│ /app/adb/ (from main repo)              │
│ /app/templates/ (from submodule)        │
│ /app/static/ (from submodule)           │
└─────────────────────────────────────────┘
```

## Git Workflow

### Initialize Submodule
```bash
git submodule add ../web-scrcpy web-scrcpy
```

Creates two things:
1. **web-scrcpy/** directory - contains submodule working tree
2. **.gitmodules** file - tracks submodule config

Content of .gitmodules:
```ini
[submodule "web-scrcpy"]
	path = web-scrcpy
	url = ../web-scrcpy
```

### Clone with Submodules
```bash
git clone --recurse-submodules <repo-url>
# Automatically initializes all submodules
```

OR:
```bash
git clone <repo-url>
git submodule update --init --recursive
# Manually initialize submodules
```

### Update Submodule
```bash
cd web-scrcpy
git pull origin main
cd ..
git add web-scrcpy
git commit -m "Update web-scrcpy submodule"
```

## Directory Structure in CI/CD

### GitHub Actions Example
```yaml
- name: Checkout with submodules
  uses: actions/checkout@v3
  with:
    submodules: 'recursive'

- name: Build Docker image
  run: docker build -t web-scrcpy:latest .
```

### Docker Build Command
```bash
docker build --build-arg BUILDKIT_CONTEXT_KEEP_GIT_DIR=1 -t web-scrcpy:latest .
```

Note: BUILDKIT_CONTEXT_KEEP_GIT_DIR needed to preserve .git in build context

## File Ownership

| File | Owner | Location | Notes |
|------|-------|----------|-------|
| app.py | web-scrcpy repo | `web-scrcpy/app.py` | Edit in submodule |
| scrcpy.py | web-scrcpy repo | `web-scrcpy/scrcpy.py` | Edit in submodule |
| templates/ | web-scrcpy repo | `web-scrcpy/templates/` | Edit in submodule |
| static/ | web-scrcpy repo | `web-scrcpy/static/` | Edit in submodule |
| adb_manager.py | Docker repo | `./adb_manager.py` | Docker-specific code |
| adb/ | Docker repo | `./adb/` | Docker-specific binaries |
| Dockerfile | Docker repo | `./Dockerfile` | Docker-specific config |
| requirements.txt | Docker repo | `./requirements.txt` | Shared dependencies |

## Comparison: Three Approaches

### Approach 1: Copy Files (Current - Before)
```
❌ Files duplicated
❌ Hard to keep in sync
❌ Can't easily contribute back to upstream
✅ Simpler to clone
```

### Approach 2: Git Submodule (Recommended Solution)
```
✅ Single source of truth
✅ Easy to update
✅ Easy to contribute back
✅ Works well with Docker
❌ Users must use --recurse-submodules flag
```

### Approach 3: Git Subtree
```
✅ Can merge changes back
✅ Simpler clone (no special flags needed)
❌ More complex history
❌ Harder to track upstream updates
```

**Why Submodule is best:** Docker uses COPY, which automatically gets submodule contents. Users cloning will naturally work with --recurse-submodules.

## Integration Checklist at a Glance

- [x] Dockerfile updated to use submodule
- [ ] `git submodule add ../web-scrcpy web-scrcpy`
- [ ] `git add .gitmodules web-scrcpy`
- [ ] `git commit -m "Add web-scrcpy submodule"`
- [ ] `git rm app.py scrcpy.py templates/ static/`
- [ ] `git commit -m "Remove duplicated files"`
- [ ] Test Docker build
- [ ] Test Docker run
- [ ] Update README with clone instructions
- [ ] Push to remote

## Troubleshooting

### Q: Why add `.git/` to safe.directory in Dockerfile?
A: Alpine Linux has strict git security. Without this, `git submodule update` fails with "fatal: not a git repository".

### Q: Do I need to clone web-scrcpy separately?
A: No! With `git clone --recurse-submodules`, everything is fetched automatically.

### Q: Can I edit files in web-scrcpy submodule and commit?
A: Yes! The submodule is a full git repo. Changes can be committed and pushed normally.

### Q: How do I update to latest web-scrcpy?
A: Run `git submodule update --remote` to fetch latest from web-scrcpy main branch.

### Q: What if web-scrcpy repo URL changes?
A: Update `.gitmodules` and run `git submodule sync`, or re-add the submodule.
