# File Processing & Parsing

This document explains how Banana Slides processes uploaded files to extract content.

## 📂 Supported File Types

| Category | Extensions | Description |
|----------|-----------|-------------|
| **Documents** | `.pdf` | PDF documents |
| **Word** | `.docx`, `.doc` | Microsoft Word documents |
| **PowerPoint** | `.pptx`, `.ppt` | Microsoft PowerPoint presentations |
| **Excel** | `.xlsx`, `.xls` | Microsoft Excel spreadsheets |
| **Text** | `.txt`, `.md` | Plain text and Markdown files |
| **Data** | `.csv` | Comma-separated values |

## 🔄 Processing Workflow

```
Upload File
    ↓
Save to Disk
    ↓
Create ReferenceFile Record (status: "pending")
    ↓
Start Background Parsing Task
    ↓
Try MinerU API (if configured)
    ↓ (fallback)
Try markitdown Library
    ↓
Extract Text & Images
    ↓
Convert to Markdown
    ↓
Save to Database (status: "completed")
    ↓
Available for AI Context
```

## 🛠️ Parsing Implementation

### FileParserService

Located in `backend/services/file_parser_service.py`

**Main Functions**:

```python
class FileParserService:
    def parse_file_async(self, file_path: str, file_id: str):
        """Parse file in background thread"""
        thread = Thread(target=self._parse_file_worker, args=(file_path, file_id))
        thread.daemon = True
        thread.start()
    
    def _parse_file_worker(self, file_path: str, file_id: str):
        """Worker function for parsing"""
        try:
            # Update status
            ref_file = ReferenceFile.query.get(file_id)
            ref_file.parse_status = 'parsing'
            db.session.commit()
            
            # Try MinerU first
            if MINERU_TOKEN:
                content = self._parse_with_mineru(file_path)
            else:
                # Fallback to markitdown
                content = self._parse_with_markitdown(file_path)
            
            # Save result
            ref_file.markdown_content = content
            ref_file.parse_status = 'completed'
            db.session.commit()
            
        except Exception as e:
            ref_file.parse_status = 'failed'
            ref_file.error_message = str(e)
            db.session.commit()
```

### MinerU Integration

**MinerU** is an advanced document parsing service with better layout recognition.

**Configuration**:
```env
MINERU_TOKEN=your-token-here
MINERU_API_BASE=https://mineru.net
```

**API Call**:
```python
def _parse_with_mineru(self, file_path: str) -> str:
    """Parse file using MinerU API"""
    url = f"{MINERU_API_BASE}/api/parse"
    
    with open(file_path, 'rb') as f:
        files = {'file': f}
        headers = {'Authorization': f'Bearer {MINERU_TOKEN}'}
        response = requests.post(url, files=files, headers=headers)
    
    if response.status_code == 200:
        data = response.json()
        return data['markdown_content']
    else:
        raise Exception(f"MinerU failed: {response.status_code}")
```

**Features**:
- Better PDF layout recognition
- Table extraction
- Image extraction
- Multi-column support

### markitdown Library

**markitdown** is the fallback parser using local processing.

**Installation**:
```bash
pip install markitdown[all]
```

**Usage**:
```python
from markitdown import MarkItDown

def _parse_with_markitdown(self, file_path: str) -> str:
    """Parse file using markitdown library"""
    md = MarkItDown()
    result = md.convert(file_path)
    return result.text_content
```

**Features**:
- Works offline
- No API limits
- Supports all file types
- Fast for simple documents

## 📊 Markdown Output Format

### Text Extraction

```markdown
# Document Title

## Section 1

Paragraph text with **bold** and *italic*.

- Bullet point 1
- Bullet point 2

## Section 2

More content...
```

### Image Extraction

Images are extracted and saved locally, then referenced in markdown:

```markdown
![Chart showing sales data](/files/mineru/project-123/image-1.png)

Caption: Sales growth over Q1-Q4
```

