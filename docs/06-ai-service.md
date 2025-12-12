# AI Service & Prompts System

This document explains how AI generation works in Banana Slides, including the prompt engineering, provider architecture, and generation pipeline.

## 🤖 Overview

Banana Slides uses AI for multiple tasks:
1. **Text Generation**: Outline and description creation (Gemini/OpenAI)
2. **Image Generation**: Slide visuals (nano banana pro / DALL-E)
3. **Image Editing**: Natural language modifications
4. **Content Refinement**: Iterative improvements

## 🏗️ AI Service Architecture

### Provider Pattern

The system uses a **pluggable provider pattern** to support multiple AI services:

```
AIService
├── TextProvider (Abstract)
│   ├── GeminiTextProvider
│   └── OpenAITextProvider
└── ImageProvider (Abstract)
    ├── GeminiImageProvider
    └── OpenAIImageProvider
```

### Configuration

Set in `.env`:
```env
AI_PROVIDER_FORMAT=gemini  # or "openai"

# Gemini Configuration
GOOGLE_API_KEY=your-key
GOOGLE_API_BASE=https://generativelanguage.googleapis.com
TEXT_MODEL=gemini-2.5-flash
IMAGE_MODEL=gemini-3-pro-image-preview

# OpenAI Configuration  
OPENAI_API_KEY=your-key
OPENAI_API_BASE=https://api.openai.com/v1
TEXT_MODEL=gpt-4
IMAGE_MODEL=dall-e-3
```

## 📝 Text Generation

### 1. Outline Generation

**Purpose**: Convert an idea into a structured outline

**Input**:
- `idea_prompt`: User's idea
- `reference_files_content`: Parsed documents (optional)

**Prompt Template** (`get_outline_generation_prompt`):

```
You are a helpful assistant that generates an outline for a PPT.

You can organize the content in two ways:

1. Simple format (for short PPTs):
[{"title": "title1", "points": ["point1", "point2"]}, ...]

2. Part-based format (for longer PPTs with sections):
[
  {
    "part": "Part 1: Introduction",
    "pages": [
      {"title": "Welcome", "points": ["point1", "point2"]},
      ...
    ]
  },
  ...
]

<uploaded_files>
  <file name="research.pdf">
    <content>...</content>
  </file>
</uploaded_files>

User's idea: {idea_prompt}

Generate a comprehensive outline. Return only valid JSON.
```

**Output**: JSON array of pages/parts

**Processing**:
```python
# 1. Build prompt
prompt = get_outline_generation_prompt(project_context)

# 2. Call text provider
response = text_provider.generate_text(prompt)

# 3. Parse JSON
outline_data = json.loads(response)

# 4. Create Page records
for item in outline_data:
    if 'part' in item:
        # Handle part structure
        for page_data in item['pages']:
            page = Page(part=item['part'], ...)
    else:
        # Handle simple structure
        page = Page(...)
    db.session.add(page)
```

---

### 2. Description Generation

**Purpose**: Convert outline points into detailed content

**Input**:
- `page.outline_content`: {title, points}
- `project_context`: Project info and reference files
- `extra_requirements`: Additional constraints

**Prompt Template** (`get_page_description_prompt`):

```
You are creating detailed content for a PowerPoint slide.

Title: {title}
Key Points:
- {point1}
- {point2}
...

<uploaded_files>...</uploaded_files>

Requirements:
{extra_requirements}

Generate detailed content in markdown format.
Include image descriptions in format: [Include image of: description]

Return JSON:
{
  "title": "...",
  "content": "...",
  "images": [
    {
      "description": "...",
      "position": "center",
      "size": "medium"
    }
  ]
}
```

**Output**: JSON with title, markdown content, and image specs

**Parallel Processing**:
```python
# Use ThreadPoolExecutor for parallel generation
with ThreadPoolExecutor(max_workers=MAX_DESCRIPTION_WORKERS) as executor:
    futures = []
    for page in pages:
        future = executor.submit(ai_service.generate_page_description, page)
        futures.append((page, future))
    
    for page, future in futures:
        description = future.result()
        page.set_description_content(description)
```

---

### 3. Content Refinement

**Purpose**: Modify outline or descriptions based on natural language instructions

**Input**:
- `current_outline/descriptions`: Current content
- `user_requirement`: Natural language modification request
- `previous_requirements`: History of changes

**Prompt Template** (`get_outline_refinement_prompt`):

```
You are refining a PPT outline based on user feedback.

Current Outline:
[{current_outline_json}]

Previous Modifications:
1. {previous_requirement_1}
2. {previous_requirement_2}
...

New Requirement:
{user_requirement}

Modify the outline accordingly. Return only valid JSON.
```

**Smart Modification**:
- Maintains structure
- Applies changes contextually
- Preserves unaffected content

---

## 🎨 Image Generation

