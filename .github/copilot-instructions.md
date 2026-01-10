# Copilot Instructions - Angles (ComfyUI Camera Angle Selector)

## Project Overview

This is a **ComfyUI custom node** that provides an interactive 3D interface for selecting camera angles. The main deliverable is in `ComfyUI_CameraAngleSelector/` - a Python + JavaScript node that renders a Three.js 3D viewport inline within ComfyUI.

The `_bmad/` folder contains BMad Method tooling for project planning and documentation - it's NOT part of the shipped product.

## Architecture

### ComfyUI Node Structure
```
ComfyUI_CameraAngleSelector/
├── __init__.py                      # Node registration (WEB_DIRECTORY, NODE_CLASS_MAPPINGS)
├── camera_angle_selector.py         # Python backend - legacy ComfyUI API
└── web/js/camera_angle_extension.js # Three.js frontend widget (~740 lines)
```

### Key Patterns

**ComfyUI Extension Pattern** - The JavaScript uses `nodeCreated` callback to inject the 3D widget:
```javascript
app.registerExtension({
    name: "Comfy.CameraAngleSelector",
    async nodeCreated(node) {
        if (node.comfyClass !== "CameraAngleSelector") return;
        // Create and attach 3D widget via node.addDOMWidget()
    }
});
```

**96 Camera Angle Combinations** - Generated from constants:
- `VIEW_DIRECTIONS` (8): front, front-right quarter, right, back-right quarter, back, back-left quarter, left, front-left quarter
- `HEIGHT_ANGLES` (4): low-angle shot, eye-level shot, elevated shot, high-angle shot
- `SHOT_SIZES` (3): close-up, medium shot, wide shot

**Output Format**: `<sks> {direction} {height} {size}` (e.g., `<sks> front view low-angle shot close-up`)

### Frontend Widget Pattern

The JavaScript extension in `web/js/camera_angle_extension.js`:
- Loads Three.js dynamically from CDN (`r128`)
- Uses `CameraAngleWidget` class with inline rendering
- Communicates via `selected_indices` JSON string parameter
- Color-coded cameras: height by body color, size by cone/indicator color

## Development Workflow

### Testing in ComfyUI
1. Copy/symlink `ComfyUI_CameraAngleSelector/` to ComfyUI's `custom_nodes/` directory
2. Restart ComfyUI server
3. Node appears as "Camera Angle Selector" in camera category
4. Browser DevTools console shows any JavaScript errors

### Key Files to Modify

| Task | File |
|------|------|
| Add new angle types | `camera_angle_selector.py` - modify constants |
| Change 3D visuals | `web/js/camera_angle_extension.js` - Three.js scene code |
| Modify output format | `camera_angle_selector.py` - `execute()` method |
| Add UI controls | `camera_angle_extension.js` - `CameraAngleWidget` class |

## Conventions

- **Prompt prefix**: All outputs use `<sks>` prefix (standard for camera angle prompts)
- **Color scheme**: Dark theme matching ComfyUI (`#1a1a2e` background, `#0f3460` accents)
- **Widget sizing**: Minimum 240px height for 3D viewport
- **JSON state**: Selection state stored as JSON array of indices `"[0, 5, 12]"`

## BMad Tooling (Non-Product)

The `_bmad/` directory contains planning framework - ignore when working on the node itself. Relevant outputs:
- `_bmad-output/planning-artifacts/prd.md` - Product requirements (reference only)
- `_bmad-output/planning-artifacts/product-brief-*.md` - Vision document

## Testing Notes

- No automated test suite currently exists
- Manual testing: verify 3D scene loads, click selection works, output connects to downstream nodes
- Check browser console for Three.js or ComfyUI extension errors
- Test both ComfyUI v1.0+ and legacy versions for API compatibility
