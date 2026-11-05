---
name: hyperbutler
description: Use when the user runs Docker on Windows 11 with hyperbutler (a Docker-compatible engine in a Hyper-V VM, no WSL), or when a docker command on Windows fails to connect, hangs, cannot bind a port, or cannot mount a folder and hyperbutler is installed (`hyperbutler.exe` on PATH, a `hyperbutler` docker context, or the pipe `docker_engine_hyperbutler`). Covers starting and stopping the engine, settings, ports, bind mounts, reaching the host, tools that ignore docker contexts, moving from Docker Desktop, the MCP server, and troubleshooting. Do not use for Docker Desktop, Rancher Desktop, Podman, or Docker inside WSL.
---

# hyperbutler

hyperbutler runs Docker Engine in its own small Hyper-V VM on Windows 11, without WSL. The
docker CLI and Compose work as usual; hyperbutler adds a few commands of its own to start
the engine, change settings and troubleshoot. Docs: https://hyperbutler.dev and
https://github.com/Higangssh/hyperbutler (the `docs/` folder is the long form).

## First, find out the state

Run these before changing anything; they are read-only:

```powershell
hyperbutler status          # running or stopped, the API pipe, which docker.exe runs, the context
hyperbutler doctor          # what this PC is missing, each with its fix
docker context ls           # which engine a plain `docker` reaches
```

`status`, `version`, `config list/get/path`, `doctor`, `diagnose` and `uninstall` take
`--json` (every document has `"schema": 1`). Prefer `--json` when you parse the output.

## Everyday commands

```powershell
hyperbutler start           # boots the VM; creates and selects the `hyperbutler` docker context
hyperbutler stop            # stops containers and the VM; the context stays selected
hyperbutler restart
hyperbutler logs            # the engine log
hyperbutler gui             # a small window: state, containers, storage, settings
```

The Docker API is `npipe:////./pipe/docker_engine_hyperbutler` (or the pipe set with
`config set pipe`). It is not Docker Desktop's `\\.\pipe\docker_engine`, so both can be
installed at once; `docker context use desktop-linux` goes back to Docker Desktop.

## Settings

```powershell
hyperbutler config list                       # value, default, source, when it applies
hyperbutler config set memory 8GiB            # prints what restart it needs; --apply does it
hyperbutler config set cpus 4
hyperbutler config path                       # settings.json and daemon.json
hyperbutler config daemon apply               # after editing daemon.json; restarts dockerd only
```

Some `daemon.json` keys belong to hyperbutler (`hosts`, `iptables`, `data-root`,
`live-restore` and a few more) and are refused with the reason. Proxy and CA settings
(`proxy.https`, `proxy.http`, `ca.extraFile`, `ca.import`) apply to dockerd's own pulls,
not to containers or `docker build`, as with Docker Desktop.

Ask the user before changing `hostGateway.*`, `pipe`, `data.dir`, `autostart`, proxy or CA
settings: they change what containers can reach or what other programs see.

## Behaviour that differs from what users expect

- **Ports**: `docker run -p 8080:80` listens on `127.0.0.1:8080` and `[::1]:8080` only, not
  on the LAN. If another program holds the port on any address, Docker's "Ports are not
  available" error names that process.
- **Bind mounts**: folders on local drives only (`C:\src`). UNC paths, `\\wsl$\...`, device
  paths and mapped network drives are refused with an explanation; copy the files to a
  local folder or use a named volume.
- **Reaching the host**: `host.docker.internal` reaches services on the host's `127.0.0.1`
  and `[::1]`, any port, and nothing else. `config set hostGateway.enabled false` or
  `hostGateway.ports 5432,8000-8100` limits it at once.
- **Tools that ignore docker contexts** (Testcontainers, the Docker SDKs, some IDEs) use
  `DOCKER_HOST` or Docker Desktop's pipe. Point them at hyperbutler with
  `$env:DOCKER_HOST = "npipe:////./pipe/docker_engine_hyperbutler"` for one shell. hyperbutler
  never sets `DOCKER_HOST` itself; setting it for the user's account is the user's choice.
- **Moving from Docker Desktop**: `hyperbutler import --from-desktop --dry-run` lists the
  images and volumes; without `--dry-run` it copies the chosen ones and leaves Docker
  Desktop as it was.

## Troubleshooting

See [references/troubleshooting.md](references/troubleshooting.md) for the common errors
and their fixes. In short:

- `failed to connect to the docker API at npipe:////./pipe/docker_engine_hyperbutler`:
  the engine is stopped. Run `hyperbutler start`.
- docker commands hang: check `hyperbutler status` for `paused:`. The volume with the data
  disk is almost full; the VM resumes by itself once space is freed.
- anything else: `hyperbutler doctor`, then `hyperbutler diagnose`, which writes a local zip
  for a bug report. It redacts secrets but keeps paths with the user name, so the user
  should look through it before attaching it to an issue.

## AI tools: the MCP server

`hyperbutler mcp` is a Model Context Protocol server on stdio. It is read-only by default
(status, containers, logs, images, volumes, disk and memory use, settings, diagnose).
`--allow-write` adds start, stop, run, pull and a few settings; `--allow-delete` adds the
deletes. See [references/mcp.md](references/mcp.md) for the client configuration and what
each flag allows. Do not add `--allow-write` or `--allow-delete` to the user's MCP
configuration without asking.

## Destructive commands: ask first

Never run these without the user's explicit go-ahead, and show `--dry-run` output first
where there is one:

- `hyperbutler delete`: removes the VM state and the data disk with every image, container
  and volume; keeps settings.
- `hyperbutler uninstall` (`--dry-run` lists everything; `--keep-data` keeps images and
  volumes).
- `docker system prune`, `docker volume rm`, `docker rm -f` on containers you did not start.

## Not supported

- Windows 10 and ARM64 (the guest image is x86_64).
- Windows containers: hyperbutler runs Linux containers only.
- The installed MSI and the zip need different prerequisites; `hyperbutler doctor` names
  what is missing on this machine. Do not run elevated commands, enable Windows features or
  add the user to Hyper-V Administrators yourself; tell the user the exact command and why.