### 1. Initial Image Generation

**Purpose**: Create slide images from descriptions

**Input**:
- `page.description_content`: Content and layout
- `template_image`: Style reference (optional)
- `reference_images`: Content images from description

**Prompt Template** (`get_image_generation_prompt`):

```
Create a professional PowerPoint slide for a presentation.

Title: {title}
Content: {content}

Visual Elements:
{image_descriptions}

Style Requirements:
- 16:9 aspect ratio
- Professional design
- Clear typography
- Similar style to reference image

Additional Requirements:
{extra_requirements}
```

**Context Images**:
```python
context_images = []

# Add template for style consistency
if template_path:
    context_images.append(Image.open(template_path))

# Add reference images from markdown
for img_url in extract_image_urls_from_markdown(content):
    img = load_image(img_url)
    context_images.append(img)
```

**Generation**:
```python
# Call image provider
image_data = image_provider.generate_image(
    prompt=generation_prompt,
    context_images=context_images,
    aspect_ratio="16:9",
    resolution="2K"
)

# Save image
save_path = f"uploads/{project_id}/pages/{page_id}/v{version}.png"
with open(save_path, 'wb') as f:
    f.write(image_data)
```

**Parallel Processing**:
```python
# Generate all slides in parallel
with ThreadPoolExecutor(max_workers=MAX_IMAGE_WORKERS) as executor:
    futures = {
        executor.submit(generate_single_image, page): page
        for page in pages
    }
    
    for future in as_completed(futures):
        page = futures[future]
        image_url = future.result()
        page.image_url = image_url
```

---

### 2. Image Editing

**Purpose**: Modify existing images with natural language

**Input**:
- `current_image`: Existing slide image
- `edit_instruction`: Natural language edit request
- `context_images`: Additional references

**Prompt Template** (`get_image_edit_prompt`):

```
You are editing an existing PowerPoint slide.

Current Slide Content:
Title: {title}
Content: {content}

Edit Instruction:
{edit_instruction}

Modify the slide while maintaining:
- Overall structure
- 16:9 aspect ratio
- Professional style
- Text readability

Apply the requested changes precisely.
```

**Edit Mode**:
```python
# Include current image in context
context_images = [Image.open(current_image_path)]

# Add user-provided references
if uploaded_images:
    context_images.extend(uploaded_images)

# Add template for style consistency
if template_path:
    context_images.append(Image.open(template_path))

# Generate edited version
edited_image = image_provider.edit_image(
    prompt=edit_prompt,
    context_images=context_images
)

# Save as new version
version_number = get_next_version(page)
save_path = f"uploads/{project_id}/pages/{page_id}/v{version_number}.png"
```

---

## 🔧 Provider Implementation

### Text Provider Interface

```python
class TextProvider(ABC):
    @abstractmethod
    def generate_text(self, prompt: str, **kwargs) -> str:
        """Generate text from prompt"""
        pass
```

### Gemini Text Provider

```python
class GeminiTextProvider(TextProvider):
    def __init__(self, api_key: str, model: str):
        self.client = genai.Client(api_key=api_key)
        self.model = model
    
    def generate_text(self, prompt: str, **kwargs) -> str:
        response = self.client.models.generate_content(
            model=self.model,
            contents=prompt,
            config={
                'temperature': kwargs.get('temperature', 0.7),
                'max_output_tokens': kwargs.get('max_tokens', 8192)
            }
        )
        return response.text
```

### OpenAI Text Provider

```python
class OpenAITextProvider(TextProvider):
    def __init__(self, api_key: str, api_base: str, model: str):
        self.client = OpenAI(api_key=api_key, base_url=api_base)
        self.model = model
    
    def generate_text(self, prompt: str, **kwargs) -> str:
        response = self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": prompt}],
            temperature=kwargs.get('temperature', 0.7),
            max_tokens=kwargs.get('max_tokens', 4096)
        )
        return response.choices[0].message.content
```

### Image Provider Interface

```python
class ImageProvider(ABC):
    @abstractmethod
    def generate_image(
        self, 
        prompt: str, 
        context_images: List[Image.Image] = None,
        **kwargs
    ) -> bytes:
        """Generate image from prompt"""
        pass
    
    @abstractmethod
    def edit_image(
        self,
        prompt: str,
        context_images: List[Image.Image] = None,
        **kwargs
    ) -> bytes:
        """Edit image based on instruction"""
        pass
```

---

## 🎯 Prompt Engineering Best Practices

### 1. Structure

- **Clear role definition**: "You are a helpful assistant..."
- **Explicit format**: JSON schemas, markdown structure
- **Context first**: Provide all info before the task
- **Examples**: Show expected output format

### 2. Context Management

