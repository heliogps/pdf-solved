# PDF Master - Project Specification & Architecture Guide

## Project Overview
A multiplatform desktop application that runs locally only, is totally free and open-source, and solves all PDF problems (Read, Write, Convert, OCR, etc.). The primary goals are maximum speed and efficiency.

## Technology Stack

### Backend (Core & System Layer)
- **Language**: Rust
  - Rationale: Near C++ performance with memory safety, no garbage collection pauses, excellent cross-platform support, small binary sizes.
  
- **Framework**: Tauri
  - Rationale: Uses native OS webviews (WebView2 on Windows, WebKit on macOS, GTK WebKit on Linux), resulting in ~10MB binaries vs 150MB+ for Electron. Lower memory footprint and better security for local-only apps.

### Frontend (Presentation Layer)
- **Language**: TypeScript
- **Framework**: React or Vue 3 (to be decided during scaffolding)
- **Styling**: TailwindCSS
- **State Management**: Zustand (React) or Pinia (Vue)
- **Data Fetching**: TanStack Query

### Key Rust Crates
| Purpose | Crate |
|---------|-------|
| PDF Parsing/Reading | `pdf-rs`, `lopdf` |
| PDF Writing/Editing | `lopdf` |
| OCR | `tesseract-rs` + `leptonica-sys` |
| Image Processing | `image` |
| Async Runtime | `tokio` |
| Serialization | `serde`, `serde_json` |
| PDF Conversion | `poppler` bindings, `pdf2image` |

## Architecture Pattern

### Clean Architecture + CQRS (Command Query Responsibility Segregation)

```
src/
├── core/                   # Domain logic, entities, value objects
│   ├── entities/           # PDF Document, Page, Annotation, etc.
│   ├── value_objects/      # Color, Rectangle, Rotation, etc.
│   └── errors/             # Domain-specific errors
│
├── application/            # Use cases, commands, queries
│   ├── commands/           # Write operations (Edit, Convert, OCR, etc.)
│   ├── queries/            # Read operations (GetPage, SearchText, etc.)
│   └── services/           # Application-level services
│
├── infrastructure/         # External implementations
│   ├── pdf/                # PDF library implementations (lopdf, pdf-rs)
│   ├── ocr/                # Tesseract integration
│   ├── filesystem/         # File I/O operations
│   └── cache/              # Caching mechanisms
│
├── interface/              # Tauri commands, API layer
│   ├── commands.rs         # Tauri @commands
│   └── dto/                # Data Transfer Objects
│
└── presentation/           # Frontend (separate package)
    ├── src/
    │   ├── components/     # UI components
    │   ├── hooks/          # Custom React/Vue hooks
    │   ├── stores/         # State management
    │   └── views/          # Page views
    └── package.json
```

### Layer Responsibilities

1. **Core Layer**
   - Pure business logic
   - No external dependencies
   - Platform-agnostic
   - Highly testable

2. **Application Layer**
   - Orchestrates use cases
   - Defines Command/Query interfaces
   - Coordinates between core and infrastructure
   - Transaction management

3. **Infrastructure Layer**
   - Implements repository interfaces
   - Integrates with PDF libraries
   - Handles file system operations
   - Manages OCR engine integration

4. **Interface Layer**
   - Tauri IPC commands
   - Request/Response mapping
   - Input validation
   - Error translation

5. **Presentation Layer**
   - UI components
   - User interaction handling
   - State management
   - Async communication with backend

## Core Features

### Phase 1: Foundation
- [ ] PDF Reading & Rendering
- [ ] Basic Navigation (zoom, pan, page navigation)
- [ ] Text Selection & Copy
- [ ] Thumbnail generation

### Phase 2: Editing
- [ ] Merge PDFs
- [ ] Split PDFs
- [ ] Rotate pages
- [ ] Delete pages
- [ ] Reorder pages
- [ ] Add/Remove bookmarks

### Phase 3: Advanced Features
- [ ] OCR (Optical Character Recognition)
- [ ] PDF to Image conversion
- [ ] Image to PDF conversion
- [ ] PDF to Text extraction
- [ ] Compress PDFs
- [ ] Add watermarks
- [ ] Add annotations (highlight, underline, strike-through)
- [ ] Add comments/notes