**Image Storage**:
```
uploads/
└── mineru/
    └── {project_id}/
        ├── image-1.png
        ├── image-2.png
        └── ...
```

### Table Extraction

Tables are converted to markdown format:

```markdown
| Column 1 | Column 2 | Column 3 |
|----------|----------|----------|
| Data 1   | Data 2   | Data 3   |
| Data 4   | Data 5   | Data 6   |
```

## 🎨 Using Parsed Content

### In AI Prompts

Parsed content is included in AI generation prompts:

```xml
<uploaded_files>
  <file name="research.pdf">
    <content>
      # Research Paper Title
      
      ## Introduction
      Extracted content from PDF...
      
      ![Figure 1](/files/mineru/project-123/image-1.png)
    </content>
  </file>
  
  <file name="data.xlsx">
    <content>
      ## Sheet 1: Sales Data
      
      | Month | Revenue | Growth |
      |-------|---------|--------|
      | Jan   | $10,000 | 5%     |
      | Feb   | $12,000 | 20%    |
    </content>
  </file>
</uploaded_files>
```

### In Slide Generation

The AI uses this content to:
- Extract key points for outline
- Generate detailed descriptions
- Identify relevant images
- Include data in slides

## 🔍 Image Recognition

Images extracted from documents can be analyzed using AI.

**Image Caption Generation**:
```python
def generate_image_caption(image_path: str) -> str:
    """Generate description of image using AI"""
    with Image.open(image_path) as img:
        prompt = "Describe this image in detail:"
        response = vision_model.analyze(img, prompt)
        return response.text
```

**Integration**:
```markdown
![Bar chart](/files/mineru/project-123/chart.png)

*AI-generated caption: Bar chart showing quarterly revenue growth from Q1 to Q4 2024, with values ranging from $50K to $200K.*
```

## ⚡ Performance Considerations

### File Size Limits

```python
# In config.py
MAX_CONTENT_LENGTH = 200 * 1024 * 1024  # 200MB
```

### Async Processing

```python
# Parsing happens in background thread
# User can continue working while file is parsed
# Status updates: pending → parsing → completed
```

### Caching

```python
# Parsed content is cached in database
# No need to re-parse on subsequent accesses
```

## 🐛 Error Handling

### Common Errors

1. **Corrupted File**:
   ```
   Error: File is corrupted or invalid format
   ```

2. **MinerU API Failure**:
   ```
   MinerU parsing failed, falling back to markitdown
   ```

3. **Unsupported Format**:
   ```
   Error: File type not supported
   ```

4. **Parsing Timeout**:
   ```
   Error: Parsing took too long
   ```

### Recovery

```python
# Auto-retry with fallback
try:
    content = parse_with_mineru(file_path)
except MinerUException:
    logger.warning("MinerU failed, using markitdown")
    content = parse_with_markitdown(file_path)
```

## 🎯 Best Practices

### For Users

1. **Use clean PDFs**: Scanned PDFs may have OCR errors
2. **Structure documents**: Use headings and lists
3. **Optimize images**: Compress large images
4. **Test parsing**: Check parsed content before generating

### For Developers

1. **Validate files**: Check format before parsing
2. **Handle timeouts**: Set reasonable limits
3. **Clean up**: Delete temporary files
4. **Log errors**: Track parsing failures

## 🔮 Future Enhancements

Planned improvements:

- [ ] OCR for scanned documents
- [ ] Better table recognition
- [ ] Multi-language support
- [ ] Video/audio extraction
- [ ] Streaming large files
- [ ] Progress indicators
- [ ] Preview before upload

## 📚 Related Documentation

- [Architecture](./02-architecture.md) - System design
- [Workflow](./03-workflow.md) - How files fit in workflow
- [AI Service](./06-ai-service.md) - How content is used

---

**Next**: Check the [Troubleshooting Guide](./09-troubleshooting.md) for file parsing issues.
