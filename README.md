# tgws-cli

Headless CLI build of [Flowseal/tg-ws-proxy](https://github.com/Flowseal/tg-ws-proxy) for
[zapret-ui](https://github.com/Dezuron/zapret-ui).

Upstream ships a **tray GUI** for Windows and its binary ignores `--host/--port/--secret`
(settings live in `%APPDATA%\TgWsProxy\config.json`). This repository bundles only the
upstream **console entry point** (`proxy.tg_ws_proxy:main`), so an application can supervise
the proxy as a child process and drive it entirely through CLI flags.

The upstream sources are used **unmodified**; the only addition is `entry.py` (a launcher that
imports the upstream module, because it uses relative imports and cannot be frozen directly)
plus the PyInstaller spec.

## Releases

Tag format: `v<upstream>-<revision>`, for example `v1.11.2-1`.

The build workflow checks out `Flowseal/tg-ws-proxy` at `v<upstream>`, freezes the console
entry point, smoke-tests it (`--help` plus a real port bind) and publishes:

- `tgws-cli-windows-x64.exe` — console build, no tray icon, no GUI dialogs;
- `SHA256SUMS.txt`.

A daily schedule keeps the mirror in sync: it looks up the newest
`Flowseal/tg-ws-proxy` release and, when this repository has no build for that upstream
version yet, publishes `v<upstream>-1` on its own. A manual run (`workflow_dispatch`) does
the same for a given `upstream` tag.

`zapret-ui` downloads the binary from this repository's latest release, verifies the GitHub
SHA-256 `digest` and stores it in `%LOCALAPPDATA%\zapret-ui\tgws\`.

## Local build

```powershell
git clone https://github.com/Flowseal/tg-ws-proxy.git upstream
git -C upstream checkout v1.11.2
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -e upstream pyinstaller==6.11.1
.\.venv\Scripts\python.exe -m PyInstaller --clean --noconfirm `
  --distpath tgws-artifacts --workpath build/tgws tgws-cli.spec
```

## Usage

```text
tgws-cli-windows-x64.exe --host 127.0.0.1 --port 1443 --secret <32-hex> \
  [--dc-ip 2:149.154.167.220] [--buf-kb 256] [--pool-size 4] [-v]
  [--no-cfproxy] [--no-h2] [--cfproxy-domain <domain>] [--fake-tls-domain <domain>]
```

The connection link printed at startup is `tg://proxy?server=<host>&port=<port>&secret=dd<hex>`
(or `ee<hex><hex-domain>` when fake TLS is enabled).

## License

Upstream `Flowseal/tg-ws-proxy` is MIT licensed; the MIT text is reproduced in `LICENSE`.
This repository is an independent build wrapper and is not affiliated with Flowseal.
