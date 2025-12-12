# Frontend Components Guide

This document explains the frontend structure, key components, and how they work together.

## 🎨 Frontend Stack

- **React 18**: UI framework with hooks
- **TypeScript**: Type-safe development
- **Vite**: Fast build tool
- **Tailwind CSS**: Utility-first styling
- **Zustand**: Lightweight state management
- **React Router**: Client-side routing
- **Axios**: HTTP client

## 📁 Directory Structure

```
frontend/src/
├── api/              # API layer
├── components/       # UI components
├── pages/            # Route pages
├── store/            # State management
├── types/            # TypeScript types
├── utils/            # Utility functions
└── App.tsx           # Root component
```

## 🔌 API Layer (`src/api/`)

### `client.ts` - HTTP Client

Configures Axios with base URL and interceptors:

```typescript
const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:5000',
  headers: {
    'Content-Type': 'application/json'
  }
});

// Request interceptor (can add auth tokens here)
apiClient.interceptors.request.use(config => {
  // Add auth header if needed
  return config;
});

// Response interceptor (handle errors globally)
apiClient.interceptors.response.use(
  response => response,
  error => {
    console.error('API Error:', error);
    return Promise.reject(error);
  }
);
```

### `endpoints.ts` - API Functions

All API calls are defined here:

```typescript
// Project operations
export const createProject = async (data: CreateProjectRequest) => {
  const response = await apiClient.post('/api/projects', data);
  return response.data;
};

export const getProject = async (projectId: string) => {
  const response = await apiClient.get(`/api/projects/${projectId}`);
  return response.data;
};

// Generation operations
export const generateOutline = async (projectId: string) => {
  const response = await apiClient.post(
    `/api/projects/${projectId}/generate/outline`, 
    {}
  );
  return response.data;
};

// ... more endpoints
```

**Benefits**:
- Centralized API logic
- Type-safe requests/responses
- Easy to mock for testing
- Consistent error handling

## 🗂️ State Management (`src/store/`)

### `useProjectStore.ts` - Zustand Store

Global state for project management:

```typescript
interface ProjectStore {
  // State
  currentProject: Project | null;
  pages: Page[];
  isLoading: boolean;
  isGlobalLoading: boolean;
  
  // Actions
  initializeProject: (data: CreateProjectRequest) => Promise<void>;
  fetchProject: (projectId: string) => Promise<void>;
  updatePageOrder: (pageIds: string[]) => Promise<void>;
  addPage: (pageData: Partial<Page>) => Promise<void>;
  deletePage: (pageId: string) => Promise<void>;
  // ... more actions
}

const useProjectStore = create<ProjectStore>((set, get) => ({
  currentProject: null,
  pages: [],
  isLoading: false,
  isGlobalLoading: false,
  
  initializeProject: async (data) => {
    set({ isGlobalLoading: true });
    try {
      const response = await createProject(data);
      set({ currentProject: response.data });
      localStorage.setItem('currentProjectId', response.data.project_id);
      // Start polling for outline generation
      // ...
    } finally {
      set({ isGlobalLoading: false });
    }
  },
  
  // ... more implementations
}));
```

**Usage in Components**:
```typescript
function MyComponent() {
  const { currentProject, fetchProject, isLoading } = useProjectStore();
  
  useEffect(() => {
    fetchProject('project-id');
  }, []);
  
  return (
    <div>
      {isLoading ? 'Loading...' : currentProject?.idea_prompt}
    </div>
  );
}
```

## 📄 Pages (`src/pages/`)

### `Home.tsx` - Project Creation

Entry point for creating new presentations.

**Key Features**:
- Three creation modes (idea, outline, descriptions)
- Template selector
- Reference file upload
- Material preview

**State Management**:
```typescript
const [activeTab, setActiveTab] = useState<CreationType>('idea');
const [content, setContent] = useState('');
const [selectedTemplate, setSelectedTemplate] = useState<File | null>(null);
const [referenceFiles, setReferenceFiles] = useState<ReferenceFile[]>([]);
```

**Creation Flow**:
```typescript
const handleCreate = async () => {
  const projectData = {
    creation_type: activeTab,
    idea_prompt: activeTab === 'idea' ? content : undefined,
    outline_text: activeTab === 'outline' ? content : undefined,
    description_text: activeTab === 'description' ? content : undefined
  };
  
  await initializeProject(projectData);
  navigate(`/outline/${projectId}`);
};
```

