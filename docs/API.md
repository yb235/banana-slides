# API Documentation

## Table of Contents
- [Overview](#overview)
- [Base URL](#base-url)
- [Authentication](#authentication)
- [Response Format](#response-format)
- [Error Handling](#error-handling)
- [Endpoints](#endpoints)
  - [Health Check](#health-check)
  - [Projects](#projects)
  - [Pages](#pages)
  - [Tasks](#tasks)
  - [Materials](#materials)
  - [Templates](#templates)
  - [Reference Files](#reference-files)
  - [Export](#export)
  - [File Serving](#file-serving)

## Overview

Banana Slides API is a RESTful API that provides endpoints for creating, managing, and exporting AI-generated presentations.

**API Version**: 1.0.0

**Content Type**: `application/json` (except file uploads)

## Base URL

### Development
```
http://localhost:5000
```

### Production
```
http://your-domain.com
```

## Authentication

Currently, the API does not require authentication. In production, consider implementing:
- API keys
- OAuth 2.0
- JWT tokens

## Response Format

### Success Response

```json
{
  "success": true,
  "data": {
    // Response data
  }
}
```

### Error Response

```json
{
  "success": false,
  "error": "Error message"
}
```

### Paginated Response

```json
{
  "success": true,
  "data": {
    "items": [...],
    "total": 100,
    "limit": 20,
    "offset": 0
  }
}
```

## Error Handling

### HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | OK - Request successful |
| 201 | Created - Resource created successfully |
| 400 | Bad Request - Invalid input |
| 404 | Not Found - Resource not found |
| 500 | Internal Server Error - Server error |

### Error Response Format

```json
{
  "success": false,
  "error": "Detailed error message"
}
```

## Endpoints

### Health Check

#### Check API Health

```http
GET /health
```

**Response**:
```json
{
  "status": "ok",
  "message": "Banana Slides API is running"
}
```

---

### Projects

#### Create Project

Create a new presentation project.

```http
POST /api/projects
```

**Request Body**:
```json
{
  "creation_type": "idea",  // "idea" | "outline" | "descriptions"
  "idea_prompt": "Create a presentation about AI trends in 2024",
  "outline_text": null,  // Used when creation_type is "outline"
  "description_text": null  // Used when creation_type is "descriptions"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "project_id": "uuid-here",
    "idea_prompt": "Create a presentation about AI trends in 2024",
    "creation_type": "idea",
    "status": "DRAFT",
    "template_image_url": null,
    "pages": [
      {
        "page_id": "uuid-here",
        "order_index": 0,
        "outline_content": {
          "title": "AI Trends 2024",
          "key_points": ["GPT-4", "Computer Vision"]
        },
        "description_content": null,
        "image_url": null
      }
    ],
    "created_at": "2024-01-01T00:00:00",
    "updated_at": "2024-01-01T00:00:00"
  }
}
```

**Notes**:
- When `creation_type` is "idea", outline is automatically generated
- When `creation_type` is "outline", pages are created from outline
- When `creation_type` is "descriptions", outline is generated from descriptions

---

#### List Projects

Get all projects with pagination.

```http
GET /api/projects?limit=50&offset=0
```

**Query Parameters**:
- `limit` (optional): Number of results (default: 50)
- `offset` (optional): Offset for pagination (default: 0)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "projects": [
      {
        "project_id": "uuid",
        "idea_prompt": "...",
        "status": "DRAFT",
        "created_at": "2024-01-01T00:00:00",
        "updated_at": "2024-01-01T00:00:00"
      }
    ],
    "total": 10
  }
}
```

---

#### Get Project

Get project details including all pages.

```http
GET /api/projects/{project_id}
```

**Path Parameters**:
- `project_id`: UUID of the project

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "project_id": "uuid",
    "idea_prompt": "...",
    "status": "DRAFT",
    "pages": [
      {
        "page_id": "uuid",
        "order_index": 0,
        "outline_content": {...},
        "description_content": {...},
        "image_url": "/files/project-id/pages/page-id/image.png"
      }
    ],
    "created_at": "2024-01-01T00:00:00",
    "updated_at": "2024-01-01T00:00:00"
  }
}
```

---

#### Update Project

Update project metadata.

```http
PUT /api/projects/{project_id}
```

**Path Parameters**:
- `project_id`: UUID of the project

**Request Body**:
```json
{
  "idea_prompt": "Updated idea",
  "status": "COMPLETED",
  "extra_requirements": "Use blue color scheme",
  "pages_order": ["page-id-1", "page-id-2", "page-id-3"]  // For reordering
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "project_id": "uuid",
    "idea_prompt": "Updated idea",
    "status": "COMPLETED",
    ...
  }
}
```

---

#### Delete Project

Delete a project and all associated data.

```http
DELETE /api/projects/{project_id}
```

**Path Parameters**:
- `project_id`: UUID of the project

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "message": "Project deleted successfully"
  }
}
```

---

#### Upload Template

Upload template image for a project.

```http
POST /api/projects/{project_id}/template
```

**Path Parameters**:
- `project_id`: UUID of the project

**Request Body** (multipart/form-data):
- `template_image`: Image file

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "template_image_url": "/files/project-id/template/image.png"
  }
}
```

---

#### Generate Outline

Generate outline from idea prompt.

```http
POST /api/projects/{project_id}/generate-outline
```

**Path Parameters**:
- `project_id`: UUID of the project

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "project_id": "uuid",
    "pages": [
      {
        "page_id": "uuid",
        "outline_content": {
          "title": "Introduction",
          "key_points": ["Point 1", "Point 2"]
        }
      }
    ]
  }
}
```

---

#### Refine Outline

Refine outline using natural language.

```http
POST /api/projects/{project_id}/refine-outline
```

**Path Parameters**:
- `project_id`: UUID of the project

**Request Body**:
```json
{
  "refinement_prompt": "Add a page about case studies after slide 3"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "project_id": "uuid",
    "pages": [...]  // Updated pages
  }
}
```

---

#### Generate Descriptions

Generate descriptions for all pages (async task).

```http
POST /api/projects/{project_id}/generate-descriptions
```

**Path Parameters**:
- `project_id`: UUID of the project

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "task_id": "uuid",
    "task_type": "generate_descriptions",
    "status": "pending"
  }
}
```

**Notes**:
- This is an async operation
- Poll task status using GET /api/tasks/{task_id}

---

#### Refine Descriptions

Refine descriptions using natural language.

```http
POST /api/projects/{project_id}/refine-descriptions
```

**Path Parameters**:
- `project_id`: UUID of the project

**Request Body**:
```json
{
  "refinement_prompt": "Make all slides more visual with more images"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "task_id": "uuid",
    "task_type": "refine_descriptions",
    "status": "pending"
  }
}
```

---

#### Generate Images

Generate images for all pages (async task).

```http
POST /api/projects/{project_id}/generate-images
```

**Path Parameters**:
- `project_id`: UUID of the project

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "task_id": "uuid",
    "task_type": "generate_images",
    "status": "pending"
  }
}
```

**Notes**:
- This is an async operation
- Can take several minutes depending on number of pages
- Poll task status using GET /api/tasks/{task_id}

---

#### Generate from Description

Generate outline and pages from description text.

```http
POST /api/projects/{project_id}/generate-from-description
```

**Path Parameters**:
- `project_id`: UUID of the project

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "project_id": "uuid",
    "pages": [...]
  }
}
```

---

### Pages

#### Add Page

Add a new blank page to project.

```http
POST /api/projects/{project_id}/pages
```

**Path Parameters**:
- `project_id`: UUID of the project

**Request Body** (optional):
```json
{
  "order_index": 5,  // Position to insert (optional)
  "outline_content": {
    "title": "New Page",
    "key_points": []
  }
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "page_id": "uuid",
    "project_id": "uuid",
    "order_index": 5,
    "outline_content": {...},
    "created_at": "2024-01-01T00:00:00"
  }
}
```

---

#### Update Page

Update page data.

```http
PUT /api/pages/{page_id}
```

**Path Parameters**:
- `page_id`: UUID of the page

**Request Body**:
```json
{
  "outline_content": {
    "title": "Updated Title",
    "key_points": ["Point 1", "Point 2"]
  },
  "description_content": {
    "layout": "Title and Content",
    "text_elements": [...]
  }
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "page_id": "uuid",
    "outline_content": {...},
    "description_content": {...},
    "updated_at": "2024-01-01T00:00:00"
  }
}
```

---

#### Update Page Outline

Update only the outline content of a page.

```http
PUT /api/pages/{page_id}/outline
```

**Path Parameters**:
- `page_id`: UUID of the page

**Request Body**:
```json
{
  "title": "Updated Title",
  "subtitle": "Optional subtitle",
  "key_points": ["Point 1", "Point 2"]
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "page_id": "uuid",
    "outline_content": {...},
    "updated_at": "2024-01-01T00:00:00"
  }
}
```

---

#### Update Page Description

Update only the description content of a page.

```http
PUT /api/pages/{page_id}/description
```

**Path Parameters**:
- `page_id`: UUID of the page

**Request Body**:
```json
{
  "layout": "Title and Content",
  "text_elements": [
    {
      "type": "title",
      "content": "Page Title"
    },
    {
      "type": "body",
      "content": "Body text..."
    }
  ],
  "image_suggestions": ["Chart showing growth", "Team photo"],
  "color_scheme": "blue and white"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "page_id": "uuid",
    "description_content": {...},
    "updated_at": "2024-01-01T00:00:00"
  }
}
```

---

#### Delete Page

Delete a page from project.

```http
DELETE /api/pages/{page_id}
```

**Path Parameters**:
- `page_id`: UUID of the page

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "message": "Page deleted successfully"
  }
}
```

---

#### Generate Page Description

Generate description for a single page.

```http
POST /api/pages/{page_id}/generate-description
```

**Path Parameters**:
- `page_id`: UUID of the page

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "task_id": "uuid",
    "status": "pending"
  }
}
```

---

#### Generate Page Image

Generate image for a single page.

```http
POST /api/pages/{page_id}/generate-image
```

**Path Parameters**:
- `page_id`: UUID of the page

**Query Parameters**:
- `force_regenerate` (optional): Set to "true" to regenerate even if image exists

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "task_id": "uuid",
    "status": "pending"
  }
}
```

---

#### Edit Page Image

Edit existing page image with natural language prompt.

```http
POST /api/pages/{page_id}/edit-image
```

**Path Parameters**:
- `page_id`: UUID of the page

**Request Body** (multipart/form-data):
- `edit_prompt`: Text prompt for editing (e.g., "Change background to blue")
- `use_template`: Boolean, whether to use project template as reference
- `desc_image_urls`: JSON array of description image URLs to use as reference
- `uploaded_files`: Image files to use as reference

**Example Request**:
```javascript
const formData = new FormData();
formData.append('edit_prompt', 'Change background to blue and add more charts');
formData.append('use_template', 'true');
formData.append('desc_image_urls', JSON.stringify(['/files/image1.png']));
formData.append('uploaded_files', file1);
formData.append('uploaded_files', file2);
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "task_id": "uuid",
    "status": "pending"
  }
}
```

---

#### Get Page Versions

Get version history of a page's images.

```http
GET /api/pages/{page_id}/versions
```

**Path Parameters**:
- `page_id`: UUID of the page

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "versions": [
      {
        "version_id": "uuid",
        "version_number": 1,
        "image_url": "/files/...",
        "edit_prompt": "Original generation",
        "created_at": "2024-01-01T00:00:00"
      },
      {
        "version_id": "uuid",
        "version_number": 2,
        "image_url": "/files/...",
        "edit_prompt": "Change background to blue",
        "created_at": "2024-01-01T00:05:00"
      }
    ]
  }
}
```

