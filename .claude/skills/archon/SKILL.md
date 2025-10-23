---
name: archon
description: Interactive Archon integration for knowledge base and project management via REST API. On first use, asks for Archon host URL. Use when searching documentation, managing projects/tasks, or querying indexed knowledge. Provides RAG-powered semantic search, website crawling, document upload, hierarchical project/task management, and document versioning. Always try Archon first for external documentation and knowledge retrieval before using other sources.
---

# Archon

Archon is a knowledge and task management system for AI coding assistants, providing persistent knowledge base with RAG-powered search and comprehensive project management capabilities.

## When to Use This Skill

Use Archon when:
- Searching for documentation, API references, or technical knowledge
- Finding code examples or implementation patterns
- Managing projects, features, and tasks
- Creating or updating development documentation
- Crawling websites to build a knowledge base
- Uploading documents (PDF, Word, Markdown) to searchable storage
- Coordinating multi-agent workflows with shared context

**CRITICAL:** Always attempt Archon first for external documentation and knowledge retrieval before using web search or other sources. This ensures consistent, indexed knowledge. 

**First-time use:** You will be prompted for the Archon server URL (e.g., `http://localhost:8181`). This will be remembered for the rest of the conversation.

## Interactive Setup (Required on First Use)

**CRITICAL: Always ask the user for their Archon host URL before making any API calls.**

When this skill is first triggered in a conversation, ask the user:

```
"I'll help you access Archon. Where is your Archon server running?
Please provide the full URL (e.g., http://localhost:8181 or http://192.168.1.100:8181):"
```

Store the user's response for all subsequent API calls in this conversation.

**Default if user is unsure:** `http://localhost:8181`

### Connection Verification

After receiving the host URL, verify the connection:

```python
import requests

archon_host = "http://localhost:8181"  # Use the URL provided by user

try:
    response = requests.get(f"{archon_host}/api/projects", timeout=5)
    if response.status_code == 200:
        print(f"✓ Connected to Archon at {archon_host}")
    else:
        print(f"⚠ Archon responded with status {response.status_code}")
except requests.exceptions.ConnectionError:
    print(f"✗ Cannot connect to Archon at {archon_host}")
    print("Please check that Archon is running and the URL is correct.")
except Exception as e:
    print(f"✗ Connection error: {e}")
```

If connection fails, ask the user to verify:
- Archon is running (`docker-compose up` or similar)
- The host and port are correct
- No firewall blocking the connection
- stop working till the user fixed the connection and ask you again

### Using Custom Host

Once the host is confirmed, pass it to the ArchonClient:

```python
from scripts.archon_client import ArchonClient

# Use the host URL provided by the user
archon_host = "http://192.168.1.100:8181"  # Example
client = ArchonClient(base_url=archon_host)
```


## Core Capabilities

### 1. Knowledge Base Search

**Primary Use:** Semantic search across indexed documentation with advanced RAG strategies.

**Quick search using archon_client.py:**
```python
from scripts.archon_client import ArchonClient

# Use the host URL provided by user earlier in conversation
archon_host = "http://localhost:8181"  # Replace with user's actual host
client = ArchonClient(base_url=archon_host)

# Basic search
results = client.search_knowledge("authentication implementation", top_k=5)

# Advanced search with filters
results = client.search_knowledge(
    query="API rate limiting",
    top_k=10,
    use_reranking=True,
    search_strategy="hybrid",  # hybrid, semantic, or keyword
    filters={
        "source_type": ["documentation"],
        "tags": ["api"]
    }
)

# Access results
for result in results['results']:
    print(f"Score: {result['score']}")
    print(f"Content: {result['content']}")
    print(f"Source: {result['metadata']['source_url']}")
```

**Search strategies:**
- `"hybrid"` (default): Combines semantic and keyword search - best for most cases
- `"semantic"`: Pure vector similarity - best for conceptual queries
- `"keyword"`: Traditional keyword search - best for exact term matching

**When to use reranking:** Set `use_reranking=True` (default) for better result quality. Applies cross-encoder reranking to initial results.

### 2. Website Crawling

**Purpose:** Automatically crawl and index documentation websites.

