---
title: tsvsheet.web
---

**tsvsheet.web** is the grid editor for [tsvsheet](https://github.com/tsvsheet/tsvsheet) — a spreadsheet in plain text. It is a standalone, fully client-side webapp: open a `.tsvt` file into an editable computed grid, edit cells and formulas, and save canonical text back. The tsvsheet engine runs in the page as WebAssembly, so every cell is computed exactly as every other tsvsheet frontend computes it, and the file never leaves your machine.

- Source: `tsvsheet/tsvsheet.web` (not yet public)
- Desktop app: [tsvsheet on the desktop](desktop.md)
- Language: [tsvsheet/tsvsheet](https://github.com/tsvsheet/tsvsheet)

## Open, edit, save

**Open** picks a `.tsvt` file and renders it as a computed grid with column letters and row numbers — values where the sheet has values, computed results where it has `=formulas`. Editing a cell shows its _raw source_ (the formula, not its result); committing the edit recomputes the whole sheet through the engine. On browsers with the File System Access API, **Save** writes straight back to the file you opened; on browsers without it, opening uses the classic file picker and saving delivers the file as a download.

What lands on disk follows one promise: **saving a document you have not edited writes back the exact text you opened, byte for byte.** Once you have edited, the saved file is the canonical `.tsvt` source emitted by the engine — formulas stay formulas, and computed values are never written into your file.

## Keyboard

The grid is fully keyboard operable, following the [ARIA grid pattern](https://www.w3.org/WAI/ARIA/apg/patterns/grid/): arrow keys move the selection, Tab steps across cells, Enter opens the selected cell's source in an editor, Enter commits, and Escape cancels without changing anything. **Ctrl+D** (Cmd on macOS) fills the selected cell from the cell above and **Ctrl+R** from the cell to its left — the copy's relative references shift to the new position while `$`-pinned coordinates stay put, exactly as fill works in Excel; on the top row or first column there is no source cell, so the keys do nothing.

## Select, copy, paste

Hold **Shift** with the arrow keys to grow the selection into a rectangle, click a cell to select it, Shift+click to extend to it, and **Ctrl+A** (Cmd on macOS) to select the whole sheet. A range that crosses hidden rows or columns includes them — hiding changes what you see, never what an edit means.

**Ctrl+C** copies the selection to the clipboard as plain tab-separated text carrying each cell's _source_ — formulas as formulas, not their results — so a copied block pastes into a text editor, a terminal, or another spreadsheet as-is. A dashed outline marks what was copied until your next edit.

**Ctrl+V** pastes at the selection's top-left. Pasting a block you copied in the editor works the way a spreadsheet should: each formula's relative references shift by the distance of the move, `$`-pinned coordinates stay put, and a reference pushed off the sheet becomes `#REF!`. Pasting text from anywhere else lands exactly as written — plain values become values, and anything starting with `=` becomes that formula, unshifted. A whole pasted block is one edit: one Undo removes it, and a block containing a malformed formula is refused whole, leaving the sheet untouched. The editor never reads your clipboard on its own — the only read is the paste you perform.

**Delete** (or Backspace) clears every cell in the selection, as one undoable edit. **Cut is not available yet**: cutting in a spreadsheet _moves_ cells, which follows different reference rules than copying, so until the editor can do that correctly it declines — copy, paste, then Delete the original.

## The view a sheet declares

A `.tsvt` can declare what a viewport should do with it, on `#.` lines — which rows and columns to hide, which carry headers, which stay pinned while the rest scrolls. The editor honours all three: hidden rows and columns are not drawn, header rows read as headers, and frozen rows and columns stay put as you scroll.

Hiding never renumbers. Row 7 is row 7 whether or not rows 3–6 are hidden, because every formula still addresses them, so the gutter marks the skip rather than closing it up.

**Show hidden** appears in the toolbar when a sheet hides something. It reveals without unhiding: the revealed cells are marked as hidden rather than shown as ordinary data, the document does not become dirty, and the file is untouched — so a sheet you share still carries the view its author declared.

## Undo and redo

Every committed edit — cell changes, row and column inserts and deletes — is one step in the history. **Undo** and **Redo** walk that history; undoing all the way back returns the document to its exact opened text, restoring the byte-identical save promise.

## Diagnostics

Cells the engine cannot compute are marked in place, and a diagnostics strip below the grid lists every diagnostic as `cell: message`. The grid also indicates when a sheet is _volatile_ — when it wraps a cell in `volatile()`, marking that its value re-evaluates over time, so it can differ between computations and the view refreshes on a schedule.

## Offline and self-contained

The editor is static files only — HTML, CSS, JavaScript, and the WebAssembly engine, all bundled together. It makes no network requests after loading, talks to no server, and needs nothing installed: serve the built app from any static file host, or run it entirely offline. Your spreadsheets are opened, computed, and saved locally in the browser.
