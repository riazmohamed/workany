# WorkAny — Full Architecture Guide

## What is WorkAny?

WorkAny is a **desktop AI agent application** that lets you type tasks in plain English and have AI execute them. Think of it as a smart assistant that lives on your computer, can write code, manage files, and talk to AI models like Claude.

---

## The Big Picture: Three Main Parts

The app is split into **3 separate codebases** that work together (this is called a **monorepo** — one repository containing multiple projects):

```
workany/
├── src/          ← Frontend (what you see)
├── src-api/      ← Backend API (the brain)
├── src-tauri/    ← Desktop wrapper (makes it a real app)
```

Think of it like a restaurant:
- **Frontend** = the dining room (menus, tables, the look & feel)
- **Backend API** = the kitchen (does the real work)
- **Desktop wrapper** = the building itself (walls, doors, electricity)

---

## 1. Frontend — `src/` (The User Interface)

**What it does:** Everything you see and interact with — buttons, text inputs, task lists, settings panels.

| Concept | Technology | What It Means |
|---------|-----------|---------------|
| **UI Framework** | React 19 | A library for building user interfaces out of reusable "components" (like LEGO blocks). Each button, sidebar, or page is a component. |
| **Language** | TypeScript 5.8 | JavaScript with "types" — it catches bugs before code runs by defining what kind of data each variable holds (text, number, etc.). |
| **Build Tool** | Vite 7 | Converts your source code into optimized files browsers can understand. Also provides instant updates while developing ("hot reload"). |
| **Styling** | Tailwind CSS 4 | Instead of writing CSS in separate files, you add style classes directly in HTML like `class="text-red-500 p-4"`. |
| **UI Components** | shadcn/ui + Radix UI | Pre-built accessible components (dialogs, dropdowns, tooltips) so you don't build from scratch. |
| **Routing** | React Router 7 | Controls which page shows when you navigate — `/` shows Home, `/task/123` shows a task detail page. |
| **Icons** | Lucide React | A library of small SVG icons (search, settings, etc.). |
| **Markdown** | react-markdown | Renders formatted text (bold, lists, code blocks) from plain text with special syntax. |

### Key Pages

- **Home** (`src/app/pages/Home.tsx`) — Where you type tasks
- **TaskDetail** (`src/app/pages/TaskDetail.tsx`) — Shows AI executing your task in real-time
- **Library** (`src/app/pages/Library.tsx`) — Browse generated files
- **Setup** (`src/app/pages/Setup.tsx`) — Install dependencies

### Frontend Directory Structure

```
src/
├── main.tsx                    # React entry point (initializes settings & providers)
├── app/
│   ├── App.tsx                 # Root component
│   ├── router.tsx              # React Router configuration
│   └── pages/
│       ├── Home.tsx            # Main task input page
│       ├── TaskDetail.tsx      # Detailed task execution view
│       ├── Library.tsx         # File library view
│       └── Setup.tsx           # Setup/dependency installation page
├── components/
│   ├── artifacts/              # Output artifact rendering
│   ├── home/                   # Home page components
│   ├── layout/                 # Layout components (header, sidebar)
│   ├── library/                # Library view components
│   ├── settings/               # Settings UI components
│   ├── task/                   # Task display components
│   ├── ui/                     # Base UI components (shadcn/ui)
│   ├── common/                 # Shared components
│   ├── shared/                 # Utility components
│   └── setup-guard.tsx         # Setup routing guard
├── shared/
│   ├── db/
│   │   ├── database.ts         # IndexedDB + SQLite abstraction
│   │   ├── settings.ts         # Settings persistence
│   │   └── types.ts            # Database type definitions
│   ├── hooks/
│   │   ├── useAgent.ts         # Agent communication hook
│   │   ├── useVitePreview.ts   # Preview server management
│   │   └── useProviders.ts     # Provider management
│   ├── lib/
│   │   ├── utils.ts            # Utility functions
│   │   ├── background-tasks.ts # Background task management
│   │   ├── attachments.ts      # File attachment handling
│   │   ├── session.ts          # Session management
│   │   └── api/                # API communication layer
│   └── providers/
│       ├── theme-provider.tsx  # Dark/light theme context
│       └── language-provider.tsx # i18n context
├── config/
│   ├── locale/
│   │   ├── messages/           # Internationalization (en, zh)
│   │   └── index.ts
│   ├── style/
│   │   ├── global.css          # Global styles
│   │   └── theme.css           # Theme variables
│   └── index.ts
├── core/
│   └── i18n/
│       └── translations.ts     # Translation utilities
├── types/
│   └── react-markdown.d.ts     # Type definitions
└── assets/
```