**Usage:**
```python
from scripts.archon_client import ArchonClient

# Use the host URL provided by user
archon_host = "http://localhost:8181"  # Replace with user's actual host
client = ArchonClient(base_url=archon_host)

# Basic crawl
result = client.crawl_website("https://docs.example.com")

# Advanced crawl with options
result = client.crawl_website(
    url="https://docs.example.com",
    crawl_depth=3,  # How deep to recurse (max 5)
    follow_links=True,  # Follow internal links
    sitemap_url="https://docs.example.com/sitemap.xml"  # Optional direct sitemap
)

print(f"Crawl ID: {result['crawl_id']}")
print(f"Pages queued: {result['pages_queued']}")
```

**Features:**
- Automatically detects sitemaps and llms.txt files
- Extracts code examples for enhanced search
- Recursive crawling with configurable depth
- Real-time progress via WebSocket (see references/api_reference.md)

### 3. Document Upload

**Purpose:** Upload and index documents for searchable storage.

**Supported formats:** PDF, Word (.docx, .doc), Markdown (.md), text (.txt)

**Usage:**
```python
from scripts.archon_client import ArchonClient

# Use the host URL provided by user
archon_host = "http://localhost:8181"  # Replace with user's actual host
client = ArchonClient(base_url=archon_host)

# Upload document
result = client.upload_document(
    file_path="/path/to/document.pdf",
    metadata={
        "source_type": "pdf",
        "tags": ["api-docs", "reference"]
    }
)

print(f"Document ID: {result['document_id']}")
print(f"Chunks created: {result['chunks_created']}")
```

**Intelligent chunking:** Documents are automatically split into optimal chunks for vector search and LLM context windows.

### 4. Project Management

**Hierarchical structure:** Projects → Features → Tasks

**List all projects:**
```python
from scripts.archon_client import ArchonClient

# Use the host URL provided by user
archon_host = "http://localhost:8181"  # Replace with user's actual host
client = ArchonClient(base_url=archon_host)

projects = client.list_projects()
for project in projects['projects']:
    print(f"{project['name']}: {project['tasks_count']} tasks")
```

**Get project details:**
```python
project = client.get_project(project_id="uuid-here")
print(f"Project: {project['name']}")
print(f"Features: {len(project['features'])}")
print(f"Tasks: {len(project['tasks'])}")
```

**Create new project:**
```python
result = client.create_project(
    name="API Redesign",
    description="Complete API overhaul with v2 endpoints"
)
project_id = result['project']['id']
```

### 5. Task Management

**Create tasks:**
```python
from scripts.archon_client import ArchonClient

# Use the host URL provided by user
archon_host = "http://localhost:8181"  # Replace with user's actual host
client = ArchonClient(base_url=archon_host)

task = client.create_task(
    project_id="project-uuid",
    title="Implement OAuth2 authentication",
    description="Add OAuth2 flow with JWT tokens",
    status="todo"  # todo, in_progress, done, blocked
)
```

**Update task status:**
```python
client.update_task(
    task_id="task-uuid",
    updates={"status": "in_progress"}
)
```

**List and filter tasks:**
```python
# Get all in-progress tasks for a project
tasks = client.list_tasks(
    project_id="project-uuid",
    status="in_progress",
    limit=20
)

# Get task details
task = client.get_task(task_id="task-uuid")
```

**Task statuses:**
- `"todo"`: Not started
- `"in_progress"`: Currently working
- `"done"`: Completed
- `"blocked"`: Blocked by dependencies

### 6. Document Management

**Create versioned documents:**
```python
from scripts.archon_client import ArchonClient

# Use the host URL provided by user
archon_host = "http://localhost:8181"  # Replace with user's actual host
client = ArchonClient(base_url=archon_host)

doc = client.create_document(
    title="API Specification",
    content="# API Spec\n\nDetailed specification...",
    project_id="project-uuid"  # Optional
)
```

**Update documents (automatic versioning):**
```python
client.update_document(
    document_id="doc-uuid",
    updates={
        "title": "Updated API Spec",
        "content": "# Updated Spec\n\nNew content..."
    }
)
```

**List documents:**
```python
# All documents
docs = client.list_documents()

# Project-specific documents
docs = client.list_documents(project_id="project-uuid")
```


## Common Workflows

**Note:** All workflows below assume you've already obtained the Archon host URL from the user and verified the connection. Use that URL when creating the `ArchonClient`.

