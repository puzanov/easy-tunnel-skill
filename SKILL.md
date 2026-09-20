---
name: easy-tunnel
description: Expose an already-running local HTTP service through a temporary Cloudflare Quick Tunnel using Docker. Use only when the user explicitly invokes `$easy-tunnel`; never activate it from project context or an inferred need for a tunnel.
---

# Easy Tunnel

Create a temporary public URL for an already-running local web service. Use the official `cloudflare/cloudflared` Docker image, install nothing on the host, return a verified `trycloudflare.com` URL, and keep the tunnel running until the user asks to stop it or stops it themselves.

Interpret the request in any language. Explicit invocation of `$easy-tunnel` is the activation signal; surrounding words do not need to be English.

## Constraints

- Do not invoke this skill implicitly, even when the project appears to need a tunnel.
- Treat explicit invocation as authorization to expose the selected local service publicly.
- Require an existing working Docker installation. Do not install or upgrade Docker, `cloudflared`, runtimes, packages, or system services.
- Do not start, restart, reconfigure, or stop the user's local application unless the user separately asks for that action.
- Use an accountless Cloudflare Quick Tunnel on `trycloudflare.com`; do not request Cloudflare credentials.
- Never stop unrelated containers or processes.

## Find the local origin

1. Use a port or local URL stated by the user when one is provided.
2. Otherwise, infer the likely origin from the immediate conversation, recent server output, project configuration, and active TCP listeners. Useful read-only checks include `ss -ltnpH` and `docker ps` with published-port formatting.
3. Probe only plausible HTTP candidates. Discard the response body and use a short timeout. Try HTTPS only when the project or listener indicates TLS, or when HTTP fails and HTTPS is plausible.
4. Select an origin automatically only when there is exactly one credible candidate. If no credible candidate exists, or multiple candidates remain plausible, ask the user for the port or local URL. Do not guess.
5. Confirm that the selected origin is reachable before creating the tunnel. If it is not reachable, report that fact and ask the user to start or identify the service; do not install or launch a replacement server.

## Start the tunnel

First confirm that `docker version` can reach the Docker daemon. If it cannot, report the blocker and stop.

Use a unique, recognizable container name such as `easy-tunnel-<port>-<timestamp>` and retain that exact name for cleanup.

On Linux, use host networking so a service bound only to loopback remains reachable:

```bash
docker run -d \
  --name "<container-name>" \
  --network host \
  cloudflare/cloudflared:latest \
  tunnel --no-autoupdate --url "http://127.0.0.1:<port>"
```

Preserve the selected scheme and host when they differ from the example. On Docker Desktop environments where host networking is unavailable, route the origin through `host.docker.internal` instead and keep the same port and scheme.

Read the container logs until both conditions occur, with a bounded wait:

- a URL matching `https://*.trycloudflare.com` appears;
- the tunnel reports a registered connection.

Allow a short propagation interval, then verify the public URL with an HTTP request that discards the body. Retry only a few times for transient propagation. If startup or verification fails, inspect the container logs, report the concrete failure, and do not claim success.

## Return the URL and manage shutdown

A successful invocation is not complete until the user receives the verified public URL. Present it prominently and also state the tunnel container name.

Immediately after providing the URL, explicitly ask the user to tell you when to shut the tunnel down. Do not stop it automatically. If the user wants it left running, leave it running and wait for a later instruction. A later unambiguous shutdown request continues this explicitly invoked workflow and does not require the user to invoke `$easy-tunnel` again.

When shutdown is requested, remove only the recorded tunnel container:

```bash
docker rm -f "<container-name>"
```

Confirm that the tunnel container stopped. Do not stop the local application. If the user says they stopped the tunnel themselves, verify that the recorded container is no longer running and acknowledge it.
