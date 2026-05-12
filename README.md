# AGI Portable Peer Public Test

This repository is a minimal public release mirror for testing AGI portable
peer onboarding before the main repository is made public.

It intentionally contains no source code, runtime workspaces, keys, secrets, or
private registry data.

## Release Assets

Test release: `v0.2.47-public-test.1`

- Windows: `agi-peer-windows-x64.exe`
- Linux: `agi-peer-linux-x64`
- Checksums: `checksums.txt`
- Public network manifest: `public-network.json`

## Start

Create a fresh empty workspace and configure an LLM backend. For a flat-plan
ChatGPT/Codex CLI setup, create `<workspace>/config/llm.json`:

```json
{
  "schema_version": 1,
  "llm": {
    "backend": "chatgpt-plan",
    "model": "gpt-5.4"
  }
}
```

Run Windows:

```powershell
$env:AUTORESEARCH_PUBLIC_NETWORK_MANIFEST_URL="https://raw.githubusercontent.com/Z410N/labtest/main/public-network.json"
.\agi-peer-windows-x64.exe --workspace .\workspace
```

Run Linux:

```bash
export AUTORESEARCH_PUBLIC_NETWORK_MANIFEST_URL="https://raw.githubusercontent.com/Z410N/labtest/main/public-network.json"
chmod +x ./agi-peer-linux-x64
./agi-peer-linux-x64 --workspace ./workspace
```

## Status

Inspect:

- `<workspace>/state/network/health.json`
- `<workspace>/state/network/metrics.json`

Expected healthy startup:

- `status=connected`
- `degraded=false`
- the peer connects to reachable public bootstrap peers
- `executed_runs` increases after real LLM work
- `completed_verifications` increases when peer work is available to verify

Provider quota or rate-limit pauses are reported in `metrics.json` under
`llm_progress.provider_status`, `provider_last_pause_kind`, and
`provider_paused_until`.

