# Complete Workflow Guide

This guide explains how Banana Slides works from a user's perspective and what happens under the hood at each step.

## 🎯 Overview: Three Creation Paths

Banana Slides offers three ways to create a presentation:

1. **From an Idea** (💡): Start with a simple prompt
2. **From an Outline** (📝): Provide a structured outline
3. **From Descriptions** (📄): Provide detailed page-by-page content

All three paths converge into the same workflow after the initial creation.

## 🚀 Path 1: Creating from an Idea

### User Journey

```
Home Page → Enter Idea → Create Project → Outline Generated → 
Edit Outline → Generate Descriptions → Generate Images → 
Preview & Export
```

### Step-by-Step Process

#### Step 1: Enter Your Idea (Home Page)

**What you do:**
- Navigate to the home page
- Select "Start with an idea" tab
- Enter a prompt like: "Create a presentation about sustainable energy"
- Optionally upload a template image for style consistency
- Optionally upload reference files (PDFs, DOCX, etc.)
- Click "Create Project"

**What happens behind the scenes:**

```
1. Frontend (Home.tsx):
   - Validates input
   - Calls createProject() API endpoint
   - Stores project ID in localStorage
   - Navigates to /outline/:projectId

2. Backend (project_controller.py):
   - Creates new Project record with status="DRAFT"
   - Saves idea_prompt, template, and creation_type="idea"
   - Starts async task to generate outline
   - Returns project ID immediately

3. Task Manager (task_manager.py):
   - Spawns background thread
   - Calls AIService.generate_outline()
   
4. AI Service (ai_service.py):
   - Builds prompt with idea + reference files
   - Calls text provider (Gemini/OpenAI)
   - Receives JSON outline with titles and points
   - Parses JSON response
   - Creates Page records for each slide
   - Updates task status to "completed"

5. Frontend (OutlineEditor.tsx):
   - Polls task status every 2 seconds
   - Shows loading animation
   - Updates UI when pages are ready
```

**API Flow:**

```
POST /api/projects
{
  "creation_type": "idea",
  "idea_prompt": "Create a presentation about sustainable energy"
}

Response:
{
  "success": true,
  "data": {
    "project_id": "abc-123",
    "status": "DRAFT"
  }
}

→ Auto-triggers outline generation task

GET /api/projects/abc-123/tasks/task-456
{
  "success": true,
  "data": {
    "task_id": "task-456",
    "status": "completed",
    "result": { ... }
  }
}
```

#### Step 2: Review and Edit Outline

**What you do:**
- Review the generated outline (titles and bullet points)
- Reorder pages by dragging
- Edit page titles and bullet points
- Add or delete pages
- Use AI Refine feature to modify the outline with natural language
  - Example: "Add a page about wind energy"
- Click "Next: Generate Descriptions"

**What happens:**

```
1. Frontend (OutlineEditor.tsx):
   - Displays pages in draggable list
   - Each page shows title and bullet points
   - Changes are saved automatically via API

2. Manual Edits:
   - PUT /api/projects/:projectId/pages/:pageId/outline
   - Updates outline_content in database

3. AI Refine:
   - POST /api/projects/:projectId/refine/outline
   - Sends user requirement + current outline
   - AI modifies outline based on instruction
   - Returns updated pages

4. Reordering:
   - PUT /api/projects/:projectId
   - Updates pages_order field
   - Maintains order in database
```

#### Step 3: Generate Descriptions

**What you do:**
- Click "Generate Descriptions" button
- Wait for AI to generate detailed content for each page

**What happens:**

```
1. Frontend:
   - POST /api/projects/:projectId/generate/descriptions
   - Shows loading state for each page

2. Backend:
   - Creates task for batch description generation
   - Uses ThreadPoolExecutor for parallel processing
   - Each page processed independently

3. AI Service:
   - For each page:
     - Builds prompt with title, points, context
     - Calls text provider
     - Receives structured description (markdown + images)
     - Saves to page.description_content
   - Runs up to MAX_DESCRIPTION_WORKERS (default: 5) in parallel

4. Frontend:
   - Polls task status
   - Shows progress indicators
   - Navigates to DetailEditor when done
```

**Description Structure:**

```json
{
  "title": "Sustainable Energy Solutions",
  "content": "Detailed text about sustainable energy...",
  "images": [
    {
      "description": "Solar panel installation",
      "position": "top-right",
      "size": "medium"
    }
  ]
}
```

#### Step 4: Review and Edit Descriptions

**What you do:**
- Review detailed descriptions for each page
- Edit the markdown content
- Adjust image descriptions
- Use AI Refine to modify descriptions
  - Example: "Make the content more concise"
- Click "Next: Generate Images"

**What happens:**

