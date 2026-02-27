# Featured Image Tool - Project Summary

## Overview

The Featured Image Tool is a client-side HTML/CSS/JS application for creating high-resolution featured images with customizable text, geometric lines, and image layers. Originally designed as a standalone tool, it has been extended to integrate directly with WordPress, allowing images to be sent directly to the Media Library.

---

## What We Built

### Core Tool Features

1. **Canvas-based Image Generation**
   - Customizable dimensions (width/height)
   - Portrait/Landscape orientation presets
   - Export as PNG or SVG

2. **Text Elements**
   - Multiple text lines with individual positioning
   - Rotation control
   - Custom fonts (drag & drop .ttf/.otf/.woff/.woff2 files)
   - Global font size control

3. **Geometric Lines**
   - Positionable lines with rotation
   - Adjustable length and thickness

4. **Image Layers**
   - Drag & drop images onto canvas
   - Scale, rotate, position like other elements
   - Object-fit: cover rendering

5. **Background Options**
   - Solid background color (RGB picker)
   - Background image with cover scaling

### WordPress Integration

1. **Plugin: `creativecoding-xyz-image-tool`**
   - Located in: `/web/app/plugins/creativecoding-xyz-image-tool/`
   - Adds "Create Image" menu item under Media Library
   - Media modal tab integration

2. **Communication Flow**
   ```
   WordPress → Tool (iframe via postMessage + localStorage)
   Tool saves → localStorage key: wp_import_{nonce}
   WordPress polls → Imports image to Media Library via AJAX
   ```

3. **Security**
   - Nonce-based authentication
   - Domain validation for return URLs
   - WordPress capability checks (`upload_files`)

---

## Technical Implementation

### Tool URL Configuration

The tool can be configured to work with different domains:

| Environment | URL |
|------------|-----|
| Local Development | `http://uqam-ecole-de-design.test/featured-image-tool/index.html` |
| Production | `https://edd.creativecoding.xyz/featured-image-tool/index.html` |

The WordPress plugin auto-detects the environment and sets the appropriate URL.

### WordPress Integration Parameters

| Parameter | Description |
|-----------|-------------|
| `nonce` | Security token from WordPress |
| `post_id` | Optional - set as featured image for this post |
| `return_url` | URL to redirect after sending |
| `allowed_domains` | Comma-separated whitelist for return_url validation |
| `button_label` | Custom label for the send button |

### File Structure

```
featured_image_tool/
├── index.html              # Main tool (all-in-one HTML/CSS/JS)
├── toolbox.md             # Technical documentation for future tools
├── context.md             # Original context document
└── .git/hooks/post-commit # Auto-deploys index.html to website

website/web/
├── app/plugins/creativecoding-xyz-image-tool/
│   ├── creativecoding-xyz-image-tool.php
│   └── assets/admin.js
└── featured-image-tool/
    └── index.html         # Deployed copy (via git hook)
```

---

## Errors and Bugs Encountered

### 1. Button Not Appearing in WordPress Iframe

**Issue:** The "Send to Media Library" button didn't appear when the tool was loaded in the WordPress admin iframe.

**Root Cause:** 
- The tool was opening the **local file** (`file:///...`) instead of the deployed version
- URL parameters weren't being parsed correctly in the iframe context
- postMessage wasn't reliably delivering the nonce

**Solution:**
- Added URL hash fallback for nonce: `#nonce=xxx`
- Added postMessage listener for dynamic config
- Made button always visible initially for testing, then hidden by default

**Prevention:**
- Document the deployment URL clearly
- Add visible debug indicator when developing iframe integrations
- Test in actual iframe context, not just standalone

### 2. Redirect Happening Inside Iframe

**Issue:** After clicking "Send to Media Library", the WordPress admin page loaded inside the iframe instead of the parent window.

**Root Cause:** `window.location.href` was being called on the iframe, not the parent.

**Solution:**
```javascript
// Changed from:
window.location.href = wpConfig.returnUrl;

// To:
if (window.parent && window.parent !== window) {
    window.parent.postMessage({ type: 'cc_image_import_ready' }, '*');
}
```