---

### Tasks

#### Get Task Status

Get status and progress of an async task.

```http
GET /api/tasks/{task_id}
```

**Path Parameters**:
- `task_id`: UUID of the task

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "task_id": "uuid",
    "task_type": "generate_images",
    "status": "running",  // "pending" | "running" | "completed" | "failed"
    "progress": {
      "total": 10,
      "completed": 3,
      "message": "Generating images..."
    },
    "result": null,  // Populated when completed
    "error_message": null,  // Populated if failed
    "created_at": "2024-01-01T00:00:00",
    "updated_at": "2024-01-01T00:00:05"
  }
}
```

**Notes**:
- Poll this endpoint every 2-3 seconds to check task progress
- When status is "completed", result field contains task output

---

### Materials

#### Upload Material

Upload a material (image, document, etc.).

```http
POST /api/materials/upload
```

**Request Body** (multipart/form-data):
- `file`: Material file
- `material_type`: Type of material ("image", "chart", "icon", etc.)

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "material_id": "uuid",
    "filename": "chart.png",
    "file_url": "/files/materials/uuid/chart.png",
    "material_type": "image",
    "created_at": "2024-01-01T00:00:00"
  }
}
```

---

#### Generate Material

Generate material using AI.

```http
POST /api/materials/generate
```

