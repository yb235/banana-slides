# Database Models

This document describes all database models and their relationships in Banana Slides.

## 📊 Database Overview

Banana Slides uses **SQLite** with **SQLAlchemy ORM** for data persistence. The database schema is designed to support:

- Multiple projects with version history
- Flexible page structure with parts/sections
- Async task tracking
- Material and template management
- Reference file parsing and storage

### Database Configuration

```python
# SQLite with WAL mode for better concurrency
PRAGMA journal_mode=WAL
PRAGMA synchronous=NORMAL
PRAGMA busy_timeout=30000
```

## 🗂️ Entity Relationship Diagram

```
┌─────────────┐
│   Project   │
└──────┬──────┘
       │
       │ 1:N
       │
       ├──────────────┬──────────────┬──────────────┐
       │              │              │              │
       ▼              ▼              ▼              ▼
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌────────────────┐
│   Page   │   │   Task   │   │ Material │   │ ReferenceFile  │
└─────┬────┘   └──────────┘   └──────────┘   └────────────────┘
      │
      │ 1:N
      │
      ▼
┌──────────────────┐
│ PageImageVersion │
└──────────────────┘

┌──────────────────┐
│  UserTemplate    │  (Independent)
└──────────────────┘
```

## 📋 Models

### Project

The main entity representing a PowerPoint presentation.

**Table:** `projects`

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | String(36) | UUID primary key |
| `idea_prompt` | Text | Original idea/prompt from user |
| `outline_text` | Text | User-provided outline text (for outline type) |
| `description_text` | Text | User-provided descriptions (for descriptions type) |
| `extra_requirements` | Text | Additional requirements for AI generation |
| `creation_type` | String(20) | "idea", "outline", or "descriptions" |
| `template_image_path` | String(500) | Path to template image file |
| `status` | String(50) | Project status (e.g., "DRAFT", "COMPLETE") |
| `created_at` | DateTime | Creation timestamp |
| `updated_at` | DateTime | Last update timestamp |

**Relationships:**
- `pages`: One-to-many with Page
- `tasks`: One-to-many with Task
- `materials`: One-to-many with Material

**Methods:**

```python
def to_dict(self, include_pages=False):
    """Convert to dictionary representation"""
    return {
        'project_id': self.id,
        'idea_prompt': self.idea_prompt,
        'outline_text': self.outline_text,
        'description_text': self.description_text,
        'creation_type': self.creation_type,
        'template_image_url': ...,
        'status': self.status,
        'pages': [...] if include_pages else None
    }
```

**Example:**

```python
project = Project(
    idea_prompt="Create a presentation about AI",
    creation_type="idea",
    status="DRAFT"
)
db.session.add(project)
db.session.commit()
```

---

### Page

Represents an individual slide/page in a presentation.

**Table:** `pages`

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | String(36) | UUID primary key |
| `project_id` | String(36) | Foreign key to Project |
| `order_index` | Integer | Position in presentation (0-based) |
| `part` | String(100) | Optional section/part name |
| `outline_content_json` | Text | JSON: {title, points} |
| `description_content_json` | Text | JSON: {title, content, images} |
| `image_url` | String(500) | URL to current slide image |
| `current_version_id` | String(36) | Foreign key to current PageImageVersion |
| `created_at` | DateTime | Creation timestamp |
| `updated_at` | DateTime | Last update timestamp |

**Relationships:**
- `project`: Many-to-one with Project
- `image_versions`: One-to-many with PageImageVersion

**Methods:**

```python
def get_outline_content(self):
    """Parse and return outline content"""
    if self.outline_content_json:
        return json.loads(self.outline_content_json)
    return None

def set_outline_content(self, content):
    """Set outline content from dict"""
    self.outline_content_json = json.dumps(content, ensure_ascii=False)

def get_description_content(self):
    """Parse and return description content"""
    if self.description_content_json:
        return json.loads(self.description_content_json)
    return None

def set_description_content(self, content):
    """Set description content from dict"""
    self.description_content_json = json.dumps(content, ensure_ascii=False)
```

**Outline Content Structure:**

```json
{
  "title": "Page Title",
  "points": [
    "Bullet point 1",
    "Bullet point 2",
    "Bullet point 3"
  ]
}
```

**Description Content Structure:**

```json
{
  "title": "Page Title",
  "content": "Detailed markdown content with **bold**, *italic*, etc.",
  "images": [
    {
      "description": "Chart showing growth trend",
      "position": "center",  // "top", "center", "bottom", "left", "right"
      "size": "large"        // "small", "medium", "large"
    }
  ]
}
```

---

### PageImageVersion

Tracks version history of page images for rollback functionality.