---

### `OutlineEditor.tsx` - Outline Editing

Edit outline structure before generating descriptions.

**Key Features**:
- Drag-and-drop reordering (@dnd-kit)
- Inline editing
- AI refinement
- Progress tracking

**Component Structure**:
```tsx
<div className="outline-editor">
  <Header>
    <AiRefineInput onRefine={handleRefine} />
    <Button onClick={generateDescriptions}>Next: Generate Descriptions</Button>
  </Header>
  
  <DndContext onDragEnd={handleDragEnd}>
    <SortableContext items={pages}>
      {pages.map(page => (
        <OutlineCard
          key={page.page_id}
          page={page}
          onEdit={handleEdit}
          onDelete={handleDelete}
        />
      ))}
    </SortableContext>
  </DndContext>
  
  <TaskStatusIndicator taskId={currentTaskId} />
</div>
```

**Drag and Drop**:
```typescript
const handleDragEnd = (event: DragEndEvent) => {
  const { active, over } = event;
  if (over && active.id !== over.id) {
    const oldIndex = pages.findIndex(p => p.page_id === active.id);
    const newIndex = pages.findIndex(p => p.page_id === over.id);
    const newOrder = arrayMove(pages, oldIndex, newIndex);
    updatePageOrder(newOrder.map(p => p.page_id));
  }
};
```

---

### `DetailEditor.tsx` - Description Editing

Edit detailed content for each page.

**Key Features**:
- Markdown editor for each page
- Image specification
- AI refinement
- Real-time preview

**Markdown Editing**:
```tsx
<Textarea
  value={description.content}
  onChange={(e) => updateDescription(page.page_id, {
    ...description,
    content: e.target.value
  })}
  placeholder="Enter detailed content in markdown..."
/>
```

---

### `SlidePreview.tsx` - Preview and Export

Preview generated slides and export to PPTX/PDF.

**Key Features**:
- Grid/list view of all slides
- Full-screen preview
- Image editing modal
- Version history
- Export buttons

**Export Flow**:
```typescript
const handleExportPPTX = async () => {
  const response = await exportPPTX(projectId);
  const downloadUrl = response.data.download_url_absolute;
  window.open(downloadUrl, '_blank');
};
```

---

### `History.tsx` - Project List

Browse all created projects.

**Key Features**:
- Project cards with thumbnails
- Search and filter
- Delete projects
- Navigate to any editing stage

## 🧩 Shared Components (`src/components/shared/`)

### `Button.tsx` - Reusable Button

```typescript
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  disabled?: boolean;
  onClick?: () => void;
  children: React.ReactNode;
}

export const Button: React.FC<ButtonProps> = ({
  variant = 'primary',
  size = 'md',
  loading = false,
  disabled = false,
  onClick,
  children
}) => {
  const className = cn(
    'btn',
    `btn-${variant}`,
    `btn-${size}`,
    loading && 'btn-loading',
    disabled && 'btn-disabled'
  );
  
  return (
    <button
      className={className}
      onClick={onClick}
      disabled={disabled || loading}
    >
      {loading && <Spinner />}
      {children}
    </button>
  );
};
```

---

### `Modal.tsx` - Generic Modal

```typescript
interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title?: string;
  children: React.ReactNode;
}

export const Modal: React.FC<ModalProps> = ({
  isOpen,
  onClose,
  title,
  children
}) => {
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={e => e.stopPropagation()}>
        {title && <h2 className="modal-title">{title}</h2>}
        <button className="modal-close" onClick={onClose}>×</button>
        {children}
      </div>
    </div>
  );
};
```

---

### `Toast.tsx` - Notifications

```typescript
export const useToast = () => {
  const [toasts, setToasts] = useState<Toast[]>([]);
  
  const show = (message: string, type: 'success' | 'error' | 'info' = 'info') => {
    const id = Date.now();
    setToasts(prev => [...prev, { id, message, type }]);
    
    setTimeout(() => {
      setToasts(prev => prev.filter(t => t.id !== id));
    }, 3000);
  };
  
  const ToastContainer = () => (
    <div className="toast-container">
      {toasts.map(toast => (
        <div key={toast.id} className={`toast toast-${toast.type}`}>
          {toast.message}
        </div>
      ))}
    </div>
  );
  
  return { show, ToastContainer };
};
```

