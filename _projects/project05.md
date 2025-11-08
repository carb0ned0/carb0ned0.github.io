---
layout: page
title: NovaEdit
description: A lightweight, dual-mode (CLI and GUI) text editor written in pure Python, featuring a custom-built, extensible syntax highlighting engine.
# img: assets/img/12.jpg
importance: 5
category: fun
related_publications: false
---

## Overview

Every developer's most basic tool is a text editor, yet we rarely think about how they work. Modern editors like VS Code are powerful but can be heavy and slow to load. Conversely, terminal editors like `nano` or `vim` are lightning-fast but have a steep learning curve and often lack modern, intuitive features.

My challenge was: **Could I build an editor that combines the best of both worlds?** The goal was to create a single, dependency-free Python application that could run as a fast, `nano`-inspired editor for quick terminal edits, but *also* launch as a full-featured, tabbed GUI application for larger project work. And when I was finished with the [C-Like Programming language](/projects/project04) I was dying for that colorful text editor, although I could change the settings in vscode to read it as a `C` file but the stupid idea of making my own text editor was too tempting. So I architected the entire application, from low-level terminal I/O to the high-level GUI.

## Research

This project's design process was purely architectural, focused on solving three core technical challenges:

1. **How to build a functional terminal UI from scratch?**
    * **Research:** This led me to low-level POSIX terminal controls.
    * **Decision:** I chose to use the `termios` standard library. This allowed me to put the terminal into "raw mode," which is crucial for capturing single keypresses (like `Ctrl+S`) without waiting for the user to press `Enter`. All screen rendering had to be done manually by building a string buffer of ANSI escape codes (`\x1b[...`) to position the cursor, draw text, and apply colors.

2. **How to create a modern, cross-platform GUI?**
    * **Research:** The primary constraint was to have *zero external dependencies*.
    * **Decision:** The only choice was Python's built-in `tkinter` library. I used the `ttk.Notebook` widget to create a modern, tabbed interface. A key UX decision was to use the `platform` module to detect the user's OS, enabling native-feeling shortcuts like `Command+S` on macOS versus `Ctrl+S` on Windows/Linux.

3. **How to avoid writing the core logic twice?**
    * **The Problem:** Both the CLI and GUI modes needed syntax highlighting. Writing two separate highlighting engines would be a disaster for maintenance.
    * **Solution (The "Shared Core"):** I refactored the most complex logic into a shared, abstract engine. The `get_syntax_highlighting()` function takes a line of text and a set of rules, and outputs a generic list of "highlight types" (e.g., `HL_KEYWORD1`, `HL_STRING`).
    * The CLI renderer (`refresh_screen`) translates these types into ANSI colors.
    * The GUI renderer (`highlight_syntax`) translates these *same types* into `tkinter` tags. This was the single most important architectural decision, keeping the code clean, fast, and DRY.

## Implementation

The final application is a single Python file that boots into one of two modes based on a `--gui` flag.

#### 1. The CLI Engine (Default)

The terminal mode is a classic `while True` event loop (`terminal_process_keypress`). It waits for a single keypress using `os.read()`, then processes it through a large state machine.

* **State Management:** An `EditorState` class tracks everything: cursor position, scroll offset (`row_offset`), and the text itself as a list of `EditorRow` objects.
* **Rendering:** The `refresh_screen()` function redraws the *entire* screen on every keypress. It builds a buffer (`AppendBuffer`) of ANSI escape codes to hide the cursor, clear lines, set colors, and finally, draw the status bar and position the cursor.
* **Features:** It includes a custom-built undo/redo stack (`E.undo_stack`), clipboard (`E.clipboard`), and selection logic (`set_mark()`).

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/novaedit-CLI-screenshot.png" title="NovaEdit CLI screenshot.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">NovaEdit in terminal mode, editing a C-Like file with syntax highlighting.</div>

#### 2. The GUI Engine (`--gui`)

The `GUIEditor` class is a `tkinter` application.

* **Tabbed Interface:** It's built around a `ttk.Notebook`, where each tab holds a `Text` widget and its associated metadata (filename, modified status).
* **Native Feel:** It uses `platform.system()` to provide OS-aware key bindings (e.g., `Cmd+Shift+Z` for Redo on Mac, `Ctrl+Y` on Windows).
* **Find/Replace:** A dedicated `Toplevel` window provides a full find, replace, and "replace all" dialog.
* **Undo/Redo:** This mode leverages `tkinter`'s built-in, robust undo stack (`text.edit_undo()`), which was a pragmatic trade-off versus reinventing it.

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/novaedit-GUI-screenshot.png" title="NovaEdit GUI screenshot" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">NovaEdit in GUI mode, editing a C-Like file with syntax highlighting</div>

#### 3. The Shared Syntax Highlighting Engine

This is the project's "secret sauce." A central `SYNTAX_DB` list defines syntax rules (keywords, comment styles) for different languages. The `get_syntax_highlighting()` function parses a line and returns a list of enums, which is then consumed by either the CLI or GUI. I built this to be extensible, and even added the syntax for **"CLIKE," my own custom-built programming language** from another project.

## Results

The final result is a fast, reliable, and portable text editor.

* **Metric of Success:** The editor runs on macOS, Linux, and Windows using *only* a standard Python 3 installation.
* **Outcome:** The CLI mode is genuinely useful, launching instantly for quick configuration file edits. The GUI mode is a robust, lightweight notepad replacement.
* **Lesson Learned:** The sheer complexity of terminal control. Manually handling cursor positioning, especially with tab characters (`\t`), required careful calculation (`get_rendered_col`). This gave me a profound appreciation for what `ncurses` and modern terminal emulators do.
* **What I'd do differently:** I would abstract the Undo/Redo logic into a shared engine, just as I did with the syntax highlighting. The GUI's built-in stack is great, but the CLI's custom one is simpler. A unified "UndoEngine" would make the CLI mode just as robust as the GUI.

## Reflection

* **Stack:** Python 3 and Libraries
  * **`termios` (CLI):** For low-level terminal raw mode control (Unix-only).
  * **`tkinter` (GUI):** For the cross-platform graphical user interface.
  * **`argparse`:** For parsing the `--gui` command-line argument.
  * **`platform`:** To detect the OS for native keymap-awareness.
  * **`signal`:** To gracefully handle terminal window resizing (`SIGWINCH`) in CLI mode.

The project was a personal challenge in **zero-dependency design**. Using only standard libraries forced me to solve problems from first principles, rather than importing a solution. This stack demonstrates that you can build complex, native-feeling applications without a heavy framework, and it forced me to master low-level I/O (`termios`) and GUI event loops (`tkinter`) at the same time.

> View the NovaEdit source code and documentation on [GitHub](https://github.com/carb0ned0/NovaEdit).
