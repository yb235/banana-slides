# Troubleshooting Guide

Common issues and their solutions for Banana Slides.

## 🔍 Quick Diagnosis

### Is it a frontend or backend issue?

1. **Check frontend console** (F12 in browser)
   - Look for red errors
   - Check Network tab for failed API calls

2. **Check backend logs**:
   ```bash
   # Docker
   docker compose logs -f backend
   
   # Manual
   # Check terminal where backend is running
   ```

3. **Check health endpoint**:
   ```bash
   curl http://localhost:5000/health
   # Should return: {"status": "ok", ...}
   ```

## 🚨 Common Issues

### Installation Issues

#### Port Already in Use

**Problem**: Ports 3000 or 5000 are occupied

**Solution for Docker**:
```yaml
# Edit docker-compose.yml
services:
  frontend:
    ports:
      - "3001:3000"  # Change to different port
  backend:
    ports:
      - "5001:5000"  # Change to different port
```

**Solution for Manual**:
```env
# Backend: Edit .env
PORT=5001

# Frontend: Edit vite.config.ts
export default defineConfig({
  server: { port: 3001 }
})
```

---

#### Docker Permission Denied (Linux)

**Problem**: `permission denied while trying to connect to the Docker daemon`

**Solution**:
```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Log out and log back in

# Or use sudo temporarily
sudo docker compose up -d
```

---

#### uv Installation Fails

**Problem**: Cannot install uv package manager

**Solution for Linux/Mac**:
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc  # or ~/.zshrc
```

**Solution for Windows**:
```powershell
# Use PowerShell as Administrator
irm https://astral.sh/uv/install.ps1 | iex
```

---

### API Key Issues

#### Invalid API Key

**Problem**: "Invalid API key" or "Authentication failed"

**Checks**:
1. Verify key is correct (copy-paste carefully)
2. Check for extra spaces or quotes
3. Ensure no line breaks in `.env`
4. Restart backend after changing `.env`

**Example `.env`**:
```env
# Wrong
GOOGLE_API_KEY="your-key-here"  # Remove quotes
GOOGLE_API_KEY= your-key-here   # Remove spaces

# Correct
GOOGLE_API_KEY=your-key-here
```

---

#### Rate Limit Exceeded

**Problem**: "Rate limit exceeded" or "429 Too Many Requests"

**Solution**:
1. Wait a few minutes
2. Check your API quota/billing
3. Reduce parallel workers:
   ```env
   MAX_DESCRIPTION_WORKERS=2
   MAX_IMAGE_WORKERS=3
   ```
4. Consider upgrading your API plan

---

### Database Issues

#### Database Locked

**Problem**: "database is locked" error

**Solution**:
```bash
# Stop all processes
docker compose down
# Or kill backend manually

# Delete database (WARNING: Loses all data)
rm backend/instance/database.db

# Restart
docker compose up -d
```

**Prevention**: WAL mode is enabled by default to reduce locking.

---

#### Database Corrupted

**Problem**: "database disk image is malformed"

**Solution**:
```bash
# Backup database
cp backend/instance/database.db backup/database.db

# Try to recover
sqlite3 backend/instance/database.db
> PRAGMA integrity_check;
> .dump > recovery.sql
> .quit

# Create new database from dump
rm backend/instance/database.db
sqlite3 backend/instance/database.db < recovery.sql
```

---

### Generation Issues

#### Outline Not Generating

**Problem**: Stuck on "Generating outline..."

**Checks**:
1. Check backend logs for errors
2. Verify API key is valid
3. Check network connectivity
4. Look for task status:
   ```bash
   curl http://localhost:5000/api/projects/{project_id}/tasks/{task_id}
   ```

**Common Causes**:
- API key invalid
- Network timeout
- Rate limit hit
- Model not available

---

#### AI Returns Invalid JSON

**Problem**: "Failed to parse AI response"

**Backend logs show**:
```
Invalid JSON response from AI
```

**Solution**:
1. Check if using correct model
2. Try with a different prompt
3. Check model availability
4. Increase timeout settings

---

#### Images Not Generating

**Problem**: Slides remain blank after "Generate Images"

**Checks**:
1. Verify IMAGE_MODEL is correct
2. Check image generation quota
3. Look for specific errors in backend logs
4. Ensure descriptions exist first

**Common Causes**:
- Image model not available
- Invalid image prompt
- Quota exceeded
- Network timeout

---

### File Upload Issues

#### Reference File Not Parsing

**Problem**: File stuck in "parsing" status

**Checks**:
1. Check backend logs for parsing errors
2. Verify file is not corrupted
3. Check file size (must be < 200MB)
4. Ensure file type is supported

**Manually trigger parsing**:
```bash
curl -X POST http://localhost:5000/api/reference-files/{file_id}/parse
```

**Supported formats**: PDF, DOCX, DOC, PPTX, PPT, XLSX, XLS, TXT, MD, CSV

---

#### MinerU Parsing Fails

**Problem**: "MinerU parsing failed, falling back..."

**Solution**:
1. Check MINERU_TOKEN in `.env`
2. Verify MinerU service is available
3. Check network connectivity
4. Fallback to markitdown is automatic

---

### Export Issues

#### PPTX Export Fails

**Problem**: Cannot export to PowerPoint

**Checks**:
1. Ensure all pages have images
2. Check disk space
3. Look for backend errors
4. Verify python-pptx is installed

**Manual test**:
```bash
curl http://localhost:5000/api/projects/{project_id}/export/pptx
```

---

#### PDF Export Quality Poor

**Problem**: PDF images look blurry

**Solution**: This is a known limitation. For best quality:
1. Export as PPTX instead
2. Use PowerPoint to export to PDF
3. Or increase image resolution:
   ```env
   # In .env (future enhancement)
   IMAGE_RESOLUTION=4K
   ```

---

### Frontend Issues

#### Blank Page After Login

**Problem**: Frontend loads but shows nothing

**Checks**:
1. Open browser console (F12)
2. Check for JavaScript errors
3. Verify API is reachable:
   ```javascript
   fetch('http://localhost:5000/health')
   ```
4. Clear browser cache (Ctrl+Shift+Delete)

---

#### "Network Error" on API Calls

**Problem**: All API calls fail with network error

**Checks**:
1. Is backend running?
   ```bash
   curl http://localhost:5000/health
   ```
2. CORS issue? Check backend logs for CORS errors
3. Check `CORS_ORIGINS` in `.env`:
   ```env
   CORS_ORIGINS=http://localhost:3000
   ```
4. Try from different browser

---

#### State Not Updating

**Problem**: Changes don't reflect in UI

**Solution**:
1. Hard refresh (Ctrl+F5 or Cmd+Shift+R)
2. Check Zustand store in React DevTools
3. Clear localStorage:
   ```javascript
   localStorage.clear()
   location.reload()
   ```

---

## 🐛 Debugging Tips

### Enable Debug Logging

**Backend**:
```env
# In .env
LOG_LEVEL=DEBUG
```

**Frontend**:
```typescript
// In browser console
localStorage.setItem('debug', '*')
location.reload()
```

---

### Inspect Network Requests

**Chrome DevTools**:
1. Press F12
2. Click "Network" tab
3. Reproduce the issue
4. Check failed requests (red)
5. Click request → Preview/Response

---

### Check Database Contents

**Using DB Browser**:
1. Download [DB Browser for SQLite](https://sqlitebrowser.org/)
2. Open `backend/instance/database.db`
3. Browse tables
4. Run queries

**Using Python**:
```bash
cd backend
uv run python

