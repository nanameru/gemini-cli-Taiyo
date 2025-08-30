# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🚀 Development Environment Setup

### Prerequisites
- **Node.js**: Version 20.0.0 or higher (development requires ~20.19.0 for dependencies)
- **npm**: Latest version (comes with Node.js)
- **Git**: For version control
- Optional: **Docker** or **Podman** for sandbox development

### Initial Setup
```bash
# Clone the repository
git clone https://github.com/google-gemini/gemini-cli.git
cd gemini-cli

# Install dependencies for all packages
npm install

# Build all packages
npm run build:all
```

## 📋 Essential Development Commands

### Primary Development Workflow
```bash
# Full preflight check (recommended before any commit)
npm run preflight

# Individual commands for focused development:
npm run build              # Build main application
npm run build:all          # Build all packages (including VSCode extension)
npm run test               # Run all unit tests
npm run test:e2e          # Run end-to-end integration tests
npm run typecheck         # TypeScript type checking
npm run lint              # Code linting
npm run lint:fix          # Auto-fix lint issues
npm run format            # Format code with Prettier
```

### Testing Commands
```bash
# Run specific test suites
npm run test:ci                           # CI test suite
npm run test:integration:all              # All integration tests
npm run test:integration:sandbox:docker   # Docker sandbox tests
npm run test:integration:sandbox:podman   # Podman sandbox tests
npm run test:scripts                      # Script tests

# Run single test file
npx vitest run packages/cli/src/ui/components/Header.test.tsx
```

### Development and Debugging
```bash
npm run start             # Start the CLI in development mode
npm run debug             # Start with debugger attached
npm run build-and-start   # Build then start

# Debug with React DevTools
DEV=true npm start
npx react-devtools@4.28.5
```

**Critical**: Always run `npm run preflight` before submitting changes. This validates your changes meet all quality gates.

## 🏗️ Architecture Overview

### Monorepo Structure
This is a **TypeScript monorepo** using npm workspaces with four main packages:

- **`packages/cli/`** - Main CLI application with React + Ink terminal UI
- **`packages/core/`** - Core logic, tools, services, and Gemini API integration
- **`packages/test-utils/`** - Shared test utilities across packages
- **`packages/vscode-ide-companion/`** - VSCode extension for IDE integration

### Key Architectural Patterns

#### CLI Application (`packages/cli/`)
- **Entry Point**: `index.ts` → `src/gemini.tsx`
- **UI Framework**: React with Ink for rich terminal interface
- **Command System**: Slash commands (`/help`, `/chat`, `/settings`, etc.)
- **Authentication**: Multi-method (OAuth, API key, Vertex AI)
- **State Management**: React Context with custom hooks

#### Core System (`packages/core/`)
- **Tool Registry**: Centralized tool management and execution
- **MCP Integration**: Model Context Protocol client for extensibility
- **Gemini Client**: Streaming API communication with response handling
- **Services**: File discovery, shell execution, Git operations, telemetry

### Tool System Architecture
```
User Input → Command Processor → Tool Registry → MCP Client → External Tools
                ↓
          Core Tool Scheduler → Gemini API → Streaming Response → React UI
```

Built-in tools include:
- File operations (`read-file`, `write-file`, `edit`, `glob`, `grep`)
- Shell execution (`shell`)
- Web operations (`web-fetch`, `web-search`)
- Memory management (`memory-tool`)

## 🧪 Testing Strategy

### Framework and Structure
This project uses **Vitest** as the primary testing framework:

- **Test Files**: Co-located with source (`*.test.ts`, `*.test.tsx`)
- **Configuration**: `vitest.config.ts` files in each package
- **Global Setup**: `test-setup.ts` files for common mocks

### Vitest Testing Patterns
```typescript
// Basic test structure
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';

describe('ComponentName', () => {
  beforeEach(() => {
    vi.resetAllMocks();
  });

  afterEach(() => {
    vi.restoreAllMocks();
  });

  it('should behave correctly', () => {
    // Test implementation
  });
});
```

### Mocking Guidelines
- **ES Modules**: Use `vi.mock()` with `async (importOriginal) => { ... }`
- **Critical Dependencies**: Place mocks at top of file before imports
- **Node.js Built-ins**: Common mocks for `fs`, `os`, `path`, `child_process`
- **External SDKs**: Mock `@google/genai`, `@modelcontextprotocol/sdk`

