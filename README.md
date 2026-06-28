# Fractal Scribe

Turn photos of handwritten reading notes into a structured Obsidian
Zettelkasten, automatically.

Fractal Scribe is a single static web page. You photograph your handwritten
philosophy notes, Claude reads them, and the page writes back tidy Obsidian
markdown — **literature notes** and atomic **concept notes**, cross-linked with
`[[backlinks]]`, tagged, and filed into your vault — without you typing a word.

```
 📷  handwritten pages
       │
       ▼
 Claude (vision)  ──►  extracts ideas, quotes, page numbers, book refs
       │
       ▼
 structured Obsidian notes  ──►  written to your vault on OneDrive
       │
       ▼
 Syncthing  ──►  fanned out to every device you read on
```

There is no build step and no backend. It is one `index.html` you open in a
browser.

---

## What it produces

For each batch of pages, Claude returns two kinds of note:

- **Literature notes** (`LitNotes/`) — one per source: title, author, an
  outline, historical context, chapter/section breakdown, and links to the
  atomic notes drawn from it.
- **Atomic notes** (`AtomicNotes/`) — one per distinct idea: a prose
  explanation with any quotations and page numbers, plus connections to other
  atoms, to its literature note, and to broader "nexus" concepts.

Every generated note carries the tag `fractal-scribe` so you can always tell
app-made notes from hand-made ones.

---

## Setup

You need three things: an Anthropic API key, a OneDrive access token, and two
folders in your vault. Everything is entered in the page and kept **in memory
only** — nothing is stored, logged, or sent anywhere except the two APIs the
app calls.

### 1. Anthropic API key

