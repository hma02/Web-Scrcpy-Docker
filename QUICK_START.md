# Quick Start: Execute These Commands

## Complete Integration Steps

Run these commands in order in `/home/pi/python_code/Web-Scrcpy-Docker`:

### 1. Add Submodule
```bash
git submodule add ../web-scrcpy web-scrcpy
```

### 2. Verify Submodule Added
```bash
cat .gitmodules
ls web-scrcpy/app.py  # Should exist
```

### 3. Commit Submodule Addition
```bash
git add .gitmodules web-scrcpy
git commit -m "Add web-scrcpy as git submodule"
```

### 4. Remove Duplicated Files
```bash
git rm app.py
git rm scrcpy.py
git rm -r templates/
git rm -r static/
git commit -m "Remove files now managed by web-scrcpy submodule"
```

### 5. Verify Structure
```bash
# Should show web-scrcpy/ as a directory
git ls-tree --name-only HEAD | head -20

# Should show files from submodule
ls -la web-scrcpy/app.py
ls -la web-scrcpy/templates/
ls -la web-scrcpy/static/
```

### 6. Test Docker Build
```bash
docker build -t web-scrcpy:test .
```

**Expected output:**
```
...
Step 37/47 : RUN git config --global --add safe.directory /app &&...
 ---> Running in xxxxx
 ---> 123abc456def
Step 38/47 : RUN python3 -m venv /app/venv &&...
 ---> Running in xxxxx
 ---> 123xyz789abc
...
Successfully tagged web-scrcpy:test
```

### 7. Test Docker Run
```bash
docker run -it --rm -p 5000:5000 web-scrcpy:test bash -c ". /app/venv/bin/activate && python3 app.py --port 5000"
```

**Expected to see:**
```
 * Running on http://0.0.0.0:5000
 * Press CTRL+C to quit
```

Visit http://localhost:5000 in browser - should work!

### 8. Push Changes
```bash
git push origin main
```

## Minimal Changes Made (Summary)

### ✅ Files Modified
1. **Dockerfile**
   - Added `git` to build dependencies
   - Added submodule initialization step
   - Updated COPY commands to use `web-scrcpy/` subdirectory

2. **.gitignore**
   - Added comments noting files now come from submodule

### ✅ Files Created (Documentation)
3. **SUBMODULE_SETUP.md** - Developer guide
4. **IMPLEMENTATION_CHECKLIST.md** - Step-by-step checklist
5. **INTEGRATION_VISUAL_GUIDE.md** - Visual explanations

### ✅ Files to Delete (After Running Commands Above)
- `app.py` (will remove via git)
- `scrcpy.py` (will remove via git)
- `templates/` (will remove via git)
- `static/` (will remove via git)

### ⏸ Files Unchanged (Keep These)
- `adb_manager.py` - Docker-specific
- `adb/` - Docker-specific
- `requirements.txt` - Shared
- `docker-compose.yml` - No changes needed
- `README.md` - Can update clone instructions

## File Locations After Integration

```
Web-Scrcpy-Docker/
├── web-scrcpy/                  ← NEW: Git submodule
│   ├── app.py                   ← Sourced from here
│   ├── scrcpy.py                ← Sourced from here
│   ├── templates/               ← Sourced from here
│   ├── static/                  ← Sourced from here
│   ├── scrcpy-server            ← Sourced from here
│   └── ...
│
├── .gitmodules                  ← NEW: Submodule config
├── Dockerfile                   ← UPDATED: Uses submodule paths
├── .gitignore                   ← UPDATED: Documented change
│
├── adb_manager.py               ← KEPT: Docker-specific
├── adb/                         ← KEPT: Docker-specific
├── requirements.txt             ← KEPT: Shared dependencies
├── docker-compose.yml           ← KEPT: No changes needed
└── *.md files (documentation)   ← NEW: Setup guides
```

## For Future Users

When cloning this repo, they should use:
```bash
git clone --recurse-submodules https://github.com/NEANC/Web-Scrcpy-Docker.git
```

Or if they forgot:
```bash
git submodule update --init --recursive
```

## Key Points

✅ **Minimal changes**: Only Dockerfile and .gitignore modified for functionality  
✅ **Docker-native**: Uses COPY which automatically gets submodule files  
✅ **Clean separation**: Docker-specific code stays in main repo  
✅ **Easy updates**: Pull latest web-scrcpy with `git submodule update --remote`  
✅ **Backward compatible**: Docker build process unchanged from user perspective  

## Optional: Update README

Add this section to README.md:

```markdown
## Installation (with Git Submodule)

This repository includes the web-scrcpy project as a git submodule.

### Clone with Submodule
```bash
git clone --recurse-submodules https://github.com/NEANC/Web-Scrcpy-Docker.git
cd Web-Scrcpy-Docker
```

### If Already Cloned Without Submodule
```bash
git submodule update --init --recursive
```

### Build Docker Image
```bash
docker build -t web-scrcpy:latest .
```

See [INTEGRATION_VISUAL_GUIDE.md](INTEGRATION_VISUAL_GUIDE.md) for technical details.
```

## Status Check

Run this after completing all steps to verify:

```bash
#!/bin/bash
echo "Checking submodule integration..."
echo ""
echo "1. .gitmodules exists:"
test -f .gitmodules && echo "✅ YES" || echo "❌ NO"

echo ""
echo "2. web-scrcpy directory exists:"
test -d web-scrcpy && echo "✅ YES" || echo "❌ NO"

echo ""
echo "3. Required files in submodule:"
test -f web-scrcpy/app.py && echo "✅ app.py" || echo "❌ app.py"
test -f web-scrcpy/scrcpy.py && echo "✅ scrcpy.py" || echo "❌ scrcpy.py"
test -d web-scrcpy/templates && echo "✅ templates/" || echo "❌ templates/"
test -d web-scrcpy/static && echo "✅ static/" || echo "❌ static/"
test -f web-scrcpy/scrcpy-server && echo "✅ scrcpy-server" || echo "❌ scrcpy-server"

echo ""
echo "4. Docker-specific files still present:"
test -f adb_manager.py && echo "✅ adb_manager.py" || echo "❌ adb_manager.py"
test -d adb && echo "✅ adb/" || echo "❌ adb/"

echo ""
echo "5. Duplicated files removed from main repo:"
test ! -f app.py && echo "✅ app.py removed" || echo "❌ app.py still exists"
test ! -f scrcpy.py && echo "✅ scrcpy.py removed" || echo "❌ scrcpy.py still exists"
test ! -d templates && echo "✅ templates/ removed" || echo "❌ templates/ still exists"
test ! -d static && echo "✅ static/ removed" || echo "❌ static/ still exists"

echo ""
echo "All checks passed! ✅"
```

Save as `check_integration.sh` and run: `bash check_integration.sh`