### Important Frontend Concepts Explained

- **State Management**: React tracks data (called "state") that changes over time. When state changes, the screen updates automatically. WorkAny uses React's built-in hooks (`useState`, `useContext`) for this.

- **Hooks** (`src/shared/hooks/`): Special functions in React that let components "hook into" features like talking to the API (`useAgent.ts`) or managing theme settings (`useProviders.ts`).

- **Providers/Context** (`src/shared/providers/`): A way to share data across many components without passing it through every level. WorkAny uses this for themes (dark/light mode) and language (English/Chinese).

- **IndexedDB**: A database built into web browsers for storing data locally. WorkAny stores tasks, messages, files, and sessions here when running in a browser.

---

## 2. Backend API — `src-api/` (The Brain)

**What it does:** Receives requests from the frontend, talks to AI models, runs code in sandboxes, and sends results back.

| Concept | Technology | What It Means |
|---------|-----------|---------------|
| **Web Framework** | Hono 4.7 | A lightweight server framework that handles HTTP requests. Like Express but faster and more modern. |
| **Runtime** | Node.js 20+ | Lets you run JavaScript/TypeScript outside a browser — on a server. |
| **AI Agent** | Claude Agent SDK 0.2.7 | Anthropic's official library for building AI agents that can plan and execute multi-step tasks. |
| **Code Sandbox** | @anthropic-ai/sandbox-runtime | Runs code safely in an isolated environment so it can't harm your computer. |
| **MCP** | Model Context Protocol SDK | A standard protocol that lets AI models use external tools and data sources. |
| **Validation** | Zod | Checks that incoming data has the right shape (e.g., "this field must be a string, that field must be a number"). |
| **Binary Packaging** | pkg | Compiles the Node.js API into a standalone executable — no Node.js installation needed. |

### API Endpoints (Routes)

| Route | Purpose |
|-------|---------|
| `GET /health` | Check if the server is running |
| `POST /agent` | Send a task to the AI agent |
| `POST /sandbox` | Execute code safely |
| `GET/POST /providers` | Manage AI model providers |
| `GET/POST /files` | File operations |
| `GET/POST /mcp` | MCP server configuration |
| `POST /preview` | Live preview of generated HTML/React |

### API Directory Structure

