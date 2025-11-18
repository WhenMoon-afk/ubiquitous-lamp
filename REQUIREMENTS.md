# DataForge – Requirements Specification

## Document Overview
This document establishes the concrete, testable requirements for DataForge's MVP release. Requirements are derived from analysis of comparable tools (CyberChef, DevToys/DevUtils, Chepy) and tailored to DataForge's core mission: a local, offline-first data transformation workbench with plugin extensibility.

---

## Background Summary

DataForge draws inspiration from three distinct tool categories, combining their strengths while addressing their limitations:

**CyberChef** pioneered the browser-based data transformation pipeline model with its three-pane interface (input | recipe | output) and auto-bake functionality. Its linear pipeline of composable operations serves security analysts, incident responders, and malware researchers who need to iteratively decode, deobfuscate, and analyze suspicious payloads. However, CyberChef's browser environment struggles with large inputs, its extensive operation catalog can overwhelm new users, and recipe management lacks organization for power users working across many investigations.

**DevToys and DevUtils** demonstrate the value of native desktop experiences for developer utilities. These tools excel at OS integration (hotkeys, clipboard detection, window management) and provide focused, performant individual tools. However, they lack CyberChef's core strength: reusable multi-step pipelines. Each utility is largely a one-off transformer without the ability to chain operations into repeatable workflows.

**Chepy** brings CyberChef-style operations to Python as a scriptable library, enabling automation and leveraging Python's ecosystem through plugins. While powerful for programmers, it lacks an accessible GUI for exploratory analysis and requires users to write code rather than interactively build recipes.

**DataForge synthesizes these approaches**: CyberChef's composable pipeline model, DevToys' native desktop UX and OS integration, and Chepy's plugin extensibility, all implemented in a local-first architecture that prioritizes privacy, offline capability, and maintainability. The MVP focuses on clarity over breadth—a modest, thoughtfully chosen operation catalog with a robust pipeline engine that users can extend through a simple plugin system.

---

## Functional Requirements

### FR1: Input and Output Handling

#### FR1.1: Text Input [MUST]
- The system SHALL accept text input through a dedicated input pane
- The input pane SHALL support:
  - Direct text entry via text area
  - Paste from clipboard
  - Configurable text encoding (UTF-8 default, with ASCII, UTF-16, Latin-1 options)

#### FR1.2: File Input [MUST]
- The system SHALL accept file input through:
  - File picker dialog
  - File path entry
- The system SHALL support:
  - Text files (read as text with encoding detection/specification)
  - Binary files (read as bytes)
- The system SHALL display file size and warn when files exceed 10MB

#### FR1.3: Output Display [MUST]
- The system SHALL display transformation output in a dedicated output pane
- The output pane SHALL support multiple view modes:
  - Text view (default)
  - Hex view (for binary data)
- The output SHALL update after each successful recipe execution

#### FR1.4: Output Actions [MUST]
- The system SHALL provide:
  - Copy to clipboard button
  - Save to file dialog with format/encoding options
- The system SHALL preserve binary data integrity when saving

---

### FR2: Operations and Recipes

#### FR2.1: Operation Model [MUST]
- Each operation SHALL have:
  - Unique identifier (e.g., "base64_decode")
  - Display name (e.g., "Base64 Decode")
  - Category/type (e.g., "Encoding", "Hashing", "Formatting")
  - Description of function
  - Parameter schema (if configurable)
  - Input type requirements (text, bytes, or both)
  - Output type (text or bytes)

#### FR2.2: Recipe Structure [MUST]
- A recipe SHALL be an ordered list of operations
- Operations SHALL execute sequentially: `input → op1 → op2 → ... → opN → output`
- Each operation receives the output of the previous operation as its input

#### FR2.3: Recipe Management UI [MUST]
- The recipe pane SHALL display the current recipe as an ordered list
- Users SHALL be able to:
  - Add operations from an operation catalog/menu
  - Remove operations from the recipe
  - Reorder operations (move up/down)
  - Enable/disable operations without removing them
  - Configure operation parameters through the UI

#### FR2.4: Operation Parameters [MUST]
- Operations with parameters SHALL expose them in the UI
- Parameter types SHALL include:
  - Text input
  - Dropdown selection (for enumerated options)
  - Numeric input (with min/max validation)
  - Boolean checkbox
