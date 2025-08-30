# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Tauri Codebase Guide

## Project Overview

Tauri is a framework for building tiny, blazingly fast binaries for all major desktop and mobile platforms. It combines a Rust-powered backend with any frontend framework that compiles to HTML, JS, and CSS.

**Key Characteristics:**
- **Architecture**: Webview frontend + Rust backend with IPC communication
- **Repository Type**: Polyglot monorepo (Rust + TypeScript/JavaScript)
- **Platforms**: Windows, macOS, Linux, iOS, Android
- **License**: Apache-2.0 OR MIT
- **Primary Language**: Rust (backend), TypeScript (frontend APIs)

## Repository Structure

```
tauri/
├── crates/           # Rust workspace with core libraries
│   ├── tauri/        # Main framework crate
│   ├── tauri-cli/    # Rust CLI for development and building
│   ├── tauri-bundler/# Platform-specific bundling logic
│   ├── tauri-runtime-wry/ # WRY webview runtime implementation
│   ├── tauri-utils/  # Common utilities and config parsing
│   ├── tauri-macros/ # Procedural macros
│   └── ...           # Additional supporting crates
├── packages/         # Node.js packages (PNPM workspace)
│   ├── api/          # TypeScript API for frontend (@tauri-apps/api)
│   └── cli/          # Node.js CLI wrapper (@tauri-apps/cli)
├── examples/         # Example applications demonstrating features
├── bench/            # Performance benchmarks
└── .github/workflows/ # CI/CD pipelines
```

### Core Crates

- **tauri**: Main framework - brings everything together, handles configuration, API hosting
- **tauri-cli**: Development CLI (`cargo tauri dev`, `cargo tauri build`)
- **tauri-bundler**: Creates platform-specific app bundles (.app, .dmg, .deb, .exe, etc.)
- **tauri-runtime**: Abstraction layer between tauri and webview libraries
- **tauri-runtime-wry**: WRY-specific implementation of the runtime
- **tauri-utils**: Configuration parsing, platform detection, asset management
- **tauri-macros**: Compile-time code generation
- **tauri-codegen**: Asset embedding and configuration processing

### Key TypeScript Packages

- **@tauri-apps/api**: Frontend TypeScript API for communicating with Rust backend
- **@tauri-apps/cli**: Node.js wrapper around the Rust CLI

## Development Commands

### Prerequisites
- Rust (1.77.2+) with cargo
- Node.js with PNPM (for frontend packages)
- Platform-specific dependencies (see Tauri docs)

### Common Workspace Commands

```bash
# Install all dependencies
pnpm install

# Build all packages (JS first, then API)  
pnpm build

# Format code (Prettier)
pnpm format
pnpm format:check  # Check formatting without fixing

# Type checking across workspace
pnpm ts:check

# Linting across workspace
pnpm eslint:check

# Run tests across workspace
pnpm test

# Development builds (faster compilation)
pnpm build:debug
```

### Rust Development

```bash
# Build all crates
cargo build

# Run tests for core
cargo test

# Run specific example
cargo run --example helloworld

# Check formatting
cargo fmt --check

# Run clippy
cargo clippy --all-targets --all-features -- -D warnings

# Install CLI for development
cd crates/tauri-cli && cargo install --path . --debug
```

### TypeScript/JavaScript Development

```bash
# Build the API package
pnpm build:api

# Build CLI package  
pnpm build:cli

# Run API example with dev server
pnpm example:api:dev
```

## Architecture & Key Concepts

### Core Architecture
1. **Frontend**: Web technologies (HTML/CSS/JS) rendered in system webview
2. **Backend**: Rust application with Tauri APIs
3. **Communication**: IPC (Inter-Process Communication) via JSON messages
4. **Runtime**: TAO (window management) + WRY (webview rendering)

### Security Model
- **CSP (Content Security Policy)**: Strict content restrictions
- **Isolation Pattern**: Separate execution contexts for enhanced security  
- **ACL (Access Control Layer)**: Fine-grained permission system
- **Capability System**: Defines what APIs frontend can access

### Plugin System
- Plugins provide additional functionality (filesystem, network, etc.)
- Can include Rust code, mobile implementations, and JS APIs
- Located in `src-tauri/capabilities/` and configured in `tauri.conf.json`

### Mobile Support
- **iOS**: Swift + Rust integration via XCode
- **Android**: Kotlin + Rust integration via Gradle
- Templates available in `crates/tauri-cli/templates/mobile/`

## Configuration

### Main Config File: `tauri.conf.json`
Located in `src-tauri/tauri.conf.json` in applications, this file controls:
- App metadata (name, version, identifier)
- Build settings (frontend dist, dev server)
- Security settings (CSP, capabilities)
- Window configuration
- Platform-specific settings

### Capabilities System
- Defined in `src-tauri/capabilities/` directory
- TOML or JSON files defining API permissions
- Referenced in main config under `app.security.capabilities`

## Development Workflow

### Testing Your Changes

#### Core Development
```bash
# Test with helloworld example
cargo run --example helloworld

# Test your own app by pointing to local tauri
# In your app's Cargo.toml:
# tauri = { path = "/path/to/local/tauri/crates/tauri" }
```

