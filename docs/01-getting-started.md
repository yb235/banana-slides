# Getting Started with Banana Slides

This guide will help you set up and run Banana Slides on your local machine.

## 📋 Prerequisites

### For Docker Installation (Recommended)
- **Docker Desktop** (Windows/Mac) or **Docker Engine** (Linux)
- **Docker Compose** (usually bundled with Docker Desktop)
- At least 4GB of free RAM
- 2GB of free disk space

### For Manual Installation
- **Python 3.10 or higher**
- **Node.js 16 or higher** with npm
- **uv** - Python package manager
- At least 2GB of free disk space

### API Keys Required
You'll need API keys from one of these providers:
- **Google Gemini API** (recommended) - Get it from [Google AI Studio](https://ai.google.dev/)
- **OpenAI API** - Get it from [OpenAI Platform](https://platform.openai.com/)
- Or use [AIHubMix](https://aihubmix.com/?aff=17EC) as a unified API provider

## 🚀 Method 1: Docker Installation (Recommended)

This is the easiest and fastest way to get started.

### Step 1: Clone the Repository

```bash
git clone https://github.com/Anionex/banana-slides.git
cd banana-slides
```

### Step 2: Configure Environment Variables

Copy the example environment file:

```bash
cp .env.example .env
```

Edit the `.env` file with your preferred text editor:

```bash
nano .env    # or use vim, code, notepad, etc.
```

**Minimal Configuration for Gemini:**

```env
# AI Provider Format: "gemini" or "openai"
AI_PROVIDER_FORMAT=gemini

# Gemini Configuration
GOOGLE_API_KEY=your-gemini-api-key-here
GOOGLE_API_BASE=https://generativelanguage.googleapis.com

# Models
TEXT_MODEL=gemini-2.5-flash
IMAGE_MODEL=gemini-3-pro-image-preview

# Server Configuration
PORT=5000
CORS_ORIGINS=http://localhost:3000
```

**Alternative Configuration for OpenAI:**

```env
# AI Provider Format
AI_PROVIDER_FORMAT=openai

# OpenAI Configuration
OPENAI_API_KEY=your-openai-api-key-here
OPENAI_API_BASE=https://api.openai.com/v1

# Models (adjust based on your provider)
TEXT_MODEL=gpt-4
IMAGE_MODEL=dall-e-3

# Server Configuration
PORT=5000
CORS_ORIGINS=http://localhost:3000
```

### Step 3: Start the Application

```bash
docker compose up -d
```

This will:
- Pull the necessary Docker images
- Build the frontend and backend containers
- Start both services in the background

### Step 4: Access the Application

Open your browser and navigate to:
- **Frontend UI**: http://localhost:3000
- **Backend API**: http://localhost:5000

You should see the Banana Slides home page!

### Step 5: Check Logs (Optional)

To monitor the application:

```bash
# View all logs
docker compose logs -f

# View backend logs only
docker compose logs -f backend

# View frontend logs only
docker compose logs -f frontend
```

### Stopping the Application

```bash
docker compose down
```

### Updating the Application

```bash
git pull
docker compose down
docker compose build --no-cache
docker compose up -d
```

## 🔧 Method 2: Manual Installation

For developers who want to work on the code directly.

### Step 1: Clone the Repository

```bash
git clone https://github.com/Anionex/banana-slides.git
cd banana-slides
```

### Step 2: Set Up Backend

#### Install uv (Python package manager)

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

For Windows, use PowerShell:
```powershell
irm https://astral.sh/uv/install.ps1 | iex
```

#### Configure Environment

```bash
cp .env.example .env
# Edit .env with your API keys (same as Docker method)
```

#### Install Backend Dependencies

```bash
# From project root
uv sync
```

This automatically creates a virtual environment and installs all Python dependencies.

#### Start Backend Server

```bash
cd backend
uv run python app.py
```

The backend will start on http://localhost:5000

### Step 3: Set Up Frontend

Open a new terminal window:

```bash
cd frontend
npm install
npm run dev
```

The frontend will start on http://localhost:3000

### Verification

1. Visit http://localhost:5000/health - Should return `{"status": "ok"}`
2. Visit http://localhost:3000 - Should show the Banana Slides interface

## 🧪 Testing the Installation

1. Open http://localhost:3000 in your browser
2. Click on "Start with an idea"
3. Enter a simple prompt like: "Create a presentation about climate change"
4. Click "Create Project"
5. If everything is set up correctly, you should be redirected to the outline editor

## 🔍 Troubleshooting

### Port Already in Use

If ports 3000 or 5000 are already in use:

**For Docker:**
Edit `docker-compose.yml` to change the port mappings:
```yaml
ports:
  - "3001:3000"  # Frontend
  - "5001:5000"  # Backend
```

**For Manual Setup:**
- Backend: Change `PORT` in `.env`
- Frontend: Edit `vite.config.ts` to set a different port

### Docker Permission Errors (Linux)

If you get permission errors:
```bash
sudo docker compose up -d
```

Or add your user to the docker group:
```bash
sudo usermod -aG docker $USER
# Log out and log back in
```

### API Key Errors

If you see errors about API keys:
1. Check your `.env` file has the correct keys
2. Verify the keys are valid by testing them directly with the API provider
3. Make sure there are no extra spaces or quotes around the keys
4. Restart the backend after changing `.env`

### Module Not Found Errors (Manual Setup)

If Python can't find modules:
```bash
# Make sure you're in the project root
uv sync --reinstall
```

If Node modules are missing:
```bash
cd frontend
rm -rf node_modules package-lock.json
npm install
```

### Database Errors

If you encounter SQLite database errors:
```bash
# Delete the database and restart
rm -rf backend/instance/database.db
# Then restart the backend
```

### Docker Build Failures

If Docker build fails:
```bash
# Clear Docker cache and rebuild
docker compose down
docker system prune -a
docker compose build --no-cache
docker compose up -d
```

## 📦 What Gets Installed?

### Backend Dependencies
- **Flask**: Web framework
- **SQLAlchemy**: Database ORM
- **google-genai**: Gemini API client
- **openai**: OpenAI API client
- **python-pptx**: PowerPoint generation
- **Pillow**: Image processing
- **reportlab**: PDF generation
- **markitdown**: File parsing (PDF, DOCX, etc.)

### Frontend Dependencies
- **React**: UI framework
- **TypeScript**: Type safety
- **Vite**: Build tool
- **Tailwind CSS**: Styling
- **Zustand**: State management
- **React Router**: Navigation
- **Axios**: HTTP client
- **@dnd-kit**: Drag and drop

## 🎯 Next Steps

Now that you have Banana Slides running:

1. **Learn the workflow**: Read the [Workflow Guide](./03-workflow.md)
2. **Explore the architecture**: Check out [Architecture Overview](./02-architecture.md)
3. **Try creating a presentation**: Follow the UI and experiment!
4. **Review API documentation**: See [API Reference](./04-api-reference.md) if you want to understand or extend the backend

## 💡 Tips for Development

- **Hot Reload**: Both frontend (Vite) and backend (Flask debug mode) support hot reload
- **Database Browser**: Use tools like [DB Browser for SQLite](https://sqlitebrowser.org/) to inspect the database
- **API Testing**: Use Postman or curl to test backend endpoints directly
- **Console Logs**: Check browser console (F12) for frontend errors and terminal for backend logs

## 🌐 Environment Variables Reference

Here's a complete list of available environment variables:

```env
# === AI Provider Configuration ===
AI_PROVIDER_FORMAT=gemini          # "gemini" or "openai"

# === Gemini Settings ===
GOOGLE_API_KEY=                    # Your Gemini API key
GOOGLE_API_BASE=                   # Gemini API base URL

# === OpenAI Settings ===
OPENAI_API_KEY=                    # Your OpenAI API key
OPENAI_API_BASE=                   # OpenAI API base URL

# === Model Configuration ===
TEXT_MODEL=gemini-2.5-flash        # Model for text generation
IMAGE_MODEL=gemini-3-pro-image-preview  # Model for image generation
IMAGE_CAPTION_MODEL=gemini-2.5-flash    # Model for image captioning

# === Server Configuration ===
PORT=5000                          # Backend port
FLASK_ENV=development              # development or production
SECRET_KEY=                        # Flask secret key (auto-generated if not set)
CORS_ORIGINS=http://localhost:3000 # Allowed CORS origins

# === Advanced Settings ===
MAX_DESCRIPTION_WORKERS=5          # Parallel workers for description generation
MAX_IMAGE_WORKERS=8                # Parallel workers for image generation
LOG_LEVEL=INFO                     # Logging level: DEBUG, INFO, WARNING, ERROR

# === Optional: MinerU File Parsing ===
MINERU_TOKEN=                      # MinerU API token (for advanced file parsing)
MINERU_API_BASE=https://mineru.net # MinerU API base URL
```

---

**Need help?** Check the [Troubleshooting Guide](./09-troubleshooting.md) or [open an issue](https://github.com/Anionex/banana-slides/issues).
