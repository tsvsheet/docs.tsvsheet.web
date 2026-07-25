---
title: Desktop
---

**tsvsheet.ui** puts the [tsvsheet.web](_index.md) grid editor on the desktop as a native app, making `.tsvt` files first-class: double-click one and it opens in the editor. The app embeds the web editor unchanged — same grid, same keyboard behavior, same engine, same byte-identical save promise — and adds what only a native app can:

- **OS file association** — `.tsvt` files are registered to the app, so opening one from Finder or Explorer launches the editor with the file loaded.
- **Native dialogs and menus** — Open and Save use the operating system's file dialogs (filtered to `.tsvt`), driven from a native menu with the standard Open and Save accelerators.
- **Recent files** — a recents submenu in the File menu reopens recently edited spreadsheets.
- **Single-instance opens** — opening a `.tsvt` while the app is already running delivers the file into the running window instead of launching another copy.
- **Unsaved-changes protection** — closing the window or quitting with unsaved edits asks for confirmation before anything is discarded.

Files are read and written verbatim: the app hands the editor your file's exact bytes and writes back exactly what the editor produces, nothing more.

- Source: `tsvsheet/tsvsheet.ui` (not yet public)
- The editor it embeds: `tsvsheet/tsvsheet.web` (not yet public)
- Language: [tsvsheet/tsvsheet](https://github.com/tsvsheet/tsvsheet)
