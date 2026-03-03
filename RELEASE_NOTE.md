# Release Note — OpenClaw XHS Improvements

Repository: https://github.com/Noga-1/openclaw-xhs
Based on upstream project: https://github.com/zhjiang22/openclaw-xhs

## About this release
This repository is an **improvement layer built on top of the original work by @zhjiang22**.
The goal is to improve reliability in cloud/server environments and reduce flaky interaction failures.

Thanks to the original author for the solid wrapper foundation around `xiaohongshu-mcp`.

## Improvements included

### 1) Server startup reliability (`Xvfb` auto-management)
- Auto-start `Xvfb` when running `start-mcp.sh --headless=false` on no-desktop servers.
- Reuse existing display when available.
- Clear installation hints when `Xvfb` is missing.
- Track `Xvfb` PID/log for better operations.
- Optional `Xvfb` cleanup in `stop-mcp.sh`.

**Commit:** `025e425`  
**Message:** `fix: auto-manage Xvfb for non-headless server startup`

### 2) Interaction stability (like/comment retry flow)
- Added robust retry detection in `scripts/mcp-call.sh` for:
  - JSON-RPC top-level `error`
  - `result.isError = true`
- Added timeout/retry controls:
  - `XHS_MCP_TIMEOUT`
  - `XHS_MCP_RETRIES`
- Added `scripts/interact-safe.sh` to make interactive actions safer:
  - automatic retry
  - automatic MCP restart fallback
  - like-API param compatibility handling

**Commit:** `b2cca6a`  
**Message:** `fix: add resilient retry flow for like/comment interactions`

## Compatibility notes
- This repo is still a wrapper and depends on external `xiaohongshu-mcp` binaries.
- Interaction success can still be affected by upstream UI changes / anti-bot behavior.
- The added logic focuses on recovery and retry, not bypassing platform protections.

## Attribution
- Original wrapper project: @zhjiang22 / openclaw-xhs
- Upstream runtime project: @xpzouying / xiaohongshu-mcp
