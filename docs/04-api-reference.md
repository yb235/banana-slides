# Backend API Reference

Complete reference for all Banana Slides API endpoints.

## 📋 Table of Contents

- [General Information](#general-information)
- [Project Management](#project-management)
- [Page Operations](#page-operations)
- [Generation Endpoints](#generation-endpoints)
- [Refinement Endpoints](#refinement-endpoints)
- [Material Management](#material-management)
- [Template Management](#template-management)
- [Reference Files](#reference-files)
- [Export Operations](#export-operations)
- [Task Status](#task-status)
- [File Serving](#file-serving)

## General Information

### Base URL

```
http://localhost:5000
```

### Response Format

All API responses follow this structure:

**Success Response:**
```json
{
  "success": true,
  "message": "Operation successful",
  "data": { ... }
}
```

**Error Response:**
```json
{
  "success": false,
  "message": "Error description",
  "error": "Detailed error message"
}
```

### Common Status Codes

- `200 OK`: Successful request
- `201 Created`: Resource created successfully
- `400 Bad Request`: Invalid input
- `404 Not Found`: Resource not found
- `500 Internal Server Error`: Server error

## Project Management

### Create Project

Creates a new PPT project.

**Endpoint:** `POST /api/projects`

**Request Body:**
```json
{
  "creation_type": "idea",  // "idea" | "outline" | "descriptions"
  "idea_prompt": "Create a presentation about climate change",
  "outline_text": null,     // For outline type
  "description_text": null  // For descriptions type
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "project_id": "abc-123-def-456",
    "creation_type": "idea",
    "status": "DRAFT",
    "created_at": "2024-01-15T10:30:00Z"
  }
}
```

**Notes:**
- For `creation_type="idea"`, automatically starts outline generation task
- For `creation_type="outline"`, parses outline_text and creates pages immediately
- For `creation_type="descriptions"`, parses and creates pages with descriptions

---

### Get Project

Retrieves project details with all pages.

**Endpoint:** `GET /api/projects/:projectId`

**Response:**
```json
{
  "success": true,
  "data": {
    "project_id": "abc-123",
    "idea_prompt": "Create a presentation about climate change",
    "creation_type": "idea",
    "status": "DRAFT",
    "template_image_url": "/files/abc-123/template/image.png",
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-15T10:35:00Z",
    "pages": [
      {
        "page_id": "page-1",
        "order_index": 0,
        "part": null,
        "outline_content": {
          "title": "Introduction",
          "points": ["Point 1", "Point 2"]
        },
        "description_content": {
          "title": "Introduction",
          "content": "Detailed content...",
          "images": []
        },
        "image_url": "/files/abc-123/pages/page-1/v1.png"
      }
    ]
  }
}
```

---

### List Projects

Lists all projects (most recent first).

**Endpoint:** `GET /api/projects`

**Query Parameters:**
- `limit` (optional): Number of projects to return (default: 50)
- `offset` (optional): Number of projects to skip (default: 0)

**Response:**
```json
{
  "success": true,
  "data": {
    "projects": [
      {
        "project_id": "abc-123",
        "idea_prompt": "Create a presentation...",
        "creation_type": "idea",
        "status": "DRAFT",
        "created_at": "2024-01-15T10:30:00Z",
        "updated_at": "2024-01-15T10:35:00Z"
      }
    ],
    "total": 100
  }
}
```

---

### Update Project

Updates project properties.

**Endpoint:** `PUT /api/projects/:projectId`

**Request Body:**
```json
{
  "idea_prompt": "Updated prompt",
  "extra_requirements": "Make it professional",
  "pages_order": ["page-2", "page-1", "page-3"]  // Reorder pages
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "project_id": "abc-123",
    // ... updated project data
  }
}
```

---

### Delete Project

Deletes a project and all associated data.

**Endpoint:** `DELETE /api/projects/:projectId`

**Response:**
```json
{
  "success": true,
  "message": "Project deleted successfully"
}
```

**Notes:**
- Cascading delete removes all pages, tasks, materials, and files
- Physical files are also deleted from disk

---

### Upload Template

Uploads a template image for the project.

**Endpoint:** `POST /api/projects/:projectId/template`

**Request:** multipart/form-data
- `template_image`: Image file (PNG, JPG, JPEG, GIF, WEBP)

**Response:**
```json
{
  "success": true,
  "data": {
    "template_image_url": "/files/abc-123/template/image.png"
  }
}
```

## Page Operations

### Add Page

Adds a new page to the project.

**Endpoint:** `POST /api/projects/:projectId/pages`

**Request Body:**
```json
{
  "order_index": 2,
  "part": "Introduction",  // Optional
  "outline_content": {
    "title": "New Page",
    "points": ["Point 1", "Point 2"]
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "page_id": "page-new",
    "order_index": 2,
    // ... page data
  }
}
```

---

### Update Page

Updates page properties.

**Endpoint:** `PUT /api/projects/:projectId/pages/:pageId`

**Request Body:**
```json
{
  "order_index": 1,
  "part": "Main Content"
}
```

---

### Update Page Outline

Updates page outline content.

**Endpoint:** `PUT /api/projects/:projectId/pages/:pageId/outline`

**Request Body:**
```json
{
  "outline_content": {
    "title": "Updated Title",
    "points": ["New point 1", "New point 2"]
  }
}
```

---

### Update Page Description

Updates page description content.

**Endpoint:** `PUT /api/projects/:projectId/pages/:pageId/description`

**Request Body:**
```json
{
  "description_content": {
    "title": "Page Title",
    "content": "Detailed markdown content...",
    "images": [
      {
        "description": "Chart showing data",
        "position": "center",
        "size": "large"
      }
    ]
  }
}
```

---

### Delete Page

Deletes a page from the project.

**Endpoint:** `DELETE /api/projects/:projectId/pages/:pageId`

**Response:**
```json
{
  "success": true,
  "message": "Page deleted successfully"
}
```

## Generation Endpoints

### Generate Outline

Generates outline from project idea.

**Endpoint:** `POST /api/projects/:projectId/generate/outline`

**Request Body:** `{}`

**Response:**
```json
{
  "success": true,
  "data": {
    "task_id": "task-123",
    "status": "running"
  }
}
```

**Notes:**
- Async operation, poll task status for completion
- Uses idea_prompt and reference files

---

### Generate Descriptions

Generates detailed descriptions for all pages.

**Endpoint:** `POST /api/projects/:projectId/generate/descriptions`

**Request Body:** `{}`

**Response:**
```json
{
  "success": true,
  "data": {
    "task_id": "task-456",
    "status": "running"
  }
}
```

**Notes:**
- Processes pages in parallel (MAX_DESCRIPTION_WORKERS)
- Generates description_content for each page

---

### Generate Single Page Description

Generates description for a specific page.

**Endpoint:** `POST /api/projects/:projectId/pages/:pageId/generate/description`

**Request Body:**
```json
{
  "force_regenerate": false
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "task_id": "task-789",
    "status": "running"
  }
}
```

---

### Generate Images

Generates images for all pages with descriptions.

**Endpoint:** `POST /api/projects/:projectId/generate/images`

**Request Body:** `{}`

**Response:**
```json
{
  "success": true,
  "data": {
    "task_id": "task-111",
    "status": "running"
  }
}
```

**Notes:**
- Processes pages in parallel (MAX_IMAGE_WORKERS)
- Uses nano banana pro or DALL-E
- Creates version history

---

### Generate Single Page Image

Generates image for a specific page.

**Endpoint:** `POST /api/projects/:projectId/pages/:pageId/generate/image`

**Request Body:**
```json
{
  "force_regenerate": false
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "task_id": "task-222",
    "status": "running"
  }
}
```

---

### Edit Page Image

Edits an existing page image using natural language.

**Endpoint:** `POST /api/projects/:projectId/pages/:pageId/edit/image`

**Request Body (JSON):**
```json
{
  "edit_instruction": "Change background to blue",
  "context_images": {
    "use_template": true,
    "desc_image_urls": ["/files/image1.png"]
  }
}
```

**Request Body (multipart/form-data):**
```
edit_instruction: "Change background to blue"
use_template: "true"
desc_image_urls: ["url1", "url2"]
context_images: [File1, File2]  // Uploaded files
```

**Response:**
```json
{
  "success": true,
  "data": {
    "task_id": "task-333",
    "status": "running"
  }
}
```

## Refinement Endpoints

### Refine Outline

Modifies outline based on natural language instruction.

**Endpoint:** `POST /api/projects/:projectId/refine/outline`

**Request Body:**
```json
{
  "user_requirement": "Add a conclusion page and make it more technical",
  "previous_requirements": ["Previous instruction 1"]
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "pages": [
      {
        "page_id": "page-1",
        "outline_content": { ... }
      }
    ],
    "message": "Outline refined successfully"
  }
}
```

---

### Refine Descriptions

Modifies descriptions based on natural language instruction.

**Endpoint:** `POST /api/projects/:projectId/refine/descriptions`

**Request Body:**
```json
{
  "user_requirement": "Make all content more concise and add statistics",
  "previous_requirements": []
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "pages": [
      {
        "page_id": "page-1",
        "description_content": { ... }
      }
    ],
    "message": "Descriptions refined successfully"
  }
}
```

## Material Management

### Generate Material

Generates a new material image.

**Endpoint:** `POST /api/projects/:projectId/materials/generate`

**Request:** multipart/form-data
- `prompt`: Text prompt for generation
- `ref_image` (optional): Reference image file
- `extra_images` (optional): Additional reference images

**Response:**
```json
{
  "success": true,
  "data": {
    "task_id": "task-444",
    "status": "running"
  }
}
```

---

### Upload Material

Uploads a material image.

**Endpoint:** `POST /api/projects/:projectId/materials/upload`

OR (for global materials):

**Endpoint:** `POST /api/materials/upload`

**Request:** multipart/form-data
- `file`: Image file

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "material-123",
    "project_id": "abc-123",
    "filename": "image.png",
    "url": "/files/materials/image.png",
    "created_at": "2024-01-15T10:30:00Z"
  }
}
```

---

### List Materials

Lists materials for a project or globally.

**Endpoint:** `GET /api/projects/:projectId/materials`

OR:

**Endpoint:** `GET /api/materials?project_id=all|none`

**Response:**
```json
{
  "success": true,
  "data": {
    "materials": [
      {
        "id": "material-123",
        "filename": "image.png",
        "url": "/files/materials/image.png",
        "created_at": "2024-01-15T10:30:00Z"
      }
    ],
    "count": 10
  }
}
```

---

### Delete Material

Deletes a material.

**Endpoint:** `DELETE /api/materials/:materialId`

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "material-123"
  }
}
```

## Template Management

### Upload User Template

Uploads a reusable template.

**Endpoint:** `POST /api/user-templates`

**Request:** multipart/form-data
- `template_image`: Image file
- `name` (optional): Template name

**Response:**
```json
{
  "success": true,
  "data": {
    "template_id": "template-123",
    "name": "My Template",
    "template_image_url": "/files/templates/image.png",
    "created_at": "2024-01-15T10:30:00Z"
  }
}
```

---

### List User Templates

Lists all user templates.

**Endpoint:** `GET /api/user-templates`

**Response:**
```json
{
  "success": true,
  "data": {
    "templates": [
      {
        "template_id": "template-123",
        "name": "My Template",
        "template_image_url": "/files/templates/image.png",
        "created_at": "2024-01-15T10:30:00Z"
      }
    ]
  }
}
```

---

### Delete User Template

Deletes a user template.

**Endpoint:** `DELETE /api/user-templates/:templateId`

**Response:**
```json
{
  "success": true,
  "message": "Template deleted successfully"
}
```

## Reference Files

### Upload Reference File

Uploads a document for parsing.

**Endpoint:** `POST /api/reference-files/upload`

**Request:** multipart/form-data
- `file`: Document file (PDF, DOCX, etc.)
- `project_id` (optional): Associate with project

**Response:**
```json
{
  "success": true,
  "data": {
    "file": {
      "id": "file-123",
      "project_id": "abc-123",
      "filename": "document.pdf",
      "file_size": 1048576,
      "file_type": "application/pdf",
      "parse_status": "pending",
      "created_at": "2024-01-15T10:30:00Z"
    }
  }
}
```

**Notes:**
- Parsing starts automatically in the background
- Check parse_status: pending → parsing → completed/failed

---

### Get Reference File

Gets reference file details.

**Endpoint:** `GET /api/reference-files/:fileId`

**Response:**
```json
{
  "success": true,
  "data": {
    "file": {
      "id": "file-123",
      "filename": "document.pdf",
      "parse_status": "completed",
      "markdown_content": "# Document Content\n\n...",
      "error_message": null
    }
  }
}
```

---

### List Project Reference Files

Lists files for a project.

**Endpoint:** `GET /api/reference-files/project/:projectId`

OR (for global files):

**Endpoint:** `GET /api/reference-files/project/global`

**Response:**
```json
{
  "success": true,
  "data": {
    "files": [
      {
        "id": "file-123",
        "filename": "document.pdf",
        "parse_status": "completed",
        "created_at": "2024-01-15T10:30:00Z"
      }
    ]
  }
}
```

---

### Delete Reference File

Deletes a reference file.

**Endpoint:** `DELETE /api/reference-files/:fileId`

**Response:**
```json
{
  "success": true,
  "message": "File deleted successfully"
}
```

---

### Trigger File Parse

Manually triggers file parsing.

**Endpoint:** `POST /api/reference-files/:fileId/parse`

**Response:**
```json
{
  "success": true,
  "data": {
    "file": { ... },
    "message": "Parsing started"
  }
}
```

## Export Operations

### Export PPTX

Exports project as PowerPoint file.

**Endpoint:** `GET /api/projects/:projectId/export/pptx`

**Response:**
```json
{
  "success": true,
  "data": {
    "download_url": "/files/exports/abc-123.pptx",
    "download_url_absolute": "http://localhost:5000/files/exports/abc-123.pptx"
  }
}
```

**Notes:**
- File is generated on-demand
- Each slide is added as an image
- 16:9 aspect ratio

---

### Export PDF

Exports project as PDF file.

**Endpoint:** `GET /api/projects/:projectId/export/pdf`

**Response:**
```json
{
  "success": true,
  "data": {
    "download_url": "/files/exports/abc-123.pdf",
    "download_url_absolute": "http://localhost:5000/files/exports/abc-123.pdf"
  }
}
```

## Task Status

### Get Task Status

Gets the status of an async task.

**Endpoint:** `GET /api/projects/:projectId/tasks/:taskId`

**Response (Running):**
```json
{
  "success": true,
  "data": {
    "task_id": "task-123",
    "status": "running",
    "progress": "Processing page 2 of 5"
  }
}
```

**Response (Completed):**
```json
{
  "success": true,
  "data": {
    "task_id": "task-123",
    "status": "completed",
    "result": {
      "pages_created": 5,
      "images_generated": 5
    }
  }
}
```

**Response (Failed):**
```json
{
  "success": true,
  "data": {
    "task_id": "task-123",
    "status": "failed",
    "error": "AI service error: Rate limit exceeded"
  }
}
```

**Task Statuses:**
- `pending`: Task queued
- `running`: Task in progress
- `completed`: Task finished successfully
- `failed`: Task encountered an error

## File Serving

### Get File

Serves static files (images, exports, etc.).

**Endpoint:** `GET /files/*path`

**Examples:**
- `/files/abc-123/template/image.png` - Template image
- `/files/abc-123/pages/page-1/v1.png` - Page image
- `/files/materials/image.png` - Material image
- `/files/exports/abc-123.pptx` - Export file

**Response:** Binary file content with appropriate Content-Type header

---

## Error Responses

### Common Errors

**400 Bad Request:**
```json
{
  "success": false,
  "message": "Invalid input",
  "error": "Missing required field: idea_prompt"
}
```

**404 Not Found:**
```json
{
  "success": false,
  "message": "Resource not found",
  "error": "Project abc-123 not found"
}
```

**500 Internal Server Error:**
```json
{
  "success": false,
  "message": "Internal server error",
  "error": "Database connection failed"
}
```

---

## Rate Limiting

Currently, there is no built-in rate limiting. However, external AI services (Gemini, OpenAI) have their own rate limits:

- **Gemini**: Varies by tier and model
- **OpenAI**: Varies by tier and model

Monitor your API usage through the respective provider dashboards.

## Testing the API

### Using curl

```bash
# Create project
curl -X POST http://localhost:5000/api/projects \
  -H "Content-Type: application/json" \
  -d '{"creation_type":"idea","idea_prompt":"Test presentation"}'

# Get project
curl http://localhost:5000/api/projects/abc-123

# Upload file
curl -X POST http://localhost:5000/api/reference-files/upload \
  -F "file=@document.pdf" \
  -F "project_id=abc-123"
```

### Using Postman

Import the following collection base URL: `http://localhost:5000`

Create requests for each endpoint using the documentation above.

---

**Next**: Learn about [Database Models](./05-database-models.md) to understand data structures.
