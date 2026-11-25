# Windows desktop

## Purpose and facilities
Turns a web application into a Windows program that people install and open like any other, with its own window and icon, using Electron. The window shows the application's own web build, packed inside the program and loaded from a private address (such as `ourpoker://app/`), so it works without a web host and nothing on the internet can pose as the application.

It is built to be safe by default. The page inside the window gets no access to the computer's files or programs. It may talk to one server, the application's own. Links go to the system browser only when they point to an approved address and the person has just clicked. Downloads, screen recording and opening other pages inside the window are refused. The camera and microphone can be used only after the person has interacted with the window, and never while the window is a shared board (see the display-mode component).

The component also builds the desktop version of the web client and packages it, and includes the checks a Windows release runs on the finished program.

Facilities: `desktop-host` (Turns the web application into an installable Windows program with its own window, locked down by default).

## User inputs
There are no installer settings. The application gives these values in code to `start` and `build` (or through the optional `windows-desktop-configuration` port); all are required:

- `appName`: the program and window title, for example `OurPoker`.
- `scheme`: the private address scheme the window loads from: lower-case letters, digits and hyphens, starting with a letter, and not `http`, `https`, `file`, `data` or `javascript`. For example `ourpoker`.
- `serverOrigin`: the exact address of the application's server, such as `https://ourpoker-server.example.workers.dev`, with no path, user name or trailing slash. It must be https, except `http://localhost`, `http://127.0.0.1` or `http://[::1]` for a server on the same computer.
- `externalOrigins`: the addresses the program may open in the system browser, each https with no path (for example `["https://ourapps.app"]`); an empty list allows none.
- `partition`: the name under which the window keeps its storage (preferences, sign-in token), for example `persist:ourpoker`. A name starting `persist:` keeps it between runs.
- `modeEvent`: the browser event display-mode sends when the window switches between player and board, for example `app:mode`; the same name as display-mode's.
- `webRoot`: for `start`, the folder holding the packaged web build (inside the program, `resources/web`); for `build`, the web project to build. An absolute path.
- `outputRoot`: where `build` writes the desktop web build (`desktop/build/web`), an absolute path outside `webRoot`.

## Artefacts and dependencies
Every implementation file maps to `components/windows-desktop/` with its `implementation/` prefix removed: `desktop/src/main.cjs` (`start`), `policy.cjs` (the rules above), `shell.cjs` (the window), `desktop/scripts/build.cjs` (`build`) and the verification helpers, including `verification/result.mjs`, which reads and writes aepm's check context and results. SOURCE.md records where each file came from and SOURCE.json pins the source hashes. The application supplies Electron itself; `examples/desktop` is a small independent program with a locked Electron 45.0.0 and electron-builder 26.15.3, used by the checks. `tools.lock.json` pins Node 26.5 and npm 11.17.

## Provided interfaces
`interfaces/desktop-host.json` is an OpenRPC 1.3.2 document for `ae/desktop-host` version 1.0.0: one method per exported function of `desktop/src/main.cjs`, `policy.cjs`, `shell.cjs`, `desktop/scripts/build.cjs` and `verification/desktop.cjs`, plus the functions of the returned policy (`Policy.*`) and shell (`Shell.*`), with parameters, results and the errors thrown as JSON Schema 2020-12.

## Required interfaces
`windows-desktop-configuration` (`ae/windows-desktop-configuration` version 1.0.0) is optional: the values above are the application's own and can be passed in code, so a design may leave it unconnected. `interfaces/required-windows-desktop-configuration.json` is the JSON Schema 2020-12 document of its fields.

## Integration
1. Build the desktop web client: `build({ webRoot: <web project>, outputRoot: <desktop/build/web> })` runs the web project's `npm run build:desktop` with `VITE_DESKTOP=1`. Set `APP_VERSION` and `VITE_APP_VERSION` for the version the program shows.
2. Package that output with electron-builder as `resources/web`, together with the `components/windows-desktop/desktop` sources, so Electron can be found from there.
3. In the program's main file, call `start({ appName, scheme, serverOrigin, externalOrigins, partition, modeEvent, webRoot })` with `webRoot` the packaged `resources/web`.
4. In the web client, publish the mode with display-mode before anything uses the camera or microphone; a window whose mode is unknown is refused capture.

`examples/consumer.cjs` shows the calls.

## Visual definition and connected examples
`visual/definition.json` is the component block. `visual/examples/connected.json` wires an illustrative configuration provider and consumer. `visual/output/connected.svg` and `connected.png` are the rendered diagram and `visual/output/description.md` its text equivalent.

## Infrastructure
None: no accounts and no services. Building the installer needs Node and npm; it can be done on a Mac or on Windows. The installer is not code-signed by this component.

## Verification
The required check (`node --test tests/policy.test.cjs tests/result.test.cjs tests/runtime.test.cjs tests/package-staging.test.cjs`) runs without Electron. It checks that the page has no access to the computer, that only the player's own window can use the camera and microphone and only after an interaction, that a switch to board mode cancels a pending request, that downloads, screen capture, redirects and pages opened inside the window are refused, that only the exact configured addresses are allowed, that packaged files cannot be read from outside the web folder, that the web build is made and served after the program is moved, and that packaging copies exactly the right files and refuses links and changed files.

Two Windows checks run on the finished program: `node scripts/platform-check.cjs package` builds the unsigned installer from `examples/desktop` and checks its contents, and `node scripts/platform-check.cjs runtime` (Windows x64 only) starts the program, loads the page, and checks navigation, the mode switch and camera access with test devices before and after a switch to board mode.

## Scenarios and presentation
A family installs the game on the living-room computer. It opens in its own window, connects to the game's server and plays like the web version. A link to the store's page opens in the browser after a click; a link to anywhere else does nothing. When the computer is turned into the shared table view, its camera and microphone stop and stay off until it becomes a player again. If someone copies the program folder elsewhere, it still finds its pages.

## Updates and removal
Updates replace the component's files. The window's stored data lives in the `partition` on the person's computer and is not deleted when the program is updated or the component removed; uninstalling the program follows the installer's own rules.
