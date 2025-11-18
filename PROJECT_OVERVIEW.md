# DataForge – Project Overview

## Project Name
**DataForge** (working name) – A local, offline-first data transformation workbench

## High-Level Goal
Build a desktop application that provides security analysts, developers, and power users with a local, privacy-respecting alternative to CyberChef. DataForge enables chaining data transformation operations in a simple linear pipeline, running entirely offline on the user's machine.

## Target Users
- **Security Analysts & Incident Responders**: Need to decode, decode, hash, and transform data during investigations without sending sensitive information to external services
- **Developers**: Require quick access to common encoding/decoding operations during development and debugging
- **Reverse Engineers**: Analyze file formats, protocols, and binary data through repeated transformations
- **Privacy-Conscious Power Users**: Want full control over their data without cloud dependencies

## Core Features (What We're Building)

### 1. Simple 3-Pane Desktop UI
- **Left pane**: Input data (text area, file drop zone, or file picker)
- **Middle pane**: Recipe builder showing ordered list of operations with configurable parameters
- **Right pane**: Output display with copy/save capabilities

### 2. Linear Pipeline Engine
- Operations execute in sequence: `input → op1 → op2 → ... → opN → output`
- Each operation receives the output of the previous operation
- Clear error messages when operations fail (e.g., "Invalid Base64 at position 42")
- Parameter configuration for each operation in the pipeline

### 3. Core Operations Catalog (MVP)
Essential transformations that cover common use cases:
- **Encoding/Decoding**: Base64, Hex, URL encoding
- **Data Formatting**: JSON pretty-print, JSON minify
- **Hashing**: SHA-256 (with room to add MD5, SHA-1, SHA-512, etc.)
- **Text Operations**: To/from uppercase/lowercase, trim whitespace

### 4. Plugin-Style Extension Model
- New operations can be added by dropping files into a `plugins/` directory
- Plugins register themselves with metadata (name, description, parameters)
- Core engine discovers and loads plugins at startup
- Adding an operation should NOT require modifying core code

### 5. Offline-First Architecture
- All built-in operations work without network access
- No telemetry, no "phone home" behavior
- Data never leaves the user's machine unless explicitly saved/exported

## Non-Goals for MVP

The following are explicitly OUT of scope for the initial release to keep the project focused and achievable:

- **Full CyberChef Parity**: We're not trying to replicate every CyberChef feature; we're building a simpler, more maintainable tool
- **Advanced Cryptographic Operations**: No encryption/decryption, advanced crypto analysis, or key management in MVP
- **Complex Forensic Tooling**: No file carving, memory analysis, or deep binary inspection (may add later)
- **Heavy Visual Features**: No charts, graphs, or complex visualizations beyond text output
- **Multi-User Collaboration**: No shared recipes, cloud sync, or team features
- **Web-Based Architecture**: Desktop-first; not building a web app
- **Branching/Conditional Pipelines**: Linear only for MVP; branching is a stretch goal
- **Scripting Integration**: No embedded Python/JavaScript execution in MVP
- **Advanced UX**: No drag-and-drop recipe building, keyboard shortcuts library, or theming in MVP

## Key Assumptions and Constraints

### Technology Choices
- **Primary Language**: Python (for rapid development, extensive libraries, and cross-platform support)
- **UI Framework**: TBD in later phases (candidates: PyQt6, Tkinter, or a lightweight web UI with PyWebView)
- **Desktop-Style UX**: Optimized for keyboard and mouse on desktop/laptop, not mobile
- **Packaging**: Single-file executables for Windows, macOS, and Linux (using PyInstaller or similar)

### Design Principles
1. **Simplicity over feature richness**: A tool that does 20 things well beats one that does 200 things poorly
2. **Offline by default**: Network operations (if any) must be explicitly opt-in
3. **Clear mental model**: Users should immediately understand "input → operations → output"
4. **Fail loudly and clearly**: Better to show an error message than silently produce wrong results
5. **Extensibility through convention**: Plugins follow simple file/folder conventions rather than complex APIs

### Scope Boundaries
- **MVP Timeline**: Focus on getting a working tool with 5-10 operations and a functional UI
- **Plugin Security**: MVP assumes trusted plugins; sandboxing is a future enhancement
- **Performance**: Optimize for files up to ~10MB; larger files are stretch goals
- **Cross-Platform**: Design for cross-platform but test primarily on Linux and Windows for MVP

## Success Criteria for MVP

The MVP will be considered successful when:

1. **Basic Workflow Works**: A user can start the app, paste or load input data, build a recipe with 2-4 operations, and see the transformed output

2. **Offline Functionality**: All built-in operations work without any network connection

3. **Plugin Extensibility**: A developer can add a new operation by creating a Python file in the `plugins/` directory without modifying core engine code

4. **Error Handling**: The tool gracefully handles common error cases:
   - Invalid Base64 input shows clear error message
   - Malformed JSON displays parse error with location
   - File I/O errors are caught and reported

5. **Cross-Platform Packaging**: The tool can be packaged as a standalone executable for at least two platforms (e.g., Windows .exe and Linux AppImage)

6. **Usability Baseline**: A new user can understand how to use the tool within 2 minutes without reading documentation (through intuitive UI design)

## Success Metrics (Qualitative)

- **"It just works"**: Users can accomplish common tasks (decode Base64, format JSON, compute hash) without friction
- **"I trust it"**: No network calls, clear about what it's doing, doesn't crash on bad input
- **"I can extend it"**: Plugin API is simple enough that a developer can add an operation in <30 minutes

## Future Vision (Post-MVP)

After proving the core concept with the MVP, potential enhancements include:
- More operations (compression, regex, advanced crypto, file format parsers)
- Recipe library with save/load/share capabilities
- Branching pipelines with conditional logic
- CLI mode for scripting and automation
- Advanced UI features (operation search, keyboard shortcuts, themes)
- Performance optimization for large files (streaming, chunking)
- Plugin sandboxing and security model
- Integration with external tools (hex editors, debuggers)

## Project Principles

1. **Local First**: Privacy and offline capability are non-negotiable
2. **User Respect**: No dark patterns, no telemetry, no surprises
3. **Developer Friendly**: Clean code, good documentation, easy to contribute
4. **Sustainable**: Modest dependencies, long-term maintainability over quick hacks
5. **Focused**: Better to have 10 operations that work perfectly than 100 that are buggy