**Request Body**:
```json
{
  "prompt": "Create a chart showing sales growth over 5 years",
  "material_type": "chart"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "task_id": "uuid",
    "status": "pending"
  }
}
```

---

#### List Materials

List all materials for a project.

```http
GET /api/projects/{project_id}/materials
```

**Path Parameters**:
- `project_id`: UUID of the project

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "materials": [
      {
        "material_id": "uuid",
        "filename": "chart.png",
        "file_url": "/files/materials/uuid/chart.png",
        "material_type": "image",
        "created_at": "2024-01-01T00:00:00"
      }
    ]
  }
}
```

---

#### Delete Material

Delete a material.

```http
DELETE /api/materials/{material_id}
```

**Path Parameters**:
- `material_id`: UUID of the material

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "message": "Material deleted successfully"
  }
}
```

---

#### Associate Materials to Project

Associate existing materials with a project.

```http
POST /api/projects/{project_id}/materials/associate
```

**Path Parameters**:
- `project_id`: UUID of the project

**Request Body**:
```json
{
  "material_ids": ["uuid1", "uuid2", "uuid3"]
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "message": "Materials associated successfully"
  }
}
```

---

### Templates

#### List User Templates

List all user-uploaded templates.

```http
GET /api/user-templates
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "templates": [
      {
        "template_id": "uuid",
        "name": "Blue Corporate",
        "preview_url": "/files/templates/uuid/preview.png",
        "created_at": "2024-01-01T00:00:00"
      }
    ]
  }
}
```