**Table:** `page_image_versions`

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | String(36) | UUID primary key |
| `page_id` | String(36) | Foreign key to Page |
| `version_number` | Integer | Sequential version number |
| `image_path` | String(500) | Path to image file |
| `edit_instruction` | Text | Natural language edit instruction (if any) |
| `generation_prompt` | Text | Full prompt used for generation |
| `is_current` | Boolean | Whether this is the active version |
| `created_at` | DateTime | Creation timestamp |

**Relationships:**
- `page`: Many-to-one with Page

**Methods:**

```python
def to_dict(self):
    """Convert to dictionary"""
    return {
        'version_id': self.id,
        'version_number': self.version_number,
        'image_url': f'/files/...',
        'edit_instruction': self.edit_instruction,
        'is_current': self.is_current,
        'created_at': self.created_at.isoformat()
    }
```

**Notes:**
- Each image generation/edit creates a new version
- Users can switch between versions
- Old versions are kept for history

---

### Task

Tracks asynchronous operations (generation, parsing, etc.).

**Table:** `tasks`

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | String(36) | UUID primary key |
| `project_id` | String(36) | Foreign key to Project |
| `task_type` | String(50) | Type of task (e.g., "generate_outline") |
| `status` | String(20) | "pending", "running", "completed", "failed" |
| `progress_message` | Text | Current progress description |
| `result_json` | Text | JSON result data (when completed) |
| `error_message` | Text | Error details (when failed) |
| `created_at` | DateTime | Creation timestamp |
| `updated_at` | DateTime | Last update timestamp |

**Relationships:**
- `project`: Many-to-one with Project

**Task Types:**
- `generate_outline`: Generate outline from idea
- `generate_descriptions`: Generate all page descriptions
- `generate_description`: Generate single page description
- `generate_images`: Generate all page images
- `generate_image`: Generate single page image
- `edit_image`: Edit page image
- `generate_material`: Generate material image
- `parse_file`: Parse reference file

**Methods:**

```python
def to_dict(self):
    """Convert to dictionary"""
    return {
        'task_id': self.id,
        'task_type': self.task_type,
        'status': self.status,
        'progress_message': self.progress_message,
        'result': json.loads(self.result_json) if self.result_json else None,
        'error': self.error_message,
        'created_at': self.created_at.isoformat()
    }

def update_status(self, status, progress=None, result=None, error=None):
    """Update task status"""
    self.status = status
    if progress:
        self.progress_message = progress
    if result:
        self.result_json = json.dumps(result, ensure_ascii=False)
    if error:
        self.error_message = error
    self.updated_at = datetime.utcnow()
```

---

### Material

Represents image materials (global or project-specific).

**Table:** `materials`

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | String(36) | UUID primary key |
| `project_id` | String(36) | Foreign key to Project (nullable for global) |
| `filename` | String(255) | Original filename |
| `relative_path` | String(500) | Relative path from uploads folder |
| `prompt` | Text | Generation prompt (if AI-generated) |
| `source_filename` | String(255) | Source filename (if uploaded) |
| `created_at` | DateTime | Creation timestamp |

**Relationships:**
- `project`: Many-to-one with Project (optional)

**Methods:**

```python
def to_dict(self):
    """Convert to dictionary"""
    return {
        'id': self.id,
        'project_id': self.project_id,
        'filename': self.filename,
        'url': f'/files/{self.relative_path}',
        'relative_path': self.relative_path,
        'prompt': self.prompt,
        'source_filename': self.source_filename,
        'created_at': self.created_at.isoformat()
    }
```

**Storage Structure:**
```
uploads/
├── materials/
│   ├── global/              # Global materials
│   │   └── uuid.png
│   └── {project_id}/        # Project-specific materials
│       └── uuid.png
```

---

### UserTemplate

Represents reusable template images.

**Table:** `user_templates`

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `template_id` | String(36) | UUID primary key |
| `name` | String(100) | Template name (optional) |
| `template_image_path` | String(500) | Path to template image |
| `created_at` | DateTime | Creation timestamp |
| `updated_at` | DateTime | Last update timestamp |

**No Relationships** (independent entity)

**Methods:**

```python
def to_dict(self):
    """Convert to dictionary"""
    return {
        'template_id': self.template_id,
        'name': self.name,
        'template_image_url': f'/files/user-templates/{...}',
        'created_at': self.created_at.isoformat(),
        'updated_at': self.updated_at.isoformat()
    }
```

---

### ReferenceFile

Represents uploaded documents for content extraction.

