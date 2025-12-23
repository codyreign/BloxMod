# Deployment and Runtime

## Tool Placement
- Equip the tool.
- A translucent green preview appears under the mouse.
- Click to place the chip at the preview position.
- Preview rotates to align with surfaces (ground or walls).
- Placement is disabled while the editor is open.

## Removal
- Press `Z` to remove the most recently placed chip.
- Press `Z` again to remove earlier chips in order.

## Lifecycle
Each chip instance follows:
1) Compile / Validate
2) Initialize (`init` event)
3) Event loop (`tick`, etc.)
4) Stop / Cleanup

## Server Authority
- Projects are stored and versioned on the server.
- Client edits are proposals; only saved versions execute.
- Syntax is revalidated on placement; invalid code will not execute.

## Cleanup Guarantees
On stop (removal, error, player leave):
- Spawned instances destroyed
- Event connections disconnected
- Timers canceled
- Runtime memory released

## Determinism
- Single-threaded per chip runtime.
- No re-entrant callbacks.
- Events and ticks are serialized.

## Troubleshooting

### “Requested module experienced an error while loading”
Ensure the tool contains the full `ChipSystem` folder, including `Parser` subfolder.

### “Syntax error in main.lua”
Use “Check” in the editor and fix the reported path/line before placing.

### No preview when equipped
Make sure the tool is in StarterPack and equipped.

### Placement does not work
Verify the editor is closed; placement is disabled while open.