- Parameters SHALL have sensible defaults
- Invalid parameter values SHALL be caught before execution

---

### FR3: MVP Operation Catalog

The following operations are REQUIRED for MVP:

#### FR3.1: Encoding Operations [MUST]

**Base64 Encode**
- Input: Text or bytes
- Output: Base64-encoded text
- Parameters: None

**Base64 Decode**
- Input: Base64 text
- Output: Decoded bytes or text
- Parameters: Output format (text/bytes)
- Error: Invalid Base64 characters or padding

**Hex Encode**
- Input: Bytes or text
- Output: Hexadecimal string (e.g., "48656c6c6f")
- Parameters: Delimiter (none, space, colon), uppercase/lowercase

**Hex Decode**
- Input: Hexadecimal string
- Output: Decoded bytes or text
- Parameters: Output format (text/bytes)
- Error: Invalid hex characters

**URL Encode**
- Input: Text
- Output: URL-encoded text (percent-encoding)
- Parameters: Encode spaces as + or %20

**URL Decode**
- Input: URL-encoded text
- Output: Decoded text
- Parameters: None

#### FR3.2: Data Formatting Operations [MUST]

**JSON Pretty Print**
- Input: JSON text
- Output: Formatted JSON with indentation
- Parameters: Indent size (2 or 4 spaces)
- Error: Invalid JSON with parse error location

**JSON Minify**
- Input: JSON text
- Output: Compact JSON (no whitespace)
- Parameters: None
- Error: Invalid JSON with parse error location

#### FR3.3: Hashing Operations [MUST]

**SHA-256 Hash**
- Input: Text or bytes
- Output: SHA-256 hash as hex string
- Parameters: Output format (hex, base64)

#### FR3.4: Hash Operations [SHOULD]

**MD5 Hash** [SHOULD]
- Input: Text or bytes
- Output: MD5 hash as hex string
- Parameters: Output format (hex, base64)
- Warning: Display security warning about MD5 being cryptographically broken

**SHA-1 Hash** [SHOULD]
- Input: Text or bytes
- Output: SHA-1 hash as hex string
- Parameters: Output format (hex, base64)
- Warning: Display security warning about SHA-1 being deprecated

---

### FR4: Pipeline Execution

#### FR4.1: Manual Execution [MUST]
- The system SHALL provide a "Run" or "Execute" button
- Clicking Run SHALL execute the current recipe on the current input
- Execution SHALL be synchronous for MVP (blocking UI)

#### FR4.2: Auto-Run Mode [SHOULD]
- The system SHOULD provide an optional "Auto-run" toggle
- When enabled, the recipe SHALL re-execute automatically when:
  - Input changes
  - Recipe changes (operations added/removed/reordered)
  - Operation parameters change
- Auto-run SHALL only trigger for inputs smaller than 1MB
- Auto-run SHALL debounce changes (wait 500ms after last change before executing)

#### FR4.3: Execution State [MUST]
- The system SHALL indicate execution state:
  - Idle (ready to run)
  - Running (in progress)
  - Success (completed without errors)
  - Error (failed at some operation)

#### FR4.4: Error Handling [MUST]
- When an operation fails:
  - Execution SHALL halt at the failing operation
  - The system SHALL display a clear error message indicating:
    - Which operation failed
    - Why it failed (e.g., "Invalid Base64 character 'ñ' at position 42")
    - What the user can do to fix it
  - Downstream operations SHALL NOT execute
  - The output pane SHALL show the error, not partial/incorrect output
- The system SHALL NOT crash on operation errors

#### FR4.5: Progress Indication [SHOULD]
- For operations taking longer than 1 second:
  - The system SHOULD display a progress indicator
  - The indicator SHOULD show which operation is currently executing

---

### FR5: Plugin Operations

#### FR5.1: Plugin Discovery [MUST]
- The system SHALL scan a designated `plugins/` directory for plugin modules
- Plugin discovery SHALL occur at application startup
- Failed plugin loads SHALL log warnings but NOT crash the application