```
1. Frontend (DetailEditor.tsx):
   - Displays markdown editor for each page
   - Supports full markdown syntax
   - Auto-saves changes

2. Manual Edits:
   - PUT /api/projects/:projectId/pages/:pageId/description
   - Updates description_content in database

3. AI Refine:
   - POST /api/projects/:projectId/refine/descriptions
   - Modifies descriptions based on user input
```

#### Step 5: Generate Images

**What you do:**
- Click "Generate Images" button
- Wait for AI to generate all slide images

**What happens:**

```
1. Frontend:
   - POST /api/projects/:projectId/generate/images
   - Shows generation progress

2. Backend:
   - Creates task for batch image generation
   - Uses ThreadPoolExecutor for parallel processing
   - Each page processed independently

3. AI Service:
   - For each page:
     - Builds image prompt from description
     - Includes template image if provided
     - Includes reference images from description
     - Calls image provider (nano banana pro / DALL-E)
     - Saves image to file system
     - Creates PageImageVersion record
     - Updates page.image_url
   - Runs up to MAX_IMAGE_WORKERS (default: 8) in parallel

4. Image Storage:
   - Saved to uploads/{project_id}/pages/{page_id}/
   - Filename includes version: v1_timestamp.png
   - Keeps version history for rollback
```

**Image Prompt Example:**

```
Create a professional PowerPoint slide for a presentation about sustainable energy.

Title: "Sustainable Energy Solutions"
Content: Detailed text about sustainable energy...

Include these visual elements:
- Solar panels in a sunny landscape
- Modern architecture
- Clean, professional design

Style requirements:
- 16:9 aspect ratio
- Professional color scheme
- Clear typography
- Similar style to reference image
```

#### Step 6: Preview and Export

**What you do:**
- Review all generated slides
- Edit individual images with natural language
  - Example: "Change the background to blue"
- Reorder slides
- Export to PPTX or PDF

**What happens:**

```
1. Frontend (SlidePreview.tsx):
   - Displays all slides in a grid
   - Clicking a slide shows full-size preview
   - Edit button opens image editing modal

2. Image Editing:
   - POST /api/projects/:projectId/pages/:pageId/edit/image
   - Sends edit instruction + context images
   - AI generates new version
   - Adds to version history

3. Export PPTX:
   - GET /api/projects/:projectId/export/pptx
   - Backend uses python-pptx to create file
   - Each slide is added as an image
   - Returns download URL

4. Export PDF:
   - GET /api/projects/:projectId/export/pdf
   - Backend uses reportlab to create PDF
   - Each slide becomes a PDF page
   - Returns download URL
```

## 📝 Path 2: Creating from an Outline

### User Journey

```
Home Page → Enter Outline → Create Project → Outline Parsed → 
Edit Outline → Generate Descriptions → Generate Images → 
Preview & Export
```

### Key Differences from "From Idea"

**Step 1: Provide Outline**

**What you do:**
- Select "Start with outline" tab
- Paste or type your outline in this format:

```
# Page 1: Introduction
- Welcome to the presentation
- Overview of topics
- Goals and objectives

# Page 2: Main Content
- Key point one
- Key point two
- Supporting details
```

- Click "Create Project"

**What happens:**

```
1. Backend:
   - Parses the outline text
   - Extracts page titles and bullet points
   - Creates Page records immediately
   - No AI call needed for outline
   - Status is "READY" immediately

2. Data Structure:
   - Outline text → JSON structure
   - Each # header → new page
   - Bullet points → page points
```

The rest of the workflow (descriptions, images, export) is identical to "From Idea" path.

## 📄 Path 3: Creating from Descriptions

### User Journey

```
Home Page → Enter Descriptions → Create Project → 
Descriptions Parsed → Generate Images → Preview & Export
```

### Key Differences

**Step 1: Provide Descriptions**

**What you do:**
- Select "Start with descriptions" tab
- Paste detailed content for each slide:

```
---SLIDE 1---
Title: Introduction to AI
Content: Artificial Intelligence is transforming...
[Include image of: futuristic AI visualization]

---SLIDE 2---
Title: Machine Learning Basics
Content: Machine learning is a subset...
[Include image of: neural network diagram]
```

- Click "Create Project"

**What happens:**

```
1. Backend:
   - Parses description text
   - Splits into individual pages
   - AI converts to structured outline + descriptions
   - Creates Page records with both outline and descriptions
   - No manual description generation needed

2. Workflow:
   - Skip outline editing (already structured)
   - Skip description generation (already provided)
   - Go directly to image generation
```

## 🎨 Working with Materials and Templates

### Materials (Custom Images)

**What they are:**
- Custom images you want to include in your presentation
- Can be uploaded or AI-generated
- Can be global or project-specific

**How to use:**

```
1. Generate Material:
   - Click "Materials" button
   - Enter a prompt: "Professional diagram of renewable energy"
   - Optionally upload reference images
   - AI generates the image
   - Saved to material library

2. Upload Material:
   - Click "Materials" button
   - Drag and drop images
   - Images added to library

3. Use in Slides:
   - When editing images, select materials
   - Include in image generation prompt
   - Referenced in edit instructions
```

