# Backend Documentation

## Table of Contents
- [Overview](#overview)
- [Directory Structure](#directory-structure)
- [Application Entry Point](#application-entry-point)
- [Configuration](#configuration)
- [Models](#models)
- [Services](#services)
- [Controllers](#controllers)
- [Database](#database)
- [AI Integration](#ai-integration)
- [File Handling](#file-handling)
- [Task Management](#task-management)
- [Export System](#export-system)

## Overview

The backend is built with Flask 3.0 and Python 3.10+, following a layered architecture:

```
Controllers (API Layer) → Services (Business Logic) → Models (Data Layer) → Database
```

### Key Features
- RESTful API design
- Async task management for long-running operations
- Pluggable AI providers (Gemini/OpenAI)
- File upload and parsing
- PPTX and PDF export
- SQLite with WAL mode for concurrent access

## Directory Structure

```
backend/
├── app.py                      # Application factory and entry point
├── config.py                   # Configuration management
├── models/                     # SQLAlchemy ORM models
│   ├── __init__.py            # Database initialization
│   ├── project.py             # Project model
│   ├── page.py                # Page (slide) model
│   ├── task.py                # Async task model
│   ├── material.py            # Material model
│   ├── user_template.py       # User template model
│   ├── reference_file.py      # Reference file model
│   └── page_image_version.py  # Page version history
├── services/                   # Business logic layer
│   ├── ai_service.py          # AI generation service
│   ├── ai_providers/          # AI provider implementations
│   │   ├── __init__.py
│   │   ├── base.py            # Base provider interface
│   │   ├── gemini_provider.py # Google Gemini provider
│   │   └── openai_provider.py # OpenAI provider
│   ├── file_service.py        # File management
│   ├── file_parser_service.py # File parsing
│   ├── export_service.py      # PPTX/PDF export
│   ├── task_manager.py        # Async task management
│   └── prompts.py             # AI prompt templates
├── controllers/                # API endpoints
│   ├── __init__.py
│   ├── project_controller.py
│   ├── page_controller.py
│   ├── material_controller.py
│   ├── template_controller.py
│   ├── reference_file_controller.py
│   ├── export_controller.py
│   └── file_controller.py
├── utils/                      # Utility functions
│   ├── response.py            # Response helpers
│   ├── validators.py          # Input validation
│   └── path_utils.py          # Path utilities
└── instance/                   # Runtime data
    └── database.db            # SQLite database (auto-created)
```

## Application Entry Point

### app.py

The main application is created using the application factory pattern:

```python
def create_app():
    """Application factory"""
    app = Flask(__name__)
    
    # Load configuration
    app.config.from_object(Config)
    
    # Initialize database
    db.init_app(app)
    
    # Enable CORS
    CORS(app, origins=cors_origins)
    
    # Register blueprints (controllers)
    app.register_blueprint(project_bp)
    app.register_blueprint(page_bp)
    # ... more blueprints
    
    # Create database tables
    with app.app_context():
        db.create_all()
    
    return app
```

**Key Features**:
- Loads environment variables from `.env` file in project root
- Enables SQLite WAL mode for concurrent access
- Configures logging to stdout for Docker compatibility
- Registers all API blueprints
- Health check endpoint at `/health`

## Configuration

### config.py

Configuration is managed using environment variables with sensible defaults:

```python
class Config:
    # Database
    SQLALCHEMY_DATABASE_URI = 'sqlite:///instance/database.db'
    SQLALCHEMY_ENGINE_OPTIONS = {
        'connect_args': {'check_same_thread': False},
        'pool_pre_ping': True,
    }
    
    # AI Configuration
    AI_PROVIDER_FORMAT = 'gemini'  # or 'openai'
    GOOGLE_API_KEY = os.getenv('GOOGLE_API_KEY', '')
    OPENAI_API_KEY = os.getenv('OPENAI_API_KEY', '')
    TEXT_MODEL = 'gemini-2.5-flash'
    IMAGE_MODEL = 'gemini-3-pro-image-preview'
    
    # File Upload
    UPLOAD_FOLDER = 'uploads/'
    MAX_CONTENT_LENGTH = 200 * 1024 * 1024  # 200MB
    
    # Concurrency
    MAX_DESCRIPTION_WORKERS = 5
    MAX_IMAGE_WORKERS = 8
```

**Environment-Specific Configs**:
- `DevelopmentConfig`: DEBUG=True
- `ProductionConfig`: DEBUG=False

## Models

All models inherit from `db.Model` (Flask-SQLAlchemy).

### Project Model (`models/project.py`)

Represents a PPT project.

**Fields**:
- `id`: UUID primary key
- `idea_prompt`: User's initial idea (for 'idea' type)
- `outline_text`: User-provided outline (for 'outline' type)
- `description_text`: User-provided descriptions (for 'description' type)
- `creation_type`: 'idea', 'outline', or 'descriptions'
- `template_image_path`: Path to template image
- `status`: 'DRAFT' or 'COMPLETED'
- `created_at`, `updated_at`: Timestamps

**Relationships**:
- `pages`: One-to-many with Page
- `tasks`: One-to-many with Task
- `materials`: One-to-many with Material

**Key Methods**:
```python
def to_dict(self, include_pages=False):
    """Convert to dictionary for API responses"""
```

### Page Model (`models/page.py`)

Represents a single slide in a presentation.

**Fields**:
- `id`: UUID primary key
- `project_id`: Foreign key to Project
- `order_index`: Integer for ordering
- `part`: Optional string for grouping (e.g., "Introduction", "Main Content")
- `outline_content`: JSON field storing outline data
- `description_content`: JSON field storing description data
- `image_url`: URL/path to generated image
- `created_at`, `updated_at`: Timestamps

**JSON Field Structures**:

`outline_content`:
```json
{
  "title": "Page Title",
  "subtitle": "Optional subtitle",
  "key_points": ["Point 1", "Point 2"]
}
```

`description_content`:
```json
{
  "layout": "Title and Content",
  "text_elements": [
    {"type": "title", "content": "..."},
    {"type": "body", "content": "..."}
  ],
  "image_suggestions": ["description 1", "description 2"],
  "color_scheme": "blue and white"
}
```

**Relationships**:
- `project`: Many-to-one with Project
- `versions`: One-to-many with PageImageVersion

**Key Methods**:
```python
def get_outline_content(self):
    """Parse outline_content JSON"""
    
def get_description_content(self):
    """Parse description_content JSON"""
```

### Task Model (`models/task.py`)

Represents an asynchronous task.

**Fields**:
- `id`: UUID primary key
- `project_id`: Foreign key to Project
- `task_type`: 'generate_descriptions', 'generate_images', etc.
- `status`: 'pending', 'running', 'completed', 'failed'
- `progress`: JSON field with progress info
- `result`: JSON field with task results
- `error_message`: Error message if failed
- `created_at`, `updated_at`: Timestamps

**Progress Structure**:
```json
{
  "total": 10,
  "completed": 3,
  "message": "Generating images..."
}
```

### Material Model (`models/material.py`)

Represents uploaded reference materials.

**Fields**:
- `id`: UUID primary key
- `project_id`: Foreign key to Project
- `filename`: Original filename
- `file_path`: Absolute path to file
- `material_type`: 'image', 'document', etc.
- `created_at`: Timestamp

### ReferenceFile Model (`models/reference_file.py`)

Represents uploaded reference documents.

**Fields**:
- `id`: UUID primary key
- `project_id`: Foreign key to Project
- `filename`: Original filename
- `file_path`: Absolute path to file
- `parse_status`: 'pending', 'parsing', 'completed', 'failed'
- `markdown_content`: Parsed content in Markdown
- `created_at`: Timestamp

### PageImageVersion Model (`models/page_image_version.py`)

Stores version history of page images.

**Fields**:
- `id`: UUID primary key
- `page_id`: Foreign key to Page
- `image_url`: URL/path to image
- `edit_prompt`: Prompt used for this version
- `version_number`: Integer version number
- `created_at`: Timestamp

## Services

Services contain the business logic and interact with external APIs.

### AIService (`services/ai_service.py`)

Central service for AI-powered generation.

**Key Methods**:

#### 1. `generate_outline(project_context)`
Generates outline from idea prompt.

```python
outline = ai_service.generate_outline(project_context)
# Returns: List of outline items
# [
#   {"part": "Introduction", "pages": [{"title": "...", "key_points": [...]}]},
#   {"title": "...", "key_points": [...]}
# ]
```

#### 2. `generate_page_description(page_outline, project_context)`
Generates detailed description for a page from outline.

```python
description = ai_service.generate_page_description(
    page_outline,
    project_context,
    template_image_url=None,
    reference_images=[]
)
# Returns: Description JSON with layout, text_elements, etc.
```

#### 3. `generate_image(description, project_context)`
Generates slide image from description.

```python
image_data = ai_service.generate_image(
    description=page_description,
    project_context=project_context,
    template_image_url=None,
    additional_context_images=[]
)
# Returns: bytes of generated image
```

#### 4. `edit_image(current_image_url, edit_prompt, project_context)`
Edits existing image based on prompt.

```python
new_image_data = ai_service.edit_image(
    current_image_url="path/to/image.png",
    edit_prompt="Change background to blue",
    project_context=project_context,
    template_image_url=None,
    additional_context_images=[]
)
# Returns: bytes of edited image
```

**AI Provider Architecture**:

The service uses pluggable AI providers via the factory pattern:

```python
# Get providers based on AI_PROVIDER_FORMAT env var
text_provider = get_text_provider(model="gemini-2.5-flash")
image_provider = get_image_provider(model="gemini-3-pro-image-preview")
```

**Supported Providers**:
- **GeminiProvider**: Uses Google's GenAI SDK
- **OpenAIProvider**: Uses OpenAI SDK (compatible with OpenAI-like APIs)

### TaskManager (`services/task_manager.py`)

Manages asynchronous tasks using ThreadPoolExecutor.

**Key Features**:
- Thread pool for parallel execution
- Task status tracking in database
- Progress reporting
- Error handling and logging

**Common Task Functions**:

#### 1. `generate_descriptions_task(project_id)`
Generates descriptions for all pages without descriptions.

```python
def generate_descriptions_task(project_id: str):
    # Get project and pages
    # For each page without description:
    #   - Generate description using AIService
    #   - Update page in database
    #   - Report progress
    # Mark task as completed
```

#### 2. `generate_images_task(project_id)`
Generates images for all pages without images.

```python
def generate_images_task(project_id: str):
    # Get project and pages
    # For each page without image:
    #   - Generate image using AIService
    #   - Save image to disk
    #   - Update page.image_url
    #   - Report progress
    # Mark task as completed
```

**Usage Pattern**:
```python
# Create task in database
task = Task(project_id=project_id, task_type='generate_images')
db.session.add(task)
db.session.commit()

# Submit to thread pool
future = task_manager.submit_task(
    generate_images_task,
    project_id=project_id,
    task_id=task.id
)

# Frontend polls task status via API
```

### ExportService (`services/export_service.py`)

Handles PPTX and PDF export.

#### 1. `create_pptx_from_images(image_paths)`
Creates PowerPoint from images.

```python
pptx_bytes = ExportService.create_pptx_from_images(
    image_paths=['/path/to/slide1.png', '/path/to/slide2.png']
)
# Returns: bytes of PPTX file
```

**Implementation**:
- Uses python-pptx library
- Sets 16:9 aspect ratio
- Each image becomes a full-slide
- Images fitted to slide dimensions

#### 2. `create_pdf_from_images(image_paths)`
Creates PDF from images.

```python
pdf_bytes = ExportService.create_pdf_from_images(
    image_paths=['/path/to/slide1.png', '/path/to/slide2.png']
)
# Returns: bytes of PDF file
```

**Implementation**:
- Uses ReportLab and Pillow
- Maintains 16:9 aspect ratio
- High-quality image rendering

### FileParserService (`services/file_parser_service.py`)

Parses uploaded reference files.

**Supported Formats**:
- PDF
- DOCX, DOC
- PPTX, PPT
- XLSX, XLS
- TXT, MD
- Images (extracts with captions)

**Key Methods**:

#### `parse_reference_file(file_path, filename)`
Parses file and returns markdown content.

```python
markdown_content = FileParserService.parse_reference_file(
    file_path='/path/to/document.pdf',
    filename='document.pdf'
)
# Returns: Markdown string with extracted content
```

**Parsing Strategies**:
1. **Local Parsing** (default): Uses markitdown library
2. **MinerU API** (optional): For advanced parsing with better table/image extraction

**Configuration**:
- Set `MINERU_TOKEN` and `MINERU_API_BASE` to enable MinerU
- Falls back to local parsing if MinerU fails

### FileService (`services/file_service.py`)

Manages file uploads and storage.

**Key Features**:
- Secure filename generation
- Directory management
- File validation
- Path resolution

**Key Methods**:
```python
def save_uploaded_file(file, project_id, file_type):
    """Save uploaded file and return path"""
    
def delete_file(file_path):
    """Delete file from disk"""
    
def get_file_url(file_path, project_id):
    """Generate public URL for file"""
```

## Controllers

Controllers handle HTTP requests and responses.

### ProjectController (`controllers/project_controller.py`)

**Endpoints**:

#### `POST /api/projects`
Create new project.

**Request Body**:
```json
{
  "creation_type": "idea",
  "idea_prompt": "Create a presentation about AI",
  "outline_text": null,
  "description_text": null
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "project_id": "uuid",
    "status": "DRAFT",
    "pages": [...]
  }
}
```

#### `GET /api/projects`
List all projects.

**Query Parameters**:
- `limit`: Number of results (default: 50)
- `offset`: Offset for pagination (default: 0)

#### `GET /api/projects/{project_id}`
Get project details.

#### `PUT /api/projects/{project_id}`
Update project.

#### `DELETE /api/projects/{project_id}`
Delete project.

#### `POST /api/projects/{project_id}/generate-outline`
Generate outline from idea.

#### `POST /api/projects/{project_id}/generate-descriptions`
Generate descriptions for all pages.

#### `POST /api/projects/{project_id}/generate-images`
Generate images for all pages.

### PageController (`controllers/page_controller.py`)

**Endpoints**:

#### `POST /api/projects/{project_id}/pages`
Add new page to project.

#### `PUT /api/pages/{page_id}`
Update page data.

#### `DELETE /api/pages/{page_id}`
Delete page.

#### `POST /api/pages/{page_id}/generate-description`
Generate description for single page.

#### `POST /api/pages/{page_id}/generate-image`
Generate image for single page.

#### `POST /api/pages/{page_id}/edit-image`
Edit page image with prompt.

**Request Body** (edit-image):
```json
{
  "edit_prompt": "Change background to blue",
  "use_template": true,
  "desc_image_urls": ["url1", "url2"],
  "uploaded_files": [/* FormData files */]
}
```

### ExportController (`controllers/export_controller.py`)

**Endpoints**:

#### `POST /api/export/pptx`
Export project as PPTX.

**Request Body**:
```json
{
  "project_id": "uuid"
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "download_url": "/api/export/download/filename.pptx"
  }
}
```

#### `POST /api/export/pdf`
Export project as PDF.

#### `GET /api/export/download/{filename}`
Download exported file.

### MaterialController (`controllers/material_controller.py`)

**Endpoints**:

#### `POST /api/projects/{project_id}/materials`
Upload material for project.

#### `GET /api/projects/{project_id}/materials`
List project materials.

#### `DELETE /api/materials/{material_id}`
Delete material.

#### `POST /api/materials/generate`
Generate material using AI.

### ReferenceFileController (`controllers/reference_file_controller.py`)

**Endpoints**:

#### `POST /api/reference-files/upload`
Upload reference file.

#### `POST /api/reference-files/{file_id}/parse`
Trigger file parsing.

#### `GET /api/reference-files/{file_id}`
Get file details and parsed content.

#### `POST /api/reference-files/{file_id}/associate`
Associate file with project.

## Database

### SQLite Configuration

**WAL Mode** (Write-Ahead Logging):
- Enabled for better concurrent access
- Multiple readers don't block writers
- Better performance under concurrent load

**Connection Settings**:
```python
SQLALCHEMY_ENGINE_OPTIONS = {
    'connect_args': {
        'check_same_thread': False,  # Allow cross-thread usage
        'timeout': 30
    },
    'pool_pre_ping': True,  # Check connection before use
    'pool_recycle': 3600,   # Recycle connections every hour
}
```

**Migrations**:
- Database tables created automatically on startup
- Schema changes require manual migration scripts

### Best Practices

1. **Session Management**: Always use `db.session` within request context
2. **Transactions**: Wrap multiple operations in try-except blocks
3. **Relationships**: Use lazy loading to avoid N+1 queries
4. **Indexing**: Add indexes on frequently queried fields

## AI Integration

### Prompt Engineering

Prompts are defined in `services/prompts.py`:

**Key Prompts**:
1. **Outline Generation**: `get_outline_generation_prompt()`
2. **Page Description**: `get_page_description_prompt()`
3. **Image Generation**: `get_image_generation_prompt()`
4. **Image Editing**: `get_image_edit_prompt()`
5. **Outline Refinement**: `get_outline_refinement_prompt()`

**Example Prompt Structure**:
```python
def get_image_generation_prompt(description, extra_requirements=None):
    prompt = f"""
    Generate a PowerPoint slide image based on:
    {description}
    
    Requirements:
    - 16:9 aspect ratio
    - Professional design
    - Clear typography
    {extra_requirements or ''}
    """
    return prompt
```

### AI Provider Interface

Base interface in `services/ai_providers/base.py`:

```python
class TextProvider(ABC):
    @abstractmethod
    def generate_text(self, prompt: str, **kwargs) -> str:
        pass

class ImageProvider(ABC):
    @abstractmethod
    def generate_image(self, prompt: str, reference_images: List, **kwargs) -> bytes:
        pass
```

## Error Handling

### Response Format

**Success Response**:
```json
{
  "success": true,
  "data": {...}
}
```

**Error Response**:
```json
{
  "success": false,
  "error": "Error message"
}
```

### Common Errors

- `400 Bad Request`: Invalid input data
- `404 Not Found`: Resource not found
- `500 Internal Server Error`: Server error

### Logging

Uses Python's `logging` module:

```python
logger = logging.getLogger(__name__)
logger.info("Info message")
logger.error("Error message", exc_info=True)
```

**Log Levels**:
- INFO: General information
- WARNING: Warning messages
- ERROR: Error messages with traceback
- DEBUG: Detailed debugging (development only)

## Performance Considerations

1. **Parallel Processing**: Use ThreadPoolExecutor for independent tasks
2. **Caching**: Consider caching AI responses for identical inputs
3. **Connection Pooling**: SQLAlchemy manages connection pool
4. **File Cleanup**: Regularly clean up old export files
5. **Image Optimization**: Consider compressing generated images

## Testing

Currently, the backend has minimal automated tests. Recommended test coverage:

1. **Unit Tests**: Test individual functions in services
2. **Integration Tests**: Test API endpoints
3. **Database Tests**: Test model relationships and queries
4. **AI Tests**: Mock AI providers for consistent testing

## Deployment

### Development
```bash
cd backend
uv run python app.py
```

### Production with Docker
```bash
docker compose up -d backend
```

### Environment Variables
See `.env.example` for required variables.

## Troubleshooting

### Common Issues

**1. Database Locked**
- Ensure WAL mode is enabled
- Check for long-running transactions
- Increase `busy_timeout`

**2. AI API Errors**
- Verify API keys in `.env`
- Check API rate limits
- Review model names

**3. File Upload Errors**
- Check `MAX_CONTENT_LENGTH` setting
- Verify upload folder permissions
- Check disk space

**4. Memory Issues**
- Limit concurrent workers
- Optimize image sizes
- Monitor thread pool size