---

#### Upload User Template

Upload a custom template.

```http
POST /api/user-templates/upload
```

**Request Body** (multipart/form-data):
- `template_file`: Template image file
- `name`: Template name

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "template_id": "uuid",
    "name": "Blue Corporate",
    "preview_url": "/files/templates/uuid/preview.png",
    "created_at": "2024-01-01T00:00:00"
  }
}
```

---

### Reference Files

#### Upload Reference File

Upload a reference document (PDF, DOCX, etc.).

```http
POST /api/reference-files/upload
```

**Request Body** (multipart/form-data):
- `file`: Reference file

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "file_id": "uuid",
    "filename": "document.pdf",
    "file_path": "/path/to/file",
    "parse_status": "pending",
    "created_at": "2024-01-01T00:00:00"
  }
}
```

---

#### Trigger File Parse

Trigger parsing of uploaded reference file.

```http
POST /api/reference-files/{file_id}/parse
```

**Path Parameters**:
- `file_id`: UUID of the file

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "task_id": "uuid",
    "status": "pending"
  }
}
```

---

#### Get Reference File

Get reference file details and parsed content.

```http
GET /api/reference-files/{file_id}
```

**Path Parameters**:
- `file_id`: UUID of the file

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "file_id": "uuid",
    "filename": "document.pdf",
    "parse_status": "completed",
    "markdown_content": "# Document Title\n\nContent...",
    "created_at": "2024-01-01T00:00:00"
  }
}
```

---

#### Associate File to Project

Associate parsed reference file with a project.

```http
POST /api/reference-files/{file_id}/associate
```

**Path Parameters**:
- `file_id`: UUID of the file

**Request Body**:
```json
{
  "project_id": "uuid"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "message": "File associated successfully"
  }
}
```

---

### Export

#### Export PPTX

Export project as PowerPoint file.

```http
POST /api/export/pptx
```

**Request Body**:
```json
{
  "project_id": "uuid"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "download_url": "/api/export/download/project-name-20240101.pptx"
  }
}
```

---

#### Export PDF

Export project as PDF file.

```http
POST /api/export/pdf
```

**Request Body**:
```json
{
  "project_id": "uuid"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "download_url": "/api/export/download/project-name-20240101.pdf"
  }
}
```

---

#### Download Export File

Download exported file.

```http
GET /api/export/download/{filename}
```

**Path Parameters**:
- `filename`: Name of the export file

**Response**:
- File download (Content-Type: application/vnd.openxmlformats-officedocument.presentationml.presentation or application/pdf)

---

### File Serving

#### Get File

Serve uploaded files (images, materials, etc.).

```http
GET /files/{project_id}/{file_type}/{filename}
```

**Path Parameters**:
- `project_id`: UUID of the project
- `file_type`: Type of file ("pages", "template", "materials", etc.)
- `filename`: Name of the file

**Example**:
```
GET /files/abc123/pages/page-id/slide.png
GET /files/abc123/template/template.png
```

**Response**:
- File download with appropriate Content-Type

---

## Rate Limiting

Currently, no rate limiting is implemented. For production:
- Consider implementing rate limiting per IP
- Set appropriate limits based on AI API quotas
- Return 429 Too Many Requests when limit exceeded

## Webhooks

Currently not supported. Future implementation could include:
- Task completion webhooks
- Export completion webhooks
- Error notification webhooks

## SDK / Client Libraries

Official client libraries:
- **JavaScript/TypeScript**: Available in frontend (`/frontend/src/api/endpoints.ts`)
- **Python**: Not yet available
- **Other languages**: Not yet available

## Changelog

### Version 1.0.0 (Current)
- Initial API release
- Project CRUD operations
- AI-powered outline and description generation
- Image generation and editing
- Material and template management
- PPTX and PDF export

## Support

For API support:
- GitHub Issues: https://github.com/Anionex/banana-slides/issues
- Email: (to be added)
