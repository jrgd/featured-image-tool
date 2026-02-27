# Design Tools Toolbox

A standardized framework for creating canvas-based design tools that share common principles and can communicate together.

---

## Core Architecture

### File Structure
```
tool-name/
├── index.html       # Single file containing HTML, CSS, JS
└── toolbox.md       # This documentation
```

### Technology Stack
- Pure HTML/CSS/JS (no frameworks)
- HTML5 Canvas for rendering
- CSS Flexbox for layout
- JSON for data persistence

---

## Element System

### Base Element Interface
All design elements follow a common structure:

```javascript
{
    id: string,           // Unique identifier
    type: string,         // 'text' | 'line' | 'image' | 'gradient' | etc.
    x: number,            // X position (center-based)
    y: number,            // Y position (center-based)
    angle: number,       // Rotation in degrees (-180 to 180)
    // Type-specific properties...
}
```

### Element Types

#### 1. Text Element
```javascript
{
    type: 'text',
    text: string,         // Content
    fontSize: number,     // In pixels (global setting)
    fontFamily: string   // Font name (global setting)
}
```

#### 2. Line Element (Geometric)
```javascript
{
    type: 'line',
    length: number,       // Line length in pixels
    thickness: number   // Line weight
}
```

#### 3. Image Layer
```javascript
{
    type: 'image',
    image: Image,        // JavaScript Image object
    width: number,      // Original width
    height: number,     // Original height
    scale: number       // 0.1 to 5
}
```

#### 4. Background Image
```javascript
{
    type: 'bgImage',
    image: Image,
    // Rendered with object-fit: cover semantics
}
```

---

## Rendering Pipeline

### Render Function Pattern
```javascript
function render() {
    const width = parseInt(document.getElementById('imgWidth').value);
    const height = parseInt(document.getElementById('imgHeight').value);
    
    canvas.width = width;
    canvas.height = height;
    
    // 1. Clear and draw background
    drawBackground(ctx, width, height);
    
    // 2. Draw background image (if present)
    if (bgImage) drawBackgroundImage(ctx, width, height);
    
    // 3. Draw elements in z-order (back to front)
    imageLayers.forEach(drawImageLayer);
    geometricLines.forEach(drawLine);
    textElements.forEach(drawText);
}
```

### Drawing Helpers
```javascript
function drawElement(ctx, element, drawFn) {
    ctx.save();
    ctx.translate(element.x, element.y);
    ctx.rotate(element.angle * Math.PI / 180);
    drawFn(ctx, element);
    ctx.restore();
}
```

---

## Interaction System

### Hit Testing
Each element type needs a hit test function:

```javascript
// For rotated rectangles (images, text boxes)
function findElementAtPoint(x, y) {
    for (let i = elements.length - 1; i >= 0; i--) {
        const el = elements[i];
        const rad = -el.angle * Math.PI / 180;
        
        // Transform point to element's local space
        const dx = x - el.x;
        const dy = y - el.y;
        const localX = dx * Math.cos(rad) - dy * Math.sin(rad);
        const localY = dx * Math.sin(rad) + dy * Math.cos(rad);
        
        // Check bounds
        if (isInBounds(localX, localY, el)) {
            return i;
        }
    }
    return null;
}
```

### Drag State Management
```javascript
let draggingElement = null;
let draggingElementType = null;  // 'text' | 'line' | 'image'
let dragOffsetX = 0;
let dragOffsetY = 0;
let lastMouseX = 0;
let lastMouseY = 0;
let activeKey = null;  // 'x' | 'y' | 'r' | 's' | etc.
```

### Keyboard Modifiers
```javascript
document.addEventListener('keydown', (e) => {
    if (['x', 'y', 'r', 's', 'l', 't'].includes(e.key.toLowerCase())) {
        activeKey = e.key.toLowerCase();
        keyIndicator.textContent = activeKey.toUpperCase();
        keyIndicator.classList.add('active');
    }
});

document.addEventListener('keyup', (e) => {
    const key = e.key.toLowerCase();
    if (['x', 'y', 'r', 's', 'l', 't'].includes(key)) {
        activeKey = null;
        keyIndicator.classList.remove('active');
    }
});
```

### Modifier Actions by Element Type
| Key | Text | Line | Image |
|-----|------|------|-------|
| `x` | Constrain X | Constrain X | Constrain X |
| `y` | Constrain Y | Constrain Y | Constrain Y |
| `r` | Rotate | Rotate | Rotate |
| `s` | Font size | - | Scale |
| `l` | - | Length | - |
| `t` | - | Thickness | - |

---

## UI Components