**Usage**:
```typescript
const { show, ToastContainer } = useToast();

const handleSuccess = () => {
  show('Project created successfully!', 'success');
};

return (
  <>
    <Button onClick={handleSuccess}>Create</Button>
    <ToastContainer />
  </>
);
```

---

### `Markdown.tsx` - Markdown Renderer

```typescript
import ReactMarkdown from 'react-markdown';
import remarkGfm from 'remark-gfm';
import remarkBreaks from 'remark-breaks';
import rehypeRaw from 'rehype-raw';

export const Markdown: React.FC<{ content: string }> = ({ content }) => (
  <ReactMarkdown
    remarkPlugins={[remarkGfm, remarkBreaks]}
    rehypePlugins={[rehypeRaw]}
    components={{
      img: ({ src, alt }) => (
        <img src={src} alt={alt} className="markdown-image" loading="lazy" />
      ),
      code: ({ inline, children }) =>
        inline ? (
          <code className="inline-code">{children}</code>
        ) : (
          <pre className="code-block"><code>{children}</code></pre>
        )
    }}
  >
    {content}
  </ReactMarkdown>
);
```

---

## 🎨 Styling with Tailwind CSS

### Configuration (`tailwind.config.js`)

```javascript
export default {
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        primary: {...},
        secondary: {...}
      },
      animation: {
        shimmer: 'shimmer 2s infinite',
      }
    }
  },
  plugins: []
};
```

### Usage Examples

```tsx
// Utility classes
<div className="flex items-center justify-between p-4 bg-white rounded-lg shadow-md">
  <h1 className="text-2xl font-bold text-gray-900">Title</h1>
  <Button className="ml-auto">Action</Button>
</div>

// Responsive design
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {/* Cards */}
</div>

// Custom utilities with clsx
import { clsx } from 'clsx';

const buttonClass = clsx(
  'px-4 py-2 rounded',
  isActive && 'bg-blue-500',
  isDisabled && 'opacity-50 cursor-not-allowed'
);
```

## 🔧 TypeScript Types (`src/types/`)

### Core Types

```typescript
export interface Project {
  project_id: string;
  idea_prompt: string | null;
  outline_text: string | null;
  description_text: string | null;
  creation_type: 'idea' | 'outline' | 'descriptions';
  template_image_url: string | null;
  status: string;
  pages?: Page[];
  created_at: string;
  updated_at: string;
}

export interface Page {
  page_id: string;
  project_id: string;
  order_index: number;
  part: string | null;
  outline_content: {
    title: string;
    points: string[];
  } | null;
  description_content: {
    title: string;
    content: string;
    images: Array<{
      description: string;
      position: string;
      size: string;
    }>;
  } | null;
  image_url: string | null;
  created_at: string;
  updated_at: string;
}

export interface Task {
  task_id: string;
  task_type: string;
  status: 'pending' | 'running' | 'completed' | 'failed';
  progress_message: string | null;
  result: any;
  error: string | null;
}

export interface ApiResponse<T = any> {
  success: boolean;
  message?: string;
  data?: T;
  error?: string;
}
```

## 🚀 Performance Optimization

### Lazy Loading

```typescript
// Lazy load pages
const Home = lazy(() => import('./pages/Home'));
const OutlineEditor = lazy(() => import('./pages/OutlineEditor'));

// Use with Suspense
<Suspense fallback={<Loading />}>
  <Routes>
    <Route path="/" element={<Home />} />
    <Route path="/outline/:id" element={<OutlineEditor />} />
  </Routes>
</Suspense>
```

### Memoization

```typescript
// Memoize expensive computations
const sortedPages = useMemo(() => {
  return pages.sort((a, b) => a.order_index - b.order_index);
}, [pages]);

// Memoize components
const PageCard = memo(({ page }: { page: Page }) => {
  return <div>{page.outline_content?.title}</div>;
});
```

### Debouncing

```typescript
import { debounce } from 'lodash';

const debouncedSearch = useMemo(
  () => debounce((query: string) => {
    // Perform search
  }, 500),
  []
);
```

## 📚 Related Documentation

- [Architecture](./02-architecture.md) - System overview
- [Workflow](./03-workflow.md) - User journey
- [API Reference](./04-api-reference.md) - Backend API

---

**Next**: Check out the [Troubleshooting Guide](./09-troubleshooting.md) for common issues.
