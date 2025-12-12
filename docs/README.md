# Banana Slides - Complete Documentation

Welcome to the Banana Slides documentation! This guide will help you understand the entire codebase, architecture, workflow, APIs, and everything you need to know as a first-time user.

## 📚 Documentation Structure

This documentation is organized into several sections:

1. **[Getting Started](./01-getting-started.md)** - Quick start guide to set up and run the application
2. **[Architecture Overview](./02-architecture.md)** - High-level architecture and tech stack
3. **[Workflow Guide](./03-workflow.md)** - Complete workflow from idea to PPT export
4. **[Backend API Reference](./04-api-reference.md)** - Detailed API endpoints documentation
5. **[Database Models](./05-database-models.md)** - Database schema and models explanation
6. **[AI Service & Prompts](./06-ai-service.md)** - How AI generation works
7. **[Frontend Components](./07-frontend-components.md)** - Frontend structure and components
8. **[File Processing](./08-file-processing.md)** - How files are parsed and processed
9. **[Troubleshooting](./09-troubleshooting.md)** - Common issues and solutions
10. **[Contributing Guide](./10-contributing.md)** - How to contribute to the project

## 🎯 What is Banana Slides?

Banana Slides is an AI-native PowerPoint generation application built on top of the nano banana pro🍌 model. It transforms your ideas, outlines, or detailed descriptions into complete, professional PPT presentations.

### Key Features

- **Multiple Creation Paths**: Start from an idea, outline, or detailed description
- **AI-Powered Generation**: Uses Gemini/OpenAI models for content and nano banana for images
- **Natural Language Editing**: Modify slides using conversational instructions
- **Smart File Parsing**: Upload PDFs, Word docs, and more for content extraction
- **Material Management**: Generate and organize custom materials and templates
- **Export Ready**: Export to PPTX or PDF format, ready for presentation

## 🏗️ Quick Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                         Frontend                             │
│  React + TypeScript + Vite + Tailwind CSS + Zustand         │
│                                                              │
│  Pages: Home → OutlineEditor → DetailEditor → SlidePreview  │
└──────────────────────┬───────────────────────────────────────┘
                       │ REST API (Axios)
┌──────────────────────┴───────────────────────────────────────┐
│                         Backend                              │
│           Flask + Python + SQLAlchemy + SQLite               │
│                                                              │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │ Controllers │  │   Services   │  │    Models    │       │
│  │  (Routes)   │→│ (Business    │→│  (Database)  │       │
│  │             │  │   Logic)     │  │              │       │
│  └─────────────┘  └──────────────┘  └──────────────┘       │
│                                                              │
│  Services:                                                   │
│  • AI Service (Text + Image generation)                     │
│  • File Parser (PDF, DOCX, etc.)                            │
│  • Export Service (PPTX, PDF)                               │
│  • Task Manager (Async operations)                          │
└──────────────────────┬───────────────────────────────────────┘
                       │
┌──────────────────────┴───────────────────────────────────────┐
│                    External Services                         │
│                                                              │
│  • Google Gemini API / OpenAI API (Text generation)         │
│  • nano banana pro (Image generation)                       │
│  • MinerU (Advanced file parsing - optional)                │
└──────────────────────────────────────────────────────────────┘
```

## 🚀 Getting Started

### Prerequisites

- **Docker + Docker Compose** (recommended) OR
- **Python 3.10+** and **Node.js 16+** (for development)
- Valid API keys for Gemini or OpenAI

### Quick Start with Docker

```bash
# Clone the repository
git clone https://github.com/Anionex/banana-slides
cd banana-slides

# Copy and configure environment variables
cp .env.example .env
# Edit .env with your API keys

# Start the application
docker compose up -d

# Access the application
# Frontend: http://localhost:3000
# Backend: http://localhost:5000
```

For detailed installation instructions, see **[Getting Started Guide](./01-getting-started.md)**.

## 📖 How to Use This Documentation

### If you're a first-time user:
1. Start with **[Getting Started](./01-getting-started.md)** to set up the application
2. Read **[Workflow Guide](./03-workflow.md)** to understand how to use the app
3. Check **[Troubleshooting](./09-troubleshooting.md)** if you encounter issues

### If you're a developer:
1. Review **[Architecture Overview](./02-architecture.md)** to understand the system design
2. Study **[Backend API Reference](./04-api-reference.md)** and **[Database Models](./05-database-models.md)**
3. Explore **[AI Service & Prompts](./06-ai-service.md)** to understand AI integration
4. Check **[Frontend Components](./07-frontend-components.md)** for UI structure

### If you want to contribute:
1. Read **[Contributing Guide](./10-contributing.md)** for guidelines
2. Understand the architecture and workflow
3. Check existing issues or create new ones

## 🎓 Learning Path

```
┌─────────────────┐
│ Getting Started │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Architecture   │ ← Understand the big picture
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│    Workflow     │ ← How data flows through the app
└────────┬────────┘
         │
         ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  API Reference  │────▶│ Database Models │────▶│  AI Service     │
└─────────────────┘     └─────────────────┘     └─────────────────┘
         │
         ▼
┌─────────────────┐
│ Frontend Guide  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Contributing   │
└─────────────────┘
```

## 💡 Key Concepts

Before diving deep, here are some important concepts:

- **Project**: The main entity representing a PPT presentation
- **Page**: Individual slides within a project
- **Outline**: High-level structure with titles and bullet points
- **Description**: Detailed content for each slide
- **Material**: Images and assets (global or project-specific)
- **Template**: Style reference images for consistent design
- **Reference File**: Uploaded documents (PDF, DOCX, etc.) for content extraction
- **Task**: Async operations tracked by the backend

## 🔗 Quick Links

- [GitHub Repository](https://github.com/Anionex/banana-slides)
- [Main README](../README.md)
- [Issue Tracker](https://github.com/Anionex/banana-slides/issues)
- [Pull Requests](https://github.com/Anionex/banana-slides/pulls)

## 📝 License

This project is licensed under the MIT License. See [LICENSE](../LICENSE) for details.

---

**Ready to dive in?** Start with the [Getting Started Guide](./01-getting-started.md)! 🚀