#### FR5.2: Plugin Interface [MUST]
- Each plugin operation SHALL conform to a standard interface:
  - Metadata: name, description, category, parameter schema, input type, output type
  - Execution function: `run(input_data, parameters) -> output_data`
- The interface specification SHALL be documented in `docs/plugin-api.md` (Phase E)

#### FR5.3: Plugin Operation Registration [MUST]
- Successfully loaded plugins SHALL register their operations with the core engine
- Plugin operations SHALL appear in the operation catalog alongside built-in operations
- Plugin operations SHALL be indistinguishable from built-in operations in the UI

#### FR5.4: Plugin Error Handling [MUST]
- Plugin load errors SHALL:
  - Log a warning with the plugin file name and error details
  - Continue loading other plugins
  - NOT crash the application
- Plugin execution errors SHALL be handled identically to built-in operation errors

#### FR5.5: Plugin Directory Structure [SHOULD]
- The `plugins/` directory SHOULD contain:
  - Individual `.py` files for simple operations
  - Subdirectories for complex multi-file plugins
  - A `README.md` with plugin development guidance
  - An `example_operation.py` template

---

## Non-Functional Requirements

### NFR1: Local-First and Offline Capability

#### NFR1.1: No Network Dependency [MUST]
- All core operations and UI features SHALL work without network connectivity
- The application SHALL NOT make network requests during normal operation
- Operations requiring network access (future) MUST be explicitly opt-in and clearly marked

#### NFR1.2: Data Privacy [MUST]
- User data SHALL NOT be transmitted over the network
- The application SHALL NOT include telemetry, analytics, or "phone home" behavior
- All data processing SHALL occur locally on the user's machine

---

### NFR2: Performance and Scalability

#### NFR2.1: Typical Use Case Performance [MUST]
- The application SHALL handle text payloads up to 1MB without UI freezing
- Pipeline execution for common recipes (2-4 operations) on 100KB input SHALL complete within 1 second on typical hardware (2015+ laptop)

#### NFR2.2: Large Input Handling [SHOULD]
- The application SHOULD display warnings for inputs exceeding 10MB
- The application SHOULD remain responsive (not freeze) during execution of long-running operations
- For MVP, inputs exceeding 100MB MAY be rejected with a clear error message

#### NFR2.3: Future Performance Goals [COULD]
- Post-MVP enhancements COULD include:
  - Streaming/chunked processing for large files
  - Asynchronous execution with cancellation
  - Multi-threading for parallelizable operations

---

### NFR3: Portability

#### NFR3.1: Cross-Platform Support [MUST]
- The application SHALL run on:
  - Windows 10 and later
  - Linux (Ubuntu 20.04+ and equivalent)
- The application SHOULD run on:
  - macOS 11 (Big Sur) and later

#### NFR3.2: Implementation Language [MUST]
- Core engine and operations SHALL be implemented in Python 3.9+
- The codebase SHALL use Python standard library where possible to minimize dependencies

#### NFR3.3: UI Framework [MUST]
- The GUI framework SHALL be cross-platform compatible
- Framework selection (PyQt6, Tkinter, or PyWebView) will be finalized in Phase B
- The UI SHALL be separable from the core engine (MVC pattern)

#### NFR3.4: Packaging [MUST]
- The application SHALL be packaged as a standalone executable for Windows (.exe) and Linux (AppImage, .deb, or equivalent)
- The executable SHALL run on systems without Python installed
- The packaged application SHALL bundle all required dependencies

---

### NFR4: Maintainability

#### NFR4.1: Modular Architecture [MUST]
- The codebase SHALL be organized into clearly separated modules:
  - Core pipeline engine
  - Built-in operations library
  - Plugin loader and registry
  - GUI layer
  - CLI (future)
- Dependencies between modules SHALL be minimized and documented

#### NFR4.2: Code Quality [MUST]
- Code SHALL follow PEP 8 style guidelines
- Critical modules (engine, operations) SHALL have unit test coverage exceeding 80%
- All public APIs SHALL be documented with docstrings

#### NFR4.3: Plugin API Stability [MUST]
- The plugin API SHALL be versioned
- Breaking changes to the plugin API SHALL be avoided in minor releases
- Plugin API documentation SHALL be maintained in `docs/plugin-api.md`

