# Examined Projects

This file lists every open-source project analysed against the problem pattern
repository. It prevents redundant analysis across sessions.

## Guidelines

- Add new projects at the end of the relevant category.
- Format: `name` → `org/repo` → brief description of architectural relevance.
- When re-entering a session, check this file first. If a project is listed,
  its mapping already exists in INDEX.md (or a prior session draft) and does
  not need re-analysis unless the user specifically asks for re-examination.

---

## CLI / Terminal Tools

| # | Project | Repository | Notes |
|---|---------|-----------|-------|
| 1 | **fd** | sharkdp/fd | File search: parallel walk, regex/glob filter, exec, format |
| 2 | **bat** | sharkdp/bat | `cat` clone: syntax highlighting, git diff, paging, decor |
| 3 | **tokei** | XAMPPRocky/tokei | Code counter: lang detection, state-machine counting, embedding |
| 4 | **dust** | bootandy/dust | Disk usage: tree walk, proportional bars, filter, progress |
| 5 | **bottom** | ClementTsang/bottom | System monitor TUI: time-series widgets, process table, metric polling |

---

## Snippet / Utility

| # | Project | Repository | Notes |
|---|---------|-----------|-------|
| 6 | **pet** | knqyf263/pet | CLI snippet manager: CRUD, TUI dialog, sync, exec, search |

---

## Web Frameworks

| # | Project | Repository | Notes |
|---|---------|-----------|-------|
| 7 | **tide** | http-rs/tide | Rust async web: routing, middleware, listener, typed extraction |
| 8 | **express** | expressjs/express | Node.js web: middleware stack, request/response extension, routing |
| 9 | **htmx** | bigskysoftware/htmx | Hypermedia HTML: declarative AJAX attributes, swap strategies |

---

## GUI (non-web)

| # | Project | Repository | Notes |
|---|---------|-----------|-------|
| 10 | **ratatui** | ratatui-org/ratatui | TUI framework: widgets, layout, backend abstraction, buffer |
| 11 | **egui** | emilk/egui | Immediate-mode GUI (Rust): frame context, GPU tessellation, input |
| 12 | **rnote** | flxzt/rnote | Vector drawing (Rust+GTK4): stroke engine, camera, file format |
| 13 | **MonitorControl** | MonitorControl/MonitorControl | macOS display control: DDC protocol, media keys, OSD |

---

## Web UI Libraries / Frameworks

| # | Project | Repository | Notes |
|---|---------|-----------|-------|
| 14 | **React** | facebook/react | Declarative UI: fiber reconciler, scheduler, server components |
| 15 | **Vue** | vuejs/core | Template-compiled: SFC compiler, reactivity graph, SSR |
| 16 | **Svelte** | sveltejs/svelte | Compile-first: runes, codegen, transitions, motion |

---

## Games

| # | Project | Repository | Notes |
|---|---------|-----------|-------|
| 17 | **Mindustry** | Anuken/Mindustry | Tower defense (Java): annotation-processed ECS, RPC, mod system |
| 18 | **doukutsu-rs** | doukutsu-rs/doukutsu-rs | Cave Story reimpl (Rust): scene stack, map system, engine extraction |
| 19 | **OpenTTD** | OpenTTD/OpenTTD | Transport sim (C++): economy model, NewGRF, save compat, AI |
| 20 | **Endless Sky** | endless-sky/endless-sky | Space sim (C++): data-driven everything, conversation state machine |
| 21 | **Veloren** | veloren/veloren | Voxel RPG (Rust): client/server split, ECS core, agent AI |

---

## Low-Level / Systems

| # | Project | Repository | Notes |
|---|---------|-----------|-------|
| 22 | **tock** | tock/tock | Embedded OS (Rust): kernel/arch/board layering, MPU, syscall |
| 23 | **jemalloc** | jemalloc/jemalloc | Memory allocator: arenas, tcache, size classes, extent management |
| 24 | **picolibc** | keith-packard/picolibc | C library for embedded: newlib-derived, TLS errno, minimal stdio |
| 25 | **libfuse** | libfuse/libfuse | Userspace filesystem: kernel→userspace bridge, high/low API |
| 26 | **fio** | axboe/fio | I/O benchmarking: engine abstraction (io_uring, libaio, sync), job |

---

## Embedded / RTOS

| # | Project | Repository | Notes |
|---|---------|-----------|-------|
| 27 | **FreeRTOS-Kernel** | FreeRTOS/FreeRTOS-Kernel | RTOS: task scheduler, queue, timers, portable abstraction |
| 28 | **RIOT** | RIOT-OS/RIOT | IoT OS: modular, 200+ boards, network stack (6LoWPAN, CoAP, LoRaWAN) |
| 29 | **littlefs** | littlefs-project/littlefs | Fail-safe embedded fs: copy-on-write, wear levelling, bounded RAM/ROM |
| 30 | **mbedtls** | Mbed-TLS/mbedtls | TLS/crypto for embedded: PSA Crypto, X.509, TLS 1.3, configurable |
| 31 | **Contiki-NG** | contiki-ng/contiki-ng | IoT OS: IPv6/6LoWPAN, RPL, CoAP, TSCH/6TiSCH |

---

## Graphics / Rendering

| # | Project | Repository | Notes |
|---|---------|-----------|-------|
| 32 | **SDL** | libsdl-org/SDL | Cross-platform multimedia: video/audio/input abstraction, GPU renderer |
| 33 | **HarfBuzz** | harfbuzz/harfbuzz | Text shaping: OpenType/AAT, subsetting, rasterisation, GPU output |
| 34 | **stb** | nothings/stb | Single-file libraries: image I/O, font raster, rect pack, voxel render |
| 35 | **Freetype** | freetype/freetype | Font rendering engine: hinting, anti-alias, glyph caching, GX/AAT/OpenType |
| 36 | **Cairo** | freedesktop/cairo | (not yet fetched — GitLab Anubis blocks, GitHub mirror requires auth)

---

## Editors / IDEs

| # | Project | Repository | Notes |
|---|---------|-----------|-------|
| 37 | **Helix** | helix-editor/helix | Modal editor: rope document, tree-sitter incremental parse, LSP client, multiple selections, undo transaction |
| 38 | **grbl** | grbl/grbl | CNC motion control: look-ahead trajectory planning, acceleration profiles, stepper pulse generation, spindle PWM |
| 39 | **ArduPilot** | ArduPilot/ardupilot | Autopilot: cascaded PID attitude/position control, EKF sensor fusion, failsafe state machines, mode hierarchy |
| 40 | **Lite XL** | lite-xl/lite-xl | Lightweight Lua editor: line-based document model, undo stack, command/compositor/view architecture, plugin system |
