# Feature: Logical playlists (Flashback lobby host vars)

> **Status:** Spec + host JSON runtime shipped in Flashback; `.sw` example for future WASM embed  
> **Related:** [state_driven_songs.md](./state_driven_songs.md)

## Goal

A **song** may be an endless nested playlist of other songs/tracks. Host apps feed realtime variables; the song decides what plays via `when`.

Flashback lobby OST uses this model:

| Host var | Source | Meaning |
|----------|--------|---------|
| `season` | calendar | `christmas` Nov 15–Jan 2, else default |
| `theater` | host | movie / external player active |
| `enabled` | host | user/APK preference |

Default nested song: **`publicDomain`** (PD/CC classical). Christmas overrides by date.

## Flashback v1 (no WASM)

Portable JSON: `flashback-logical-playlist/v1`  
(`fan-edits/hooks/client/lobby/lobby.playlist.json`)

Web/APK interpreters evaluate `when` and play Bunny HTTPS MP3s with fade in/out.

## Future `.sw` shape

See [`lobby.sw`](../../../fan-edits/hooks/client/lobby/lobby.sw) (example). Desired engine support:

- `loadWav` / `loadSong` from HTTPS URLs
- Nested playlist songs (shuffle + loop)
- Host state: `state.season`, `state.theater`, `state.enabled`
- Crossfade on song change (lobby fades already in host)
