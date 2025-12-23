# In-Game Editor

The Chip editor is a project-based IDE available in-game.

## Open / Close
- Press `E` to toggle the editor while the tool is equipped.
- Click the red `X` in the top-right to close.

## Projects
- `New` creates a project with a default entry script.
- `Duplicate` clones the current project.
- `Delete` removes the current project.
- `Save` persists the project on the server.

## File Explorer
- Right-click inside the tree to open the context menu.
- `New File` and `New Folder` create items immediately with a default name.
- Rename:
  - Press `F2` on a selected item, or
  - Click an item, pause, click again (rename)
- `Delete` removes a file or folder.
- Folders can be collapsed/expanded by clicking them.

## Editor
- Syntax highlighting with custom selection overlay.
- Line numbers and blinking caret.
- Undo/redo: `Ctrl+Z` / `Ctrl+Y`.
- “Check” validates syntax across all files.

## Entry File
- Right-click a file and choose “Set Entry”.
- The entry file is marked with a `*` in the tree.

## Errors
- Syntax errors appear in the error panel and block execution on placement.
- Runtime errors halt the chip and show a message to the owner.
