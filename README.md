# HudCitizen

💫 HudCitizen – Overlay for Star Citizen to keep eyes on FPS, webpage, trade route and more in the future...

Multi‑window overlay for **Star Citizen**: shortcut dock at the bottom of the screen, movable widgets (FPS, network, kill feed, embedded browser, etc.), window pinning and a global shortcut to show or hide the overlay.

## Language translated

- French (100%)
- English (100%)
- German (100%)
- Spanish (100%)

## Screenshot

<details>
  <img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/5aef53c0-338b-4a13-a0c9-f371abd3600b" />
  <img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/cf23da01-c8f6-4993-8e5d-238e03ebeff0" />
  <img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/a6ccbe8a-e84c-4ec2-87ef-7fc0221c3f55" />
</details>

## Prerequisites

- **Node.js** (LTS recommended, e.g. 20 or 22)
- **npm** (comes with Node)
- On **Windows** (main target of the project)

Optional for full features:

- **[PresentMon](https://github.com/GameTechDev/PresentMon)** (FPS widget): either a **bundled copy** (place `PresentMon.exe` in `vendor/presentmon/` before `npm run electron:build` — MIT license, see `vendor/presentmon/LICENSE-PresentMon.txt`), or installation in the `PATH` / absolute path in **Settings**
- Game `**Game.log`** file for session time and kill feed (default path like `...\StarCitizen\LIVE\Game.log`)

### Caution
- Activating ***global shortcut*** can be detected by windows as a keylogger

## Installing dependencies

At the root of the repo:

```bash
npm install
```

## Run in development

The command first chains an **initial compilation** (`tsc`) to produce `dist-electron/` without a race with Electron, then starts in parallel: **watch** compilation of main/preload, the **Vite** server, and **Electron** only after `http://localhost:5173` responds (`wait-on`).

```bash
npm run electron:dev
```

- **Default shortcut** to show / hide the overlay: `Ctrl+Shift+O` (configurable in **Settings**).
- The bottom dock only appears when the overlay is **shown**.
- A **tray icon** (taskbar) also offers **Show / hide overlay** — useful when the keyboard shortcut does not respond (see below).

If you prefer to launch the steps manually (3 terminals):

1. **Terminal 1**: `npm run dev:main` — wait for the **first** successful compilation (presence of `dist-electron/main/index.js`).
2. **Terminal 2**: `npm run dev` — Vite on port **5173**; keep the server running.
3. **Terminal 3**: `npm run start` — this script automatically chains:

- `**npm run build:electron`**: compiles main/preload once (ensures `dist-electron/` even if terminal 1 is late);
- `**wait-on`**: waits for `http://localhost:5173` to respond (avoids blank windows if Electron starts before Vite);
- launching Electron with `VITE_DEV_SERVER_URL=http://localhost:5173`.

**Do not**: run `npm start` alone while Vite is not started — renderer URLs will be unreachable.

**If you get a “main not found” error** (file missing under `dist-electron/`): run `npm run build:electron` at the project root.

## Checks (lint and build)

Before a commit or to diagnose a compilation problem:

```bash
npm run check
```

This command chains: Electron compilation (`build:electron`), **ESLint** (`lint`), then **production Vite build**.

For lint only:

```bash
npm run lint
```

## Vite server (`http://localhost:5173`)

### What is it used for?

`http://localhost:5173` is the **Vite development server** address. In this project, it is only used to **serve the UI files** (HTML, JS, CSS) to the **Electron process** during development, with fast reload.

In `electron:dev` mode, windows load URLs like:

- `http://localhost:5173/shell.html` — full‑screen overlay (veil + dock)
- `http://localhost:5173/widgets/fps.html` — example widget
- `http://localhost:5173/settings.html` — settings

The root `**.env`** file defines `VITE_PORT` and `VITE_DEV_SERVER_URL`: they must match the actual Vite port (default **5173**).

### Summary

| Context                                  | Role of `localhost:5173`                                                   |
| ---------------------------------------- | -------------------------------------------------------------------------- |
| **Development**                          | Serves the pages loaded by Electron (`shell.html`, widgets, etc.).        |
| **Browser on `/`**                       | Nothing planned: no home page at the root.                                |
| **Production build** (`npm run build`)   | No need for this server anymore: the app reads files from `dist/` locally. |

## Build for production (local files)

Generates the UI in `dist/` and the Electron code in `dist-electron/`:

```bash
npm run build
```

Then start the app **without** the dev server (loads files from `dist/`):

```bash
npx cross-env electron .
```

(Under PowerShell, you can clear the environment variable if it lingers:  
`$env:VITE_DEV_SERVER_URL = $null` then `npx electron .`)

## Create a Windows installer (optional)

After `npm run build`:

```bash
npm run electron:build
```

The result (NSIS installer) is placed in the `**release/**` folder (see `electron-builder` configuration in `package.json`).

## Repository structure

```
├── package.json              # npm scripts, dependencies, electron-builder target
├── eslint.config.js          # ESLint (flat config, TypeScript)
├── vite.config.ts            # Multi-page build (dock, widgets, settings)
├── tsconfig.electron.json    # Main process (ESM)
├── tsconfig.preload.json     # Preload → CommonJS, renamed to `preload/index.cjs`
├── scripts/rename-cjs.mjs
├── src/
│   ├── main/                 # Electron main process
│   │   ├── index.ts          # Entry point, services, timers
│   │   ├── tray.ts           # Tray icon (show / hide)
│   │   ├── window-manager.ts # Windows: dock, widgets, visibility / pinning
│   │   ├── ipc-setup.ts      # IPC, global shortcuts / low-level hook, external URLs
│   │   ├── low-level-toggle-shortcut.ts  # Windows option: WinKeyServer + combo
│   │   ├── accelerator-to-combo.ts       # Parse accelerator → detectable keys
│   │   ├── resolve-win-key-server-path.ts
│   │   ├── store.ts          # Persistence (electron-store)
│   │   ├── types.ts
│   │   └── services/       # Game.log, PresentMon, network stats
│   ├── preload/              # Secure bridge exposed to renderer (`window.overlay`)
│   └── renderer/             # React UI + Vite
│       ├── shell.html        # Full-screen overlay (veil + dock)
│       ├── shell/
│       ├── dock/             # Dock component (icon bar)
│       ├── settings.html     # Settings window
│       ├── settings/
│       ├── widgets/          # One HTML entry + folder per widget (fps, kill-feed, …)
│       ├── components/
│       └── styles/
├── dist/                     # Vite output (generated, do not version)
└── dist-electron/            # Compiled main + preload JS (generated)
```

The `**dist/**`, `**dist-electron/**`, `**release/**` and `**node_modules/**` folders are normally ignored by Git (see `.gitignore`).

## npm scripts (reference)

| Script                   | Role                                                       |
| ------------------------ | ---------------------------------------------------------- |
| `npm run dev`            | Vite server only (web UI).                                 |
| `npm run dev:main`       | Continuous compilation of Electron code (`tsc --watch`).   |
| `npm run electron:dev`   | Full development environment (recommended).                |
| `npm run build:electron` | One‑off main/preload compilation to `dist-electron/`.      |
| `npm run build`          | `vite build` + `build:electron`.                           |
| `npm start`              | Use only if **Vite is already running**.                   |
| `npm run lint`           | ESLint (`eslint.config.js`).                               |
| `npm run check`          | `build:electron` + `lint` + `vite build`.                  |
| `npm run electron:build` | Build + installer creation in `release/`.                  |
| `npm run preview`        | Preview the Vite build (without Electron).                 |


## Troubleshooting: global shortcut while in game

With **Star Citizen** in the foreground in **fullscreen** or **borderless**, the game may capture the keyboard before Windows forwards global shortcuts registered by the overlay (`Ctrl+Shift+O`, etc.). This is not a local configuration issue: it is a **common limitation** with games and the API used by Electron for hotkeys.

**Ideas:**

1. **Tray icon**: left click or context menu → **Show / hide overlay** (works even when the shortcut doesn’t go through in game).
2. **Alt+Tab** to desktop or another window, then use the global shortcut.
3. **Relaunch the overlay executable** while the app is already running: the second instance is ignored and the first one **re‑shows** the overlay (single‑instance lock). You can bind a Windows shortcut or macro to this launch.
4. **Different key combo** in **Settings**: useful if another app uses the same key; often **not enough** if the game blocks the whole keyboard in the foreground.
5. **Low-level keyboard hook (Windows, experimental)**: in **Settings**, tick the option to use the same shortcut via a small helper executable (`[node-global-key-listener](https://www.npmjs.com/package/node-global-key-listener)`), similar to a `WH_KEYBOARD_LL` hook. **Disabled by default**: risk of antivirus false positives, no guarantee with anti‑cheat or the game. If `WinKeyServer.exe` is not found in the packaged app, the overlay automatically falls back to the standard global shortcut.

## License and third‑party game

Respect [RSI’s Terms of Service](https://robertsspaceindustries.com/tos) and their rules on third‑party tools. This tool does not modify the game client; it reads log files and system metrics external to the game process.
