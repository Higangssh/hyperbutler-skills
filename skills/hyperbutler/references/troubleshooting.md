# Troubleshooting hyperbutler

Start with `hyperbutler status` and `hyperbutler doctor`; both are read-only.

## docker cannot connect

```
failed to connect to the docker API at npipe:////./pipe/docker_engine_hyperbutler; check if the path is correct and if the daemon is running: open //./pipe/docker_engine_hyperbutler: The system cannot find the file specified.
```

The `hyperbutler` context is selected and the engine is stopped (after `hyperbutler stop`
or a reboot). Run `hyperbutler start`. It starts at sign-in only with
`hyperbutler config set autostart true`; ask before turning that on, since it creates a
Task Scheduler task.

To use another engine instead: `docker context use desktop-linux` (Docker Desktop) or
`docker context use default` (`DOCKER_HOST` or `\\.\pipe\docker_engine`).

## `hyperbutler start` says the pipe is taken

Another installation or another Windows user's hyperbutler serves the same pipe name. Give
this one its own: `hyperbutler config set pipe \\.\pipe\docker_engine_<name>`. Ask first:
the pipe name is visible to every program on the machine.

## docker commands hang

`hyperbutler status` shows something like
`paused:  C: has 900.0 MiB free; it resumes at 1.2 GiB free`. The free-space guard paused
the VM because the volume with the data disk is almost full. Free space on that volume and
the VM goes on where it stopped; nothing is lost. `docker system df` shows what the engine
uses; pruning is a deletion, so ask before running it.

## "Ports are not available"

Another program (Docker Desktop, a dev server, another user's hyperbutler) listens on that
port on some address. The error names the process. Use another host port, or stop that
program. Published ports are on 127.0.0.1 and [::1] only; for LAN access the user has to
forward the port themselves.

## A bind mount is refused

hyperbutler shares folders on local drives. Network paths (`\\server\share`), WSL paths
(`\\wsl$\...`), device paths and mapped network drives are refused before the container
is created. Copy the files to a local folder such as `C:\src`, or use a named volume.

## A tool does not see hyperbutler

Testcontainers, the Docker SDKs and some IDEs ignore docker contexts. Set
`DOCKER_HOST=npipe:////./pipe/docker_engine_hyperbutler` for that shell or tool. While
`DOCKER_HOST` is set, `docker context use` has no effect in that shell.

## Compose is not found

The zip ships the compose plugin in `clients\cli-plugins`. `hyperbutler start` adds that
folder to the docker CLI's `cliPluginsExtraDirs` only while no other `docker-compose.exe`
is installed; Docker Desktop's compose wins when both exist. With
`context.manage false` nothing is added.

## Containers cannot reach a service on the host

Use `host.docker.internal` (or `--add-host NAME:host-gateway`). It reaches the host's
`127.0.0.1` and `[::1]` only. Check `hyperbutler config get hostGateway.enabled` and
`hostGateway.ports`. A service that listens only on the LAN address is not reachable this
way.

## Corporate proxy or TLS interception

dockerd trusts the Windows root stores' CAs by default (`ca.import`). A proxy for pulls:
`hyperbutler config set proxy.https http://proxy.corp:3128 --apply`. Containers and
`docker build` do not inherit these; put the CA into the image and set the build proxy in
the docker CLI's `config.json` (`proxies`).

## Collecting a report

`hyperbutler diagnose` writes a local zip; nothing is uploaded. It redacts passwords,
tokens, registry auth and proxy credentials but keeps paths (with the user name): the user
should look through it before attaching it to an issue at
https://github.com/Higangssh/hyperbutler/issues.