>>> from models import db, Project, Page
>>> from app import app
>>> with app.app_context():
...     projects = Project.query.all()
...     print(projects)
```

---

### Monitor Resource Usage

**Docker**:
```bash
docker stats
```

**Manual**:
```bash
# CPU and memory
top  # or htop

# Disk space
df -h

# Network
netstat -tuln | grep 5000
```

---

## 🔧 Advanced Troubleshooting

### Reset Everything

**Nuclear option** (deletes all data):
```bash
# Docker
docker compose down -v
rm -rf backend/instance/
rm -rf uploads/
docker compose up -d

# Manual
rm -rf backend/instance/
rm -rf uploads/
# Restart backend
```

---

### Check Dependencies

**Backend**:
```bash
cd backend
uv pip list
uv pip check
```

**Frontend**:
```bash
cd frontend
npm list
npm audit
```

---

### Update Everything

```bash
# Pull latest code
git pull

# Docker
docker compose down
docker compose build --no-cache
docker compose up -d

# Manual
cd backend && uv sync
cd ../frontend && npm install
```

---

## 📞 Getting Help

### Before Asking for Help

1. **Check the logs** (frontend console + backend logs)
2. **Search existing issues** on GitHub
3. **Try to reproduce** the issue consistently
4. **Gather information**:
   - Error messages (full text)
   - Steps to reproduce
   - Environment (OS, Docker version, etc.)
   - Screenshots if applicable

### Where to Get Help

1. **GitHub Issues**: [Open an issue](https://github.com/Anionex/banana-slides/issues)
2. **Discussions**: Community discussions
3. **Documentation**: Re-read relevant docs

### What to Include in Bug Report

```markdown
## Description
Brief description of the issue

## Steps to Reproduce
1. Step 1
2. Step 2
3. ...

## Expected Behavior
What should happen

## Actual Behavior
What actually happens

## Environment
- OS: Windows 10 / macOS 13 / Ubuntu 22.04
- Installation: Docker / Manual
- Backend: Python 3.11, Flask 3.0
- Frontend: Node 18.x
- Browser: Chrome 120

## Logs
```
Paste relevant logs here
```

## Screenshots
[Attach screenshots if applicable]
```

---

## 🎓 Common Mistakes

### Forgetting to Restart

**Issue**: Made changes but they don't apply

**Remember**:
- Change `.env` → Restart backend
- Change code → Restart backend (or wait for hot reload)
- Update dependencies → Restart everything

---

### Wrong Directory

**Issue**: Commands don't work

**Remember**:
```bash
# Project root: docker compose, git
cd /path/to/banana-slides

# Backend: python, uv
cd /path/to/banana-slides/backend

# Frontend: npm
cd /path/to/banana-slides/frontend
```

---

### Mixing Installation Methods

**Issue**: Conflicts between Docker and manual install

**Solution**: Choose one method and stick with it
- Docker: Don't run manual backend
- Manual: Don't use docker compose

---

## 📚 Related Documentation

- [Getting Started](./01-getting-started.md) - Installation guide
- [Architecture](./02-architecture.md) - System overview
- [API Reference](./04-api-reference.md) - API endpoints

---

**Still stuck?** [Open an issue](https://github.com/Anionex/banana-slides/issues) with details!
