# Web-Scrcpy-Docker Submodule Integration Checklist

## ✅ Changes Already Made

### 1. Dockerfile Updates
- [x] Added `git` to build dependencies (needed for submodule initialization)
- [x] Added `git config --global --add safe.directory /app` (security setting for Docker)
- [x] Added `git submodule update --init --recursive` to initialize submodule during build
- [x] Updated COPY commands to reference `web-scrcpy/` subdirectory:
  - `app.py` ← `/app/web-scrcpy/app.py`
  - `scrcpy.py` ← `/app/web-scrcpy/scrcpy.py`
  - `scrcpy-server` ← `/app/web-scrcpy/scrcpy-server`
  - `templates/` ← `/app/web-scrcpy/templates/`
  - `static/` ← `/app/web-scrcpy/static/`

### 2. .gitignore Updates
- [x] Added comments noting these files now come from submodule

### 3. Documentation
- [x] Created `SUBMODULE_SETUP.md` with complete developer guide

## ⏳ Next Steps (Execute In Order)

### Step 1: Add Submodule to Git
```bash
cd /home/pi/python_code/Web-Scrcpy-Docker
git submodule add ../web-scrcpy web-scrcpy
```

**Expected output:**
- New `web-scrcpy/` directory created
- New `.gitmodules` file created

**Verify:**
```bash
ls -la | grep .gitmodules  # Should exist
cat .gitmodules            # Should show submodule config
ls web-scrcpy/app.py       # Should exist
```

### Step 2: Initialize Submodule
```bash
git submodule update --init --recursive
```

**Expected output:**
- Submodule initialized
- Files downloaded from web-scrcpy repo

### Step 3: Commit Submodule Addition
```bash
git add .gitmodules web-scrcpy
git commit -m "Add web-scrcpy as git submodule"
```

### Step 4: Remove Duplicated Files from Docker Repo
```bash
git rm app.py
git rm scrcpy.py
git rm -r templates/
git rm -r static/
git commit -m "Remove files now managed by web-scrcpy submodule"
```

**Keep these files** (they are Docker-specific):
- `adb_manager.py` ← Docker-specific ADB manager
- `adb/` ← Contains platform-specific ADB binaries
- `requirements.txt` ← Shared dependencies

### Step 5: Test Docker Build
```bash
docker build -t web-scrcpy:test .
```

**Should succeed with:**
- Git submodule initialization completing
- All files copied from submodule correctly
- Virtual environment created
- Dependencies installed

### Step 6: Test Docker Run
```bash
docker run -it --rm -p 5000:5000 web-scrcpy:test python3 app.py --port 5000
```

**Expected:**
- Flask app starts
- No ImportError for scrcpy, templates, or static files
- Accessible at http://localhost:5000

### Step 7: Push to Remote
```bash
git push origin main  # or your default branch
```

## 📋 Files to Verify After Implementation

| File/Folder | Location | Status | Notes |
|------------|----------|--------|-------|
| app.py | web-scrcpy/app.py | Submodule | Imported by app |
| scrcpy.py | web-scrcpy/scrcpy.py | Submodule | Core streaming |
| templates/ | web-scrcpy/templates/ | Submodule | Flask templates |
| static/ | web-scrcpy/static/ | Submodule | JS/CSS assets |
| scrcpy-server | web-scrcpy/scrcpy-server | Submodule | Binary |
| adb_manager.py | Web-Scrcpy-Docker/adb_manager.py | Local | Docker-specific |
| adb/ | Web-Scrcpy-Docker/adb/ | Local | Docker-specific |
| requirements.txt | Web-Scrcpy-Docker/requirements.txt | Local | Should work with both repos |
| Dockerfile | Web-Scrcpy-Docker/Dockerfile | Updated ✅ | References submodule |
| .gitmodules | Web-Scrcpy-Docker/.gitmodules | New | Auto-created |
| .gitignore | Web-Scrcpy-Docker/.gitignore | Updated ✅ | Documented change |

## 🔄 Ongoing Maintenance

### To Update web-scrcpy to Latest
```bash
cd web-scrcpy
git pull origin main  # Get latest changes
cd ..
git add web-scrcpy
git commit -m "Update web-scrcpy submodule to latest"
git push
```

### To Make Changes to web-scrcpy
```bash
cd web-scrcpy
# Edit files here (e.g., app.py, templates/, etc.)
git add .
git commit -m "Your change description"
git push origin main
cd ..
git add web-scrcpy
git commit -m "Update web-scrcpy submodule"
git push
```

### To Clone This Repo Later
```bash
git clone --recurse-submodules <repo-url>
```

## ⚠️ Potential Issues & Solutions

### Issue: Docker build fails with "fatal: not a git repository"
**Solution:** Ensure `git config --global --add safe.directory /app` is in Dockerfile ✅ (already added)

### Issue: Submodule files not found after build
**Solution:** Verify `.gitmodules` exists and points to correct path: `web-scrcpy`

### Issue: "No module named 'app'" after Docker starts
**Solution:** Check that COPY commands in Dockerfile reference the submodule path correctly ✅ (already updated)

### Issue: Can't clone the Docker repo without submodules
**Solution:** Document that users should use `git clone --recurse-submodules` - update README

## 📚 Documentation Updates Needed

Update `README.md` in Web-Scrcpy-Docker to add:

```markdown
## Installation with Submodule

This repository uses git submodules. When cloning, be sure to include submodules:

### Clone with Submodules
\`\`\`bash
git clone --recurse-submodules https://github.com/NEANC/Web-Scrcpy-Docker.git
cd Web-Scrcpy-Docker
\`\`\`

### Update Submodules
To get the latest version of web-scrcpy:
\`\`\`bash
git submodule update --remote
\`\`\`

See [SUBMODULE_SETUP.md](SUBMODULE_SETUP.md) for more details.
```

## ✨ Benefits of This Setup

- ✅ Single source of truth for web-scrcpy code
- ✅ Easy to keep Docker fork synchronized with upstream
- ✅ Clean separation between Docker-specific and shared code
- ✅ Reduces file duplication
- ✅ Easier to track and update versions
- ✅ Other users can benefit from Docker improvements without forking