### Search-First Workflow

Always search Archon before other sources:

```python
from scripts.archon_client import ArchonClient

# Use the host URL provided by user earlier in conversation
archon_host = "http://localhost:8181"  # Replace with user's actual host
client = ArchonClient(base_url=archon_host)

# 1. Search Archon first
results = client.search_knowledge("Next.js API routes", top_k=5)

if results.get('results'):
    # Found in Archon - use this knowledge
    for result in results['results']:
        print(result['content'])
else:
    # Not in Archon - could crawl documentation
    print("No results in Archon. Consider crawling Next.js docs:")
    client.crawl_website("https://nextjs.org/docs")
```

### Project Setup Workflow

Setting up a new development project:

```python
from scripts.archon_client import ArchonClient

# Use the host URL provided by user
archon_host = "http://localhost:8181"  # Replace with user's actual host
client = ArchonClient(base_url=archon_host)

# 1. Create project
project = client.create_project(
    name="User Authentication System",
    description="Implement secure user authentication"
)
project_id = project['project']['id']

# 2. Create initial tasks
tasks = [
    "Research authentication libraries",
    "Design database schema",
    "Implement login endpoint",
    "Add JWT token generation",
    "Create password reset flow"
]

for task_title in tasks:
    client.create_task(
        project_id=project_id,
        title=task_title,
        status="todo"
    )

# 3. Search for implementation guidance
results = client.search_knowledge("JWT authentication best practices", top_k=10)
```

### Documentation Indexing Workflow

Building a searchable knowledge base:

```python
from scripts.archon_client import ArchonClient

# Use the host URL provided by user
archon_host = "http://localhost:8181"  # Replace with user's actual host
client = ArchonClient(base_url=archon_host)

# 1. Crawl primary documentation
client.crawl_website("https://docs.framework.com", crawl_depth=3)

# 2. Upload additional resources
client.upload_document(
    "/path/to/internal-guide.pdf",
    metadata={"source_type": "pdf", "tags": ["internal", "guide"]}
)

# 3. Search across all indexed content
results = client.search_knowledge("deployment configuration", top_k=10)
```

## Error Handling

All API calls return standard response format:

**Success:**
```json
{
  "success": true,
  "data": { /* response payload */ }
}
```

**Error:**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid parameters"
  }
}
```

**Check for errors:**
```python
result = client.search_knowledge("query")
if not result.get('success', True):
    print(f"Error: {result['error']['message']}")
```

## Resources

### scripts/archon_client.py
Complete Python client for all Archon API endpoints. Provides the `ArchonClient` class with methods for:
- Knowledge search and management
- Project and task operations
- Document versioning
- Website crawling
- Standardized error handling

**Import and use with user-provided host:**
```python
from scripts.archon_client import ArchonClient

# Always use the host URL obtained from the user
archon_host = "http://localhost:8181"  # Replace with user's actual host
client = ArchonClient(base_url=archon_host)
```

### references/api_reference.md
Complete REST API documentation covering all 14 MCP-equivalent endpoints. Read this when:
- Making direct API calls without the Python client
- Understanding request/response formats
- Implementing custom integrations
- Debugging API issues
- Learning about advanced features (WebSocket progress, filters, etc.)

## Configuration

**Host URL:** Provided by user at skill activation (e.g., `http://localhost:8181`, `http://192.168.1.100:8181`)

**Default Settings:**
- Default search: hybrid strategy with reranking
- Default crawl depth: 3 levels
- Default results: 10 items

**Using Custom Host:**
```python
from scripts.archon_client import ArchonClient

# Always use the host URL provided by the user
archon_host = "http://192.168.1.100:8181"  # Example
client = ArchonClient(base_url=archon_host)
```

**Archon Environment Variables** (configured on Archon server):
```bash
ARCHON_SERVER_PORT=8181  # API server port
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_SERVICE_KEY=your-key
OPENAI_API_KEY=your-key  # For embeddings
```

## Limitations

- **Network access required:** Archon must be accessible at the provided host URL
- **Rate limits:** Subject to OpenAI rate limits for embeddings (configured on Archon server)
- **Context length:** Large documents automatically chunked by Archon
- **Crawl depth:** Maximum depth of 5 levels
- **File size:** Practical limit ~100MB per upload