**Table:** `reference_files`

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | String(36) | UUID primary key |
| `project_id` | String(36) | Foreign key to Project (nullable for global) |
| `filename` | String(255) | Original filename |
| `file_path` | String(500) | Path to uploaded file |
| `file_size` | Integer | File size in bytes |
| `file_type` | String(100) | MIME type |
| `parse_status` | String(20) | "pending", "parsing", "completed", "failed" |
| `markdown_content` | Text | Extracted markdown content |
| `error_message` | Text | Error details (if parsing failed) |
| `created_at` | DateTime | Creation timestamp |
| `updated_at` | DateTime | Last update timestamp |

**No Direct Relationship** with Project (uses project_id field)

**Methods:**

```python
def to_dict(self):
    """Convert to dictionary"""
    return {
        'id': self.id,
        'project_id': self.project_id,
        'filename': self.filename,
        'file_size': self.file_size,
        'file_type': self.file_type,
        'parse_status': self.parse_status,
        'markdown_content': self.markdown_content,
        'error_message': self.error_message,
        'created_at': self.created_at.isoformat(),
        'updated_at': self.updated_at.isoformat()
    }
```

**Supported File Types:**
- PDF: `.pdf`
- Word: `.docx`, `.doc`
- PowerPoint: `.pptx`, `.ppt`
- Excel: `.xlsx`, `.xls`
- Text: `.txt`, `.md`
- CSV: `.csv`

---

## 🔄 Common Queries

### Get Project with All Pages

```python
project = Project.query.get(project_id)
pages = project.pages.order_by(Page.order_index).all()
```

### Get Page with Version History

```python
page = Page.query.get(page_id)
versions = page.image_versions.order_by(
    PageImageVersion.version_number.desc()
).all()
```

### Get Active Tasks for Project

```python
tasks = Task.query.filter_by(
    project_id=project_id,
    status='running'
).all()
```

### Get Global Materials

```python
materials = Material.query.filter_by(project_id=None).all()
```

### Get Completed Reference Files

```python
files = ReferenceFile.query.filter_by(
    project_id=project_id,
    parse_status='completed'
).all()
```

---

## 🗄️ Database Migrations

Currently, Banana Slides uses `db.create_all()` for schema creation, which works for SQLite but doesn't support migrations.

### Adding a New Field

1. Update the model class:
```python
class Project(db.Model):
    # ... existing fields
    new_field = db.Column(db.String(100), nullable=True)
```

2. Delete and recreate database (development only):
```bash
rm backend/instance/database.db
# Restart backend - tables are recreated
```

3. For production, consider using **Flask-Migrate** (Alembic):
```bash
pip install Flask-Migrate
flask db init
flask db migrate -m "Add new field"
flask db upgrade
```

---

## 💾 Backup and Restore

### Backup

```bash
# SQLite database is a single file
cp backend/instance/database.db backup/database_$(date +%Y%m%d).db

# Also backup uploads
tar -czf backup/uploads_$(date +%Y%m%d).tar.gz uploads/
```

### Restore

```bash
# Stop backend
docker compose down

# Restore database
cp backup/database_20240115.db backend/instance/database.db

# Restore uploads
tar -xzf backup/uploads_20240115.tar.gz

# Start backend
docker compose up -d
```

---

## 🔍 Database Inspection

### Using DB Browser for SQLite

1. Download [DB Browser for SQLite](https://sqlitebrowser.org/)
2. Open `backend/instance/database.db`
3. Browse tables, run queries, view data

### Using SQL Queries

```python
# In Flask shell
flask shell

>>> from models import db, Project, Page
>>> Project.query.count()
10
>>> Page.query.filter_by(project_id='abc-123').all()
[<Page 1>, <Page 2>, ...]
```

---

## 📊 Performance Considerations

### Indexing

Current indexes (auto-created by SQLAlchemy):
- Primary keys on all `id` fields
- Foreign keys on relationship fields

Consider adding indexes for:
```python
class Page(db.Model):
    # Add index for common query
    __table_args__ = (
        db.Index('idx_project_order', 'project_id', 'order_index'),
    )
```

### Query Optimization

```python
# Bad: N+1 query problem
project = Project.query.get(project_id)
for page in project.pages:
    print(page.image_url)  # Triggers separate query

# Good: Eager loading
project = Project.query.options(
    db.joinedload(Project.pages)
).get(project_id)
```

---

## 🔐 Data Integrity

### Cascading Deletes

When a project is deleted, all related data is automatically removed:

```python
class Project(db.Model):
    pages = db.relationship('Page', cascade='all, delete-orphan')
    tasks = db.relationship('Task', cascade='all, delete-orphan')
    materials = db.relationship('Material', cascade='all, delete-orphan')
```

This ensures:
- No orphaned records
- Clean database
- Automatic cleanup

---

**Next**: Learn about the [AI Service & Prompts](./06-ai-service.md) system.
