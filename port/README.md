# ESP32 port

The glue that runs gwenesis on the esp-cpp emulators: `genesis.cpp` drives a
frame (68000 + VDP on core 0, Z80 + sound on core 1, display-aware frame
skip, pacing, save states), `genesis_shared_memory.cpp` places the core's
buffers. It is shared by:

- [esp-box-emu](https://github.com/esp-cpp/esp-box-emu) (ESP32-S3)
- [tab5-emu](https://github.com/esp-cpp/tab5-emu) (ESP32-P4)

Each project builds `src/` and `port/` as its `genesis` component (that
project keeps the CMakeLists.txt and linker fragment) and supplies
`box-emu.hpp`, a `BoxEmu` singleton with the video, audio and input calls
used here (`push_frame`, `frame_slot_free`, `frame_buffer0/1`, `play_audio`,
`gamepad_state`, ...), plus the `shared_memory` and `statistics` components.

## Compile-time options

Where the two chips need different trade-offs the default follows the target
and a define overrides it:

| Define | Default | Effect |
|---|---|---|
| `GENESIS_HOT_IN_PSRAM` | 1 on the P4, else 0 | All hot buffers in PSRAM instead of internal RAM first |
| `GENESIS_DUAL_CORE` | 1 | Z80 + sound on core 1 |
| `GENESIS_SKIP_UNDISPLAYABLE_FRAMES` | 1 | Skip the render of frames the display could not take, and (bounded) while behind real time |
| `GENESIS_MAX_CONSECUTIVE_SKIPS` | 2 | Longest run of skipped renders |
| `GENESIS_PERF` | 1 | `[genesis perf]` per-frame time breakdown every 300 frames |
