# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **Home Assistant custom component** (integration) for controlling Japanese SESAME smart locks via the CANDY HOUSE Web API v3. It is a maintained fork of the archived [thematrixdev/home-assistant-sesame-lock-jp](https://github.com/thematrixdev/home-assistant-sesame-lock-jp) repository, updated to work with modern Home Assistant releases.

The component supports SESAME 3, 4, and 5 devices — all use the same API endpoint (`app.candyhouse.co/api/sesame2/{uuid}`).

## Project Structure

```
custom_components/sesame_jp/
├── __init__.py      # Minimal package marker
├── lock.py          # All integration logic: entity class, API calls, crypto signing
└── manifest.json    # HA integration metadata, dependency declarations
```

All meaningful code lives in `lock.py`. There are no tests, no CI configuration, and no build tooling.

## Development Workflow

There is no standalone development server or test runner. To test changes:

1. Copy `custom_components/sesame_jp/` into a Home Assistant instance's `config/custom_components/` directory.
2. Restart Home Assistant.
3. Check logs under **Settings → System → Logs** for errors.

### Configuration (YAML only — no UI config flow)

This integration uses YAML-based `PLATFORM_SCHEMA` configuration, not a config flow. Add to `configuration.yaml`:

```yaml
lock:
  - platform: sesame_jp
    name: Sesame
    device_id: 'SESAME_UUID'
    api_key: 'SESAME_API_KEY'
    client_secret: 'SESAME_SECRET'
    status_refresh_rate: 1200  # optional, seconds between API polls
```

## Architecture and Key Conventions

### Authentication

Commands require an AES-CMAC signature computed from the current Unix timestamp (little-endian, bytes 2–4 of the 4-byte representation) signed with the device's 16-byte `secret_key`. See `_sesame_command()` in `lock.py`. The status GET endpoint only requires the `x-api-key` header.

### API Commands

| Action | `cmd` value |
|--------|-------------|
| LOCK   | `82`        |
| UNLOCK | `83`        |

Commands POST to `https://app.candyhouse.co/api/sesame2/{uuid}/cmd`. Status polls GET from `https://app.candyhouse.co/api/sesame2/{uuid}`.

### Status Polling

`async_update()` enforces client-side rate limiting: it only calls the API if `status_refresh_rate` seconds have elapsed since `_last_update`. Default is 1200 seconds (20 minutes). This avoids hammering the CANDY HOUSE cloud API.

### Lock State Management

After issuing a command, the integration optimistically updates local state (`_is_locked`, `_is_locking`, `_is_unlocking`) without waiting for a confirming poll. The `available` property reflects whether the last status poll succeeded (`_responsive`).

### Unique ID

The entity's `unique_id` is `sesame_jp_{uuid}`, enabling UI-based area assignment and renaming.

### Dependency

`pycryptodome>=3.18.0` is declared in `manifest.json` and installed automatically by Home Assistant. It provides `Crypto.Cipher.AES` and `Crypto.Hash.CMAC` used for signing.

## Manifest Versioning

When incrementing functionality, update `"version"` in `manifest.json`. Home Assistant requires this field (since 2021.6) or the integration will be blocked from loading.
