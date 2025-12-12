# Architecture Overview

This document provides a comprehensive overview of Banana Slides' architecture, technology stack, and design decisions.

## 🏗️ High-Level Architecture

Banana Slides follows a **client-server architecture** with clear separation of concerns:

```
┌──────────────────────────────────────────────────────────────────┐
│                        User's Browser                             │
│                                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                    React Frontend                          │ │
│  │  (TypeScript, Vite, Tailwind CSS, Zustand)                │ │
│  │                                                            │ │
│  │  Components → Pages → Store → API Client                  │ │
│  └────────────────┬───────────────────────────────────────────┘ │
│                   │                                              │
└───────────────────┼──────────────────────────────────────────────┘
                    │ HTTP/REST API (JSON)
                    │ WebSocket for async updates (planned)
┌───────────────────┼──────────────────────────────────────────────┐
│                   ▼                                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                   Flask Backend                            │ │
│  │             (Python 3.10+, Flask 3.0)                      │ │
│  │                                                            │ │
│  │  Controllers ──→ Services ──→ Models ──→ Database         │ │
│  │       ▲              │                        │            │ │
│  │       │              ▼                        ▼            │ │
│  │       │      Task Manager          SQLite (SQLAlchemy)    │ │
│  │       │    (Async operations)                             │ │
│  └───────┼────────────┬───────────────────────────────────────┘ │
│          │            │                                          │
│  ┌───────┴────────────┴──────────────────────────────────────┐  │
│  │                    File System                            │  │
│  │                                                           │  │
│  │  • uploads/ - User uploaded files                        │  │
│  │  • backend/instance/ - SQLite database                   │  │
│  │  • backend/exports/ - Generated PPTX/PDF files           │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└──────────────────────┬───────────────────────────────────────────┘
                       │
┌──────────────────────┴───────────────────────────────────────────┐
│                   External AI Services                           │
│                                                                  │
│  ┌────────────────────┐    ┌──────────────────────────────┐    │
│  │  Text Generation   │    │    Image Generation          │    │
│  │                    │    │                              │    │
│  │  • Gemini API      │    │  • nano banana pro (Gemini)  │    │
│  │  • OpenAI API      │    │  • DALL-E (OpenAI)           │    │
│  └────────────────────┘    └──────────────────────────────┘    │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         Optional: MinerU File Parsing Service            │  │
│  │         (Advanced PDF/DOCX parsing)                      │  │
│  └──────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

## 🎨 Frontend Architecture

### Technology Stack

- **React 18**: UI framework with hooks
- **TypeScript**: Type-safe JavaScript
- **Vite 5**: Fast build tool and dev server
- **Tailwind CSS**: Utility-first CSS framework
- **Zustand**: Lightweight state management
- **React Router v6**: Client-side routing
- **Axios**: HTTP client for API calls
- **@dnd-kit**: Drag and drop functionality
- **Lucide React**: Icon library

### Directory Structure

```
frontend/
├── src/
│   ├── api/                    # API communication layer
│   │   ├── client.ts           # Axios configuration
│   │   └── endpoints.ts        # All API endpoint functions
│   │
│   ├── components/             # Reusable UI components
│   │   ├── shared/             # Common components
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Modal.tsx
│   │   │   ├── Toast.tsx
│   │   │   └── ...
│   │   ├── outline/            # Outline-specific components
│   │   ├── preview/            # Preview-specific components
│   │   └── history/            # History-specific components
│   │
│   ├── pages/                  # Main application pages
│   │   ├── Home.tsx            # Project creation
│   │   ├── OutlineEditor.tsx   # Outline editing
│   │   ├── DetailEditor.tsx    # Description editing
│   │   ├── SlidePreview.tsx    # Slide preview and export
│   │   └── History.tsx         # Project history
│   │
│   ├── store/                  # State management
│   │   └── useProjectStore.ts  # Zustand store for project state
│   │
│   ├── types/                  # TypeScript type definitions
│   │   └── index.ts            # Shared types
│   │
│   ├── utils/                  # Utility functions
│   │   ├── index.ts
│   │   └── projectUtils.ts
│   │
│   ├── styles/                 # Global styles
│   ├── constants/              # Constants and configurations
│   └── App.tsx                 # Root component with routing
│
├── public/                     # Static assets
├── index.html                  # HTML entry point
├── vite.config.ts              # Vite configuration
├── tailwind.config.js          # Tailwind CSS configuration
├── tsconfig.json               # TypeScript configuration
└── package.json                # Dependencies and scripts
```

### State Management

Banana Slides uses **Zustand** for state management, providing a simple and efficient way to manage application state:

```typescript
// useProjectStore.ts
interface ProjectStore {
  currentProject: Project | null;
  pages: Page[];
  isLoading: boolean;
  
