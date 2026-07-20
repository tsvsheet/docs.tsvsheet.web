# The host file port (`FilePort`)

The editor in [tsvsheet.web](https://github.com/tsvsheet/tsvsheet.web) is host-agnostic: everything a host can differ in — how files open, how they save, how destructive actions are confirmed — sits behind one interface, the host file port. A new shell (desktop, mobile, anything that can host a webview) implements this interface and changes zero editor code. This page is the contract for anyone building such a shell.

## The interface

The authoritative definition lives at [src/ports/port.ts](https://github.com/tsvsheet/tsvsheet.web/blob/main/src/ports/port.ts):

```typescript
/** An opened document: its text and enough identity to save back. */
interface PortDocument {
	readonly name: string;
	/** The `.tsvt` source, verbatim. */
	readonly text: string;
	/** Opaque host token for saveExisting; null when saves cannot re-target. */
	readonly handle: unknown;
}

/** Everything the editor asks of its host. Methods reject with PortError. */
interface FilePort {
	/** Interactive open. Resolves null when the user cancels. */
	open(): Promise<PortDocument | null>;
	/** Save back to the document's origin. */
	saveExisting(doc: PortDocument, text: string): Promise<void>;
	/** Interactive save-as. Resolves the new document, or null on cancel. */
	saveAs(suggestedName: string, text: string): Promise<PortDocument | null>;
	/** Ask the user to confirm discarding unsaved changes. */
	confirmDiscard(name: string): Promise<boolean>;
	/** Host-initiated opens (OS file association); the editor subscribes once. */
	onOpened(listener: (doc: PortDocument) => void): void;
	/** Host-initiated teardown; the listener answers whether it may proceed. */
	onCloseRequested(listener: () => Promise<boolean>): void;
}
```

`handle` is whatever the host needs to save back to the document's origin — a File System Access handle in the browser, an absolute path in a native shell — and is opaque to the editor. A `null` handle means the document cannot be re-targeted, and `saveExisting` rejects with `not-retargetable`.

`onCloseRequested` lets a host that can intercept its own teardown (window close, app backgrounding) ask the editor whether closing is safe; the editor's listener runs its own `confirmDiscard` flow when the document is dirty and answers with a verdict. Hosts that cannot intercept teardown never fire it.

## `PortError` codes

Every failure a port reports is a `PortError` carrying a stable `code`; the editor branches on codes, never on message strings.

| Code | Meaning |
| --- | --- |
| `cancelled` | The user backed out of an interactive operation. |
| `denied` | The host refused access (permissions, security policy). |
| `io` | The read or write itself failed. |
| `not-retargetable` | `saveExisting` was called on a document whose `handle` is `null`. |

Interactive cancellation is usually expressed as a `null` resolution (`open`, `saveAs`); the `cancelled` code exists for hosts whose APIs surface cancellation as a failure.

## Implementations

| Host | `open` / `saveAs` | `saveExisting` | `onOpened` | `onCloseRequested` |
| --- | --- | --- | --- | --- |
| Browser — shipped in [tsvsheet.web](https://github.com/tsvsheet/tsvsheet.web) ([src/ports/browser.ts](https://github.com/tsvsheet/tsvsheet.web/blob/main/src/ports/browser.ts)) | File System Access API where available; `<input type=file>` picker + anchor-download fallback elsewhere | writes through the FS Access handle; `not-retargetable` in fallback mode | never fires (no OS association on the web) | never fires (the native `beforeunload` prompt covers teardown) |
| Wails desktop shell — shipped in [tsvsheet.ui](https://github.com/tsvsheet/tsvsheet.ui) | native dialogs via the Go bridge | Go writes the absolute path held as `handle` | OS file-association and recents opens → listener | window-close intercept → listener verdict |
| iOS WKWebView shell — future | `UIDocument` document browser | `UIDocument` save via a WKWebView message handler | document-browser open events | scene-teardown intercept |

## Rules for shell implementers

- **Hosts are injected.** The port is handed to the editor once, at bootstrap, by the composition root — the only host-aware code in the app. Editor code never sniffs host globals mid-flight.
- **The editor owns serialization.** It writes _verbatim text_ through the port; the byte-identical unedited-save promise is the editor's responsibility, not the port's. **Ports never transform content** — no newline normalization, no encoding changes, no trimming. Read exact bytes in, write exact bytes out.
- **A shell exposes exactly the port's methods, nothing more.** A bridge with extra editor-facing methods is scope creep to reject at review; the contract is the whole surface.
- **Recent files are shell-native, not a port method.** The shell renders its own recents menu, reads the chosen file itself, and delivers it through `onOpened` — exactly like an OS file-association open. The editor neither knows nor cares that a recents list exists.
