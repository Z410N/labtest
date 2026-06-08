# AGI Portable Peer Public Test

This repository is the minimal public download page for testing AGI portable
peer onboarding before the main AGI repository is made public.

It intentionally contains no source code, runtime workspaces, private Git
registry data, API keys, peer keys, or user secrets. The only public files here
are:

- this README;
- `public-network.json`, the public network manifest;
- `checksums.txt`, the SHA256 digests for the current binaries;
- release assets attached to the GitHub release.

Current public test release:
[`v0.2.105-public-test.1`](https://github.com/Z410N/labtest/releases/tag/v0.2.105-public-test.1)

## What This Portable Does

The portable peer is a single executable that joins the AGI public test network.
When it is running with a working LLM backend, it:

- joins discovered peers through libp2p rendezvous nodes, uses DHT as a
  fallback after initial contact, or starts locally in seed mode while
  continuing to look for peers;
- advertises and discovers peers over libp2p rendezvous instead of the older
  HTTP peer-directory endpoint;
- uses libp2p circuit relay / hole-punch support so NAT-bound peers can join
  through reachable relay peers instead of needing direct inbound reachability;
- tries UPnP port mapping during public startup and advertises only addresses
  that are usable by the libp2p network;
- asks interactive users which approved public runtime to join; each runtime
  maps to a stable canonical shared experiment, and the normal public menu does
  not create new shared experiments for now;
- asks your configured LLM to propose real research patches;
- runs the local experiment loop and records signed run results;
- receives run results from other peers over libp2p;
- verifies non-self peer runs in community-verifier mode;
- keeps the private Git registry disabled for this public onboarding path.
- prints compact `[network]` and `[experiment]` status lines so you can see
  how many peers are connected, whether the peer is waiting correctly, and whether real research
  work is progressing.

For the first public test phase, staking/slashing verifier registration is not
enabled. Each public peer behaves as both a research worker and a community
verifier so more users can participate with one simple portable file.

## Requirements

You need:

- Windows 10/11 or a recent Linux x86_64 system;
- normal outbound network access;
- normal outbound access to the public rendezvous/relay peers; inbound TCP
  `4101`, UPnP, or manual port forwarding is useful if you want your machine
  to be reachable directly, but NAT-bound peers can still join through relay
  when at least one configured relay peer is reachable;
- one working LLM backend, either API-key based or a local flat-plan CLI login;
- enough local disk space for the workspace, run records, logs, and temporary
  project files.

Supported LLM choices:

| Backend | Auth method | Notes |
| --- | --- | --- |
| `chatgpt-plan` | local `codex` CLI login | Recommended for flat-plan ChatGPT/Codex users. |
| `claude-plan` | local `claude` CLI login | Uses an existing Claude CLI subscription/login. |
| `gemini-plan` | local `gemini` CLI or `npx @google/gemini-cli` login | Uses an existing Gemini CLI login. |
| `openai-compatible` | `OPENAI_API_KEY` | API-key backend. |
| `claude-api` | `ANTHROPIC_API_KEY` | API-key backend. |
| `gemini-api` | `GEMINI_API_KEY` | API-key backend. |

Your LLM provider may charge you or count usage against a flat-plan quota. The
portable records quota/rate-limit pauses and keeps syncing, but it cannot bypass
provider limits.

## Download

Download the binary for your platform from the current release:

- Windows:
  [`agi-peer-windows-x64.exe`](https://github.com/Z410N/labtest/releases/download/v0.2.105-public-test.1/agi-peer-windows-x64.exe)
- Linux:
  [`agi-peer-linux-x64`](https://github.com/Z410N/labtest/releases/download/v0.2.105-public-test.1/agi-peer-linux-x64)
- Checksums:
  [`checksums.txt`](https://github.com/Z410N/labtest/blob/main/checksums.txt)
- Public network manifest:
  [`public-network.json`](https://github.com/Z410N/labtest/blob/main/public-network.json)

Expected SHA256:

```text
115f9ee2c6a320a67506ee75e8206e3de164a8537689ce5309a94b86dc1afe45  agi-peer-linux-x64
c15d287c257fd2eec045d96bbbce204afd36efbf91129c5b9885a50787247871  agi-peer-windows-x64.exe
```

## Verify The Download

Windows PowerShell:

```powershell
Get-FileHash .\agi-peer-windows-x64.exe -Algorithm SHA256
```

Linux:

```bash
sha256sum ./agi-peer-linux-x64
```

The printed hash must match `checksums.txt`. If it does not match, delete the
file and download it again.

## Create A Fresh Workspace

Keep the binary and workspace in a clean folder. The workspace is where your
local config, peer identity, run records, logs, and metrics are stored.

Windows PowerShell:

```powershell
mkdir agi-public-test
cd agi-public-test
mkdir workspace
mkdir workspace\config
```

Linux:

```bash
mkdir -p agi-public-test/workspace/config
cd agi-public-test
```

Do not reuse an old workspace when you want a clean test. Reusing a workspace
also reuses the same peer identity and local history, which is useful for a
long-running peer but not for a fresh onboarding check.

## Configure Your LLM

The portable can guide you through this on first start. If
`workspace/config/llm.json` does not exist, the launcher scans the supported
LLM backends, shows the full backend list, and asks which one to use. It asks
even when exactly one backend is already ready, so you can explicitly choose
between a flat-plan CLI login and an API-key backend. After setup it saves only
the backend choice and model in `workspace/config/llm.json`; API keys stay in
your environment or in the optional local secret file created by guided setup.
If `workspace/config/llm.json` already exists, the launcher still shows the LLM
choice menu and marks the saved backend as the default; press Enter to keep it
or choose another listed backend to replace the saved config.

You can also create `workspace/config/llm.json` yourself before starting the
peer. The file stores the backend choice, not the API key itself.

### Option A: ChatGPT/Codex Flat Plan

Use this if you already have the `codex` CLI installed and logged in.

Check login:

```powershell
codex login status
```

Create `workspace/config/llm.json`:

```json
{
  "schema_version": 1,
  "llm": {
    "backend": "chatgpt-plan",
    "model": "gpt-5.4"
  }
}
```

### Option B: OpenAI-Compatible API Key

Set the API key in the terminal that will launch the peer.

Windows PowerShell:

```powershell
$env:OPENAI_API_KEY = "your_api_key_here"
```

Linux:

```bash
export OPENAI_API_KEY="your_api_key_here"
```

Create `workspace/config/llm.json`:

```json
{
  "schema_version": 1,
  "llm": {
    "backend": "openai-compatible",
    "model": "gpt-5.4",
    "api_key_env": "OPENAI_API_KEY"
  }
}
```

### Option C: Claude Or Gemini

For subscription/flat-plan CLI auth, use:

```json
{
  "schema_version": 1,
  "llm": {
    "backend": "claude-plan",
    "model": "claude-sonnet-4-6"
  }
}
```

or:

```json
{
  "schema_version": 1,
  "llm": {
    "backend": "gemini-plan",
    "model": "gemini-2.5-pro"
  }
}
```

For API keys, use `claude-api` with `ANTHROPIC_API_KEY` or `gemini-api` with
`GEMINI_API_KEY`.

## Start The Peer

The portable embeds this public manifest by default:

```text
https://raw.githubusercontent.com/Z410N/labtest/main/public-network.json
```

That manifest currently has no official static bootstrap peers and leaves
`registry_git_url` empty, so normal users do not need GitHub credentials or the
private AGI repository. Discovery uses two libp2p rendezvous peers, enables
DHT fallback after initial contact, enables circuit relay / hole-punching, and
uses no HTTP peer-directory URL. Your portable advertises itself through
rendezvous and DHT, reserves relay paths when reachable relay peers are
available, then also syncs peer knowledge with peers after the first libp2p
connection. If no active peer is discovered immediately, your portable starts
in seed mode for the selected approved runtime and keeps checking
rendezvous/DHT/relay paths for later peers.

Windows PowerShell:

```powershell
.\agi-peer-windows-x64.exe --workspace .\workspace
```

Linux:

```bash
chmod +x ./agi-peer-linux-x64
./agi-peer-linux-x64 --workspace ./workspace
```

If you want the interactive guided setup instead of config-file startup, run:

```powershell
.\agi-peer-windows-x64.exe --workspace .\workspace --manual
```

or:

```bash
./agi-peer-linux-x64 --workspace ./workspace --manual
```

## Choose An Approved Runtime

Interactive startup shows the approved public runtimes from
`public-network.json`, currently:

- `gpt2-tinystories`
- `astrophysics`
- `financial-analysis`
- `ecdsa-secp256k1`

It then asks which approved runtime to join. The rows are sorted by active peer
count, so row `1` is the recommended runtime when peers are already visible.
When a runtime has no visible peer yet, selecting it starts your portable as the
first seed for that runtime's canonical shared experiment. For example,
selecting `2` for `astrophysics`, `3` for `financial-analysis`, or `4` for
`ecdsa-secp256k1` is valid even when the active peer count is `0`.

`ecdsa-secp256k1` wraps the ECDSA.fail benchmark. It is selectable from the
portable, but real local runs require Bash, Rust/Cargo/rustup, and a C compiler.
If those tools are not available yet, use one of the Python runtimes first.

The normal public portable menu does not create new shared experiments for now.
Users join approved runtimes only. This keeps public startup simple and avoids
letting one machine define a research task that other peers cannot reproduce.

A completely new runtime/project is different. It must be packaged in a future
portable release with project config, training/evaluation entry points, dataset
access, metric definition, resource limits, and verification/replay rules that
every peer can reproduce.

## What Healthy Startup Looks Like

At startup the peer prints:

- the workspace path;
- the network config source;
- the libp2p rendezvous peer count;
- whether DHT fallback is enabled;
- the selected LLM backend;
- whether static bootstrap peers are configured;
- the selected project and experiment;
- the verification mode.

The expected public-test path is:

```text
network-config=https://raw.githubusercontent.com/Z410N/labtest/main/public-network.json
startup-mode=automatic
libp2p-rendezvous-peers=2
libp2p-dht-discovery=enabled
libp2p-relay-service=enabled
libp2p-hole-punching=enabled
verification-mode=community
starting peer for project=gpt2-tinystories
status-feedback=enabled interval=30s signals=[network,experiment]
```

With the current public manifest, there are no official static bootstrap peers.
A first peer can start in seed mode, register with the configured libp2p
rendezvous peers, and participate in DHT discovery after it has an initial
network contact. Later peers keep checking rendezvous/DHT and can join through
any dialable address they find. If your machine is behind NAT or a firewall, it
may publish no reusable direct inbound address; it can still discover and dial
a reachable public peer, and can be reached through a configured relay when a
relay reservation is available.

## Live Console Feedback

The portable prints two compact status lines while it runs:

```text
[network] OK status=connected connected_peers=2 peer_id=12D3KooW...abcd degraded=no connected to 2 peers
[experiment] WORKING project=gpt2-tinystories mode=unified_peer runs=1 verifications=0 llm=1/2 failures=0 best_metric=3.42 llm_call=in_progress since=...
```

`[network] OK` means the peer is connected to at least one other peer.
`[network] READY status=seed` is also valid when this is the first visible
peer: it is listening and waiting for other peers to discover it. `[network]
CHECK` means the health file reports degraded state and the line includes the
reason.

`[experiment] WORKING` means the peer is actively calling the selected LLM or
waiting for the first completed run. `[experiment] OK` means run or
verification progress has been recorded. `[experiment] PAUSED` means the LLM
provider is paused, usually for quota, rate limit, auth, or temporary provider
availability.

The default refresh interval is 30 seconds. Advanced users can set
`AUTORESEARCH_PUBLIC_STATUS_INTERVAL_SECONDS` to a different value, or set
`AUTORESEARCH_PUBLIC_STATUS=0` to disable these console summaries.

## Check Status

The two most useful files are:

```text
workspace/state/network/health.json
workspace/state/network/metrics.json
```

Healthy network status with peers usually shows:

```json
{
  "status": "connected",
  "degraded": false,
  "connected_peer_ids": ["..."]
}
```

Healthy research and verification progress is visible in `metrics.json`:

```json
{
  "executed_runs": 1,
  "completed_verifications": 1,
  "network_status": "connected",
  "llm_progress": {
    "attempt_count": 1,
    "success_count": 1,
    "failure_count": 0,
    "provider_status": "ready"
  },
  "registry_publish": {
    "enabled": false
  },
  "peer_directory_publish": {
    "enabled": false
  }
}
```

The exact counts depend on how long the peer has been running and how much
peer work is available to verify. For a basic smoke test, look for:

- `status=seed` when no peers are visible, or `status=connected` after a
  reachable peer address is discovered;
- `degraded=false`;
- `executed_runs` increasing after a real LLM call completes;
- `completed_verifications` increasing when peer runs are available;
- `llm_progress.success_count` increasing;
- `llm_progress.failure_count=0` during a healthy provider window;
- no repeated libp2p TCP, DHT, signed-record, or relay recovery spam in normal
  console output;
- `registry_publish.enabled=false` for the public onboarding path.

## Provider Quota And Rate Limits

Provider quota/rate-limit/auth issues are not P2P failures. When the LLM
provider stops accepting work, the peer records the pause in `metrics.json`:

```json
{
  "llm_progress": {
    "provider_status": "paused",
    "provider_last_pause_kind": "quota_exhausted",
    "provider_paused_until": "..."
  }
}
```

Expected pause kinds include quota exhaustion, rate limiting, auth problems, or
a temporarily unresponsive provider. The peer should keep syncing with the
network while paused and resume LLM work after the provider is usable again.

## Stop The Peer

For a normal terminal run, press `Ctrl+C`.

After stopping, you can confirm no local process remains:

Windows PowerShell:

```powershell
Get-Process agi-peer* -ErrorAction SilentlyContinue
```

Linux:

```bash
pgrep -af agi-peer || true
```

## Files Created In The Workspace

The workspace is local to your machine. Important paths:

| Path | Purpose |
| --- | --- |
| `workspace/config/llm.json` | Your selected LLM backend and model. |
| `workspace/config/llm-secrets.json` | Optional local API-key secret file created only by guided setup. |
| `workspace/keys/` | Local worker and peer identity keys. Keep private. |
| `workspace/state/network/health.json` | Current network health. |
| `workspace/state/network/metrics.json` | Research, verification, LLM, and sync counters. |
| `workspace/state/runs/` | Local signed run records. |
| `workspace/state/verifications/` | Local signed verification records. |

Do not publish your workspace, `config/llm-secrets.json`, or `keys/` directory.

## Current Test Evidence

This release was tested before publication through the private and public
portable paths:

- `v0.2.102` passed the relay-cache recovery production-readiness gate from
  GitHub release artifacts: a fresh Linux peer imported the best known run from
  the bounded relay cache instead of restarting from baseline, relay restart
  persistence recovered the same best from a second fresh Linux peer, and a
  30-minute relay-backup soak with two Linux peers plus one Windows peer
  survived staged peer restarts, stayed `degraded=false`, kept
  `connected_peer_count=3` on all normal peers, and converged to
  `best_metric=3.2399` with
  `best_experiment_id=b4354201c4863320fb3efdf3a0902b927e1f3f51abfdd567c61ce1c9f33ec2b6`;
- `v0.2.99` passed a fresh staggered discovery/best-frontier gate with three
  Linux droplets and one local Windows portable. Peers started 5 minutes apart,
  resolved peer-id-less libp2p rendezvous endpoints dynamically, reached
  `connected_peer_count=3` on all four nodes, reported initial peer/state sync,
  adopted the current network best instead of starting isolated, completed real
  LLM-backed research calls, and improved the public runtime metric from
  baseline `3.448` to best `3.2152`;
- `v0.2.99` keeps public bootstrap peers empty and uses peer-id-less rendezvous
  entrypoints so stale static `/p2p/<peer-id>` values do not break discovery
  after a seed/rendezvous node refresh;
- `v0.2.92` passed the PP-13 controlled public-pilot chaos gate for 2 hours
  with two managed rendezvous/relay nodes plus a local Windows user portable
  and a Linux droplet user portable. The gate included 8 random restarts on
  each user peer, recovered to full user mesh after restart windows, produced
  real LLM-backed research runs, completed community verifications, and ended
  with zero LLM failures, zero push failures, zero stable isolated samples, and
  quiet checked relay/user logs;
- `v0.2.83` passed the dedicated relay/hole-punch NAT dial-through gate with
  two routable Linux rendezvous+relay peers, one Linux NAT-sim peer, and one
  Windows NAT-sim peer;
- the four-peer relay gate held `3/3` connected peers on all four nodes for
  every monitored sample, both NAT-sim peers advertised `/p2p-circuit`
  addresses, and Windows recorded successful circuit dialing to the Linux
  NAT-sim peer;
- normal stderr stayed quiet on all four relay-gate hosts after transient
  libp2p relay/DHT recovery noise was filtered;
- `v0.2.75-public-test.1` passed the previous 60-minute rendezvous/DHT public
  mirror gate with two rendezvous nodes, fresh Windows and Linux research
  peers, real LLM calls, run propagation, community verification progress, and
  private registry disabled;
- `v0.2.73` was also live-tested with DHT fallback: a fresh Windows peer used
  DHT plus one initial seed contact to discover a Linux research peer and
  exchange runs/verifications;
- anonymous Windows and Linux release downloads are verified against
  `checksums.txt`;
- the Windows `--print-config` smoke confirms `bootstrap_peers=[]`,
  `peer_directory_urls=[]`, two libp2p rendezvous peers, DHT enabled, relay
  service enabled, hole-punching enabled, community verification enabled, and
  private registry disabled;
- the public launcher still asks interactive users to choose the LLM backend
  even when a saved config exists or only one ready backend is found;
- ChatGPT/Codex prompts are sent to `codex exec` as UTF-8 and model shortcuts
  like `5.5` are normalized to `gpt-5.5`;
- local peer JSON files such as `llm.json`, `host_key.json`,
  `address_book.json`, and `metrics.json` are accepted even if a Windows editor
  writes a UTF-8 BOM;
- stale peer addresses and repeated provider/connection failures are backed
  off so ordinary users should not see endless reconnect spam;
- live `[network]` and `[experiment]` console feedback distinguishes connected
  peers, valid seed-mode waiting, active LLM work, recorded runs/verifications,
  and provider pauses.

Known public-test caveat:

- NAT-only peers still need at least one reachable rendezvous/relay peer for
  first contact. If every known rendezvous/relay peer is offline and your
  machine is not publicly reachable, your portable can still run locally in
  seed mode, but unrelated NAT-only peers cannot discover it without some
  shared discovery/relay contact.

## Troubleshooting

### The binary does not start on Windows

Windows may mark downloaded executables as coming from the internet. If needed,
right-click the file, open Properties, and use Unblock. Then run it from
PowerShell again.

### No LLM backend was detected

Either create `workspace/config/llm.json`, log in to the selected flat-plan CLI,
or set the required API key environment variable in the same terminal.

### Connected peers stay at zero

With the current no-official-static-bootstrap manifest, a first peer can
legitimately start alone in seed mode. Later peers should discover it through
the configured libp2p rendezvous peers and, after initial network contact, DHT
fallback. If it never becomes visible, check whether your peer has a dialable
public address or can at least make outbound connections to the rendezvous
peers.

### `executed_runs` does not increase

Check `llm_progress.provider_status`, `provider_last_pause_kind`,
`provider_last_pause_reason`, and `provider_paused_until` in `metrics.json`.
Most early stalls are provider quota, provider auth, or provider timeout
problems rather than P2P problems.

### Verifications stay at zero

The peer needs non-self peer runs to verify. In a one-peer test, research can
run locally while community verification waits for peer work.

## Security And Privacy Notes

- Keep API keys and workspace keys private.
- Do not upload your workspace directory.
- The public mirror is intentionally small so release assets and public network
  configuration are easy to inspect.
- The current public test is for network onboarding and early token-distribution
  readiness. It is not the final staking/slashing production mode.