  // Actions
  initializeProject: (data: CreateProjectRequest) => Promise<void>;
  fetchProject: (projectId: string) => Promise<void>;
  updatePageOrder: (pageIds: string[]) => Promise<void>;
  // ... more actions
}
```

### Routing Structure

```
/ (Home)                     → Project creation interface
├── /outline/:projectId      → Outline editing page
├── /detail/:projectId       → Description editing page
├── /preview/:projectId      → Slide preview and export
└── /history                 → Project history listing
```

### Component Design Principles

1. **Composition over Inheritance**: Small, reusable components
2. **Type Safety**: Everything is strongly typed with TypeScript
3. **Separation of Concerns**: Components, logic, and API calls are separate
4. **Responsive Design**: Mobile-friendly using Tailwind CSS utilities
5. **Accessibility**: Semantic HTML and ARIA attributes

## 🔧 Backend Architecture

### Technology Stack

- **Flask 3.0**: Lightweight web framework
- **Python 3.10+**: Modern Python features
- **SQLAlchemy**: ORM for database operations
- **SQLite**: Embedded database with WAL mode
- **uv**: Fast Python package manager
- **google-genai**: Google Gemini API client
- **openai**: OpenAI API client
- **python-pptx**: PowerPoint file generation
- **Pillow**: Image processing
- **reportlab**: PDF generation
- **markitdown**: File parsing (PDF, DOCX, etc.)

### Directory Structure

```
backend/
├── app.py                      # Application entry point
├── config.py                   # Configuration management
│
├── models/                     # Database models (SQLAlchemy)
│   ├── __init__.py
│   ├── project.py              # Project model
│   ├── page.py                 # Page/Slide model
│   ├── task.py                 # Async task tracking
│   ├── material.py             # Image materials
│   ├── user_template.py        # User templates
│   ├── reference_file.py       # Uploaded reference files
│   └── page_image_version.py  # Page image version history
│
├── controllers/                # API route handlers (blueprints)
│   ├── __init__.py
│   ├── project_controller.py   # Project CRUD and generation
│   ├── page_controller.py      # Page operations
│   ├── material_controller.py  # Material management
│   ├── template_controller.py  # Template operations
│   ├── reference_file_controller.py  # File upload/parsing
│   ├── export_controller.py    # PPTX/PDF export
│   └── file_controller.py      # Static file serving
│
├── services/                   # Business logic layer
│   ├── __init__.py
│   ├── ai_service.py           # Main AI service orchestrator
│   ├── prompts.py              # AI prompt templates
│   ├── task_manager.py         # Async task management
│   ├── file_service.py         # File management
│   ├── file_parser_service.py  # File parsing logic
│   ├── export_service.py       # Export generation
│   └── ai_providers/           # AI provider abstractions
│       ├── text/               # Text generation providers
│       │   ├── base.py
│       │   ├── genai_provider.py
│       │   └── openai_provider.py
│       └── image/              # Image generation providers
│           ├── base.py
│           ├── genai_provider.py
│           └── openai_provider.py
│
├── utils/                      # Utility functions
│   ├── __init__.py
│   ├── response.py             # Standard API response format
│   ├── validators.py           # Input validation
│   └── path_utils.py           # Path manipulation
│
├── instance/                   # Runtime data (auto-created)
│   └── database.db             # SQLite database file
│
└── exports/                    # Generated files (auto-created)
    ├── pptx/                   # PowerPoint exports
    └── pdf/                    # PDF exports
```

### Design Patterns

#### 1. **MVC-like Architecture**

```
Controller (Routes) → Service (Business Logic) → Model (Database)
```

- **Controllers**: Handle HTTP requests/responses
- **Services**: Contain business logic and external API calls
- **Models**: Define database schema and relationships

#### 2. **Blueprint Pattern**

Each controller is a Flask Blueprint for modular routing:

```python
# project_controller.py
project_bp = Blueprint('projects', __name__)

@project_bp.route('/api/projects', methods=['POST'])
def create_project():
    # Handle request
    pass
