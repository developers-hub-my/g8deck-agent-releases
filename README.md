# G8Deck agent

**The small daemon G8Deck runs on the servers it manages.**

The agent enrols with a G8Deck control plane, sends a heartbeat, and runs work the control plane has signed. The connection is outbound only, and the agent listens on no port.

By **Developers Hub Sdn Bhd** · https://devhub.my · https://g8deck.app · a G8Deck product

This repo holds the compiled downloads only. The source code is private. Get the files from **[Releases](https://github.com/developers-hub-my/g8deck-agent-releases/releases/latest)**.

## You do not install this yourself

The G8Deck control plane downloads a pinned version from these releases and checks it against `SHA256SUMS` and its own pinned checksum. It then serves the binary to your servers, either during first boot (cloud-init) or over its existing SSH session. Your servers never download it from GitHub.

## Downloads

| File | For |
|---|---|
| `g8deck-agent-<ver>-linux-amd64` | x86-64 servers |
| `g8deck-agent-<ver>-linux-arm64` | ARM servers (Graviton, Ampere) |
| `SHA256SUMS` | Checksums |

These are static binaries with no runtime and no dependencies.

## Verify a download

```sh
VER=0.1.0
curl -fLO "https://github.com/developers-hub-my/g8deck-agent-releases/releases/download/v$VER/g8deck-agent-$VER-linux-amd64"
curl -fLO "https://github.com/developers-hub-my/g8deck-agent-releases/releases/download/v$VER/SHA256SUMS"
sha256sum --ignore-missing -c SHA256SUMS
```

## Security model

| Rule | How |
|---|---|
| Outbound only | The agent makes every connection itself, over HTTPS to the control plane. It listens on nothing |
| Keys stay on the server | The agent generates its Ed25519 and X25519 keys locally. Only the public halves are sent, at enrolment |
| Signed requests | Every call carries a timestamped signature, and the control plane rejects replays |
| Signed jobs | A job runs only if it carries the pinned control-plane signature, names this agent, has not expired and has not been seen before |
| Sealed secrets | Secrets reach the agent as sealed boxes addressed to its own key. They are opened in memory and never logged |
| Root | The agent runs as root, because its jobs install packages and manage services. Whoever holds the control-plane signing key can run commands on every enrolled server, and that key lives only on the control plane |