### Phase 4: Premium Features (Free!)
- [ ] Form filling
- [ ] Form creation
- [ ] Digital signatures
- [ ] Batch processing
- [ ] Password protection
- [ ] PDF/A conversion

## Performance Optimizations

1. **Threading Model**
   - Heavy operations (OCR, conversion) run on Rust worker threads
   - Non-blocking IPC communication with frontend
   - Progress reporting for long-running tasks

2. **Memory Management**
   - Stream large PDFs instead of loading entirely into memory
   - Lazy loading of PDF pages
   - Automatic cleanup of cached resources
   - Reference counting for shared resources

3. **Caching Strategy**
   - Cache rendered page images
   - Cache OCR results per page
   - LRU cache with configurable size limits
   - Persistent cache for frequently accessed files

4. **Rendering Optimization**
   - GPU-accelerated rendering via WebGL where possible
   - Progressive rendering (low-res first, then high-res)
   - Virtual scrolling for large documents
   - Background page pre-rendering

5. **Build Optimization**
   - Release builds with LTO (Link Time Optimization)
   - Strip debug symbols for distribution
   - Platform-specific optimizations
   - Minimal dependency tree

## Development Rules

### Code Quality
- All code must be thoroughly documented
- Unit tests required for core and application layers
- Integration tests for infrastructure layer
- E2E tests for critical user flows
- Clippy warnings must be resolved
- rustfmt for consistent formatting

### Security (Local-Only Focus)
- No telemetry or analytics
- No external network calls by default
- All data stays on user's machine
- Secure handling of temporary files
- Sandboxed file access permissions

### Open Source Compliance
- MIT or Apache 2.0 license
- Clear contribution guidelines
- Document all third-party dependencies
- Respect all upstream licenses
- Maintain CHANGELOG.md

### Version Control
- Feature branch workflow
- Conventional Commits specification
- Pull request reviews required
- Semantic versioning (SemVer)

## Build & Distribution

### Development
```bash
# Install Tauri CLI
cargo install tauri-cli

# Run in development mode
cargo tauri dev

# Run frontend only
cd src/presentation && npm run dev
```

### Production Build
```bash
# Build for current platform
cargo tauri build

# Build with optimizations
cargo tauri build --release
```

### Target Platforms
- Windows 10/11 (x64, ARM64)
- macOS 11+ (Intel, Apple Silicon)
- Linux (AppImage, deb, rpm)

## Project Commands Reference

```bash
# Initialize project
cargo create-tauri-app pdf-master --template react-ts

# Add dependencies
cargo add lopdf pdf-rs tesseract-rs image tokio serde serde_json

# Run tests
cargo test
cargo test --all-features

# Check code quality
cargo clippy --all-features -- -D warnings
cargo fmt --check

# Build documentation
cargo doc --open
```

## Future Considerations

1. **Plugin Architecture**: Allow community extensions
2. **Scripting Support**: Lua or JavaScript scripting for automation
3. **Cloud Sync Optional**: User-controlled sync via WebDAV/Nextcloud
4. **Mobile Companion**: Future iOS/Android apps using same core
5. **CLI Tool**: Separate command-line interface for power users

---

## Decision Log

### Date: Initial Setup
- **Decision**: Rust + Tauri stack
- **Rationale**: Best performance-to-size ratio, memory safety, growing ecosystem
- **Alternatives Considered**: 
  - Electron + Node.js (rejected: too large, high memory usage)
  - Qt + C++ (rejected: steeper learning curve, larger binaries)
  - Flutter Desktop (rejected: immature PDF ecosystem, larger than Tauri)

### Date: Initial Setup
- **Decision**: Clean Architecture + CQRS
- **Rationale**: Clear separation of concerns, testability, maintainability
- **Alternatives Considered**:
  - MVC (rejected: less suitable for complex domain logic)
  - Hexagonal Architecture (similar, but CQRS better fits command-heavy PDF ops)

---

*This document serves as the single source of truth for project architecture and decisions. Update this file when making significant architectural changes.*
