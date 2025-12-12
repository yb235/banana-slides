# Frontend Documentation

## Table of Contents
- [Overview](#overview)
- [Directory Structure](#directory-structure)
- [Technology Stack](#technology-stack)
- [State Management](#state-management)
- [Pages](#pages)
- [Components](#components)
- [API Integration](#api-integration)
- [Routing](#routing)
- [Styling](#styling)
- [User Workflows](#user-workflows)

## Overview

The frontend is a React 18 application built with TypeScript and Vite, providing a modern, responsive UI for creating and editing AI-generated presentations.

### Key Features
- Three creation modes: Idea, Outline, and Description
- Real-time outline and description editing
- Drag-and-drop slide reordering
- Live slide preview with editing
- Material and template management
- File upload and parsing
- Export to PPTX and PDF

## Directory Structure

```
frontend/
├── src/
│   ├── pages/                  # Page components (routes)
│   │   ├── Home.tsx           # Project creation
│   │   ├── OutlineEditor.tsx  # Outline editing
│   │   ├── DetailEditor.tsx   # Description editing
│   │   ├── SlidePreview.tsx   # Slide viewing/editing
│   │   └── History.tsx        # Project history
│   ├── components/             # Reusable components
│   │   ├── shared/            # Shared UI components
│   │   │   ├── Button.tsx
│   │   │   ├── Card.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Textarea.tsx
│   │   │   ├── Modal.tsx
│   │   │   ├── Loading.tsx
│   │   │   ├── Toast.tsx
│   │   │   ├── Markdown.tsx
│   │   │   ├── MaterialSelector.tsx
│   │   │   ├── MaterialGeneratorModal.tsx
│   │   │   └── TemplateSelector.tsx
│   │   ├── preview/           # Slide preview components
│   │   │   ├── SlideCard.tsx
│   │   │   └── DescriptionCard.tsx
│   │   ├── outline/           # Outline components
│   │   │   └── OutlineCard.tsx
│   │   ├── layout/            # Layout components
│   │   └── history/           # History components
│   ├── store/                  # State management
│   │   └── useProjectStore.ts # Zustand store
│   ├── api/                    # API layer
│   │   ├── client.ts          # Axios configuration
│   │   └── endpoints.ts       # API functions
│   ├── types/                  # TypeScript types
│   │   └── index.ts           # Type definitions
│   ├── utils/                  # Utility functions
│   │   ├── index.ts
│   │   └── projectUtils.ts
│   ├── constants/              # Constants
│   ├── styles/                 # Global styles
│   ├── App.tsx                 # Root component
│   └── main.tsx               # Entry point
├── public/                     # Static assets
├── index.html                  # HTML template
├── package.json                # Dependencies
├── vite.config.ts              # Vite configuration
├── tailwind.config.js          # Tailwind CSS config
├── tsconfig.json               # TypeScript config
└── Dockerfile                  # Docker image
```

## Technology Stack

### Core Technologies

| Technology | Version | Purpose |
|-----------|---------|---------|
| React | 18.x | UI library |
| TypeScript | 5.x | Type safety |
| Vite | 5.x | Build tool and dev server |
| React Router | 6.x | Client-side routing |

### State Management

| Library | Purpose |
|---------|---------|
| Zustand | Lightweight state management |

### UI & Styling

| Library | Purpose |
|---------|---------|
| Tailwind CSS | Utility-first CSS framework |
| Lucide React | Icon library |
| @dnd-kit | Drag-and-drop functionality |

### HTTP & Data

| Library | Purpose |
|---------|---------|
| Axios | HTTP client |
| React Markdown | Markdown rendering |

## State Management

### Zustand Store (`store/useProjectStore.ts`)

The application uses a single Zustand store for global state management.

#### State Structure

```typescript
interface ProjectState {
  // Core state
  currentProject: Project | null;
  isGlobalLoading: boolean;
  activeTaskId: string | null;
  taskProgress: { total: number; completed: number } | null;
  error: string | null;
  
  // Page-level task tracking
  pageGeneratingTasks: Record<string, string>;  // pageId -> taskId
  pageDescriptionGeneratingTasks: Record<string, boolean>;
  
  // Actions
  setCurrentProject: (project: Project | null) => void;
  setGlobalLoading: (loading: boolean) => void;
  setError: (error: string | null) => void;
  
  // ... (see below for detailed actions)
}
```

#### Key Actions

##### Project Operations

```typescript
// Initialize new project
initializeProject(
  type: 'idea' | 'outline' | 'description',
  content: string,
  templateImage?: File
): Promise<void>

// Sync project from server
syncProject(projectId?: string): Promise<void>
```

##### Outline & Description Generation

```typescript
// Generate outline from idea
generateOutline(): Promise<void>

// Generate descriptions for all pages
generateDescriptions(): Promise<void>

// Generate description for single page
generatePageDescription(pageId: string): Promise<void>

// Generate outline from descriptions
generateFromDescription(): Promise<void>
```

##### Image Generation

```typescript
// Generate images for all pages
generateImages(): Promise<void>

// Generate image for single page
generatePageImage(pageId: string, forceRegenerate?: boolean): Promise<void>

// Edit page image with prompt
editPageImage(
  pageId: string,
  editPrompt: string,
  contextImages?: {
    useTemplate?: boolean;
    descImageUrls?: string[];
    uploadedFiles?: File[];
  }
): Promise<void>
```

##### Page Management

```typescript
// Update page locally (with debounced API sync)
updatePageLocal(pageId: string, data: any): void

// Save all pending changes
saveAllPages(): Promise<void>

// Reorder pages (drag-and-drop)
reorderPages(newOrder: string[]): Promise<void>

// Add new blank page
addNewPage(): Promise<void>

// Delete page
deletePageById(pageId: string): Promise<void>
```

##### Export

```typescript
// Export as PPTX
exportPPTX(): Promise<void>

// Export as PDF
exportPDF(): Promise<void>
```

#### Task Polling

The store implements automatic task polling for async operations:

```typescript
pollTask(taskId: string): Promise<void> {
  // Poll task status every 2 seconds
  // Update taskProgress state
  // Call syncProject when task completes
}
```

#### Debounced Updates

Local page updates are debounced to avoid excessive API calls:

```typescript
updatePageLocal(pageId: string, data: any) {
  // Update local state immediately (optimistic update)
  // Debounce API call (1 second delay)
}
```

## Pages

### Home Page (`pages/Home.tsx`)

Entry point for creating new projects.

**Features**:
- Three creation modes (tabs):
  - **Idea**: Single text input for idea prompt
  - **Outline**: Text area for structured outline
  - **Description**: Text area for page descriptions
- Template selection
- Material upload
- Reference file upload and parsing

**UI Flow**:
```
1. User selects creation mode
2. User enters content
3. (Optional) Upload template image
4. (Optional) Upload reference files
5. (Optional) Add materials
6. Click "Create Project"
7. Navigate to OutlineEditor or DetailEditor
```

**Key Components**:
```typescript
<Home>
  <Tabs> {/* Idea, Outline, Description */}
  <Textarea content={content} />
  <TemplateSelector onSelect={setSelectedTemplate} />
  <ReferenceFileList files={referenceFiles} />
  <MaterialGeneratorModal />
  <Button onClick={handleCreate}>Create Project</Button>
</Home>
```

### Outline Editor (`pages/OutlineEditor.tsx`)

Edit and refine the presentation outline.

**Features**:
- View/edit outline structure
- AI-powered outline refinement
- Natural language modification
- Drag-and-drop page reordering
- Part/section grouping
- Add/delete pages

**UI Components**:
```typescript
<OutlineEditor>
  <Header>
    <Button onClick={generateDescriptions}>Generate Descriptions</Button>
  </Header>
  
  <OutlineRefinementModal> {/* Natural language editing */}
    <Textarea placeholder="e.g., Add a page about case studies" />
  </OutlineRefinementModal>
  
  <DndContext onDragEnd={handleDragEnd}>
    {pages.map(page => (
      <OutlineCard 
        page={page}
        onUpdate={updatePageLocal}
        onDelete={deletePageById}
      />
    ))}
  </DndContext>
</OutlineEditor>
```

**Natural Language Refinement**:
Users can say things like:
- "Add a page about case studies after slide 3"
- "Change the second slide title to 'Introduction'"
- "Remove the conclusion slide"

### Detail Editor (`pages/DetailEditor.tsx`)

Edit detailed page descriptions.

**Features**:
- View/edit page descriptions
- AI-powered description refinement
- Material selector integration
- Description regeneration
- Visual description preview

**UI Components**:
```typescript
<DetailEditor>
  <Header>
    <Button onClick={generateImages}>Generate Images</Button>
  </Header>
  
  <DescriptionRefinementModal> {/* Natural language editing */}
    <Textarea placeholder="e.g., Make slide 2 more visual" />
  </DescriptionRefinementModal>
  
  {pages.map(page => (
    <DescriptionCard
      page={page}
      onUpdate={updatePageLocal}
      onRegenerate={generatePageDescription}
      materials={materials}
    />
  ))}
</DetailEditor>
```

**Description Structure**:
Each description includes:
- Layout type
- Text elements (title, subtitle, body)
- Image suggestions
- Color scheme
- Design notes

### Slide Preview (`pages/SlidePreview.tsx`)

View and edit generated slides.

**Features**:
- Grid view of all slides
- Full-screen preview
- Image editing with prompts
- Version history
- Material selector for editing
- Export buttons

**UI Components**:
```typescript
<SlidePreview>
  <Header>
    <Button onClick={exportPPTX}>Export PPTX</Button>
    <Button onClick={exportPDF}>Export PDF</Button>
  </Header>
  
  <Grid>
    {pages.map(page => (
      <SlideCard
        page={page}
        onClick={() => setSelectedPage(page)}
        onEdit={handleEditImage}
        onRegenerate={generatePageImage}
      />
    ))}
  </Grid>
  
  <FullScreenModal page={selectedPage}>
    <ImageEditModal
      onSubmit={(prompt, materials) => editPageImage(page.id, prompt, materials)}
    />
    <VersionHistory versions={page.versions} />
  </FullScreenModal>
</SlidePreview>
```

**Image Editing**:
Users can:
- Enter natural language edit prompts
- Select reference materials
- Use template as reference
- View previous versions

### History Page (`pages/History.tsx`)

View and manage all projects.

**Features**:
- List all projects
- Sort by date
- Search/filter
- Open project
- Delete project
- View project metadata

**UI Components**:
```typescript
<History>
  <Header>
    <Button onClick={createNew}>Create New Project</Button>
    <Input placeholder="Search projects..." />
  </Header>
  
  <ProjectGrid>
    {projects.map(project => (
      <ProjectCard
        project={project}
        onClick={() => navigate(`/project/${project.id}`)}
        onDelete={deleteProject}
      />
    ))}
  </ProjectGrid>
</History>
```

## Components

### Shared Components

#### Button (`components/shared/Button.tsx`)

Reusable button component with variants.

```typescript
<Button 
  variant="primary" | "secondary" | "danger"
  size="sm" | "md" | "lg"
  loading={isLoading}
  disabled={isDisabled}
  icon={<IconComponent />}
  onClick={handleClick}
>
  Button Text
</Button>
```

#### Card (`components/shared/Card.tsx`)

Container component for content.

```typescript
<Card 
  title="Card Title"
  subtitle="Optional subtitle"
  actions={<Button>Action</Button>}
>
  Card content
</Card>
```

#### Modal (`components/shared/Modal.tsx`)

Modal dialog component.

```typescript
<Modal
  isOpen={isModalOpen}
  onClose={() => setIsModalOpen(false)}
  title="Modal Title"
  size="sm" | "md" | "lg" | "full"
>
  Modal content
</Modal>
```

#### Loading (`components/shared/Loading.tsx`)

Loading spinner component.

```typescript
<Loading 
  size="sm" | "md" | "lg"
  text="Loading..."
/>
```

#### Toast (`components/shared/Toast.tsx`)

Toast notification hook.

```typescript
const { show, ToastContainer } = useToast();

// Show toast
show('Success message', 'success');
show('Error message', 'error');

// In component
return (
  <>
    {ToastContainer}
    {/* rest of component */}
  </>
);
```

#### MaterialSelector (`components/shared/MaterialSelector.tsx`)

Material selection component.

```typescript
<MaterialSelector
  projectId={projectId}
  selectedMaterials={selectedMaterials}
  onSelect={setSelectedMaterials}
/>
```

#### TemplateSelector (`components/shared/TemplateSelector.tsx`)

Template image selector.

```typescript
<TemplateSelector
  onSelect={(template) => setSelectedTemplate(template)}
  selectedTemplateId={selectedTemplateId}
/>
```

### Preview Components

#### SlideCard (`components/preview/SlideCard.tsx`)

Slide thumbnail card.

```typescript
<SlideCard
  page={page}
  index={pageIndex}
  onClick={handleClick}
  onEdit={handleEdit}
  onRegenerate={handleRegenerate}
  isGenerating={isGenerating}
/>
```

Features:
- Slide preview image
- Page title
- Loading state
- Edit button
- Regenerate button

#### DescriptionCard (`components/preview/DescriptionCard.tsx`)

Description editor card.

```typescript
<DescriptionCard
  page={page}
  onUpdate={(data) => updatePageLocal(page.id, data)}
  onRegenerate={() => generatePageDescription(page.id)}
  materials={materials}
/>
```

Features:
- Rich text description
- Material selector
- Regenerate button
- Auto-save

### Outline Components

#### OutlineCard (`components/outline/OutlineCard.tsx`)

Outline editor card with drag-and-drop.

```typescript
<OutlineCard
  page={page}
  index={pageIndex}
  onUpdate={(data) => updatePageLocal(page.id, data)}
  onDelete={() => deletePageById(page.id)}
  isDragging={isDragging}
/>
```

Features:
- Editable title
- Editable key points
- Part/section grouping
- Drag handle
- Delete button

## API Integration

### API Client (`api/client.ts`)

Configured Axios instance with interceptors.

```typescript
const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL || 'http://localhost:5000',
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor (add auth token, etc.)
apiClient.interceptors.request.use(config => {
  // Modify config
  return config;
});

// Response interceptor (handle errors)
apiClient.interceptors.response.use(
  response => response,
  error => {
    // Handle error globally
    return Promise.reject(error);
  }
);
```

### API Endpoints (`api/endpoints.ts`)

Type-safe API functions.

#### Project Endpoints

```typescript
// Create project
createProject(data: CreateProjectRequest): Promise<ApiResponse<Project>>

// Get project
getProject(projectId: string): Promise<ApiResponse<Project>>

// Update project
updateProject(projectId: string, data: Partial<Project>): Promise<ApiResponse<Project>>

// Delete project
deleteProject(projectId: string): Promise<ApiResponse>

// List projects
listProjects(limit?: number, offset?: number): Promise<ApiResponse<ProjectList>>
```

#### Generation Endpoints

```typescript
// Generate outline
generateOutline(projectId: string): Promise<ApiResponse<Task>>

// Generate descriptions
generateDescriptions(projectId: string): Promise<ApiResponse<Task>>

// Generate images
generateImages(projectId: string): Promise<ApiResponse<Task>>

// Generate single page description
generatePageDescription(projectId: string, pageId: string): Promise<ApiResponse<Task>>

// Generate single page image
generatePageImage(pageId: string, forceRegenerate?: boolean): Promise<ApiResponse<Task>>
```

#### Page Endpoints

```typescript
// Update page
updatePage(projectId: string, pageId: string, data: any): Promise<ApiResponse<Page>>

// Update page outline
updatePageOutline(projectId: string, pageId: string, outline: any): Promise<ApiResponse<Page>>

// Update page description
updatePageDescription(projectId: string, pageId: string, description: any): Promise<ApiResponse<Page>>

// Edit page image
editPageImage(pageId: string, editPrompt: string, contextImages?: any): Promise<ApiResponse<Task>>

// Delete page
deletePage(pageId: string): Promise<ApiResponse>

// Add page
addPage(projectId: string): Promise<ApiResponse<Page>>
```

#### Task Endpoints

```typescript
// Get task status
getTask(taskId: string): Promise<ApiResponse<Task>>
```

#### Export Endpoints

```typescript
// Export PPTX
exportPPTX(projectId: string): Promise<ApiResponse<{ download_url: string }>>

// Export PDF
exportPDF(projectId: string): Promise<ApiResponse<{ download_url: string }>>
```

#### Material & Template Endpoints

```typescript
// Upload material
uploadMaterial(file: File, materialType: string): Promise<ApiResponse<Material>>

// Associate materials to project
associateMaterialsToProject(projectId: string, materialIds: string[]): Promise<ApiResponse>

// Upload reference file
uploadReferenceFile(file: File): Promise<ApiResponse<ReferenceFile>>

// Trigger file parse
triggerFileParse(fileId: string): Promise<ApiResponse>

// Associate file to project
associateFileToProject(fileId: string, projectId: string): Promise<ApiResponse>
```

## Routing

### Route Configuration (`App.tsx`)

```typescript
<BrowserRouter>
  <Routes>
    <Route path="/" element={<Home />} />
    <Route path="/project/:projectId/outline" element={<OutlineEditor />} />
    <Route path="/project/:projectId/details" element={<DetailEditor />} />
    <Route path="/project/:projectId/preview" element={<SlidePreview />} />
    <Route path="/history" element={<History />} />
  </Routes>
</BrowserRouter>
```

### Navigation Flow

```
Home (/)
  ↓ Create Project
OutlineEditor (/project/:id/outline)
  ↓ Generate Descriptions
DetailEditor (/project/:id/details)
  ↓ Generate Images
SlidePreview (/project/:id/preview)
  ↓ Export or Edit More
```

### Programmatic Navigation

```typescript
const navigate = useNavigate();

// Navigate to outline editor
navigate(`/project/${projectId}/outline`);

// Navigate with state
navigate(`/project/${projectId}/preview`, { state: { from: 'details' } });

// Go back
navigate(-1);
```

## Styling

### Tailwind CSS

The app uses Tailwind CSS for styling.

**Common Patterns**:
```typescript
// Container
<div className="container mx-auto px-4 py-8">

// Card
<div className="bg-white rounded-lg shadow-md p-6">

// Button
<button className="bg-blue-500 hover:bg-blue-600 text-white font-medium py-2 px-4 rounded-lg transition-colors">

// Grid
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">

// Flexbox
<div className="flex items-center justify-between">
```

**Responsive Design**:
- Mobile-first approach
- Breakpoints: `sm`, `md`, `lg`, `xl`, `2xl`
- Grid columns adjust based on screen size

### Custom Styles

Custom CSS in component-specific files or `styles/` directory.

## User Workflows

### Workflow 1: Create from Idea

```
1. Home page → Select "Idea" tab
2. Enter idea: "Create a presentation about AI trends"
3. (Optional) Upload template image
4. Click "Create Project"
5. Backend generates outline automatically
6. Navigate to Outline Editor
7. Review/edit outline
8. Click "Generate Descriptions"
9. Navigate to Detail Editor
10. Review/edit descriptions
11. Click "Generate Images"
12. Navigate to Slide Preview
13. Review slides, edit if needed
14. Export to PPTX or PDF
```

### Workflow 2: Create from Outline

```
1. Home page → Select "Outline" tab
2. Enter outline text:
   ```
   # Introduction
   - What is AI
   - Why it matters
   
   # Applications
   - Healthcare
   - Finance
   ```
3. Click "Create Project"
4. Navigate to Outline Editor (outline already populated)
5. Continue from step 7 in Workflow 1
```

### Workflow 3: Create from Description

```
1. Home page → Select "Description" tab
2. Enter descriptions for each slide
3. Click "Create Project"
4. Backend generates outline from descriptions
5. Navigate to Detail Editor (descriptions already populated)
6. Continue from step 11 in Workflow 1
```

### Workflow 4: Edit Existing Slide

```
1. Slide Preview page
2. Click on a slide
3. Click "Edit" button
4. Enter edit prompt: "Change background to blue"
5. (Optional) Select reference materials
6. Click "Apply Edit"
7. Wait for regeneration
8. View new version
9. Can revert to previous version from history
```

## TypeScript Types

### Core Types (`types/index.ts`)

```typescript
interface Project {
  project_id: string;
  idea_prompt?: string;
  outline_text?: string;
  description_text?: string;
  creation_type: 'idea' | 'outline' | 'descriptions';
  template_image_url?: string;
  status: 'DRAFT' | 'COMPLETED';
  pages?: Page[];
  created_at: string;
  updated_at: string;
}

interface Page {
  page_id: string;
  project_id: string;
  order_index: number;
  part?: string;
  outline_content?: OutlineContent;
  description_content?: DescriptionContent;
  image_url?: string;
  created_at: string;
  updated_at: string;
}

interface OutlineContent {
  title: string;
  subtitle?: string;
  key_points?: string[];
}

interface DescriptionContent {
  layout: string;
  text_elements: TextElement[];
  image_suggestions?: string[];
  color_scheme?: string;
  design_notes?: string;
}

interface Task {
  task_id: string;
  task_type: string;
  status: 'pending' | 'running' | 'completed' | 'failed';
  progress?: {
    total: number;
    completed: number;
    message?: string;
  };
  result?: any;
  error_message?: string;
}

interface Material {
  material_id: string;
  filename: string;
  file_url: string;
  material_type: string;
}
```

## Development

### Environment Variables

Create `.env.local`:
```
VITE_API_BASE_URL=http://localhost:5000
```

### Running Dev Server

```bash
cd frontend
npm install
npm run dev
```

Access at `http://localhost:3000`

### Building for Production

```bash
npm run build
```

Output in `dist/` directory.

### Docker Development

```bash
docker compose up frontend
```

## Best Practices

### Performance

1. **Lazy Loading**: Use React.lazy for route-based code splitting
2. **Memoization**: Use useMemo and useCallback for expensive computations
3. **Debouncing**: Debounce API calls (already implemented in store)
4. **Optimistic Updates**: Update UI immediately, sync with server later

### State Management

1. **Single Source of Truth**: Use Zustand store for shared state
2. **Local State**: Use useState for component-specific state
3. **Avoid Prop Drilling**: Access store directly in nested components

### Error Handling

1. **Try-Catch**: Wrap API calls in try-catch
2. **Error Boundaries**: Use React Error Boundaries for component errors
3. **User Feedback**: Show toast notifications for errors

### Code Organization

1. **Component Structure**: One component per file
2. **Type Safety**: Define types for all props and state
3. **Code Splitting**: Separate business logic from UI logic
4. **Naming**: Use descriptive names for components and functions

## Troubleshooting

### Common Issues

**1. API Connection Error**
- Check `VITE_API_BASE_URL` environment variable
- Verify backend is running
- Check CORS configuration

**2. State Not Updating**
- Check if using Zustand store correctly
- Verify API call is successful
- Check browser console for errors

**3. Build Errors**
- Clear `node_modules` and reinstall: `rm -rf node_modules && npm install`
- Check TypeScript errors: `npm run type-check`
- Update dependencies: `npm update`

**4. Slow Performance**
- Check for unnecessary re-renders
- Use React DevTools Profiler
- Optimize images and assets