#### CLI Development  
```bash
cd crates/tauri-cli
cargo install --path . --debug
# Now `cargo tauri` commands use your local version
```

#### API Development
```bash
cd examples/api
pnpm tauri dev  # Uses local @tauri-apps/api automatically
```

### Release Process

**IMPORTANT**: Changes requiring version bumps must include change files.

1. **Add Change Files**: Create `.md` file in `.changes/` directory
2. **Format** (see `.changes/README.md` for details):
   ```md
   ---
   'tauri': 'patch:enhance'
   'tauri-cli': 'minor'
   ---

   Description of changes
   ```
3. **Available Tags**: `enhance`, `breaking`, `deprecation` (see `.changes/config.json`)
4. **Commit Signing Required**: All commits must be signed
5. **Covector**: Automated versioning system handles releases from change files

### CI/CD Pipeline

Key workflows:
- `test-core.yml`: Tests core Rust crates
- `test-cli-rs.yml`: Tests CLI functionality  
- `lint-rust.yml`: Clippy and formatting
- `lint-js.yml`: ESLint and Prettier
- Platform-specific testing (Windows, macOS, Linux)

## Common Development Patterns

### IPC Commands
```rust
// Backend (Rust)
#[tauri::command]
fn greet(name: &str) -> String {
    format!("Hello, {}!", name)
}

// Frontend (JS/TS)
import { invoke } from '@tauri-apps/api/core'
const greeting = await invoke('greet', { name: 'World' })
```

### Event System
```rust
// Backend emit
app.emit("frontend-event", payload)?;

// Frontend listen
import { listen } from '@tauri-apps/api/event'
const unlisten = await listen('frontend-event', (event) => {
    console.log(event.payload)
})
```

## Key Files & Directories

### Essential Config Files
- `Cargo.toml`: Workspace configuration
- `pnpm-workspace.yaml`: PNPM workspace setup
- `package.json`: Root package with scripts
- `rustfmt.toml`: Rust formatting rules

### Important Documents  
- `ARCHITECTURE.md`: Detailed architecture overview
- `CONTRIBUTING.md`: Development guidelines
- `SECURITY.md`: Security reporting procedures
- `.github/CONTRIBUTING.md`: Pull request guidelines

## Development Tips

### Testing Strategies
- **Frontend changes**: Use `examples/api` - includes comprehensive UI for testing APIs
- **Core Rust changes**: Use `cargo run --example helloworld` for quick iteration
- **CLI changes**: Install local CLI with `cd crates/tauri-cli && cargo install --path . --debug`
- **Cross-platform**: Always test on multiple platforms for core changes
- **Mobile**: Additional Xcode/Android Studio setup required

### Debugging Best Practices
- **Development builds**: Auto-enable dev tools and detailed logging
- **Logging**: Use `console.log` (frontend) and `tracing::info!` (Rust backend)
- **Diagnostics**: Run `tauri info` to check environment and dependencies
- **Webview debugging**: Check both browser console and terminal output
- **Mobile debugging**: Use Xcode/Android Studio debugging tools

### Common Pitfalls & Solutions
- **Rust rebuilds**: Window closes during recompilation - this is expected behavior
- **CSP violations**: Check browser console for blocked resources
- **Mobile gotchas**: Platform-specific permissions and capabilities required
- **Path handling**: Use `tauri::api::path` for cross-platform file paths
- **IPC errors**: Verify command signatures match between Rust and TypeScript

### Performance Optimization
- **Release profile**: Highly optimized with `opt-level = "s"` and LTO
- **Bundle size**: Target 10-20MB final binaries
- **Development**: Use `--debug` flag for faster compilation during development
- **Testing production**: Always test with `cargo build --release` before shipping

## External Dependencies

### Key Upstream Projects
- **TAO**: Cross-platform windowing (fork of winit)
- **WRY**: Cross-platform webview rendering
- Both maintained by Tauri team for specific needs

### Build Tools
- **PNPM**: JavaScript package manager
- **Rollup**: JavaScript bundling (for @tauri-apps/api)
- **napi-rs**: Node.js native bindings (for @tauri-apps/cli)

## Quick Reference

### Essential Files to Understand First
1. `ARCHITECTURE.md` - Complete technical architecture overview
2. `examples/api/src-tauri/src/main.rs` - Example of typical Tauri app setup
3. `crates/tauri/src/lib.rs` - Main framework entry point
4. `packages/api/src/core.ts` - TypeScript API for IPC communication

### Development Workflow Checklist
- [ ] Install prerequisites (Rust, Node.js, PNPM, platform tools)
- [ ] Run `pnpm install && pnpm build` to set up workspace
- [ ] Test changes with appropriate example app
- [ ] Add change files to `.changes/` directory for releases
- [ ] Sign commits (required for all PRs)
- [ ] Run format/lint checks before committing

This guide provides a foundation for contributing to Tauri. For detailed API documentation, run `cargo doc --open`. For troubleshooting, consult the [Discord community](https://discord.gg/SpmNs4S) or existing GitHub issues.