```python
class ProjectContext:
    """Centralized context for AI calls"""
    def __init__(self, project, reference_files_content):
        self.idea_prompt = project.idea_prompt
        self.outline_text = project.outline_text
        self.description_text = project.description_text
        self.creation_type = project.creation_type
        self.reference_files_content = reference_files_content
```

Benefits:
- Consistent context across calls
- Easy to modify prompts
- Efficient data passing

### 3. Reference Files Integration

```xml
<uploaded_files>
  <file name="research.pdf">
    <content>
      # Extracted markdown content
      ...
    </content>
  </file>
</uploaded_files>
```

Why XML?
- Clear structure
- Easy to parse
- Works well with AI models

---

## 🚀 Performance Optimization

### 1. Parallel Processing

```python
# Don't do this (sequential)
for page in pages:
    description = generate_description(page)  # Slow!

# Do this (parallel)
with ThreadPoolExecutor(max_workers=5) as executor:
    futures = [executor.submit(generate_description, p) for p in pages]
    descriptions = [f.result() for f in futures]
```

**Benefits**:
- 5x faster for 5 pages
- Better API utilization
- Improved user experience

### 2. Caching

```python
# Cache reference file content
@lru_cache(maxsize=100)
def get_reference_files_content(project_id):
    return ReferenceFile.query.filter_by(
        project_id=project_id,
        parse_status='completed'
    ).all()
```

### 3. Streaming (Future Enhancement)

```python
# Stream tokens as they're generated
def generate_text_stream(prompt):
    for chunk in provider.generate_stream(prompt):
        yield chunk
```

---

## 🔍 Debugging AI Outputs

### Logging

```python
logger.debug(f"Prompt sent to AI:\n{prompt}")
logger.debug(f"AI response:\n{response}")
```

### Response Validation

```python
def validate_outline_response(response):
    try:
        data = json.loads(response)
        assert isinstance(data, list)
        for item in data:
            assert 'title' in item or 'part' in item
        return data
    except Exception as e:
        logger.error(f"Invalid outline response: {e}")
        raise ValueError("AI returned invalid format")
```

### Fallback Strategies

```python
def generate_with_fallback(prompt):
    try:
        # Try primary provider
        return primary_provider.generate(prompt)
    except Exception as e:
        logger.warning(f"Primary provider failed: {e}")
        # Fallback to secondary
        return secondary_provider.generate(prompt)
```

---

## 📊 Token Usage & Costs

### Estimating Costs

```python
def estimate_tokens(text):
    # Rough estimate: 1 token ≈ 4 characters
    return len(text) // 4

# Example usage
prompt_tokens = estimate_tokens(prompt)
logger.info(f"Estimated tokens: {prompt_tokens}")
```

### Monitoring

```python
class CostTracker:
    def __init__(self):
        self.total_tokens = 0
        self.total_cost = 0.0
    
    def track_generation(self, prompt, response):
        tokens = estimate_tokens(prompt + response)
        cost = tokens * COST_PER_1K_TOKENS / 1000
        self.total_tokens += tokens
        self.total_cost += cost
```

---

## 🎓 Advanced Techniques

### Chain of Thought

```python
prompt = f"""
Let's think step by step:
1. Analyze the topic: {topic}
2. Identify key points
3. Structure the outline
4. Format as JSON

Now, generate the outline:
"""
```

### Few-Shot Learning

```python
prompt = f"""
Example 1:
Input: "Presentation about AI"
Output: {{"title": "Introduction to AI", "points": [...]}}

Example 2:
Input: "Marketing strategy"
Output: {{"title": "Marketing Overview", "points": [...]}}

Now, your turn:
Input: "{user_input}"
Output:
"""
```

### Iterative Refinement

```python
# First pass: Generate
outline_v1 = generate_outline(idea)

# Second pass: Refine
refinement_prompt = f"""
Review this outline and improve it:
{outline_v1}

Make it more detailed and structured.
"""
outline_v2 = generate_outline(refinement_prompt)
```

---

## 🔐 Security Considerations

### Input Sanitization

```python
def sanitize_prompt(user_input):
    # Remove potential injection attacks
    user_input = user_input.replace("</system>", "")
    user_input = user_input.replace("<|endoftext|>", "")
    return user_input
```

### Output Validation

```python
def validate_json_output(response):
    # Ensure response is valid JSON
    try:
        data = json.loads(response)
        return data
    except json.JSONDecodeError:
        # Extract JSON if wrapped in markdown
        match = re.search(r'```json\n(.*?)\n```', response, re.DOTALL)
        if match:
            return json.loads(match.group(1))
        raise ValueError("Invalid JSON response")
```

---

## 📚 Related Documentation

- [Architecture Overview](./02-architecture.md) - System design
- [Workflow Guide](./03-workflow.md) - How AI fits in the workflow
- [API Reference](./04-api-reference.md) - Generation endpoints

---

**Next**: Learn about [Frontend Components](./07-frontend-components.md).