```

#### 3. **Factory Pattern**

The application uses a factory function for initialization:

```python
# app.py
def create_app():
    app = Flask(__name__)
    # Configure app
    # Register blueprints
    return app

app = create_app()
```

#### 4. **Provider Pattern**

AI services use pluggable providers for flexibility:

```python
# Supports both Gemini and OpenAI
text_provider = get_text_provider()  # Based on AI_PROVIDER_FORMAT
image_provider = get_image_provider()
```

### API Design Principles

1. **RESTful**: Standard HTTP methods (GET, POST, PUT, DELETE)
2. **JSON**: All requests and responses use JSON
3. **Consistent Response Format**:
   ```json
   {
     "success": true,
     "message": "Operation successful",
     "data": { ... }
   }
   ```
4. **Error Handling**: Structured error responses
5. **CORS Enabled**: Configured for cross-origin requests

### Database Design

**SQLite with WAL Mode** for better concurrency:

```python
# WAL mode configuration
PRAGMA journal_mode=WAL
PRAGMA synchronous=NORMAL
PRAGMA busy_timeout=30000
```

**Key Features**:
- Embedded database (no separate server)
- ACID compliance
- Suitable for single-server deployments
- Easy backup (single file)

## 🔄 Data Flow

### Creating a Project (from Idea)

```
┌─────────────┐
│   Browser   │
│  (Home.tsx) │
└──────┬──────┘
       │ POST /api/projects
       │ { creation_type: "idea", idea_prompt: "..." }
       ▼
┌─────────────────────────────┐
│  project_controller.py      │
│  create_project()           │
└──────┬──────────────────────┘
       │ 1. Validate input
       │ 2. Create Project record
       │ 3. Start async task
       ▼
┌─────────────────────────────┐
│  task_manager.py            │
│  generate_outline()         │
└──────┬──────────────────────┘
       │ Execute in background
       ▼
┌─────────────────────────────┐
│  ai_service.py              │
│  generate_outline()         │
└──────┬──────────────────────┘
       │ Generate prompt
       │ Call AI provider
       ▼
┌─────────────────────────────┐
│  Text Provider              │
│  (Gemini/OpenAI)            │
└──────┬──────────────────────┘
       │ Return JSON outline
       ▼
┌─────────────────────────────┐
│  task_manager.py            │
│  Parse and save pages       │
└──────┬──────────────────────┘
       │ Create Page records
       ▼
┌─────────────────────────────┐
│  Database (SQLite)          │
│  Save project & pages       │
└──────┬──────────────────────┘
       │ Task complete
       ▼
┌─────────────────────────────┐
│  Browser polls task status  │
│  GET /api/projects/{id}/    │
│      tasks/{task_id}        │
└─────────────────────────────┘
```

## 🔒 Security Considerations

1. **API Key Security**: Never expose API keys in frontend
2. **File Upload Validation**: Restrict file types and sizes
3. **Path Traversal Prevention**: Sanitize file paths
4. **SQL Injection Prevention**: Use SQLAlchemy ORM (parameterized queries)
5. **CORS Configuration**: Restrict allowed origins
6. **Input Validation**: Validate all user inputs server-side

## 📊 Scalability Considerations

### Current Limitations (Single Server)

- SQLite handles ~1000 concurrent reads well
- File storage is local (not distributed)
- No horizontal scaling

### Future Improvements

- **PostgreSQL**: For multi-server deployments
- **Object Storage**: S3/MinIO for file storage
- **Redis**: For caching and task queues
- **Load Balancer**: For horizontal scaling
- **WebSocket**: For real-time updates

## 🧩 Extension Points

### Adding a New AI Provider

1. Create provider class in `services/ai_providers/`
2. Implement the base interface
3. Update factory function
4. Add configuration to `.env`

### Adding a New Export Format

1. Create export function in `export_service.py`
2. Add route in `export_controller.py`
3. Add UI button in `SlidePreview.tsx`

### Adding a New File Type

1. Add parser in `file_parser_service.py`
2. Update `ALLOWED_REFERENCE_FILE_EXTENSIONS` in `config.py`
3. Test parsing logic

## 📚 Related Documentation

- [Workflow Guide](./03-workflow.md) - Understand the user journey
- [API Reference](./04-api-reference.md) - Detailed API documentation
- [Database Models](./05-database-models.md) - Database schema details

---

**Next**: Learn about the [complete workflow](./03-workflow.md) from idea to PPT export.