### Control Panel Structure
```html
<div class="controls">
    <!-- Settings Section -->
    <div class="control-group">
        <h3>SECTION NAME</h3>
        <div class="control-row">
            <label>LABEL</label>
            <input type="number" id="settingId">
        </div>
    </div>
    
    <!-- Foldable Elements Section -->
    <div class="control-group">
        <h3><span class="fold-icon" id="foldIcon">−</span> ELEMENTS</h3>
        <div class="foldable" id="foldable">
            <div class="element-controls" id="elementControls"></div>
            <button class="add-btn" id="addBtn">+ ADD</button>
        </div>
    </div>
</div>
```

### Fold Toggle Pattern
```javascript
document.getElementById('foldIcon').addEventListener('click', () => {
    const foldable = document.getElementById('foldable');
    const icon = document.getElementById('foldIcon');
    foldable.classList.toggle('collapsed');
    icon.textContent = foldable.classList.contains('collapsed') ? '+' : '−';
});
```

### Element Control Generation
```javascript
function createElementControls() {
    const container = document.getElementById('elementControls');
    container.innerHTML = '';
    
    elements.forEach((el, i) => {
        const group = document.createElement('div');
        group.className = 'control-group';
        group.innerHTML = `
            <h3>
                <span class="fold-item-icon" data-index="${i}">−</span>
                ELEMENT ${i + 1}
                <button class="copy-btn" data-index="${i}">⧉</button>
                <button class="remove-btn" data-index="${i}">✕</button>
            </h3>
            <div class="item-content" data-index="${i}">
                ${generateControlFields(el, i)}
            </div>
        `;
        container.appendChild(group);
    });
    
    // Bind events
    bindControlEvents();
}
```

### CSS Variables for Theming
```css
:root {
    --border-color: #000;
    --bg-color: #fff;
    --text-color: #000;
    --font-mono: monospace;
    --font-size: 10px;
    --spacing: 5px;
    --panel-width: 320px;
}
```

---

## Data Persistence

### Save Design
```javascript
function saveDesign() {
    const design = {
        version: '1.0',
        width: canvas.width,
        height: canvas.height,
        settings: { /* global settings */ },
        bgColor: { r, g, b },
        bgImage: bgImage ? bgImage.src : null,
        elements: elements.map(el => serializeElement(el))
    };
    
    const blob = new Blob([JSON.stringify(design, null, 2)], { 
        type: 'application/json' 
    });
    const url = URL.createObjectURL(blob);
    // Trigger download
}
```

### Load Design
```javascript
function loadDesign(json) {
    // 1. Restore canvas size
    canvas.width = json.width;
    canvas.height = json.height;
    
    // 2. Restore settings
    Object.assign(settings, json.settings);
    
    // 3. Restore background
    if (json.bgImage) {
        loadBackgroundImage(json.bgImage);
    }
    
    // 4. Restore elements
    elements = json.elements.map(deserializeElement);
    createElementControls();
    
    render();
}
```

### Serialization Helpers
```javascript
function serializeElement(el) {
    const base = {
        x: el.x, y: el.y, angle: el.angle
    };
    
    switch (el.type) {
        case 'text':
            return { ...base, text: el.text };
        case 'line':
            return { ...base, length: el.length, thickness: el.thickness };
        case 'image':
            return { 
                ...base, 
                scale: el.scale,
                width: el.width,
                height: el.height,
                imageData: el.image.src  // Base64
            };
    }
}
```

---

## Color System

### RGB Color Object
```javascript
const textColor = { r: 0, g: 0, b: 0 };

function rgbToCss(r, g, b) {
    return `rgb(${r}, ${g}, ${b})`;
}
```

### Color Editor UI
```html
<div class="color-wrapper">
    <div class="color-swatch" id="colorSwatch"></div>
    <div class="hsl-editor" id="colorEditor">
        <div class="hsl-row">
            <label>R</label>
            <input type="range" id="colorR" min="0" max="255">
            <span id="colorRVal">0</span>
        </div>
        <!-- G, B rows... -->
        <div class="hsl-row hex-row">
            <label>HEX</label>
            <input type="text" id="colorHex" maxlength="7">
        </div>
    </div>
</div>
```

---

## Drop Zone Pattern

### File Drop Handler
```javascript
function setupDropZone(zoneId, acceptExtensions, onDrop) {
    const zone = document.getElementById(zoneId);
    
    zone.addEventListener('dragover', (e) => {
        e.preventDefault();
        zone.classList.add('dragover');
    });
    
    zone.addEventListener('dragleave', () => {
        zone.classList.remove('dragover');
    });
    
    zone.addEventListener('drop', (e) => {
        e.preventDefault();
        zone.classList.remove('dragover');
        
        const file = e.dataTransfer.files[0];
        const ext = file.name.toLowerCase().slice(file.name.lastIndexOf('.'));
        
        if (acceptExtensions.includes(ext)) {
            onDrop(file);
        } else {
            alert(`Please drop a ${acceptExtensions.join(', ')} file`);
        }
    });
}
```

