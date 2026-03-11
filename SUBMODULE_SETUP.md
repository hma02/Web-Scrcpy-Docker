# Git Submodule Integration Guide

## Overview
The Web-Scrcpy-Docker repository now uses the web-scrcpy project as a git submodule. This allows for cleaner separation of concerns:
- **web-scrcpy submodule**: Contains app.py, scrcpy.py, templates/, static/, scrcpy-server
- **Web-Scrcpy-Docker repo**: Contains Docker configuration, adb_manager.py, and ADB binaries

## For Developers: Initial Setup

### Clone with Submodules
```bash
git clone --recurse-submodules https://github.com/yourusername/Web-Scrcpy-Docker.git
```

Or if you already cloned without submodules:
```bash
git submodule update --init --recursive
```

### Update Submodule
To get the latest version of web-scrcpy:
```bash
# Update submodule to latest commit on main branch
git submodule update --remote web-scrcpy
git add web-scrcpy
git commit -m "Update web-scrcpy submodule to latest"
```

### Making Changes to web-scrcpy Code
Since web-scrcpy is a submodule:
```bash
cd web-scrcpy
# Make your changes here
git add .
git commit -m "Your changes"
git push
cd ..
git add web-scrcpy
git commit -m "Update web-scrcpy submodule reference"
git push
```

## For Docker Builds

### Build Process
1. Docker COPY command pulls the source including submodule reference
2. `git submodule update --init --recursive` initializes submodule in container
3. Files are copied from `/app/web-scrcpy/` to `/app/` directory
4. Application runs with unified file structure

### Key Files After Build
- `/app/app.py` → from `/app/web-scrcpy/app.py`
- `/app/scrcpy.py` → from `/app/web-scrcpy/scrcpy.py`
- `/app/templates/` → from `/app/web-scrcpy/templates/`
- `/app/static/` → from `/app/web-scrcpy/static/`
- `/app/scrcpy-server` → from `/app/web-scrcpy/scrcpy-server`
- `/app/adb_manager.py` → from Docker repo (Docker-specific)
- `/app/adb/` → from Docker repo (ADB binaries)

## File Structure After Integration

```
Web-Scrcpy-Docker/
├── .gitmodules
├── Dockerfile (updated to pull from web-scrcpy/)
├── docker-compose.yml
├── app.py (DELETED - now in submodule)
├── scrcpy.py (DELETED - now in submodule)
├── templates/ (DELETED - now in submodule)
├── static/ (DELETED - now in submodule)
├── adb_manager.py (KEPT - Docker-specific)
├── adb/ (KEPT - Docker-specific)
├── requirements.txt
├── web-scrcpy/ (NEW - git submodule)
│   ├── app.py
│   ├── scrcpy.py
│   ├── templates/
│   ├── static/
│   ├── scrcpy-server
│   └── ...
└── README.md (update clone instructions)
```

## Workflow Summary

### To Create Submodule (One-time)
```bash
cd /home/pi/python_code/Web-Scrcpy-Docker
git submodule add ../web-scrcpy web-scrcpy
git add .gitmodules web-scrcpy
git commit -m "Add web-scrcpy as git submodule"
```

### To Clean Up After Submodule Added
```bash
git rm app.py scrcpy.py
git rm -r templates/ static/
git commit -m "Remove files now managed by web-scrcpy submodule"
git push
```

### To Clone This Repository
```bash
git clone --recurse-submodules <repo-url>
```

## Advantages
✅ Single source of truth for web-scrcpy code  
✅ Easy to update web-scrcpy independently  
✅ Clean separation of Docker-specific code  
✅ Reduces file duplication across repositories  
✅ Easy to track updates to web-scrcpy version

## Notes
- The `.gitmodules` file tracks submodule configuration
- Submodule updates are tracked as a reference commit in the main repo
- Docker build requires git to be installed (added to Dockerfile)
- The submodule is checked out at a specific commit (pinned version)
