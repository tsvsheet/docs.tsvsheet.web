---
title: tsvsheet.web
---

**tsvsheet.web** is the grid editor for [tsvsheet](https://github.com/tsvsheet/tsvsheet) — a spreadsheet for plain text. It is a standalone, fully client-side webapp: open a `.tsvt` file into an editable computed grid, edit cells and formulas, and save canonical text back. The tsvsheet engine runs in the page as WebAssembly, so every cell is computed exactly as every other tsvsheet frontend computes it, and the file never leaves your machine.

- Source: `tsvsheet/tsvsheet.web` (not yet public)
- Desktop app: [tsvsheet on the desktop](desktop.md)
- Language: [tsvsheet/tsvsheet](https://github.com/tsvsheet/tsvsheet)

## Open, edit, save

**Open** picks a `.tsvt` file and renders it as a computed grid with column letters and row numbers — values where the sheet has values, computed results where it has `=formulas`. Editing a cell shows its _raw source_ (the formula, not its result); committing the edit recomputes the whole sheet through the engine. On browsers with the File System Access API, **Save** writes straight back to the file you opened; on browsers without it, opening uses the classic file picker and saving delivers the file as a download.

What lands on disk follows one promise: **saving a document you have not edited writes back the exact text you opened, byte for byte.** Once you have edited, the saved file is the canonical `.tsvt` source emitted by the engine — formulas stay formulas, and computed values are never written into your file.

## Keyboard

The grid is fully keyboard operable, following the [ARIA grid pattern](https://www.w3.org/WAI/ARIA/apg/patterns/grid/): arrow keys move the selection, Tab steps across cells, Enter opens the selected cell's source in an editor, Enter commits, and Escape cancels without changing anything. **Ctrl+D** (Cmd on macOS) fills the selected cell from the cell above and **Ctrl+R** from the cell to its left — the copy's relative references shift to the new position while `$`-pinned coordinates stay put, exactly as fill works in Excel; on the top row or first column there is no source cell, so the keys do nothing.

## Undo and redo

Every committed edit — cell changes, row and column inserts and deletes — is one step in the history. **Undo** and **Redo** walk that history; undoing all the way back returns the document to its exact opened text, restoring the byte-identical save promise.

## Diagnostics

Cells the engine cannot compute are marked in place, and a diagnostics strip below the grid lists every diagnostic as `cell: message`. The grid also indicates when a sheet is _volatile_ — when it wraps a cell in `volatile()`, marking that its value re-evaluates over time, so it can differ between computations and the view refreshes on a schedule.

## Offline and self-contained

The editor is static files only — HTML, CSS, JavaScript, and the WebAssembly engine, all bundled together. It makes no network requests after loading, talks to no server, and needs nothing installed: serve the built app from any static file host, or run it entirely offline. Your spreadsheets are opened, computed, and saved locally in the browser.