**Prevention:**
- Always check `window.parent !== window` when in iframe context
- Use postMessage for cross-frame communication instead of direct navigation

### 3. Git Hook Path Issue

**Issue:** The post-commit hook needed to copy to the website directory.

**Solution:** Created `.git/hooks/post-commit` that copies `index.html` to `/web/featured-image-tool/`

**Prevention:**
- Set up deployment hooks early in projects
- Test hooks after environment changes

### 4. WordPress Plugin Constant Error

**Issue:** PHP undefined constant warning for `CC_IMAGE_TOOL_URL`.

**Solution:** Removed the reference since we're using dynamic URL detection instead.

**Prevention:**
- Clean up unused code before deployment
- Run linting on all files

---

## Lessons Learned

### For Future Tool Development

1. **Always test in context** - Tools that run in iframes need actual iframe testing, not just standalone
2. **Debug visibility** - Add visible debug indicators for cross-frame issues where console isn't accessible
3. **Multiple communication channels** - Use both URL params AND postMessage for redundancy
4. **Clean separation** - Keep debug code behind feature flags or remove entirely before deployment
5. **Document deployment flow** - Make it clear where the tool should be hosted and how to update it

---

## Relationship with Other Tools

### The Toolbox Concept

This tool was built using principles documented in `toolbox.md`, which establishes:

- **Element System** - Common interface for text, lines, images
- **Rendering Pipeline** - Standardized render function patterns
- **Interaction System** - Hit testing, drag state, keyboard modifiers
- **UI Components** - Control panels, foldable sections
- **Data Persistence** - JSON save/load schema
- **Drop Zone Pattern** - Reusable file drop handlers

### Future Related Tools

Based on this foundation, these tools could be created:

| Tool | Unique Features | Shared Features |
|------|-----------------|-----------------|
| **Gradient Maker** | Gradient presets, stops editor | Color system, canvas, export |
| **Calligraphy** | Pen presets, pressure simulation | Text elements, custom fonts |
| **Vector Editor** | Path tools, bezier curves | Elements, SVG export |
| **Pattern Generator** | Tile preview, spacing | Image layers, export |
| **Typography Tool** | Kerning, leading, ligatures | Text elements, fonts |

### Cross-Tool Communication

The WordPress integration demonstrates a powerful pattern: **tools don't exist in isolation**. By enabling communication between:

1. **Tool ↔ WordPress** - localStorage + postMessage
2. **Tool ↔ External Apps** - Could use BroadcastChannel API, Service Workers, or WebSockets
3. **Tool ↔ Tool** - Shared localStorage namespace or URL params

This opens creative possibilities:

- **Multi-tool workflows** - Create a gradient in one tool, use it in another
- **Live preview** - See changes in real-time on an external site
- **Asset libraries** - Share custom fonts, images, colors between tools
- **Batch operations** - Generate variations across multiple tools
- **Collaborative editing** - Multiple users working on the same design

### Communication Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    External Systems                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  WordPress  │  │   Browser   │  │  Other Tools/Sites  │ │
│  │   (AJAX)    │  │localStorage │  │  (postMessage)     │ │
│  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘ │
│         │                │                     │             │
│         └────────────────┼─────────────────────┘             │
│                          │                                   │
│                    Tool (iframe or standalone)               │
└─────────────────────────────────────────────────────────────┘
```

---

## Deployment

### Local Development

1. Make changes in `featured_image_tool/index.html`
2. Commit changes: `git add . && git commit -m "description"`
3. Git hook automatically copies to `/web/featured-image-tool/`

### WordPress Plugin

1. Plugin located at: `/web/app/plugins/creativecoding-xyz-image-tool/`
2. Activate in WordPress admin → Plugins
3. Access via: Media → Create Image

### Production

1. Push commits to remote
2. Deploy website (includes plugin)
3. Tool available at configured URL

---

## Credits

This tool evolved from an idea by Jérôme Rigaud, based on a design by Matthias Kreutzer for the UQAM Inventaire book.
