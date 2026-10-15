# Feature Request: State-Driven Songs and Dynamic Song Loading

> **Status:** Requested  
> **Dependent project:** game-walker (emulator host with SongWalker integration)  
> **Not part of:** songwalker-core implementation roadmap (tracked separately)

---

## Summary

Enable songs to:

1. **Load other songs** — A parent song can load and play sub-songs (e.g. `loadSong("level1_bgm")`).
2. **React to external state** — Songs respond to state changes (e.g. `when state.song_id == 3 → playSong("boss")`).
3. **Seamlessly switch** — When state changes (level, song_id), transition or swap the active song without manual intervention.

---

## Use Case: Emulator Host Integration

The **game-walker** project runs NES/SNES emulators and monitors ROM memory in real time (level, song_id, etc.). SongWalker will provide alternate BGM while the ROM’s music channels are muted (only SFX remain).

**Responsibility split:** SongWalker handles all audio scripts (`.sw` with music notation + state logic). The emulator handles all other scripts (input, macros, overlays) in a separate language — no music notation needed. The emulator feeds state; SongWalker decides what to play.

SongWalker receives state updates each frame and must:

- Choose the correct song based on `state.song_id` or `state.level`.
- Load and play that song (or a sub-song within a larger structure).
- Switch cleanly when state changes (e.g. entering a boss fight).

Without this feature, SongWalker cannot be driven by game state in real time.

---

## Proposed Capabilities

### 1. Songs as instruments

Songs and audio files **load like instruments** and play back as the current channel instrument.

- **Load from URL/path:** `.sw` loads files from URL or path sources.
- **Assign to track:** Each loaded song is the primary instrument for a track.
- **Note = playback:** A designated note (e.g. C4) = 1× playback. Other notes change playback speed. Optional: preserve frequency when changing speed (common library feature).
- **Two at once:** Load both files, trigger two tracks simultaneously with each song as primary instrument.

```sw
loadSong("world_map", "assets/zelda_overworld.mp3")
loadSong("boss", "assets/zelda_boss.mp3")
// ... when triggers track; track uses song as instrument; note triggers 1× playback
```

**Retro + prerendered:** Both NES/GB synth and prerendered MP3/WAV are interchangeable instrument types within `.sw`.

### 2. State reactivity — `when` keyword

- **60 Hz updates** — State passed each frame; songs may wait arbitrarily before executing a note.
- **`when`** — Deterministic callback; triggers a track on state *change* only. Avoids retriggering an already playing song. Forces only one song at once.
- **Combine with `switch` / `if`** — Use as needed for complex behavior (override, branching).

### 3. Seamless switching

- When the active song changes, support:
  - Immediate swap
  - Brief crossfade
  - Configurable transition behavior

### 4. Complex conditional logic (override / ignore)

Users need to **override** game requests — e.g. "don't play battle songs, keep world map."

**Example scenario:**

- World map song is playing.
- Game triggers battle song (or battle victory).
- User preference: skip battle songs.
- **Result:** Continue playing world map; ignore battle/battle-victory triggers.

**Required:** Nested `if`/`when`, `and`/`or`, negation, "keep current" / "continue" — so logic can say "when battle → don't switch."

### 5. No restart on same song

Rely on `when` — it triggers on state *change* only. If `song_id` is unchanged, `when` doesn't fire; track keeps playing. No explicit "don't restart" logic needed.

### 6. Real-time rendering + state ahead of buffer

- **Real-time render** — Engine produces audio in fixed-size blocks (process_block style). Host calls render with state before each buffer.
- **State ahead of buffer** — Variables are set/updated *before* the buffer is rendered. Engine evaluates conditionals using that state to decide what plays in the upcoming buffer.

---

## Design Decisions (Resolved)

| Topic | Decision |
|-------|----------|
| Load model | Songs load like instruments; note triggers playback (1× or speed variant). |
| Timing | 60 Hz state updates. |
| `when` | Triggers on change; combine with switch/if. Avoids retrigger. |
| Config | Default `~/clevertree/emulation/`. |
| Mapping | All in `.sw` song code. |
| Base note | Plays sample at native frequency; mixes into final stream. |
| Retro + prerendered | Both needed; interchangeable. |

## Open Questions

1. **Decoder choice** — Which MP3 decoder supports frame-by-frame decode in Rust/WASM?
2. **Preserve-frequency API** — Exact syntax for "preserve frequency when changing speed" as common library feature.

---

## References

- [game-walker PLAN.md](../../game-walker/PLAN.md) — Overall architecture and SongWalker integration
- [retro_synth_plan.md](../retro_synth_plan.md) — NES/GB oscillator presets (complementary)
