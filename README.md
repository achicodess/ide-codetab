# Project IDEcodetab
Project IDEcodetab is a fully conatained, broswer based IDE for Python. Using this, you can write, execute and debug your programs in real-time on your own browser. This project reqires no server, toolchain, Node.js or backend for functioning. Every interaction is cached. 
## How It Works
Three external libraries are loaded and they do all the processes necessary for functioning of the IDE.
The Libraries are:
* Monaco Editor (v0.45.0)
* Pyodide (v0.26.4) 
* localStorage

## More About The 3 Core Libraries
* Monaco Editor: This is the exact same editor engine that powers VS Code. It is loaded through it's AMD Module.
(require.config + require(['vs/editor/editor.main'])). A color scheme is also provided for code awareness like pink for keywords etc and not just a <textarea> with colors for real-language aware editing.
* Pyodide: CPython 3.12 is compiled to WebAssembly. It runs inside a browser *tab* with no server involved. The script tag is dynamically injected at runtime rather than just being hardcoded in the HTML head, so that the ~25MB Wasm download is initiated only after the page shell renders. This avoids a blocking load.
* localStorage: The browser's built-in key-value store, used as a lightweight persistent filesystem under the key abstractide_v3. All the existing files survive page refreshes automatically.

## The Visual Layout Structure
This project is a nested Flexbox tree that fills 100vh × 100vw

.ide  (flex-direction: column)
 ├── .titlebar          — macOS-style traffic lights + active filename
 ├── .tabbar            — open file tabs + Run button + pkg installer
 ├── .body  (flex-direction: row)
 │    ├── .activity     — icon strip (explorer, search, shortcuts)
 │    ├── .sidebar      — file tree with +new header
 │    └── .editor-area  (flex-direction: column)
 │         ├── #monaco-container   ← flex: 1, fills remaining space
 │         ├── .resize-bar         ← draggable divider
 │         └── .terminal           ← output panel, fixed height
 └── .statusbar         — branch, Python runtime indicator, line/col
 The .editor-area uses flex: 1 on the Monaco container and a fixed pixel height on the terminal, when you drag the resize bar, it adjusts the terminal's height and calls editor.layout() to tell Monaco to re-measure its container.

 ## Features Of The IDE
 1. File System (in-memory + localStorage):
All the files live in a plain JS object { filename: content }. On every keystroke, a Monaco onDidChangeContent listener syncs the model's content back into this object and calls localStorage.setItem. On load, localStorage.getItem restores your state or seeds from DEFAULTS if it's your first visit. The three default files (main.py, models.py, utils.py) are baked directly into the JS as template strings.

2. Monaco Model-per-File Architecture
This is an important design choice. Each open file gets its own monaco.ITextModel object stored in the model's dictionary. When we switch tabs, the editor calls editor.setModel(models[name]) rather than replacing text. This means that each file maintains independent undo/redo history, cursor position, scroll state, and selection — exactly like VS Code. Models are created on first open and disposed on tab close to free up memory.

3. Python Execution Pipeline
When you hit Run, the flow can be seen as:
a. The active file's content is flushed from the Monaco model into files[active].
b. Pyodide redirects sys.stdout and sys.stderr to io.StringIO capture buffers via pyodide.runPython(...).
c. the code is executed with pyodide.runPython(files[active]).
d. The captured buffers are read back, split on newlines, and rendered into the terminal as color-coded lines — *cyan for info, green for success, red for errors, yellow for warnings*.
e. sys.stdout and sys.stderr are restored to their originals.
f. Execution time (via performance.now()) is reported

Errors are caught in a try/catch synatx so a Python exception doesn't crash the runtime but you rather get a traceback in red and can keep editing.

4. Package Installation
The ⬇ pkg button opens a modal that calls micropip.install(packageName). micropip is Pyodide's pip equivalent for the browser. It fetches pure-Python wheels from PyPI that works in a WASM environment. This works for packages like numpy, pandas, requests, and many others. Packages with compiled C extensions only work if Pyodide's team has pre-built a Wasm version for it.

5. File Explorer
The sidebar renders Object.keys(files).sort() as <div> elements with color-coded extension dots (green for .py, cyan for .md, orange for .toml, etc.). Right-clicking on any file opens a positioned context menu with Open, Rename, and Delete actions. The "+" button in the header opens a modal for naming a new file i.e if you omit the extension it defaults to .py, and the new file is seeded from a language-specific template.

6. Tabs
The tab strip is a scrollable flex container (overflow-x: auto, hidden scrollbar). Each tab renders a close button that's opacity: 0 until hovered or active. This is a common VS Code pattern. "Ctrl+W" closes the active tab, and focus automatically moves to the next nearest tab in the open list.

7. Resizable Terminal
An IIFE at the bottom of the JS attaches mousedown/mousemove/mouseup listeners to implement drag-to-resize. On drag, it calculates newHeight = startH + startY − currentY, clamps it between 32px (collapsed header only) and 65vh, applies it to the terminal's inline style, and calls editor.layout() so Monaco reflows. The terminal also auto-expands if it's collapsed when new output arrives to showcase the result of the code.

8. Color Theme
A custom Monaco theme defined via monaco.editor.defineTheme is implemented. It extends vs-dark and overrides token colors with the "Dracula Palette" from the original mockup:
a. pink (#FF79C6) for keywords and operators
b. green (#50FA7B) for functions and class names
c. cyan (#8BE9FD) for variables
d. yellow (#F1FA8C) for strings
e. purple (#BD93F9) for numbers
f.slate-grey (#545F80 italic) for comments.
The colors object overrides every Monaco UI surface — cursor, selection, scrollbars, minimap, IntelliSense widget, bracket highlights and the "aestetics" is not broken.

9. Status Bar
The bottom strip part shows the current git branch label (cosmetic), a pulsing dot that turns green once Pyodide finishes loading, the active language, encoding, and live line/column position (updated on every onDidChangeCursorPosition event from Monaco).

10. Available Keyboard Shortcuts
The shortcuts can be viewed on the left hand side corner, 3rd icon.


The website can be used and found at https://idetab.netlify.app/
