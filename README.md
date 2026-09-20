# Easy Tunnel

An explicit-only, agent-agnostic [Agent Skill](https://agentskills.io/) that exposes an already-running local web server through a temporary [Cloudflare Quick Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/do-more-with-tunnels/trycloudflare/).

The skill discovers the likely local HTTP port, asks when the choice is ambiguous, runs the official `cloudflare/cloudflared` image with Docker, verifies the public URL, and manages tunnel shutdown. It does not install anything on the host.

The package follows the open [Agent Skills specification](https://agentskills.io/specification), so it can be used by any coding agent or client that implements the standard and can execute shell commands.

## Requirements

- An [Agent Skills-compatible coding agent](https://agentskills.io/clients)
- A working Docker installation
- An HTTP or HTTPS service already running locally

No Cloudflare account or locally installed `cloudflared` binary is required.

## Install

Use your agent's skill installer with:

```text
Repository: https://github.com/puzanov/easy-tunnel-skill
Skill path: easy-tunnel
```

For a manual installation, copy the `easy-tunnel` directory into your agent's user-level or project-level skills directory:

```bash
git clone https://github.com/puzanov/easy-tunnel-skill
cp -R easy-tunnel-skill/easy-tunnel /path/to/your-agent/skills/easy-tunnel
```

Skill locations and installation commands vary by client. Consult your agent's Agent Skills documentation, then restart or reload the agent if the skill does not appear immediately.

## Use

The skill is intentionally explicit-only. Select or invoke `easy-tunnel` using your agent's skill mechanism, then request the tunnel in any language. For example:

```text
Use the easy-tunnel skill to expose my local server.
```

You may provide a port directly:

```text
Use the easy-tunnel skill to expose localhost:3000.
```

If no port is supplied, the agent inspects the current project and active listeners. It asks you for the port when it cannot make an unambiguous choice.

After creating and verifying the tunnel, the agent returns the public `trycloudflare.com` URL and asks when it should shut the tunnel down. The tunnel remains active until you request shutdown or stop its Docker container yourself.

The Agent Skills standard defines the portable `SKILL.md` package format. Individual clients may use different installation paths, skill selectors, or invocation syntax.

## Security and availability

A Quick Tunnel makes the selected local service publicly reachable. It has no uptime guarantee and is intended for testing and development, not production.

## License

[MIT](LICENSE)