```
src-api/src/
├── index.ts                    # Server entry point (port 2026 dev, 2620 prod)
├── app/
│   ├── api/
│   │   ├── agent.ts            # Agent execution routes (/agent)
│   │   ├── sandbox.ts          # Sandbox execution (/sandbox)
│   │   ├── mcp.ts              # MCP config management (/mcp)
│   │   ├── preview.ts          # Preview server (/preview)
│   │   ├── providers.ts        # Model provider management (/providers)
│   │   ├── files.ts            # File operations (/files)
│   │   ├── health.ts           # Health check (/health)
│   │   └── index.ts            # Route aggregation
│   └── middleware/
│       ├── cors.ts             # CORS configuration
│       └── index.ts
├── core/
│   ├── agent/
│   │   ├── base.ts             # Base agent class
│   │   ├── plugin.ts           # Agent plugins
│   │   ├── registry.ts         # Agent registry
│   │   ├── types.ts            # Type definitions
│   │   └── index.ts
│   └── sandbox/
│       ├── pool.ts             # Sandbox instance pooling
│       ├── plugin.ts           # Sandbox plugins
│       ├── registry.ts         # Sandbox registry
│       ├── types.ts            # Type definitions
│       └── index.ts
├── config/
│   ├── loader.ts               # Configuration loader
│   ├── constants.ts            # Configuration constants
│   └── index.ts
├── shared/
│   ├── services/
│   │   ├── agent.ts            # Agent service (core business logic)
│   │   └── preview.ts          # Preview server service
│   ├── provider/
│   │   ├── manager.ts          # Provider manager
│   │   ├── loader.ts           # Provider loader
│   │   ├── registry.ts         # Provider registry
│   │   └── types.ts
│   ├── mcp/
│   │   └── loader.ts           # MCP loader
│   ├── skills/
│   │   ├── loader.ts           # Skills loader (~/.claude/skills/)
│   │   └── index.ts
│   ├── types/
│   │   └── agent.ts            # Agent type definitions
│   └── utils/
│       ├── logger.ts           # Logging utility
│       └── paths.ts            # Path utilities
└── extensions/
    ├── agent/
    │   ├── claude/             # Claude agent provider
    │   ├── codex/              # Codex sandbox provider
    │   └── deepagents/         # DeepAgents provider
    └── mcp/
        └── sandbox-server.ts   # MCP sandbox server
```

### How a Task Flows Through the System

```
User types "Build me a calculator"
        │
        ▼
   Frontend sends POST /agent
        │
        ▼
   API receives request via Hono
        │
        ▼
   Agent Service creates a plan (steps to complete the task)
        │
        ▼
   Plan sent back to user for approval (SSE streaming)
        │
        ▼
   User approves → Agent executes each step
        │
        ▼
   Results streamed back in real-time via SSE
        │
        ▼
   Frontend displays progress + final result
```

### Important Backend Concepts Explained

- **SSE (Server-Sent Events)**: A way for the server to "push" updates to the browser in real-time. Instead of the browser asking "are you done yet?" repeatedly (polling), the server sends updates as they happen. This is how you see AI responses appear word-by-word.

- **REST API**: A way of organizing server communication. Each URL (endpoint) represents a resource, and you use HTTP methods (GET = read, POST = create, PUT = update, DELETE = remove) to interact with them.

- **Middleware** (`src-api/src/app/middleware/`): Code that runs before every request — like a security guard checking IDs at the door. WorkAny uses CORS middleware to control which websites can talk to the API.

- **CORS (Cross-Origin Resource Sharing)**: A browser security feature. By default, a webpage at `localhost:1420` can't talk to a server at `localhost:2026`. CORS middleware tells the browser "it's okay, let them through."

- **Sidecar**: In production, the API server is bundled as a binary executable that runs alongside the desktop app — like a backpack attached to the main app. The desktop app starts/stops it automatically.

---

## 3. Desktop App — `src-tauri/` (The Container)

**What it does:** Wraps the web frontend into a native desktop application with access to the file system, databases, and system features.

| Concept | Technology | What It Means |
|---------|-----------|---------------|
| **Desktop Framework** | Tauri 2 | Creates desktop apps using web technologies (HTML/CSS/JS) for the UI but Rust for the backend. Much lighter than Electron. |
| **Language** | Rust | A systems programming language known for speed and memory safety. Tauri uses it for the desktop shell. |
| **Database** | SQLite | A lightweight file-based database. All your data lives in a single `workany.db` file. No separate database server needed. |
| **Serialization** | serde | Rust library that converts data structures to/from JSON (needed to communicate between Rust and JavaScript). |

### What Tauri Provides

- **Native window** (1200x800 default size) with OS title bar
- **SQLite database** via `tauri-plugin-sql`
- **File system access** via `tauri-plugin-fs`
- **Shell commands** via `tauri-plugin-shell` (used to launch the API sidecar)
- **URL/file opening** via `tauri-plugin-opener`

### Tauri Directory Structure