### Templates (Style Reference)

**What they are:**
- Reference images that define the visual style
- Applied consistently across all slides
- Can be pre-made or uploaded

**How to use:**

```
1. Upload Template:
   - On home page, click template selector
   - Upload an image or choose preset
   - Template is used for all image generation

2. Template Application:
   - Automatically included in every image prompt
   - Ensures visual consistency
   - Can be changed mid-project
```

## 📂 Working with Reference Files

### Supported File Types

- PDF documents
- Word documents (DOCX, DOC)
- PowerPoint (PPTX, PPT)
- Excel (XLSX, XLS)
- Text files (TXT, MD)
- CSV files

### File Processing Workflow

```
1. Upload:
   - Drag and drop or select file
   - POST /api/reference-files/upload
   - Creates ReferenceFile record
   - Status: "pending"

2. Parsing (Automatic):
   - Background task starts immediately
   - Extracts text content
   - Extracts images
   - Converts to markdown format
   - Status: "parsing" → "completed"

3. Using Content:
   - When generating outline/descriptions
   - Content is included in AI prompts
   - Images are extracted and available
   - Provides context for generation

4. MinerU (Optional):
   - Advanced parsing service
   - Better for complex layouts
   - Requires MINERU_TOKEN in .env
   - Falls back to markitdown if not available
```

### File Content in AI Prompts

```xml
<uploaded_files>
  <file name="research_paper.pdf">
    <content>
      # Research Paper Title
      
      ## Introduction
      Content extracted from PDF...
      
      ![Image from PDF](image_url)
    </content>
  </file>
</uploaded_files>
```

## 🔄 Advanced Features

### AI Refinement

**Outline Refinement:**
```
User: "Add a conclusion page and remove the second page"

Backend:
- Sends current outline + user requirement to AI
- AI modifies the structure
- Returns updated pages
- Frontend updates immediately
```

**Description Refinement:**
```
User: "Make all content more technical and add statistics"

Backend:
- Sends all descriptions + user requirement
- AI modifies each description
- Returns updated pages
- Preserves structure
```

### Image Editing

**Natural Language Editing:**
```
User: "Change the background to a gradient and make the text larger"

Backend:
- Current image + edit instruction sent to AI
- Image generation with edit mode
- Creates new version
- Keeps old version in history
```

**Context Images:**
- Can include additional reference images
- Helps guide the AI
- Example: "Make it look like this style" + upload image

### Version History

**Page Image Versions:**
```
Each edit creates a new version:
- v1: Original generation
- v2: First edit
- v3: Second edit
...

User can switch between versions:
- View history
- Select previous version
- Becomes current image
```

## 📊 Workflow State Diagram

```
┌─────────────┐
│   DRAFT     │ ← Initial state after project creation
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  OUTLINE    │ ← Outline is generated/provided
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ DESCRIPTION │ ← Descriptions are generated
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   IMAGES    │ ← Images are generated
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  COMPLETE   │ ← Ready for export
└─────────────┘
```

**State Transitions:**
- States are implicit based on page data
- Frontend determines current state
- Backend doesn't enforce strict state machine
- User can jump between steps

## 🎯 Best Practices

### For Best Results

1. **Idea Creation:**
   - Be specific in your prompt
   - Include context and audience
   - Mention any specific requirements

2. **Outline Editing:**
   - Keep bullet points concise
   - 3-5 points per slide is optimal
   - Use clear, descriptive titles

3. **Description Writing:**
   - Provide detailed content
   - Mention desired visualizations
   - Include specific data/statistics

4. **Image Generation:**
   - Upload a template for consistency
   - Provide reference materials
   - Be specific about visual elements

5. **Editing:**
   - Make small, incremental edits
   - Test different versions
   - Keep good versions

### Performance Tips

1. **Parallel Generation:**
   - Descriptions and images generated in parallel
   - Faster for multi-page presentations

2. **Reusing Materials:**
   - Upload materials once, use in multiple projects
   - Faster than generating each time

3. **Template Consistency:**
   - Set template early
   - Reduces regeneration needs

## 🚫 Common Pitfalls

1. **Skipping Outline Review:**
   - Always review before generating descriptions
   - Easier to fix structure early

2. **Vague Prompts:**
   - Be specific about what you want
   - Include context and requirements

3. **Ignoring Reference Files:**
   - Upload relevant materials early
   - Better context for AI generation

4. **Too Many Edits:**
   - Each edit creates a new version
   - Plan your changes

## 📚 Related Documentation

- [API Reference](./04-api-reference.md) - Detailed API endpoints
- [Architecture](./02-architecture.md) - System design
- [Database Models](./05-database-models.md) - Data structures

---

**Next**: Explore the [API Reference](./04-api-reference.md) for technical details.
