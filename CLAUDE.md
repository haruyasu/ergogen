# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ergogen is an Ergonomic Keyboard Layout Generator that takes a configuration file and produces:
- 2D outlines (DXF/SVG) for plates and cases
- PCB files (KiCAD format)
- 3D models (JSCAD format)

Input formats supported: YAML, JSON, JavaScript, or KLE (Keyboard Layout Editor).

## Development Commands

```bash
# Install dependencies
npm install

# Run all tests (100% coverage required)
npm test

# Run specific test category
npm test -- --what=points
npm test -- --what=unit
npm test -- --what=outlines/rectangle

# Update reference files after intentional changes
npm test -- --dump

# Generate coverage report
npm run coverage

# Build bundled version (dist/ergogen.js)
npm run build

# Run CLI directly
node src/cli.js <config_file> --output out_dir --debug --svg
```

## Architecture

### Processing Pipeline (`src/ergogen.js`)

The main `process()` function orchestrates a linear pipeline:

```
Raw Input → Interpret Format → Preprocess → Parse Units → Parse Points
→ Generate Outlines → Model Cases → Scaffold PCBs → Output Results
```

### Core Modules

- **`ergogen.js`**: Main entry point, orchestrates the pipeline
- **`cli.js`**: Command-line interface entry point
- **`io.js`**: Input/output handling, format detection (YAML/JSON/JS/KLE), bundle unpacking
- **`prepare.js`**: Config preprocessing (unnest → inherit → parameterize)
- **`units.js`**: Unit variable parsing with MathJS evaluation
- **`points.js`**: Point generation with 5-level inheritance (zone → column → row → key → global)
- **`anchor.js`**: Point positioning with references, aggregators (average, intersect), and transforms
- **`outlines.js`**: 2D geometry (rectangles, circles, polygons) with boolean operations
- **`cases.js`**: 3D model generation in JSCAD format
- **`pcbs.js`**: KiCAD PCB scaffolding with footprint injection
- **`filter.js`**: Point selection with regex and logical operators (AND/OR/NOT)

### Key Subsystems

**Footprints** (`src/footprints/`): PCB component definitions. Each receives config, name, point, units, and net indexer. Returns KiCAD S-expression format.

**Templates** (`src/templates/`): KiCAD version-specific generators (v5, v8, v9).

**Extensibility**: Use `ergogen.inject('footprint'|'template', name, definition)` or bundle custom components in `.ekb` files.

### Point Naming Convention

Points follow the pattern `{{col.name}}_{{row}}` (configurable). Mirror variants get `mirror_` prefix.

## Testing Structure

- **Unit tests** (`test/unit/`): Individual module functionality
- **Integration tests** (`test/points/`, `test/outlines/`, etc.): YAML configs with `___` suffixed reference outputs
- **CLI E2E tests** (`test/cli/<scenario>/`): Each contains `command`, `reference/` dir, `log`, optional `error`

Test runner supports selective execution with `--what=category/name` and reference updates with `--dump`.

## Key Conventions

- Config preprocessing order matters: unnest → inherit → parameterize
- Underscore-prefixed outputs (e.g., `_debug`) are internal/filtered from standard output
- Mirroring uses `mirror_` prefix with asymmetry options: 'both', 'source', 'clone'
- Validation helpers in `assert.js`, utilities in `utils.js`