```
src-tauri/
├── src/
│   ├── main.rs                 # Entry point (setup, sidecar management)
│   └── lib.rs                  # Library code (migrations, window setup)
├── Cargo.toml                  # Rust dependencies
├── tauri.conf.json             # Tauri configuration
├── capabilities/
│   └── default.json            # Permission configuration
├── icons/                      # App icons (multiple sizes)
└── entitlements.plist          # macOS entitlements
```

### Database Schema (7 Migrations)

```
sessions    → Conversation sessions (prompt, task count)
tasks       → Task records (prompt, status, cost, duration)
messages    → AI messages (content, tool calls, attachments)
files       → Generated files (name, type, path, thumbnail)
settings    → Key-value config store (API keys, preferences)
```

### Important Desktop Concepts Explained

- **Migration**: A versioned change to the database structure. Migration 1 creates basic tables, Migration 2 adds new columns, etc. This lets the app upgrade its database safely when you update the app.

- **Tauri vs Electron**: Both let you build desktop apps with web tech. Electron bundles an entire Chrome browser (~150MB). Tauri uses the OS's built-in web engine (~5MB). WorkAny chose Tauri for its smaller size and better performance.

- **Plugin System**: Tauri uses plugins to add capabilities. Each plugin (sql, fs, shell) is opted into explicitly for security — the app only gets the permissions it declares.

---

## 4. AI & Agent Architecture

This is the core of what makes WorkAny special.

### Multi-Provider Support

The app isn't locked to one AI. It supports:
- **Anthropic** (Claude models)
- **OpenAI**
- **OpenRouter** (gateway to many models)

Each provider is configured with an API key and base URL in settings.

### Agent Extensions (`src-api/src/extensions/agent/`)

- `claude/` — Claude-based agent (primary)
- `codex/` — OpenAI Codex sandbox
- `deepagents/` — DeepAgents provider

### MCP (Model Context Protocol)

A standard that lets AI models use external "tools" — like giving the AI a calculator, a web browser, or access to your files. Configuration lives at `~/.workany/mcp.json`.

### Skills System

Custom agent behaviors loaded from `~/.claude/skills/`. Think of skills as "recipes" that teach the agent how to do specific things.

---

## 5. Build & Development Setup

| Command | What It Does |
|---------|-------------|
| `pnpm install` | Download all dependencies |
| `pnpm dev:all` | Start everything (frontend + API + desktop) |
| `pnpm dev:web` | Frontend only (port 1420) |
| `pnpm dev:api` | API only (port 2026) |
| `pnpm dev:app` | Desktop app (Tauri dev mode) |
| `pnpm build:app:mac-arm` | Build macOS Apple Silicon app |

**Package Manager: pnpm** — Like npm but faster and uses less disk space by sharing dependencies between projects.

### Ports

- `1420` — Frontend dev server
- `2026` — API dev server
- `2620` — API production server

---

## 6. Key Configuration Files

| File | Purpose |
|------|---------|
| `package.json` | Frontend dependencies, scripts, metadata |
| `src-api/package.json` | API dependencies and scripts |
| `src-tauri/Cargo.toml` | Rust dependencies |
| `src-tauri/tauri.conf.json` | Desktop app config (window size, app name, bundles) |
| `vite.config.ts` | Frontend build configuration |
| `tsconfig.json` | TypeScript compiler settings |
| `components.json` | shadcn/ui component library config |
| `pnpm-workspace.yaml` | Monorepo workspace definition |

---

## 7. User-Level Configuration Files

| Path | Purpose |
|------|---------|
| `~/.workany/mcp.json` | MCP server configuration |
| `~/.workany/logs/workany.log` | Application logs |
| `~/.claude/skills/` | Custom agent skills |
| `~/.claude/settings.json` | Claude settings integration |

---

## 8. Data Flow Summary

