# Language Subset

The Chip runtime uses a custom Lua-like parser. It is intentionally strict and does not allow access to Roblox or `require`.

## Supported
- `local` variable declarations
- numeric, string, boolean, and table literals
- arithmetic: `+ - * / %`
- comparisons: `== ~= < <= > >=`
- boolean operators: `and`, `or`, `not`
- `if / elseif / else / end`
- `while / end`
- `for` numeric loops: `for i = 1, 10 do ... end`
- `function` declarations and anonymous functions
- function calls and method calls (`obj:method()`)
- table indexing: `t[key]` and `t.key`
- `return`

## Not Supported
- `require`
- `debug`
- `getfenv/setfenv`
- coroutines
- metatables
- `repeat/until`
- `goto`
- `pairs/ipairs` over Roblox instances
- direct Roblox APIs or Instances

## Entry Script

The entry script is `entryPath` (default: `main.lua`). It must exist and be valid.

## Imports

Use `import(path)` to include other files in the same project.
- Path is relative to project root or current file.
- Imports are cached and cycle-detected.
- Import must return a value.

Example:
```
local utils = import("lib/utils.lua")
```

## Syntax Checking

Use the editor’s “Check” button to validate syntax before saving.
The server validates again during placement. Invalid syntax blocks execution.
