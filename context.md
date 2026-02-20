# Featured Image Tool - Context Document

## Project Overview
A client-side HTML/CSS/JS tool for creating high-resolution images with customizable text and geometric lines. The tool generates canvas-based images that can be exported as PNG or SVG, with design saving/loading capabilities.

## Features Implemented

### Core Functionality
- **Canvas-based image generation** (client-side only, no server)
- **Text elements**: Up to 4 lines of text with customizable position, rotation
- **Geometric lines**: Add/remove lines with position, rotation, length, thickness
- **Custom fonts**: Drop .ttf/.otf/.woff/.woff2 files to use
- **Default fonts**: monospace, serif, sans-serif selection
- **Background color**: RGB color picker
- **Text/lines color**: Shared color from RGB picker

### Image Controls
- **Size**: Width/Height in pixels (default: 600×800 portrait)
- **Orientation buttons**: LAND (800×600) / PORT (600×800)
- **Font size**: Single size for all text elements

### Interactions
- **Drag & drop**: Move text and lines on canvas
- **Keyboard modifiers while dragging**:
  - `x`: constrain to horizontal axis
  - `y`: constrain to vertical axis
  - `r`: rotate
  - `s`: adjust font size (text only)
  - `l`: adjust line length (lines only)
  - `t`: adjust line thickness (lines only)

### UI Features
- **Foldable sections**: TEXT, LINES, Help boxes
- **Per-element fold**: Each Text 1/2/3 and Line 1/2/3 can be collapsed
- **Copy/Remove buttons**: Duplicate or delete text/lines
- **Key indicator**: Shows active modifier key in top-right of canvas
- **Minimalist design**: 10px monospace, B/W theme, 1px borders

### Export/Save
- **PNG export**: Download as image
- **SVG export**: Vector output with optional text vectorization (requires custom font)
- **Design save**: Export parameters to JSON
- **Design load**: Restore from JSON (font file needs to be re-dropped if custom)

### Help Section
- Keyboard shortcuts listed
- Credits: "This tool evolved from an idea of Jérôme Rigaud from a design of Matthias Kreutzer for the UQAM Inventaire book."

---

## Information That Would Have Helped Work Faster

### 1. Clear Variable Mutability Intent
- **Issue**: `textColor` was declared as `const` but needed to be reassigned when loading designs
- **Better approach**: Use `let` from the start if mutation is expected, or document mutability requirements upfront

### 2. Template Literal Complexity
- **Issue**: Using template literals with `${i}` inside `forEach` loops created parsing issues with nested quotes
- **Better approach**: Use string concatenation or prepare data structure first, then build HTML

### 3. Event Binding Strategy
- **Issue**: Inline `onclick` handlers didn't work as expected; had to use direct event listeners
- **Better approach**: Decide on event delegation strategy upfront and document it

### 4. Design File Schema
- **Issue**: Loading older design files failed due to missing backward compatibility handling
- **Better approach**: Define a clear JSON schema version system from the start

---

## Debugging Lessons Learned

### 1. JavaScript Syntax Errors
- **Problem**: Extra closing braces `});` in template literal code caused "Unexpected token" errors
- **Solution**: Use Node.js `--check` to validate JavaScript syntax before testing in browser
- **Prevention**: Run syntax checks regularly, especially after making structural changes

### 2. CSS Class Naming Conflicts
- **Problem**: `.collapsed` class worked for one foldable but not another due to similar class names
- **Solution**: Use more specific selectors and verify CSS specificity
- **Prevention**: Test each UI component individually

### 3. Inline Event Handlers
- **Problem**: `onclick="function()"` didn't work reliably; needed addEventListener
- **Lesson**: Browser DOM event handling prefers addEventListener over inline handlers
- **Prevention**: Standardize on addEventListener for all interactions

### 4. Query Selector Scope
- **Problem**: `document.querySelector` searched the whole document instead of within specific containers
- **Solution**: Use `container.querySelector` to scope searches to relevant sections
- **Prevention**: Be explicit about DOM query scope, especially in functions called repeatedly

### 5. Constant vs Let Mutability
- **Problem**: Tried to reassign `const textColor` when loading design files
- **Error message**: "Assignment to constant variable"
- **Solution**: Changed to `let`
- **Prevention**: Review mutability requirements early; use `const` only for truly immutable values

---

## Technical Stack
- Pure HTML/CSS/JS (no frameworks)
- HTML5 Canvas for image generation
- opentype.js for font vectorization in SVG
- CSS Flexbox for layout

## File Structure
```
featured_image_tool/
├── index.html    # Single file containing all HTML, CSS, JS
└── context.md   # This document
```
