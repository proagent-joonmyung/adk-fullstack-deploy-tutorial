# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a production-ready fullstack template that demonstrates how to wire a Python ADK (Agent Development Kit) backend to a modern Next.js frontend with streaming responses, local development, and deployment paths to Google Cloud's Vertex AI Agent Engine and Vercel.

## Architecture

### Backend (Python ADK)
- **Core Agent**: `app/agent.py` - LlmAgent with built-in planning capabilities for goal decomposition
- **Configuration**: `app/config.py` - Handles environment variables, Vertex AI initialization, and deployment configuration
- **Deployment**: `app/agent_engine_app.py` - Handles packaging and deployment to Vertex AI Agent Engine
- **Model**: Default is `gemini-2.5-flash`, configurable via environment variable

### Frontend (Next.js)
- **Streaming Architecture**: 
  - `nextjs/src/app/api/run_sse/route.ts` - Main SSE endpoint that delegates to appropriate handler
  - `nextjs/src/lib/handlers/run-sse-local-backend-handler.ts` - Handles local ADK backend streaming
  - `nextjs/src/lib/handlers/run-sse-agent-engine-handler.ts` - Handles Agent Engine JSON fragment streaming
- **Environment Detection**: `nextjs/src/lib/config.ts` automatically detects deployment mode:
  - Local backend (default)
  - Agent Engine (when `AGENT_ENGINE_ENDPOINT` is set)
  - Cloud Run (when `CLOUD_RUN_SERVICE_URL` is set)
- **UI Components**: Chat interface with message list, streaming content, and activity timeline

## Common Development Commands

### Full Stack Development
```bash
# Install all dependencies (Python via uv, Node via npm)
make install

# Run both backend and frontend together
make dev

# Run backend only (serves at http://127.0.0.1:8000)
make dev-backend

# Run frontend only (serves at http://localhost:3000)
make dev-frontend
```

### Python Backend
```bash
# Run linting and type checking
make lint

# Start ADK web UI (for debugging)
make adk-web

# Deploy to Vertex AI Agent Engine
make deploy-adk
```

### Next.js Frontend
```bash
# Run linting
npm --prefix nextjs run lint

# Run tests
npm --prefix nextjs run test

# Build for production
npm --prefix nextjs run build
```

## Environment Configuration

### Backend (`app/.env`)
```bash
# Required for local development and deployment
GOOGLE_CLOUD_PROJECT=your-gcp-project-id
GOOGLE_CLOUD_LOCATION=us-central1
GOOGLE_CLOUD_STAGING_BUCKET=your-staging-bucket  # Required for Agent Engine deployment

# Optional
MODEL=gemini-2.5-flash
AGENT_NAME=goal-planning-agent
```

### Frontend (`nextjs/.env.local`)

For local development:
```bash
BACKEND_URL=http://127.0.0.1:8000
NODE_ENV=development
```

For Agent Engine deployment:
```bash
AGENT_ENGINE_ENDPOINT=https://us-central1-aiplatform.googleapis.com/v1/projects/your-project/locations/us-central1/reasoningEngines/YOUR_ENGINE_ID
GOOGLE_SERVICE_ACCOUNT_KEY_BASE64=<base64-encoded-service-account-json>
NODE_ENV=production
```

## Streaming Data Flow

1. **User sends message** → Frontend POST to `/api/run_sse`
2. **Route handler** detects environment and delegates to appropriate handler
3. **For local backend**: Direct SSE pass-through from ADK API server
4. **For Agent Engine**: 
   - Authenticates with Google Cloud using service account
   - Transforms JSON fragments to SSE format
   - Handles `text` and `thought` event types
5. **Frontend** processes SSE events and updates UI in real-time

## Deployment

### Backend to Agent Engine
1. Ensure Google Cloud credentials are configured: `gcloud auth application-default login`
2. Set required environment variables in `app/.env`
3. Run `make deploy-adk` which:
   - Exports dependencies to `.requirements.txt` using uv
   - Packages the app with `agent_engine_app.py`
   - Creates necessary GCS buckets
   - Outputs deployment metadata to `logs/deployment_metadata.json`

### Frontend to Vercel
1. Create service account with Vertex AI User and Service Account Token Creator roles
2. Generate JSON key and convert to base64
3. Set environment variables in Vercel dashboard
4. Import repository and deploy the `nextjs` directory

## Testing Approach

- **Python**: Uses ruff for linting, mypy for type checking, codespell for spell checking
- **TypeScript**: Uses ESLint for linting, Jest for unit tests (configured in `nextjs/jest.config.js`)
- **Test files**: Place in `src/**/__tests__/` or use `.test.ts`/`.spec.ts` suffixes

## Key Technical Decisions

- **uv** for Python dependency management (fast, reliable)
- **SSE (Server-Sent Events)** for real-time streaming instead of WebSockets
- **Environment-based routing** for seamless local/cloud deployment
- **JSON fragment processing** for Agent Engine compatibility
- **Built-in planning** in the ADK agent for structured goal decomposition