#### NFR4.4: Dependency Management [MUST]
- All dependencies SHALL be documented with rationale
- Dependency versions SHALL be pinned in `requirements.txt`
- The project SHALL minimize dependencies (prefer standard library)

---

### NFR5: Security and Safety

#### NFR5.1: Network Isolation [MUST]
- Built-in operations SHALL NOT make network calls
- Future operations requiring network access MUST be:
  - Clearly marked in the UI
  - Require explicit user opt-in
  - Documented with security implications

#### NFR5.2: Code Execution Safety [MUST]
- Operations SHALL NOT execute arbitrary user-provided code
- The pipeline engine SHALL NOT use `eval()`, `exec()`, or equivalent unsafe constructs on user input

#### NFR5.3: Error Handling [MUST]
- All operations SHALL handle malformed or malicious input gracefully
- Buffer overflows, path traversal, and other common vulnerabilities SHALL be prevented
- Operations SHALL fail safely (no data corruption or undefined behavior)

#### NFR5.4: Plugin Security [MUST for MVP awareness, COULD for enforcement]
- MVP: Plugins SHALL be treated as trusted code (no sandboxing)
- MVP: Documentation SHALL clearly warn users that plugins have full application privileges
- MVP: The UI SHOULD display a warning when loading plugins for the first time
- Post-MVP: Plugin sandboxing COULD be implemented (Phase P)

