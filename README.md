# Easy Tunnel Skill

An explicit-only Codex skill that exposes an already-running local web server through a temporary [Cloudflare Quick Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/do-more-with-tunnels/trycloudflare/).

The skill discovers the likely local HTTP port, asks when the choice is ambiguous, runs the official `cloudflare/cloudflared` image with Docker, verifies the public URL, and manages tunnel shutdown. It does not install anything on the host.

## Requirements

- Codex with skills support
- A working Docker installation
- An HTTP or HTTPS service already running locally

No Cloudflare account or locally installed `cloudflared` binary is required.

## Install

Invoke Codex's built-in skill installer and ask it to install this repository:

```text
$skill-installer Install easy-tunnel from the repository root at https://github.com/puzanov/easy-tunnel-skill
```

For a manual user-level installation:

```bash
mkdir -p ~/.agents/skills
git clone https://github.com/puzanov/easy-tunnel-skill ~/.agents/skills/easy-tunnel
```

Restart Codex if the newly installed skill does not appear immediately.

## Use

The skill is intentionally explicit-only. Invoke it by name in any language:

```text
$easy-tunnel expose my local server
```

```text
$easy-tunnel открой туннель к моему локальному серверу
```

You may provide a port directly:

```text
$easy-tunnel expose localhost:3000
```

If no port is supplied, the agent inspects the current project and active listeners. It asks you for the port when it cannot make an unambiguous choice.

After creating and verifying the tunnel, the agent returns the public `trycloudflare.com` URL and asks when it should shut the tunnel down. The tunnel remains active until you request shutdown or stop its Docker container yourself.

## Security and availability

A Quick Tunnel makes the selected local service publicly reachable. It has no uptime guarantee and is intended for testing and development, not production.

## License

[MIT](LICENSE)
