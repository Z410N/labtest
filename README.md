# AGI Portable Peer Public Test

This repository is the minimal public download page for testing AGI portable
peer onboarding before the main AGI repository is made public.

It intentionally contains no source code, runtime workspaces, private Git
registry data, API keys, peer keys, or user secrets. The only public files here
are:

- this README;
- `public-network.json`, the public bootstrap network manifest;
- `checksums.txt`, the SHA256 digests for the current binaries;
- release assets attached to the GitHub release.

Current public test release:
[`v0.2.48-public-test.1`](https://github.com/Z410N/labtest/releases/tag/v0.2.48-public-test.1)

## What This Portable Does

The portable peer is a single executable that joins the AGI public test network.
When it is running with a working LLM backend, it:

- connects to the public bootstrap peers listed in `public-network.json`;
- joins the default shared `gpt2-tinystories` research experiment;
- asks your configured LLM to propose real research patches;
- runs the local experiment loop and records signed run results;
- receives run results from other peers over libp2p;
- verifies non-self peer runs in community-verifier mode;
- keeps the private Git registry disabled for this public onboarding path.

For the first public test phase, staking/slashing verifier registration is not
enabled. Each public peer behaves as both a research worker and a community
verifier so more users can participate with one simple portable file.

## Requirements

You need:

- Windows 10/11 or a recent Linux x86_64 system;
- network access to the public bootstrap peers over TCP;
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
  [`agi-peer-windows-x64.exe`](https://github.com/Z410N/labtest/releases/download/v0.2.48-public-test.1/agi-peer-windows-x64.exe)
- Linux:
  [`agi-peer-linux-x64`](https://github.com/Z410N/labtest/releases/download/v0.2.48-public-test.1/agi-peer-linux-x64)
- Checksums:
  [`checksums.txt`](https://github.com/Z410N/labtest/blob/main/checksums.txt)
- Public network manifest:
  [`public-network.json`](https://github.com/Z410N/labtest/blob/main/public-network.json)

Expected SHA256:

```text
94df213d30fa0c2ea5a695001c3db7251a267b5627c9bc645cbec259efaaf624  agi-peer-linux-x64
b9bcb517b82e15f54933a7f55d72968243a50491d185fd824fb20cf14ffd95ef  agi-peer-windows-x64.exe
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

Create `workspace/config/llm.json`. The file stores the backend choice, not the
API key itself.

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

That manifest currently uses public bootstrap peers only and leaves
`registry_git_url` empty, so normal users do not need GitHub credentials or the
private AGI repository.

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

## What Healthy Startup Looks Like

At startup the peer prints:

- the workspace path;
- the network config source;
- the selected LLM backend;
- the number of configured public bootstrap peers;
- the selected project and experiment;
- the verification mode.

The expected public-test path is:

```text
network-config=https://raw.githubusercontent.com/Z410N/labtest/main/public-network.json
startup-mode=automatic
verification-mode=community
starting peer for project=gpt2-tinystories
```

If public bootstrap peers are online, the peer should connect through P2P. If
they are offline outside a scheduled test window, the peer may report that no
bootstrap peers are reachable and start in fallback/seed mode. That is useful
diagnostic behavior, but scheduled public tests should run with the bootstrap
peers online.

## Check Status

The two most useful files are:

```text
workspace/state/network/health.json
workspace/state/network/metrics.json
```

Healthy network status usually shows:

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
  }
}
```

The exact counts depend on how long the peer has been running and how much
peer work is available to verify. For a basic smoke test, look for:

- `status` or `network_status` becoming `connected`;
- `degraded=false` while bootstrap peers are reachable;
- `executed_runs` increasing after a real LLM call completes;
- `completed_verifications` increasing when peer runs are available;
- `llm_progress.success_count` increasing;
- `llm_progress.failure_count=0` during a healthy provider window;
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

This release was tested before publication through the public mirror path:

- anonymous Windows and Linux release downloads succeeded;
- SHA256 values matched `checksums.txt`;
- a no-env fresh Windows dry run connected to the three bootstrap peers;
- the official 60-minute five-peer public-mirror gate passed with `ok=true`;
- five peers used real LLM calls;
- Windows and Linux fresh-peer runs propagated through the bootstrap mesh;
- community verification completed;
- the private registry stayed disabled for public onboarding.

Known warning-level behavior from the accepted gate:

- direct fresh Windows to fresh Linux visibility may be absent while
  bootstrap-mediated data exchange still works;
- some retryable announcement suppressions can appear while ACK progress and
  run propagation continue;
- startup-only libp2p connection-close tracebacks may appear during inbound
  connection churn.

These are not considered gate failures when the peer remains healthy, real LLM
work progresses, runs propagate, and verifications complete.

## Troubleshooting

### The binary does not start on Windows

Windows may mark downloaded executables as coming from the internet. If needed,
right-click the file, open Properties, and use Unblock. Then run it from
PowerShell again.

### No LLM backend was detected

Either create `workspace/config/llm.json`, log in to the selected flat-plan CLI,
or set the required API key environment variable in the same terminal.

### Connected peers stay at zero

Check whether the public bootstrap peers are expected to be online for the
current test window. The peer can start alone, but real public-network testing
needs reachable bootstrap peers.

### `executed_runs` does not increase

Check `llm_progress.provider_status`, `provider_last_pause_kind`,
`provider_last_pause_reason`, and `provider_paused_until` in `metrics.json`.
Most early stalls are provider quota, provider auth, or provider timeout
problems rather than P2P problems.

### Verifications stay at zero

The peer needs non-self peer runs to verify. In a one-peer or offline-bootstrap
test, research can run locally while community verification waits for peer
work.

## Security And Privacy Notes

- Keep API keys and workspace keys private.
- Do not upload your workspace directory.
- The public mirror is intentionally small so release assets and public network
  configuration are easy to inspect.
- The current public test is for network onboarding and early token-distribution
  readiness. It is not the final staking/slashing production mode.
