# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PopOut! is a Foundry VTT module that allows users to pop out various UI elements (actor sheets, journal entries, etc.) into separate browser windows. The module intercepts window creation and migrates DOM nodes between windows while preserving functionality.

## Essential Commands

### Development
```bash
# Install dependencies
npm install

# Lint the code
make lint

# Format code with Prettier
make format

# Run tests (minimal TestCafe tests)
make test

# Build the module (transpiles to dist/popout.dist.js)
make build

# Run all checks (lint, format, test)
make all
```

## Architecture & Key Concepts

### Core Mechanism
The module works by:
1. **Proxy Interception**: Uses a Proxy on `ui.windows` to intercept all new application window creation
2. **DOM Migration**: Moves DOM nodes from main window to popup windows, preserving jQuery event handlers
3. **Window Management**: Tracks windows in `PopoutModule._popouts` map and handles lifecycle events

### Critical Code Patterns

#### Window Registry Proxy (popout.js:1318-1338)
The module intercepts window creation by replacing the window registry with a proxy that captures new application instances.

#### DOM Node Migration (popout.js:524-608)
The `PopoutModule.moveNode()` function is critical - it clones nodes with jQuery to preserve event handlers and migrates them between windows.

#### Multi-Document Registry (popout.js:1282-1316)
Manages multiple documents across windows using a proxy-based registry system.

### Module Compatibility Requirements

For a module to be compatible with PopOut:
1. Must use `sheet.element.find()` instead of global jQuery selectors
2. Cannot rely on `document.getElementById()` - must scope to the sheet element
3. Should handle the PopOut hooks if special behavior is needed

### Disabling PopOut
Applications can disable popout by:
- Setting `this._disable_popout_module = true` in the application
- Passing `popOutModuleDisable: true` in render options

## Tooltip Fix (tooltip-fix branch)

The tooltip-fix branch implements a cleaner solution for tooltips in popout windows:

### Implementation Details (popout.js:460-515)
- Creates separate `TooltipManager` instances for each popout window
- Uses Foundry's native tooltip API instead of event forwarding
- Handles tooltip activation/deactivation based on `data-tooltip` attributes
- Ensures tooltips are hidden when popout window loses focus

### Key Changes
1. Removed the brittle `document.getElementById` override
2. Removed the event dispatcher proxy mechanism
3. Each popout window manages its own tooltip lifecycle
4. Simplified event handling using native pointer events

## Known Limitations

1. **Browser Only**: Does not work in Foundry's Electron app
2. **Firefox**: Support dropped due to `instanceof` issues across windows
3. **Framework Support**: Limited support for React/Vue/Svelte-based systems
4. **Tooltips**: Partial support restored in tooltip-fix branch, but some complex tooltips (like D&D 5e AC breakdown) may still have issues

## Testing Considerations

- The module has minimal tests in `tests/popout_tests.js`
- Manual testing is critical due to the window manipulation nature
- Test in multiple browsers (Chrome, Edge, Safari)
- Test with D&D 5e system primarily (best supported)