```
┌─────────────────────────────────────────────────────┐
│                  DESKTOP (Tauri/Rust)                │
│                                                     │
│  ┌──────────────┐       ┌────────────────────────┐  │
│  │   Frontend    │──────▶│    Backend API          │  │
│  │   (React)     │◀──────│    (Hono/Node.js)      │  │
│  │              │  SSE   │                        │  │
│  │  - Pages     │       │  - Agent Service        │  │
│  │  - Components│       │  - Claude Agent SDK     │  │
│  │  - Hooks     │       │  - Sandbox Runtime      │  │
│  │  - IndexedDB │       │  - MCP Servers          │  │
│  └──────────────┘       └──────────┬─────────────┘  │
│                                    │                 │
│         ┌──────────────────────────┘                 │
│         ▼                                            │
│  ┌──────────────┐       ┌────────────────────────┐  │
│  │   SQLite DB   │       │   External AI APIs     │  │
│  │  (workany.db) │       │  (Anthropic/OpenAI/    │  │
│  │              │       │   OpenRouter)           │  │
│  └──────────────┘       └────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

---

## 9. Technology Summary

| Layer | Technology | Version |
|-------|-----------|---------|
| **Frontend Framework** | React | 19.1.0 |
| **Frontend Build** | Vite | 7.0.4 |
| **Frontend Styling** | Tailwind CSS | 4.1.18 |
| **UI Components** | shadcn/ui + Radix UI | Latest |
| **Routing** | React Router | 7.12.0 |
| **Backend Framework** | Hono | 4.7.10 |
| **Backend Runtime** | Node.js | 20+ |
| **Agent SDK** | Claude Agent SDK | 0.2.7 |
| **MCP** | @modelcontextprotocol/sdk | 1.25.2 |
| **Desktop Framework** | Tauri | 2.0 |
| **Desktop Language** | Rust | 2021 |
| **Database** | SQLite | (via Tauri plugin) |
| **Language** | TypeScript | 5.8.3 |
| **Package Manager** | pnpm | 9+ |

---

## 10. Glossary of Programming Concepts

| Term | Meaning |
|------|---------|
| **Component** | A reusable piece of UI (like a button or form). In React, each component is a function that returns HTML-like code (JSX). |
| **Hook** | A special React function (starts with `use`) that adds features to components — like state, side effects, or context. |
| **API** | Application Programming Interface — a set of rules for how software talks to other software. Here, the frontend talks to the backend via HTTP API. |
| **Endpoint** | A specific URL the API responds to, like `/agent` or `/health`. |
| **SSE** | Server-Sent Events — the server pushes updates to the browser in real-time over a single HTTP connection. |
| **TypeScript** | JavaScript with type checking. You declare what type each variable is, and the compiler catches mistakes before runtime. |
| **Monorepo** | One git repository containing multiple related projects that share code and configuration. |
| **Sidecar** | A helper process that runs alongside the main app. Here, the Node.js API server runs as a sidecar to the Tauri desktop app. |
| **Migration** | A versioned script that changes the database structure (add tables, columns, etc.) safely and incrementally. |
| **Sandbox** | An isolated environment where untrusted code can run without affecting the rest of the system. |
| **MCP** | Model Context Protocol — a standard for giving AI models access to external tools and data sources. |
| **Provider** | An AI service that the app can connect to (Anthropic, OpenAI, OpenRouter). Each has its own API key and models. |
| **Context (React)** | A way to pass data through the component tree without manually passing props at every level. Used for themes and language. |
| **CORS** | A security mechanism that controls which websites can make requests to your API server. |
| **JSX** | A syntax extension that lets you write HTML-like code inside JavaScript/TypeScript. React uses it to describe what the UI should look like. |
| **Props** | Short for "properties" — data passed from a parent component to a child component in React, like function arguments. |
| **State** | Data that a component tracks and that can change over time. When state changes, the component re-renders to reflect the new data. |
| **Dependency** | A library or package your project relies on. Listed in `package.json` (JavaScript) or `Cargo.toml` (Rust). |
| **Build** | The process of converting source code into optimized files ready for production (minified, bundled, compiled). |
| **Hot Reload** | Automatically updating the app in the browser when you save a file during development — no manual refresh needed. |
