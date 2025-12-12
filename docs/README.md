# Banana Slides Documentation

Welcome to the comprehensive documentation for Banana Slides! This guide will help you understand the architecture, codebase, APIs, and workflows of this AI-powered presentation generator.

## 📚 Documentation Index

### For First-Time Users

If you're new to Banana Slides, start here:

1. **[README.md](../README.md)** - Project overview, features, and quick start
2. **[DEPLOYMENT.md](DEPLOYMENT.md)** - Complete deployment guide (Docker, manual, production)
3. **[User Guide](#user-guide-quick-overview)** - How to use the application

### For Developers

For understanding the codebase and contributing:

1. **[ARCHITECTURE.md](ARCHITECTURE.md)** - System architecture, tech stack, and data flow
2. **[BACKEND.md](BACKEND.md)** - Backend code structure, services, controllers, and models
3. **[FRONTEND.md](FRONTEND.md)** - Frontend components, state management, and UI workflows
4. **[API.md](API.md)** - Complete API reference with all endpoints

## User Guide: Quick Overview

### What is Banana Slides?

Banana Slides is an AI-native PowerPoint presentation generator that lets you create professional, beautiful slides from:
- **💡 An idea**: Just describe what you want to present
- **📝 An outline**: Provide a structured outline
- **📄 Detailed descriptions**: Write detailed page descriptions

### Key Features

1. **Three Creation Modes**
   - Idea → AI generates outline → descriptions → images
   - Outline → AI generates descriptions → images
   - Descriptions → AI generates outline → images

2. **Natural Language Editing**
   - "Add a case study slide after slide 3"
   - "Make the background blue and add more charts"
   - "Change the title to 'Market Analysis'"

3. **Material Management**
   - Upload reference documents (PDF, DOCX, etc.)
   - Add custom images and charts
   - AI automatically extracts relevant content

4. **Professional Export**
   - Export as PPTX (PowerPoint)
   - Export as PDF
   - Ready to present, no further editing needed

### How to Use

#### 1. Installation

**Using Docker (Recommended)**:
```bash
git clone https://github.com/Anionex/banana-slides
cd banana-slides
cp .env.example .env
# Edit .env with your API key
docker compose up -d
```

Open http://localhost:3000

See [DEPLOYMENT.md](DEPLOYMENT.md) for detailed installation instructions.

#### 2. Create Your First Presentation

**Option A: From an Idea**
1. Go to Home page
2. Select "Idea" tab
3. Enter: "Create a presentation about AI trends in 2024"
4. (Optional) Upload a template image for style reference
5. Click "Create Project"
6. AI generates outline automatically
7. Review and edit outline
8. Generate descriptions
9. Generate images
10. Export as PPTX or PDF

**Option B: From an Outline**
1. Select "Outline" tab
2. Enter structured outline:
   ```
   # Introduction
   - What is AI
   - Why it matters
   
   # Current Trends
   - Large Language Models
   - Computer Vision
   
   # Future Outlook
   - Predictions
   - Challenges
   ```
3. Click "Create Project"
4. Continue from step 7 above

**Option C: From Descriptions**
1. Select "Description" tab
2. Enter detailed descriptions for each slide
3. AI generates outline from your descriptions
4. Review and generate images

#### 3. Edit and Refine

**Edit Outline**:
- Click on any page title or key point to edit
- Use natural language refinement: "Add a slide about case studies"
- Drag and drop to reorder slides

**Edit Descriptions**:
- Edit layout, text elements, and design notes
- Add material references
- Regenerate individual descriptions

**Edit Images**:
- Click "Edit" on any slide
- Enter prompt: "Change background to blue, add more charts"
- Select reference materials
- View version history and revert if needed

#### 4. Export

- Click "Export PPTX" for PowerPoint
- Click "Export PDF" for PDF
- Download and present!

## Architecture Overview

Banana Slides follows a modern client-server architecture:

```
Frontend (React + TypeScript)
    ↓ REST API
Backend (Flask + Python)
    ↓ AI API calls
Google Gemini / OpenAI
```

### Frontend
- **Framework**: React 18 + TypeScript + Vite
- **State**: Zustand for state management
- **Styling**: Tailwind CSS
- **Routing**: React Router v6

### Backend
- **Framework**: Flask 3.0 + Python 3.10+
- **Database**: SQLite with WAL mode
- **AI**: Pluggable providers (Gemini/OpenAI)
- **Export**: python-pptx, ReportLab

See [ARCHITECTURE.md](ARCHITECTURE.md) for detailed architecture.

## API Overview

### Base URL
```
http://localhost:5000
```

### Key Endpoints

**Projects**:
- `POST /api/projects` - Create project
- `GET /api/projects` - List projects
- `GET /api/projects/{id}` - Get project details
- `PUT /api/projects/{id}` - Update project
- `DELETE /api/projects/{id}` - Delete project

**Generation**:
- `POST /api/projects/{id}/generate-outline` - Generate outline
- `POST /api/projects/{id}/generate-descriptions` - Generate descriptions
- `POST /api/projects/{id}/generate-images` - Generate images

**Pages**:
- `POST /api/pages/{id}/generate-image` - Generate single image
- `POST /api/pages/{id}/edit-image` - Edit image with prompt

**Export**:
- `POST /api/export/pptx` - Export as PowerPoint
- `POST /api/export/pdf` - Export as PDF

See [API.md](API.md) for complete API reference.

## Development Workflow

### Backend Development

1. **Setup**:
   ```bash
   cd banana-slides
   uv sync
   cp .env.example .env
   # Edit .env
   ```

2. **Run**:
   ```bash
   cd backend
   uv run python app.py
   ```

3. **Project Structure**:
   - `models/` - Database models
   - `services/` - Business logic
   - `controllers/` - API endpoints
   - `utils/` - Helper functions

See [BACKEND.md](BACKEND.md) for detailed backend documentation.

### Frontend Development

1. **Setup**:
   ```bash
   cd frontend
   npm install
   ```

2. **Run**:
   ```bash
   npm run dev
   ```

3. **Project Structure**:
   - `pages/` - Route components
   - `components/` - Reusable components
   - `store/` - Zustand state management
   - `api/` - API integration

See [FRONTEND.md](FRONTEND.md) for detailed frontend documentation.

## Configuration

### Environment Variables

**Backend (.env)**:
```bash
# AI Provider (required)
AI_PROVIDER_FORMAT=gemini  # or "openai"

# Gemini Config
GOOGLE_API_KEY=your-key
GOOGLE_API_BASE=https://generativelanguage.googleapis.com

# Or OpenAI Config
OPENAI_API_KEY=your-key
OPENAI_API_BASE=https://api.openai.com/v1

# Models
TEXT_MODEL=gemini-2.5-flash
IMAGE_MODEL=gemini-3-pro-image-preview

# Server
PORT=5000
FLASK_ENV=development

# Concurrency
MAX_DESCRIPTION_WORKERS=5
MAX_IMAGE_WORKERS=8
```

**Frontend (.env.local)**:
```bash
VITE_API_BASE_URL=http://localhost:5000
```

See [DEPLOYMENT.md](DEPLOYMENT.md) for complete configuration guide.

## Common Workflows

### Workflow 1: Create from Idea

```
User enters idea
    ↓
AI generates outline (Gemini/OpenAI)
    ↓
User reviews/edits outline
    ↓
AI generates descriptions for each page
    ↓
User reviews/edits descriptions
    ↓
AI generates images for each page
    ↓
User reviews/edits images
    ↓
Export as PPTX or PDF
```

### Workflow 2: Edit Single Slide

```
User clicks "Edit" on slide
    ↓
User enters edit prompt
    ↓
(Optional) Select reference materials
    ↓
AI regenerates image
    ↓
New version saved in history
    ↓
User can revert to previous version
```

### Workflow 3: Upload Reference Document

```
User uploads PDF/DOCX
    ↓
Backend parses document (markitdown)
    ↓
Extracts text, images, tables
    ↓
Converts to Markdown
    ↓
Associates with project
    ↓
AI uses content in generation
```

## Key Technologies

### AI Integration

**Supported Providers**:
1. **Google Gemini** (default)
   - gemini-2.5-flash (text)
   - gemini-3-pro-image-preview (images)

2. **OpenAI** (alternative)
   - Compatible with OpenAI API format
   - Works with AIHubMix and other proxies

**Pluggable Architecture**:
- Easy to add new providers
- See `backend/services/ai_providers/`

### File Processing

**Supported Formats**:
- Documents: PDF, DOCX, DOC, PPTX, PPT
- Spreadsheets: XLSX, XLS, CSV
- Text: TXT, MD
- Images: PNG, JPG, JPEG, GIF, WEBP

**Parsing**:
- Local: markitdown library
- Cloud: MinerU API (optional)

### Export

**PPTX**:
- Uses python-pptx
- 16:9 aspect ratio
- Each image becomes a slide
- Ready for PowerPoint

**PDF**:
- Uses ReportLab + Pillow
- High-quality rendering
- 16:9 aspect ratio

## Database Schema

### Core Tables

**projects**:
- id, idea_prompt, outline_text, description_text
- creation_type, template_image_path, status
- created_at, updated_at

**pages**:
- id, project_id, order_index, part
- outline_content (JSON), description_content (JSON)
- image_url, created_at, updated_at

**tasks**:
- id, project_id, task_type, status
- progress (JSON), result (JSON)
- created_at, updated_at

**materials**:
- id, project_id, filename, file_path
- material_type, created_at

**reference_files**:
- id, project_id, filename, file_path
- parse_status, markdown_content
- created_at

See [BACKEND.md](BACKEND.md) for detailed schema.

## Performance Considerations

### Parallel Processing
- Descriptions generated in parallel (MAX_DESCRIPTION_WORKERS)
- Images generated in parallel (MAX_IMAGE_WORKERS)
- ThreadPoolExecutor for async tasks

### Optimization Tips
1. **Image Generation**: Most time-consuming, runs in background
2. **Caching**: Consider caching identical AI prompts
3. **Database**: SQLite with WAL mode for concurrent access
4. **File Storage**: Local filesystem (consider S3 for production)

## Troubleshooting

### Common Issues

**1. "Cannot connect to backend"**
- Check if backend is running: `curl http://localhost:5000/health`
- Verify VITE_API_BASE_URL in frontend
- Check CORS configuration

**2. "Invalid API key"**
- Verify API key in .env
- Test key with API provider documentation
- Check for trailing spaces in .env

**3. "Database locked"**
- Ensure WAL mode is enabled (automatic)
- Restart backend
- Check for long-running processes

**4. "Image generation failed"**
- Check AI model availability
- Verify API quotas
- Check backend logs: `docker compose logs backend`

**5. "File upload failed"**
- Check MAX_CONTENT_LENGTH setting
- Verify disk space
- Check upload folder permissions

See [DEPLOYMENT.md](DEPLOYMENT.md) for detailed troubleshooting.

## Security Best Practices

1. **API Keys**: Never commit to version control
2. **CORS**: Restrict to your domain in production
3. **HTTPS**: Always use in production
4. **File Uploads**: Validate types and sizes
5. **Input Validation**: Sanitize all user inputs

See [DEPLOYMENT.md](DEPLOYMENT.md) for security guide.

## Contributing

We welcome contributions! Here's how:

1. **Fork** the repository
2. **Create** a feature branch
3. **Make** your changes
4. **Test** thoroughly
5. **Submit** a pull request

### Code Style
- **Backend**: Follow PEP 8
- **Frontend**: Use TypeScript, ESLint, Prettier

### Testing
- Write unit tests for new features
- Test with different AI providers
- Test edge cases

## Resources

### Documentation
- [Architecture](ARCHITECTURE.md) - System design and structure
- [Backend Guide](BACKEND.md) - Backend development
- [Frontend Guide](FRONTEND.md) - Frontend development
- [API Reference](API.md) - Complete API docs
- [Deployment Guide](DEPLOYMENT.md) - Installation and deployment

### External Resources
- [Flask Documentation](https://flask.palletsprojects.com/)
- [React Documentation](https://react.dev/)
- [Google Gemini API](https://ai.google.dev/docs)
- [AIHubMix](https://aihubmix.com/?aff=17EC) - AI API Provider

### Community
- [GitHub Repository](https://github.com/Anionex/banana-slides)
- [Issue Tracker](https://github.com/Anionex/banana-slides/issues)
- [Pull Requests](https://github.com/Anionex/banana-slides/pulls)

## FAQ

### Q: Do I need coding experience to use Banana Slides?
**A**: No! The application has a user-friendly web interface. You only need API keys from an AI provider.

### Q: Which AI provider should I use?
**A**: We recommend starting with Google Gemini (free tier available). AIHubMix is also great for access to multiple models.

### Q: Can I use my own AI models?
**A**: Yes! The backend uses a pluggable provider architecture. Add your provider in `backend/services/ai_providers/`.

### Q: How long does it take to generate a presentation?
**A**: 
- Outline: ~10 seconds
- Descriptions: ~30 seconds (depends on page count)
- Images: ~2-5 minutes (depends on page count and model)

### Q: Can I edit generated slides?
**A**: Yes! You can edit at every stage:
- Edit outline
- Edit descriptions
- Edit images with natural language prompts
- Version history for reverting changes

### Q: What happens to my data?
**A**: All data is stored locally on your server. Nothing is sent to external services except AI API calls for generation.

### Q: Can I run this offline?
**A**: Partially. You can run the application offline, but AI generation requires API calls to Gemini/OpenAI.

### Q: How do I update to the latest version?
**A**: 
```bash
git pull
docker compose build --no-cache
docker compose up -d
```

### Q: Is this production-ready?
**A**: The application is functional and can be used in production with proper security measures (HTTPS, API key protection, etc.). See [DEPLOYMENT.md](DEPLOYMENT.md).

## License

MIT License - See [LICENSE](../LICENSE) file for details.

## Credits

- **Original Authors**: Anionex team
- **AI Models**: Google Gemini, OpenAI
- **Sponsor**: [AIHubMix](https://aihubmix.com/?aff=17EC)

## Support

Need help?
- 📖 Check the [documentation](.)
- 🐛 Report [issues](https://github.com/Anionex/banana-slides/issues)
- 💬 Ask questions in [discussions](https://github.com/Anionex/banana-slides/discussions)

---

**Happy Presenting! 🍌**