Get one at <https://console.anthropic.com> → **API Keys**. It starts with
`sk-ant-`. Paste it into the **Connection** card and click **Save**. Image
input is billed per the [Anthropic pricing](https://www.anthropic.com/pricing);
a handful of pages per session costs cents.

### 2. OneDrive access token

The app writes your notes into a OneDrive-hosted vault via the Microsoft Graph
API, so it needs a token with permission to read and write your files.

1. Open [Microsoft Graph Explorer](https://developer.microsoft.com/graph/graph-explorer).
2. Sign in with the Microsoft account that holds your vault.
3. Consent to the `Files.ReadWrite` scope (Graph Explorer → **Modify
   permissions**).
4. Copy the **Access token** from the *Access token* tab.

Paste it into the **Obsidian Vault** card.

> ⚠️ Graph Explorer tokens expire after about **one hour**. When writes start
> failing, grab a fresh token and reconnect. (A first-class OAuth login is on
> the wishlist — see [Limitations](#limitations).)

### 3. Vault folder layout

Enter your vault's path *relative to your OneDrive root*, e.g.
`Obsidian/MyVault`. Inside it, create the two subfolders the app writes to:

```
Obsidian/MyVault/
├── LitNotes/       ← literature notes land here
└── AtomicNotes/    ← atomic concept notes land here
```

Click **Connect**. The app scans existing note titles and tags (titles and
tags only — never note bodies) so Claude can suggest backlinks to notes you
already have.

### 4. Run it

It's a static file — no install:

- **Local:** open `index.html` in any modern browser, or
- **Hosted:** serve the repo anywhere static (e.g. GitHub Pages) and visit the
  URL.

---

## Using it

1. Tap **Handwritten Pages** and select up to 10 photos of your notes. Images
   are downscaled and stripped of EXIF metadata **in your browser** before they
   ever leave the device.
2. Click **Process**.
3. Watch the log as Claude reads the pages and the notes are written to your
   vault.
4. Open Obsidian to review. Done.

---

## Syncing your vault to all your devices with Syncthing

OneDrive is how Fractal Scribe *delivers* notes — it's the inbox the browser
can write to. But you read and edit in Obsidian, often on a phone, a tablet,
and a laptop. You want one vault, identical everywhere, fast, and private.

**[Syncthing](https://syncthing.net)** is the cleanest free way to do that:
open-source, peer-to-peer, end-to-end encrypted, no cloud account, no monthly
fee, no third party holding your notes. Your devices sync the vault folder
directly to each other over your network (or relayed, still encrypted).

### How the two fit together

Syncthing syncs *local folders* — it has no API a browser can POST to, so it
can't replace OneDrive as the app's write target. Instead they layer:

```
Fractal Scribe ─writes──► OneDrive ──► "hub" device (OneDrive client)
                                              │
                                        Syncthing
                                       ┌──────┼───────┐
                                       ▼      ▼       ▼
                                    laptop   phone   tablet
```

Pick one always-on machine as the **hub** (a desktop, an old laptop, a NAS).
Run the official OneDrive client on it so the vault lands in a real local
folder, then let Syncthing fan that folder out to everything else. OneDrive
stays purely as the app's ingress; Syncthing does the real cross-device work.

> If you'd rather not run OneDrive's client anywhere, OneDrive's own apps can
> sync to each device directly — but then you're logging OneDrive into every
> device and trusting the cloud for every read. The hub + Syncthing setup keeps
> your vault private and peer-to-peer once it's past the app's write step.

### 1. Install Syncthing

| Platform | How |
|---|---|
| **macOS** | `brew install syncthing` then `brew services start syncthing` |
| **Windows** | `winget install Syncthing.Syncthing`, or [SyncTrayzor](https://github.com/canton7/SyncTrayzor) for a tray UI |
| **Linux** | `apt install syncthing` (or your distro's package), then enable the user service |
| **Android** | **Syncthing** or **Syncthing-Fork** from the Play Store / F-Droid |
| **iOS / iPadOS** | No official app — use the third-party **Möbius Sync** (see caveat below) |

Each desktop install opens a web UI at <http://localhost:8384>.

### 2. Share the vault folder

1. On the **hub**, in the Syncthing UI: **Add Folder** → point it at your local
   vault folder (the one OneDrive's client syncs) → give it a memorable
   **Folder Label** and note the **Folder ID**.
2. On each other device: **Add Remote Device**, exchange **Device IDs** (each
   device shows its own under **Actions → Show ID**; scan the QR on mobile).
3. Back on the hub, edit the folder → **Sharing** tab → tick the devices you
   just added. Accept the share prompt that appears on each device.

Within seconds the vault replicates everywhere. New notes from Fractal Scribe
now flow: app → OneDrive → hub → Syncthing → every device.

### 3. Point Obsidian at the synced folder

On each device, open Obsidian and **Open folder as vault** → choose the
Syncthing-synced folder. Obsidian's mobile app can open any local folder.

**Tip — avoid config fights:** Obsidian's per-device UI state lives in
`.obsidian/workspace*.json` and churns constantly. Add a Syncthing **ignore
pattern** for it so devices stop overwriting each other's layout:

```
.obsidian/workspace
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.trash
```

(Keep the rest of `.obsidian/` synced if you want shared plugins and hotkeys;
drop all of `.obsidian/` from sync if you'd rather each device be independent.)

### iOS caveat

Syncthing has no official iOS app — Apple's background-execution limits make a
true native client impractical. The common workaround is the third-party
**Möbius Sync** app. If iPhone/iPad is your main reader, OneDrive's own iOS app
(syncing the vault directly) may be simpler for those devices, with Syncthing
handling the rest.

---

## Privacy

- **Keys never persist.** Your Anthropic key and OneDrive token live in a
  JavaScript variable for the session only. Reload the page and they're gone.
  Nothing is written to `localStorage`, cookies, or any server.
- **Photos are minimised locally.** Each image is downscaled (max 1600px) and
  re-encoded on a canvas — which strips EXIF/GPS metadata — *before* being sent
  to Anthropic.
- **Your vault stays yours.** Reading existing notes pulls titles and tags only,
  to suggest backlinks. Note bodies are never sent to Claude.
- **Two destinations, period.** Image and text go to the Anthropic API;
  markdown goes to your OneDrive. There is no Fractal Scribe server.

---

## Limitations

- **OneDrive token expiry (~1 hr).** Graph Explorer tokens are short-lived; a
  proper OAuth sign-in would remove the refresh dance.
- **Output length.** A single Claude call returns the structured notes; very
  large batches of dense pages can hit the response length ceiling. If a run
  fails to parse, split it into fewer pages.
- **OneDrive-only.** The only write backend today is Microsoft Graph. Google
  Drive, Dropbox, or local-folder targets would each need their own adapter.

---

## How it works (for hackers)

One file, no dependencies, no build:

- `index.html` — all markup, styles, and logic.
- Vision + structuring: one call to the Anthropic Messages API
  (`claude-sonnet-4-6`) with the page images and a JSON-only system prompt.
- Storage: Microsoft Graph `PUT /me/drive/root:/{path}:/content`.
- Vault indexing: Graph `children` listing + first-600-chars frontmatter read
  for tags.

PRs welcome.