#### NFR5.5: Cryptographic Operations [MUST]
- Hashing operations SHALL use well-tested libraries (Python's `hashlib`)
- Operations using deprecated/broken cryptography (MD5, SHA-1) SHALL display security warnings
- Future cryptographic operations (encryption/decryption) SHALL be reviewed for correct implementation

---

## User Stories

### US1: Security Analyst Stories

**US1.1: Decode Obfuscated Payload**
> As a security analyst investigating a phishing email, I want to paste a suspicious Base64-encoded attachment, chain together Base64 decode and JSON pretty-print operations, and see the final decoded script, so I can quickly understand whether the payload is malicious.

**Acceptance Criteria:**
- Paste Base64 text into input pane
- Add "Base64 Decode" operation to recipe
- Add "JSON Pretty Print" operation to recipe
- Execute recipe and view formatted JSON in output
- Total time from paste to result: under 30 seconds

---

**US1.2: Hash Forensic Evidence**
> As an incident responder documenting a compromised file, I want to load a suspicious binary file and compute its SHA-256 hash, so I can share the hash with threat intelligence platforms without sharing the file itself.

**Acceptance Criteria:**
- Load binary file via file picker
- Add "SHA-256 Hash" operation to recipe
- Execute and see hex hash in output
- Copy hash to clipboard with one click
- Process 10MB file within 5 seconds

---

**US1.3: Analyze Encoded Log Entries**
> As a security analyst reviewing firewall logs, I want to create a recipe that URL-decodes entries, extracts specific fields, and formats them for reporting, so I can process multiple log entries with the same recipe.

**Acceptance Criteria:**
- Build multi-step recipe (URL decode → custom processing)
- Save recipe for reuse (post-MVP feature, noted as future need)
- Apply recipe to multiple log entries
- Recipe executes reliably without errors

---

### US2: Developer Stories

**US2.1: Debug API Responses**
> As a developer debugging a REST API, I want to paste a JSON response, pretty-print it to read the structure, then hash a specific field to verify data integrity, so I can quickly validate API behavior.

**Acceptance Criteria:**
- Paste minified JSON into input
- Add "JSON Pretty Print" to recipe
- View formatted JSON
- Manually extract a field and hash it (or use future "extract" operation)

---

**US2.2: Encode Data for URLs**
> As a web developer building query parameters, I want to type or paste data and URL-encode it, so I can safely embed it in URLs without breaking special characters.

**Acceptance Criteria:**
- Type text with special characters (spaces, &, =, etc.)
- Add "URL Encode" operation
- Execute and get percent-encoded output
- Copy result to use in browser or cURL command

---

**US2.3: Quick Base64 Conversions**
> As a developer working with image data URIs, I want to quickly Base64-encode small files, so I can embed them in HTML or CSS without switching tools.

**Acceptance Criteria:**
- Load small file (e.g., icon image)
- Add "Base64 Encode" operation
- Get Base64 string in output
- Copy to clipboard
- Process multiple files quickly (restart recipe with new input)

---

### US3: Power User Stories

**US3.1: Chain Complex Transformations**
> As a power user analyzing multi-encoded data, I want to chain multiple decode operations (Base64 → Hex → URL decode) in a single recipe, so I can unpack deeply nested encodings with one execution.

**Acceptance Criteria:**
- Add 3+ operations to recipe
- Reorder operations as needed
- Execute and see final decoded output
- Disable intermediate operations to debug the recipe

---

**US3.2: Work Offline in Secure Environment**
> As a power user working in an air-gapped network, I want to use DataForge without any network connectivity, so I can process sensitive data without security concerns.

**Acceptance Criteria:**
- Disconnect from network
- Launch DataForge
- Use all built-in operations successfully
- No error messages about network unavailability
- Data never leaves local machine

---

**US3.3: Reuse Recipes Across Sessions**
> As a power user with repetitive transformation tasks, I want to save my recipes and reload them later, so I don't have to rebuild complex pipelines every time.

**Acceptance Criteria (Post-MVP):**
- Save current recipe to file
- Close and reopen application
- Load saved recipe
- Recipe executes identically to original

**MVP Note:** Recipe save/load is Phase J (post-MVP); document as high-priority enhancement.

---

### US4: Plugin Author Stories

**US4.1: Add Custom Operation**
> As a plugin author with a specialized use case, I want to create a Python file implementing a new operation, drop it into the `plugins/` directory, and have it appear in the operation catalog, so I can extend DataForge without modifying core code.

**Acceptance Criteria:**
- Write plugin file following documented API
- Place file in `plugins/` directory
- Restart DataForge (or trigger plugin reload)
- See new operation in catalog
- Use operation in recipe successfully
- Total development time for simple operation: under 30 minutes

---

**US4.2: Share Plugin with Community**
> As a plugin author who has created a useful operation, I want to share my plugin file with other users, so they can benefit from my work.

**Acceptance Criteria (Post-MVP):**
- Export plugin as standalone `.py` file
- Share file via GitHub, email, etc.
- Other users can install by copying to their `plugins/` directory
- Plugin works identically on other users' systems

**MVP Note:** Community sharing relies on simple file sharing; plugin repository/package manager is Phase K+ (stretch).

---

**US4.3: Debug Plugin Errors**
> As a plugin author developing a new operation, I want clear error messages when my plugin fails to load or execute, so I can quickly identify and fix issues.

**Acceptance Criteria:**
- Plugin with syntax error fails to load
- Application shows warning log with:
  - Plugin file name
  - Error type and message
  - Line number (if applicable)
- Application continues running (doesn't crash)
- After fixing error and restarting, plugin loads successfully

---

## Requirements Priority Summary

### MUST Have (MVP Blockers)

**Functional:**
- FR1.1-FR1.4: Input/output handling (text, file, display, actions)
- FR2.1-FR2.4: Operation model and recipe management
- FR3.1: Encoding operations (Base64, Hex, URL encode/decode)
- FR3.2: JSON formatting (pretty-print, minify)
- FR3.3: SHA-256 hashing
- FR4.1, FR4.3-FR4.4: Manual execution with error handling
- FR5.1-FR5.4: Plugin discovery, interface, registration, error handling

**Non-Functional:**
- NFR1.1-NFR1.2: Offline capability and data privacy
- NFR2.1: Performance for typical use cases (up to 1MB)
- NFR3.1-NFR3.4: Cross-platform support and packaging (Windows + Linux minimum)
- NFR4.1-NFR4.4: Modular architecture, code quality, plugin API stability
- NFR5.1-NFR5.5: Security and safety (network isolation, error handling, plugin warnings)

---

### SHOULD Have (Important for MVP, Not Blockers)

**Functional:**
- FR3.4: MD5 and SHA-1 hashing (with security warnings)
- FR4.2: Auto-run mode for small inputs
- FR4.5: Progress indication for slow operations
- FR5.5: Plugin directory structure and documentation

**Non-Functional:**
- NFR2.2: Large input handling (warnings, responsiveness)
- NFR3.1: macOS support (in addition to Windows/Linux)

---

### COULD Have (Stretch Goals, Post-MVP)

**Functional:**
- Recipe save/load (Phase J)
- Additional operations: compression, regex, advanced crypto (Phase K)
- Branching/conditional pipelines (Phase L)
- CLI mode (Phase M)
- Advanced UI features: search, shortcuts, themes (Phase O)

**Non-Functional:**
- NFR2.3: Streaming/chunked processing, multi-threading
- Plugin sandboxing (Phase P)

---

## Minimal Vertical Slice for MVP

To prove the core concept with the smallest possible implementation, the MVP vertical slice includes:

1. **Pipeline Engine:**
   - Execute ordered list of operations on byte data
   - Pass output of operation N as input to operation N+1
   - Handle errors gracefully

2. **Core Operations (Minimum 3):**
   - Base64 Decode (most common security analyst use case)
   - JSON Pretty Print (most common developer use case)
   - SHA-256 Hash (hash is essential for security workflows)

3. **Simple GUI:**
   - Text input area
   - Recipe builder (add/remove operations)
   - Text output area with copy button
   - Run button

4. **Plugin Loading:**
   - Scan `plugins/` directory
   - Load one example plugin successfully
   - Show plugin operation in catalog

5. **Error Handling:**
   - Invalid Base64 shows clear error message
   - Invalid JSON shows parse error with location

**Success Metric:** A user can paste Base64-encoded JSON, build a 2-operation recipe (Base64 Decode → JSON Pretty Print), execute it, and copy the result—all within 2 minutes on first use.

---

## Traceability to Background Analysis

| Background Insight | DataForge Requirement |
|-------------------|----------------------|
| CyberChef's linear pipeline model with auto-bake | FR2.2 (sequential pipeline), FR4.2 (auto-run mode) |
| CyberChef's three-pane layout (input \| recipe \| output) | FR1 (input handling), FR2.3 (recipe pane), FR1.3 (output pane) |
| CyberChef's pain point: cluttered UI with too many operations | FR3 MVP catalog (small, curated operation set) |
| DevToys' native desktop UX and OS integration | NFR3 (cross-platform desktop), future: clipboard integration |
| DevToys' limitation: no multi-step pipelines | FR2.2 (linear recipes as core differentiator) |
| Chepy's plugin extensibility via Python | FR5 (plugin system with Python modules) |
| Chepy's limitation: no GUI for exploration | FR1-FR4 (full GUI for interactive recipe building) |
| Common need: offline/local-first for sensitive data | NFR1 (offline capability, data privacy) |
| Common need: simple, understandable tools | NFR4 (maintainability, modular architecture) |

---

## Open Questions and Assumptions

### Assumptions Made
1. **Input size limits:** MVP targets typical use cases (text/files up to a few MB). Large file support is explicitly deferred to Phase N.
2. **Plugin trust model:** MVP assumes users only load trusted plugins. Sandboxing is post-MVP.
3. **Linear pipelines sufficient:** 80%+ of use cases can be served with linear recipes; branching is Phase L.
4. **Python performance acceptable:** For MVP operation sizes and counts, Python's performance is sufficient.

### Open Questions for Phase B (Architecture)
1. Which UI framework provides the best balance of simplicity, cross-platform support, and maintainability? (To be prototyped)
2. Should the pipeline engine support streaming/chunking from the start, or defer to Phase N?
   - **Recommendation:** Defer to Phase N; implement simple in-memory processing for MVP.
3. What is the exact plugin interface signature?
   - **Recommendation:** Define in Phase E with working prototype.

### Open Questions for User Testing (Phase I)
1. Is the lack of recipe save/load a dealbreaker for MVP users?
2. Is linear-only pipeline too limiting, or acceptable for MVP?
3. Which additional operations are most urgently needed?

---

## Document Maintenance

This requirements document is a living artifact and should be updated when:
- New requirements are identified during implementation phases
- User testing reveals gaps or incorrect assumptions
- Scope decisions change the MVP boundary

All changes should be documented with rationale and date in the revision history below.

---

## Revision History

| Date | Version | Changes | Author |
|------|---------|---------|--------|
| 2025-11-18 | 1.0 | Initial requirements specification for Phase B | Claude (Phase B) |
