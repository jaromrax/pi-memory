# Fork Changes

This file tracks changes made in the local `jaromrax/pi-memory` fork relative to the fork state before these changes.

## 2026-08-21

### Configurable automatic context-injection limits

Added environment-variable overrides for the character limits used by automatic memory-context injection:

```bash
export PI_MEMORY_LONG_TERM_MAX_CHARS=8000
export PI_MEMORY_SCRATCHPAD_MAX_CHARS=2000
export PI_MEMORY_DAILY_MAX_CHARS=1000
export PI_MEMORY_MAX_CHARS=12000
```

Variables and defaults:

| Variable | Default | Applies to |
|---|---:|---|
| `PI_MEMORY_LONG_TERM_MAX_CHARS` | `4000` | The injected `MEMORY.md` section |
| `PI_MEMORY_SCRATCHPAD_MAX_CHARS` | `2000` | Injected open scratchpad items |
| `PI_MEMORY_DAILY_MAX_CHARS` | `3000` | Each injected daily-log section—today and yesterday |
| `PI_MEMORY_MAX_CHARS` | `16000` | The overall rendered memory context |

Implementation details:

- Limits are read at context-build time, so they can be configured through the environment before starting or reloading Pi.
- Invalid, fractional, zero, negative, or non-finite values fall back to the corresponding default.
- Existing line limits and truncation modes are preserved.
- The settings affect automatic injection only; memory files on disk are not modified.
- Explicit `memory_read` and `memory_search` behavior is unchanged.
- `memory_status` reports the active effective values.

### Exit-summary success marker

Added a diagnostic marker for successful exit-summary persistence:

```text
$PI_MEMORY_DIR/.exit_memory_write_succeeded
```

Behavior:

- Created only after a non-empty exit summary has been successfully appended to the current daily log.
- Contains a human-readable timestamp.
- Its filesystem modification time records the success time.
- Removed at the next `session_start`, so a stale marker does not represent the current session.
- Marker creation failure does not affect the already-successful memory write.
- The marker is not Markdown and therefore does not become injected memory or qmd content.
- `memory_status` reports the marker timestamp.

Useful checks:

```bash
cat "$PI_MEMORY_DIR/.exit_memory_write_succeeded"
stat "$PI_MEMORY_DIR/.exit_memory_write_succeeded"
```

When `PI_MEMORY_DIR` is unset, use the default directory:

```text
~/.pi/agent/memory/.exit_memory_write_succeeded
```

### Tests and documentation

- Updated `README.md` with the new environment variables and marker behavior.
- Added unit coverage for environment parsing, per-section limits, overall limits, and marker timestamp handling.
- TypeScript build passes:

  ```text
  npm run build
  ```

- Biome checks pass:

  ```text
  npm run lint
  ```

- Unit tests pass under Bun 1.4.0:

  ```text
  187 pass
  0 fail
  ```

### Changed files

- `index.ts`
- `README.md`
- `test/unit.test.ts`
- `README_fork_changes.md`

## Maintenance notes

These changes currently exist in the local clone at:

```text
/home/ojr/.pi/agent/git/github.com/jaromrax/pi-memory
```

Commit and push the changes to the GitHub fork if they should survive a future Pi package reinstall or update. Keep using npm for dependency installation in this project; Bun is used by the existing test script and does not replace npm.