### Usage
```javascript
// Image drop zone
setupDropZone('imageDropZone', ['.jpg', '.jpeg', '.png', '.webp'], (file) => {
    const reader = new FileReader();
    reader.onload = (e) => {
        const img = new Image();
        img.onload = () => {
            imageLayers.push({ image: img, x: 100, y: 100, angle: 0, scale: 1 });
            render();
        };
        img.src = e.target.result;
    };
    reader.readAsDataURL(file);
});
```

---

## Export Options

### PNG Export
```javascript
function exportPNG() {
    const link = document.createElement('a');
    link.download = 'design.png';
    link.href = canvas.toDataURL('image/png');
    link.click();
}
```

### SVG Export Pattern
```javascript
async function exportSVG() {
    let svg = `<svg xmlns="http://www.w3.org/2000/svg" 
        width="${width}" height="${height}">`;
    
    // Background
    svg += `<rect width="100%" height="100%" fill="${bgColor}"/>`;
    
    // Elements
    for (const el of elements) {
        svg += serializeElementToSVG(el);
    }
    
    svg += '</svg>';
    // Trigger download
}
```

---

## Tool Communication

### Event-Based Communication
```javascript
// Central event bus
const ToolEvents = {
    ELEMENT_ADDED: 'tool:element-added',
    ELEMENT_REMOVED: 'tool:element-removed',
    DESIGN_LOADED: 'tool:design-loaded',
    DESIGN_SAVED: 'tool:design-saved',
    CANVAS_RESIZED: 'tool:canvas-resized'
};

function emit(event, data) {
    window.dispatchEvent(new CustomEvent(event, { detail: data }));
}

function on(event, handler) {
    window.addEventListener(event, (e) => handler(e.detail));
}
```

### Cross-Tool Sync
```javascript
// In each tool
on(ToolEvents.DESIGN_LOADED, (design) => {
    // Import design from another tool
    importDesign(design);
});

on(ToolEvents.CANVAS_RESIZED, ({ width, height }) => {
    // Sync canvas size across tools
    document.getElementById('imgWidth').value = width;
    document.getElementById('imgHeight').value = height;
    render();
});
```

---

## Tool Templates

### Minimal Tool Template
```html
<!DOCTYPE html>
<html>
<head>
    <style>
        :root {
            --border-color: #000;
            --bg-color: #fff;
            --text-color: #000;
            --font-mono: monospace;
            --font-size: 10px;
        }
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: var(--font-mono); font-size: var(--font-size); }
        .container { display: flex; gap: 5px; height: 100vh; }
        .controls { width: 320px; overflow-y: auto; }
        .preview { flex: 1; display: flex; align-items: center; justify-content: center; }
        canvas { border: 1px solid var(--border-color); }
    </style>
</head>
<body>
    <div class="container">
        <div class="controls" id="controls"></div>
        <div class="preview">
            <canvas id="canvas"></canvas>
        </div>
    </div>
    <script>
        // === STATE ===
        const state = {
            width: 600,
            height: 800,
            elements: []
        };
        
        // === CANVAS ===
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        
        // === RENDER ===
        function render() {
            canvas.width = state.width;
            canvas.height = state.height;
            ctx.fillStyle = '#fff';
            ctx.fillRect(0, 0, state.width, state.height);
            // Draw elements...
        }
        
        // === UI ===
        function initControls() {
            // Create control panel...
        }
        
        // === EVENTS ===
        function initEvents() {
            // Set up interactions...
        }
        
        // === INIT ===
        initControls();
        initEvents();
        render();
    </script>
</body>
</html>
```

---

## Roadmap: Related Tools

Based on this toolbox, these tools could be created:

| Tool | Unique Features | Shared Features |
|------|-----------------|-----------------|
| **Gradient Maker** | Gradient presets, stops editor | Color system, canvas, export |
| **Calligraphy** | Pen presets, pressure simulation | Text elements, custom fonts |
| **Vector Editor** | Path tools, bezier curves | Elements, SVG export |
| **Pattern Generator** | Tile preview, spacing | Image layers, export |
| **Typography Tool** | Kerning, leading, ligatures | Text elements, fonts |

---

## Naming Conventions

### Variables
- `camelCase` for variables and functions
- `PascalCase` for classes and constructors
- `UPPER_SNAKE_CASE` for constants

### Elements
- `textElements` or `lines` for arrays
- `createXxxControls()` for UI generation
- `drawXxx()` for render functions
- `findXxxAtPoint()` for hit testing
- `handleXxxDrop()` for file drops

