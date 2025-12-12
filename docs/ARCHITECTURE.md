# Architecture Overview

## Table of Contents
- [System Architecture](#system-architecture)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Data Flow](#data-flow)
- [Key Components](#key-components)

## System Architecture

Banana Slides follows a **client-server architecture** with clear separation between the frontend and backend:

```
┌─────────────────────────────────────────────────────────────┐
│                         Frontend                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  React 18 + TypeScript + Vite                       │   │
│  │  - Pages: Home, OutlineEditor, DetailEditor, etc.  │   │
│  │  - State Management: Zustand                        │   │
│  │  - Routing: React Router v6                        │   │
│  │  - Styling: Tailwind CSS                           │   │
│  └──────────────────────────────────────────────────────┘   │
│                          ↓ HTTP/REST API                     │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│                         Backend                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Flask 3.0 + Python 3.10+                          │   │
│  │                                                      │   │
│  │  Controllers (API Endpoints)                        │   │
│  │    ↓                                                │   │
│  │  Services (Business Logic)                          │   │
│  │    - AIService: Gemini/OpenAI Integration          │   │
│  │    - ExportService: PPTX/PDF Generation           │   │
│  │    - FileService: File Management                  │   │
│  │    - TaskManager: Async Task Handling             │   │
│  │    ↓                                                │   │
│  │  Models (Database Layer)                            │   │
│  │    - Project, Page, Material, Task, etc.          │   │
│  │    ↓                                                │   │
│  │  SQLite Database (with WAL mode)                   │   │
│  └──────────────────────────────────────────────────────┘   │
│                          ↓                                   │
│  External Services:                                         │
│  - Google Gemini API / OpenAI API (AI Generation)          │
│  - MinerU API (File Parsing - Optional)                    │
└─────────────────────────────────────────────────────────────┘
```

## Technology Stack

### Frontend Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| React | 18.x | UI framework |
| TypeScript | 5.x | Type-safe JavaScript |
| Vite | 5.x | Build tool and dev server |
| Zustand | 4.x | State management (lightweight) |
| React Router | 6.x | Client-side routing |
| Tailwind CSS | 3.x | Utility-first CSS framework |
| Axios | 1.x | HTTP client for API calls |
| @dnd-kit | 6.x | Drag-and-drop functionality |
| Lucide React | Latest | Icon library |

### Backend Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| Python | 3.10+ | Programming language |
| Flask | 3.0+ | Web framework |
| Flask-SQLAlchemy | 3.1+ | ORM for database |
| SQLite | 3.x | Database (with WAL mode) |
| Flask-CORS | 4.0+ | Cross-origin resource sharing |
| google-genai | 1.52+ | Google Gemini AI SDK |
| openai | 1.0+ | OpenAI API SDK |
| python-pptx | 1.0+ | PowerPoint file generation |
| Pillow | 12.0+ | Image processing |
| ReportLab | 4.1+ | PDF generation |
| markitdown | Latest | File parsing (PDF, DOCX, etc.) |
| uv | Latest | Fast Python package manager |

### Deployment Stack

| Technology | Purpose |
|-----------|---------|
| Docker | Containerization |
| Docker Compose | Multi-container orchestration |
| Nginx | Frontend web server (in production) |

## Project Structure

```
banana-slides/
├── frontend/                    # React frontend application
│   ├── src/
│   │   ├── pages/              # Page components (routing destinations)
│   │   │   ├── Home.tsx        # Project creation page
│   │   │   ├── OutlineEditor.tsx    # Outline editing page
│   │   │   ├── DetailEditor.tsx     # Description editing page
│   │   │   ├── SlidePreview.tsx     # Slide preview and editing page
│   │   │   └── History.tsx          # Project history page
│   │   ├── components/         # Reusable UI components
│   │   │   ├── shared/         # Shared components (Button, Modal, etc.)
│   │   │   ├── preview/        # Preview-related components
│   │   │   ├── outline/        # Outline-related components
│   │   │   ├── layout/         # Layout components
│   │   │   └── history/        # History version components
│   │   ├── store/              # Zustand state management
│   │   │   └── useProjectStore.ts  # Main project store
│   │   ├── api/                # API integration layer
│   │   │   ├── client.ts       # Axios client configuration
│   │   │   └── endpoints.ts    # API endpoint definitions
│   │   ├── types/              # TypeScript type definitions
│   │   ├── utils/              # Utility functions
│   │   └── constants/          # Constants and configuration
│   ├── public/                 # Static assets
│   ├── package.json            # npm dependencies
│   ├── vite.config.ts          # Vite configuration
│   ├── tailwind.config.js      # Tailwind CSS configuration
│   └── Dockerfile              # Frontend Docker image
│
├── backend/                    # Flask backend application
│   ├── app.py                  # Application entry point
│   ├── config.py               # Configuration management
│   ├── models/                 # Database models (SQLAlchemy)
│   │   ├── project.py          # Project model
│   │   ├── page.py             # Page (slide) model
│   │   ├── task.py             # Async task model
│   │   ├── material.py         # Reference material model
│   │   ├── user_template.py    # User template model
│   │   ├── reference_file.py   # Reference file model
│   │   └── page_image_version.py # Page version history model
│   ├── services/               # Business logic layer
│   │   ├── ai_service.py       # AI generation service
│   │   ├── ai_providers/       # AI provider implementations
│   │   ├── file_service.py     # File management service
│   │   ├── file_parser_service.py # File parsing service
│   │   ├── export_service.py   # PPTX/PDF export service
│   │   ├── task_manager.py     # Async task management
│   │   └── prompts.py          # AI prompt templates
│   ├── controllers/            # API endpoint handlers
│   │   ├── project_controller.py      # Project CRUD
│   │   ├── page_controller.py         # Page operations
│   │   ├── material_controller.py     # Material management
│   │   ├── template_controller.py     # Template management
│   │   ├── reference_file_controller.py # File upload/parsing
│   │   ├── export_controller.py       # Export operations
│   │   └── file_controller.py         # File serving
│   ├── utils/                  # Utility functions
│   │   ├── response.py         # API response helpers
│   │   ├── validators.py       # Input validation
│   │   └── path_utils.py       # Path utilities
│   ├── instance/               # SQLite database (auto-generated)
│   └── Dockerfile              # Backend Docker image
│
├── docs/                       # Documentation
│   ├── ARCHITECTURE.md         # This file
│   ├── BACKEND.md              # Backend documentation
│   ├── FRONTEND.md             # Frontend documentation
│   ├── API.md                  # API documentation
│   └── DEPLOYMENT.md           # Deployment guide
│
├── pyproject.toml              # Python project config (uv)
├── docker-compose.yml          # Docker Compose configuration
├── .env.example                # Environment variables template
└── README.md                   # Main project README
```

## Data Flow

### 1. Project Creation Flow

```
User Input (Home Page)
    ↓
Frontend: Create Project Request
    ↓ POST /api/projects
Backend: ProjectController.create_project()
    ↓
Create Project in Database
    ↓
If creation_type == 'idea':
    ↓
    AIService.generate_outline()
        ↓
    Create Pages from Outline
    ↓
Return Project ID
    ↓
Frontend: Navigate to OutlineEditor or DetailEditor
```

### 2. Image Generation Flow

```
User: Click "Generate Images"
    ↓
Frontend: POST /api/projects/{id}/generate-images
    ↓
Backend: Create Async Task
    ↓
TaskManager: generate_images_task()
    ↓
For each page (parallel processing):
    ↓
    AIService.generate_image()
        ↓
    Call Gemini/OpenAI API with:
        - Page description
        - Template image (optional)
        - Reference materials (optional)
        ↓
    Save generated image
    ↓
Update Page.image_url
    ↓
Task Complete
    ↓
Frontend: Poll task status
    ↓
Display generated slides
```

### 3. Export Flow

```
User: Click "Export PPTX" or "Export PDF"
    ↓
Frontend: POST /api/export/pptx or /api/export/pdf
    ↓
Backend: ExportController
    ↓
Collect all page images
    ↓
ExportService.create_pptx_from_images() or create_pdf_from_images()
    ↓
Generate file in exports/ directory
    ↓
Return file URL
    ↓
Frontend: Download file
```

## Key Components

### Backend Key Components

#### 1. AIService (`services/ai_service.py`)
- **Purpose**: Central AI integration service
- **Key Methods**:
  - `generate_outline()`: Generate outline from idea
  - `generate_page_description()`: Generate description for a page
  - `generate_image()`: Generate slide image using Gemini/OpenAI
  - `edit_image()`: Edit existing image with prompts
- **AI Providers**: Supports both Gemini (Google GenAI SDK) and OpenAI formats

#### 2. TaskManager (`services/task_manager.py`)
- **Purpose**: Manage long-running asynchronous tasks
- **Key Features**:
  - ThreadPoolExecutor for parallel processing
  - Task status tracking in database
  - Progress reporting
- **Common Tasks**:
  - `generate_descriptions_task`: Generate descriptions for all pages
  - `generate_images_task`: Generate images for all pages

#### 3. ExportService (`services/export_service.py`)
- **Purpose**: Export presentations to PPTX and PDF
- **Key Methods**:
  - `create_pptx_from_images()`: Create PowerPoint from images
  - `create_pdf_from_images()`: Create PDF from images
- **Library**: Uses python-pptx and ReportLab

#### 4. FileParserService (`services/file_parser_service.py`)
- **Purpose**: Parse uploaded files (PDF, DOCX, etc.)
- **Key Features**:
  - Uses markitdown library for local parsing
  - Optional MinerU API integration for advanced parsing
  - Extracts text and images from documents

### Frontend Key Components

#### 1. useProjectStore (`store/useProjectStore.ts`)
- **Purpose**: Central state management using Zustand
- **Key State**:
  - `currentProject`: Current project data
  - `isGlobalLoading`: Loading indicator
  - `activeTaskId`: Current async task
- **Key Actions**:
  - `initializeProject()`: Create new project
  - `generateOutline()`: Generate outline from idea
  - `generateDescriptions()`: Generate descriptions
  - `generateImages()`: Generate slide images
  - `exportPPTX()`, `exportPDF()`: Export presentations

#### 2. Pages
- **Home**: Project creation with 3 modes (idea, outline, description)
- **OutlineEditor**: Edit and refine outline structure
- **DetailEditor**: Edit page descriptions and materials
- **SlidePreview**: View and edit generated slides
- **History**: View project history

#### 3. API Client (`api/client.ts` and `api/endpoints.ts`)
- **Purpose**: Centralized API communication
- **Features**:
  - Axios-based HTTP client
  - Request/response interceptors
  - Type-safe endpoint definitions

## Database Schema

### Core Models

```
Project
├── id (UUID, PK)
├── idea_prompt (Text)
├── outline_text (Text)
├── description_text (Text)
├── creation_type (String: idea|outline|descriptions)
├── template_image_path (String)
├── status (String: DRAFT|COMPLETED)
├── created_at, updated_at (DateTime)
└── Relationships:
    ├── pages (One-to-Many)
    ├── tasks (One-to-Many)
    └── materials (One-to-Many)

Page
├── id (UUID, PK)
├── project_id (UUID, FK)
├── order_index (Integer)
├── part (String, optional - for grouping)
├── outline_content (JSON)
├── description_content (JSON)
├── image_url (String)
├── created_at, updated_at (DateTime)
└── Relationships:
    └── versions (One-to-Many PageImageVersion)

Task
├── id (UUID, PK)
├── project_id (UUID, FK)
├── task_type (String)
├── status (String: pending|running|completed|failed)
├── progress (JSON)
├── result (JSON)
└── created_at, updated_at (DateTime)

Material
├── id (UUID, PK)
├── project_id (UUID, FK)
├── filename (String)
├── file_path (String)
├── material_type (String)
└── created_at (DateTime)

ReferenceFile
├── id (UUID, PK)
├── project_id (UUID, FK)
├── filename (String)
├── file_path (String)
├── parse_status (String)
├── markdown_content (Text)
└── created_at (DateTime)
```

## Configuration

### Environment Variables

See `.env.example` for all available configuration options. Key variables:

- **AI Provider**: `AI_PROVIDER_FORMAT` (gemini or openai)
- **Gemini Config**: `GOOGLE_API_KEY`, `GOOGLE_API_BASE`
- **OpenAI Config**: `OPENAI_API_KEY`, `OPENAI_API_BASE`
- **Models**: `TEXT_MODEL`, `IMAGE_MODEL`
- **Server**: `PORT`, `FLASK_ENV`
- **CORS**: `CORS_ORIGINS`

## Security Considerations

1. **SQLite WAL Mode**: Enabled for better concurrent access
2. **File Upload**: Limited to 200MB, validated extensions
3. **CORS**: Configurable allowed origins
4. **API Keys**: Stored in environment variables, never in code
5. **Input Validation**: All user inputs validated before processing

## Performance Optimizations

1. **Parallel Processing**: Images and descriptions generated in parallel using ThreadPoolExecutor
2. **Debounced Updates**: Frontend debounces rapid state changes
3. **Lazy Loading**: Database relationships loaded only when needed
4. **Connection Pooling**: SQLAlchemy connection pool for database
5. **Static Asset Caching**: Nginx caches static assets in production

## Extensibility

The architecture supports easy extension:

1. **New AI Providers**: Add new provider in `services/ai_providers/`
2. **New Export Formats**: Extend `ExportService`
3. **New File Parsers**: Extend `FileParserService`
4. **New Page Types**: Extend `Page` model with new fields
5. **Custom Prompts**: Modify prompt templates in `services/prompts.py`