### React Component Testing (Ink)
```typescript
import { render } from 'ink-testing-library';
import { MyComponent } from './MyComponent';

it('renders correctly', () => {
  const { lastFrame } = render(<MyComponent />);
  expect(lastFrame()).toMatchSnapshot();
});
```

## 🛠️ Development Guidelines

### Authentication Development
```bash
# Set environment variables for local development
export GEMINI_API_KEY="your-api-key"
# OR
export GOOGLE_API_KEY="your-vertex-ai-key"
export GOOGLE_GENAI_USE_VERTEXAI=true
```

### Adding New Tools
1. Create tool in `packages/core/src/tools/`
2. Implement the tool interface and schema
3. Register in `packages/core/src/tools/tool-registry.ts`
4. Add comprehensive tests
5. Update documentation

### Adding New Commands
1. Create command in `packages/cli/src/ui/commands/`
2. Implement command handler and tests
3. Register in command system
4. Add to help system

### MCP Server Development
MCP servers extend functionality. Configure in `~/.gemini/settings.json`:
```json
{
  "mcpServers": {
    "server-name": {
      "command": "node",
      "args": ["path/to/server.js"]
    }
  }
}
```

## 🎯 Code Style and Standards

### TypeScript Guidelines
- **Strict mode**: Full TypeScript strict configuration
- **No `any`**: Use `unknown` and type narrowing instead
- **Interface over classes**: Prefer plain objects with TypeScript interfaces
- **ES modules**: Use import/export for encapsulation

### Type Narrowing Pattern
```typescript
import { checkExhaustive } from './packages/cli/src/utils/checks.ts';

switch (value.type) {
  case 'option1':
    // Handle option1
    break;
  case 'option2':
    // Handle option2
    break;
  default:
    checkExhaustive(value.type); // Ensures exhaustive checking
}
```

### React Guidelines (Ink UI)
- **Functional components**: No class components
- **Hooks**: useState, useEffect, custom hooks
- **Pure rendering**: No side effects in render
- **Context**: Use React Context for shared state
- **Testing**: Mock hooks and components appropriately

### General Style
- **Naming**: Use hyphens in flag names (`my-flag` not `my_flag`)
- **Comments**: High-value comments only, avoid explaining obvious code
- **Immutability**: Prefer array methods (`.map()`, `.filter()`, `.reduce()`)
- **Functional style**: Pure functions, no side effects where possible

## 🔧 Project-Specific Implementation Details

### Streaming Response Handling
The CLI uses streaming responses from Gemini API with real-time UI updates through React state management and custom hooks like `useGeminiStream`.

### Tool Execution Flow
1. User input parsed for tool calls in `coreToolScheduler.ts`
2. Tool registry validates and executes tools
3. Results streamed back through UI components
4. Error handling and retry logic for reliability

### Sandbox Security Model
- **macOS**: Uses system sandbox profiles (`sandbox-macos-*.sb` files)
- **Docker/Podman**: Containerized execution with controlled mounts
- **File access**: Controlled through trust dialogs and `.geminiignore`

### VSCode Integration
The companion extension (`packages/vscode-ide-companion/`) provides:
- File diff management
- IDE context sharing
- Real-time collaboration features

## 🚨 Common Issues and Solutions

### Authentication Issues
- Check API keys are properly set
- Verify Google Cloud project configuration
- Clear auth cache: `rm -rf ~/.gemini/auth`

### Build Issues
- Clean build: `npm run clean && npm install && npm run build:all`
- Check Node.js version: `node --version` (must be ≥20.0.0)
- Verify TypeScript errors: `npm run typecheck`

### Test Issues
- Reset test state: `vi.resetAllMocks()` in `beforeEach`
- Mock critical dependencies at file top
- Use `vi.useFakeTimers()` for time-sensitive tests

## 📚 Key Resources

- **Main branch**: `main`
- **Documentation**: `/docs` directory with comprehensive guides
- **Architecture**: `/docs/architecture.md` for detailed system design
- **Contributing**: `/CONTRIBUTING.md` for contribution guidelines
- **Testing**: `/docs/integration-tests.md` for testing documentation