### UI IDs
- `xxxFoldIcon`, `xxxFoldable` for fold sections
- `xxxSwatch`, `xxxEditor` for color pickers
- `xxxDropZone` for file drops
- `addXxxBtn` for add buttons

---

## WordPress Integration

### Overview
Tools can integrate with WordPress to import images directly to the Media Library. This is useful for:
- Featured image creation
- Bulk image generation
- Custom image tools for WordPress sites

### Architecture

```
┌─────────────────────┐     ┌──────────────────────────────────────────┐
│  WordPress          │     │  Tool (external URL)                     │
│                     │     │                                          │
│  ┌───────────────┐  │     │  URL params:                             │
│  │ Admin Menu   │  │────►│  ?nonce=abc                              │
│  │ Media Modal  │  │     │  &post_id=123                            │
│  │ Featured Img│  │     │  &return_url=https://...                 │
│  └───────┬───────┘  │     │  &allowed_domains=domain.com           │
│          │          │     │                                          │
│    ┌─────▼─────┐    │     │  "Send to Media Library" button        │
│    │ localStg  │◄───┼─────│  → localStorage key: wp_import_{nonce}  │
│    │ polling   │    │     │                                          │
│    └─────┬─────┘    │     │                                          │
│          │          │     │                                          │
│    AJAX upload     │     │                                          │
│    to Media Library│     │                                          │
└─────────────────────┘     └──────────────────────────────────────────┘
```

### Tool Side: URL Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `nonce` | Yes | WordPress nonce for security |
| `post_id` | No | Post ID to set as featured image |
| `return_url` | No | URL to redirect after send |
| `allowed_domains` | No | Comma-separated whitelist for return_url validation |
| `button_label` | No | Custom label for the send button |

### Tool Side: Implementation

```javascript
// Configuration
const wpConfig = {
    nonce: null,
    postId: null,
    returnUrl: null,
    allowedDomains: ['your-domain.com', 'localhost'],
    buttonLabel: 'SEND TO MEDIA LIBRARY'
};

// Parse URL params
const params = new URLSearchParams(window.location.search);
if (params.get('nonce')) wpConfig.nonce = params.get('nonce');
if (params.get('post_id')) wpConfig.postId = params.get('post_id');
if (params.get('return_url')) wpConfig.returnUrl = params.get('return_url');

// Show button if nonce present
if (wpConfig.nonce) {
    document.getElementById('sendToWpBtn').style.display = 'inline-block';
}

// Save to localStorage on send
function sendToWordPress() {
    const imageData = canvas.toDataURL('image/png');
    const storageKey = 'wp_import_' + wpConfig.nonce;
    const importData = {
        imageData: imageData,
        nonce: wpConfig.nonce,
        postId: wpConfig.postId,
        timestamp: Date.now()
    };
    localStorage.setItem(storageKey, JSON.stringify(importData));
}
```

### WordPress Plugin Requirements

1. **Menu item** - Add to Media Library submenu
2. **Media tab** - Add "Create Image" tab to media modal
3. **Import handler** - AJAX endpoint to receive and save image
4. **localStorage poller** - JavaScript to detect and import

### WordPress Plugin Example

```php
// AJAX handler
add_action('wp_ajax_cc_image_tool_import', function() {
    check_ajax_referer('cc_image_tool_import', 'nonce');
    
    $image_data = $_POST['image_data']; // base64
    $post_id = isset($_POST['post_id']) ? intval($_POST['post_id']) : null;
    
    // Decode and upload
    $data = base64_decode(preg_replace('#^data:image/\w+;base64,#i', '', $image_data));
    $upload = wp_upload_bits('image-' . time() . '.png', null, $data);
    
    // Create attachment
    $attachment_id = wp_insert_attachment($attachment, $upload['file']);
    
    // Optionally set featured image
    if ($post_id) {
        update_post_meta($post_id, '_thumbnail_id', $attachment_id);
    }
    
    wp_send_json_success(['attachment_id' => $attachment_id]);
});
```

### Security

| Layer | Implementation |
|-------|----------------|
| Nonce | WP generates, tool validates & echoes back |
| Domain validation | Tool validates return_url against whitelist |
| Capability check | WP checks `upload_files` capability |
| Single-use | localStorage cleared after import |

---

## Future Features

### Multiple Image Support
Allow sending multiple images to WordPress in one session:
- Queue images in localStorage
- Batch import via AJAX
- Progress indicator

### Direct postMessage
Replace localStorage polling with direct window.postMessage:
- More reliable
- No timing issues
- Requires same-origin or explicit message handling

### Tool URL Configuration
Allow tool URL to be configured via:
- WordPress filter: `apply_filters('cc_image_tool_url', $url)`
- PHP constant: `define('CC_IMAGE_TOOL_URL', 'https://...');`
- Admin settings page
