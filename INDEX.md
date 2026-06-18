# Pattern Index

| Planet | Pattern | File | Frame Concern |
|--------|---------|------|---------------|
| mercury | display | [patterns/mercury-display.md](patterns/mercury-display.md) | Temporal coupling between source and display |
| venus | filter | [patterns/venus-filter.md](patterns/venus-filter.md) | Predicate ownership |
| earth | sort | [patterns/earth-sort.md](patterns/earth-sort.md) | Comparator consistency |
| mars | persist | [patterns/mars-persist.md](patterns/mars-persist.md) | Durability guarantee boundary |
| jupiter | validate | [patterns/jupiter-validate.md](patterns/jupiter-validate.md) | Rule set completeness |
| saturn | route | [patterns/saturn-route.md](patterns/saturn-route.md) | Route table consistency |
| uranus | authenticate | [patterns/uranus-authenticate.md](patterns/uranus-authenticate.md) | Trust delegation |
| neptune | transform | [patterns/neptune-transform.md](patterns/neptune-transform.md) | Semantic equivalence across heterogeneous representations |
| callisto | filesystem walk | [patterns/callisto-filesystem-walk.md](patterns/callisto-filesystem-walk.md) | Gap between filesystem order and consumer order |
| io | command execution | [patterns/io-exec.md](patterns/io-exec.md) | Subprocess lifecycle vs. consumer expectations |
| ganymede | output formatting | [patterns/ganymede-output-format.md](patterns/ganymede-output-format.md) | Terminal capability detection |
| europa | configuration assembly | [patterns/europa-config-assembly.md](patterns/europa-config-assembly.md) | Merge ordering transparency |
| titan | syntax highlighting | [patterns/titan-syntax-highlight.md](patterns/titan-syntax-highlight.md) | Grammar coverage vs. source variety |
| rhea | syntax detection | [patterns/rhea-syntax-detection.md](patterns/rhea-syntax-detection.md) | Information deficit at decision time |
| iapetus | code counting | [patterns/iapetus-code-counting.md](patterns/iapetus-code-counting.md) | Grammar coverage vs. real-world source variety |
| hyperion | network listener | [patterns/hyperion-network-listener.md](patterns/hyperion-network-listener.md) | Connection establishment vs. request processing gap |
| pan | resource scheduling | [patterns/pan-resource-scheduling.md](patterns/pan-resource-scheduling.md) | Priority assignment vs. actual timeliness |
| atlas | memory allocation | [patterns/atlas-memory-allocation.md](patterns/atlas-memory-allocation.md) | Request size granularity vs. page granularity |
| dione | power management | [patterns/dione-power-management.md](patterns/dione-power-management.md) | Energy consumption vs. energy availability |
| leda | feedback control | [patterns/leda-feedback-control.md](patterns/leda-feedback-control.md) | Measurement noise vs. control stability |
| pandora | simple workpieces | [patterns/pandora-simple-workpieces.md](patterns/pandora-simple-workpieces.md) | Edit operations vs. document state consistency |

### Veloren (veloren/veloren)

A multiplayer voxel RPG written in Rust with an ECS architecture and client/server split.

| Veloren crate / module | Problem Pattern | Rationale |
|------------------------|----------------|-----------|
| `server/` | **hyperion** (network listener) + **mars** (persist) | Game server: world simulation, player persistence, NPC AI, physics |
| `client/` (voxygen/) | **mercury** (display) + **hyperion** (network listener) | Client: 3D rendering (vulkan), input handling, UI, audio |
| `common/` | **neptune** (transform) | Shared types: components, protocol messages, math, world model |
| `common/src/comp/` | **neptune** (transform) | ECS components: position, velocity, health, inventory, stats |
| `common/src/sys/` | **saturn** (route) | ECS systems: physics, combat, AI, animation, interactions |
| `common/src/terrain/` | **mars** (persist) | Voxel terrain: generation, modification, lighting, LOD |
| `common/src/trade/` | **neptune** (transform) | Trading: item economy, buy/sell, market simulation |
| `common/src/agent/` | **saturn** (route) | Agent AI: behaviour trees, pathfinding, combat tactics |
| `assets/` | **europa** (configuration assembly) | Game data: ability definitions, item stats, NPC behaviour, loot tables — in `.ron` files |

## Low-Level / Systems Projects

### tock (tock/tock)

An embedded OS for Cortex-M and RISC-V, written in Rust, with kernel/arch/board layering and MPU-based isolation.

| tock crate / module | Problem Pattern | Rationale |
|---------------------|----------------|-----------|
| `kernel/` | (orchestration) | Core kernel: scheduler, process management, grant system, callback dispatch |
| `kernel/src/sched/` | **saturn** (route) | Scheduler: cooperative round-robin, timer-based preemption, context switching |
| `kernel/src/process/` | **mars** (persist) | Process management: memory layout, stack, app state, syscall context |
| `kernel/src/grant.rs` | **uranus** (authenticate) | Grant system: per-process kernel capability allocation, memory isolation verification |
| `kernel/src/syscall.rs` | **hyperion** (network listener) + **neptune** (transform) | Syscall interface: process→kernel→driver dispatch, argument marshalling |
| `arch/` | **neptune** (transform) | Architecture support: Cortex-M, RISC-V, x86 — CSR/SFR abstractions, MPU, NVIC |
| `arch/cortex-m/src/syscall.rs` | **neptune** (transform) | Context switch: `switch_to_user`, stack save/restore, SWI handling |
| `arch/cortex-m/src/mpu.rs` | **jupiter** (validate) | MPU configuration: region programming, permission enforcement, context switch |
| `arch/riscv/src/pmp.rs` | **jupiter** (validate) | Physical Memory Protection: PMP configuration for RISC-V |
| `chips/` | **neptune** (transform) | Chip support: peripheral register maps, interrupt mapping, clock tree — HAL layer |
| `chips/sam4l/src/` | **neptune** (transform) | Specific chip: GPIO, UART, SPI, I2C, ADC, timer — peripheral driver implementations |
| `boards/` | **europa** (configuration assembly) | Board definitions: pin mappings, peripheral allocation, component wiring |
| `capsules/` | **neptune** (transform) | Drivers in Rust: virtual alarms, I2C, SPI, UART, sensors, radio, storage — in-kernel driver framework |
| `capsules/core/src/virtualizers/` | **saturn** (route) | Virtualiser: muxes a single hardware peripheral across multiple capsules (e.g., virtual alarm, virtual I2C) |
| `libraries/tickv/` | **mars** (persist) | Tock-flash: log-structured key-value store for flash storage |
| `tools/` | **jupiter** (validate) | Build tools: board dependency generation, ELF size analysis, TBF (Tock Binary Format) creation |

### jemalloc (jemalloc/jemalloc)

A general-purpose malloc(3) implementation emphasising fragmentation avoidance and scalable concurrency.

| jemalloc module | Problem Pattern | Rationale |
|-----------------|----------------|-----------|
| `include/jemalloc/internal/arena.h` | **saturn** (route) | Arena: per-CPU allocation region — routes allocation requests by thread to minimise contention |
| `include/jemalloc/internal/tcache.h` | **mars** (persist) | Thread-local cache: hot-slot cache for frequently used size classes, avoids arena locking |
| `include/jemalloc/internal/bin.h` | **saturn** (route) | Bin: per-size-class slab allocator within an arena — routes allocations to the correct free list |
| `include/jemalloc/internal/extent.h` | **neptune** (transform) | Extent: contiguous virtual memory region from OS — manages chunk allocation and deallocation |
| `include/jemalloc/internal/pa.h` | **mars** (persist) | PA (Page Allocator): physical page management, coalescing, dirty page tracking |
| `include/jemalloc/internal/ecache.h` | **mars** (persist) | Extent cache: recently freed extents kept for reuse without returning to OS |
| `include/jemalloc/internal/hpa.h` | **mars** (persist) | HPA (Huge Page Allocator): transparent 2 MiB huge page support for large allocations |
| `include/jemalloc/internal/san.h` | **jupiter** (validate) | Sanitizer support: guard pages, quarantines, UAF detection — optional safety via overhead |
| `include/jemalloc/internal/sz.h` | **neptune** (transform) | Size classes: quantisation of allocation sizes into ~200 buckets (8B…4M), with slab sizing |
| `include/jemalloc/internal/prof.h` | **mercury** (display) | Heap profiling: call-stack backtrace, allocation sampling, `jeprof` dump format |
| `include/jemalloc/internal/ctl.h` | **europa** (configuration assembly) | Runtime control: `MALLOC_CONF` env var parsing, `mallctl` namespace for live tuning |
| `include/jemalloc/internal/stats.h` | **mercury** (display) | Statistics: arena/bin/tcache stats, fragmentation analysis, printed via `malloc_stats_print()` |
| `include/jemalloc/internal/emap.h` | **neptune** (transform) | Extent map: radix tree mapping virtual address → extent metadata — fast lookup for free/alloc tracking |
| `include/jemalloc/internal/rtree.h` | **neptune** (transform) | Radix tree: compressed multi-level radix tree for virtual address to extent metadata lookup |
| `include/jemalloc/internal/sec.h` | **mars** (persist) | SEC (Sanitizer Enabled Cache): bounded cache with quarantine zones for sanitizer builds |
| `include/jemalloc/internal/eset.h` | **mars** (persist) | Extent set: non-overlapping extent data structure for coalescing buddies |

### picolibc (keith-packard/picolibc)

A C standard library for embedded systems, derived from Newlib, targeting small RAM footprints.

| picolibc module | Problem Pattern | Rationale |
|-----------------|----------------|-----------|
| `libc/` | (orchestration) | Full C library: ctype, stdio, stdlib, string, time, math — per-architecture |
| `libc/tinystdio/` | **mercury** (display) | Minimal stdio: printf/scanf without heap allocation, bare-metal file descriptors (getc/putc) |
| `libc/tinystdio/printf` | **ganymede** (output formatting) | Printf: integer + float formatting without malloc — position-based, reentrant |
| `libc/tinystdio/setvbuf.c` | **ganymede** (output formatting) | Buffering: unbuffered, line-buffered, fully-buffered modes for constrained systems |
| `libc/string/` | **neptune** (transform) | String ops: memcpy, strlen, strchr — often implemented in assembly for speed |
| `libc/machine/` | **neptune** (transform) | Machine-specific: setjmp/longjmp (assembly), TLS init — per-architecture overrides |
| `picocrt/` | **saturn** (route) | Startup code: `.init` / `.fini` / `.ctors`, stack pointer init, BSS zeroing — arch-specific |
| `libc/picolib/` | **mars** (persist) | TLS support: `__tls_get_addr`, thread-local errno (4 bytes vs traditional giant struct) |
| `libc/picolib/atexit.c` | **mars** (persist) | `atexit` chain: linked list of cleanup handlers, registered at program start |
| `newlib/libm/` | **neptune** (transform) | Math library: sin, cos, sqrt, etc — pure C or arch-specific optimised versions |
| `semihost/` | **hyperion** (network listener) + **mercury** (display) | Semihosting: debug I/O via JTAG/SWD — ARM and RISC-V breakpoint-based console |
| `scripts/` | **europa** (configuration assembly) | Build scripts: cross-compilation `.txt` files for meson — target-specific flag config |

### libfuse (libfuse/libfuse)

Reference implementation of the FUSE protocol, allowing userspace programs to export filesystems to the Linux kernel.

| libfuse module | Problem Pattern | Rationale |
|----------------|----------------|-----------|
| `lib/fuse_lowlevel.c` | **hyperion** (network listener) | Low-level API: reads kernel requests from `/dev/fuse`, dispatches, sends responses — async event loop |
| `lib/fuse.c` | **neptune** (transform) | High-level API: wraps kernel inode→path translation, maintains a path-based callback contract |
| `lib/fuse_loop.c` | **hyperion** (network listener) | Single-threaded event loop: `fuse_session_loop()` — reads request → calls handler → writes reply |
| `lib/fuse_loop_mt.c` | **hyperion** (network listener) | Multi-threaded event loop: worker thread pool, request queue, concurrency control |
| `lib/fuse_uring.c` | **hyperion** (network listener) | io_uring backend: async I/O via kernel submission/completion queues for zero-copy |
| `lib/fuse_opt.c` | **europa** (configuration assembly) | Option parsing: `fuse_opt_parse()` — mount options, sub-type arguments |
| `lib/fuse_signals.c` | **hyperion** (network listener) | Signal handling: SIGINT/SIGTERM → session exit, daemonisation |
| `lib/fuse_daemonize.c` | **neptune** (transform) | Daemonisation: fork, detach, `setsid()`, redirect stdin/stdout/stderr |
| `lib/mount.c` | **neptune** (transform) | Mount: `mount()` syscall with FUSE-specific options, `/etc/mtab` update |
| `lib/mount_util.c` | **neptune** (transform) | Mount utilities: `fusermount` path resolution, `/dev/fuse` opening |
| `util/fusermount.c` | **uranus** (authenticate) | `fusermount3` (setuid): validates mountpoint (no sticky, ownership check), `allow_other` via config |
| `include/fuse_kernel.h` | **neptune** (transform) | Kernel ABI: struct definitions for FUSE protocol messages (readdir, open, write, etc.) |
| `include/fuse.h` | **neptune** (transform) | High-level API: `fuse_operations` callback struct — 40+ operations mapped to userspace handler |
| `include/fuse_lowlevel.h` | **hyperion** (network listener) | Low-level API: `fuse_lowlevel_ops` — raw kernel message handlers with inode-level interface |
| `lib/buffer.c` | **ganymede** (output formatting) | Buffer management: segment-based buffer chain for FUSE message serialisation/deserialisation |
| `example/` | **mercury** (display) | Examples: passthrough, hello, poll, notify — reference implementations of FUSE filesystems |

### fio (axboe/fio)

Flexible I/O tester: a low-level benchmarking tool that simulates user-specified I/O workloads against storage devices.

| fio module | Problem Pattern | Rationale |
|------------|----------------|-----------|
| `backend.c` | **saturn** (route) | Job backend: rates, I/O flow engine, completion tracking — main I/O dispatch loop |
| `engines/` | **neptune** (transform) | I/O engine abstraction: sync, mmap, libaio, io_uring, nvme, sg, nbd, http, rdma — 30+ backends |
| `engines/io_uring.c` | **hyperion** (network listener) | io_uring engine: kernel submission/completion queue rings, SQ-poll, fixed buffers |
| `engines/sync.c` | **hyperion** (network listener) | Sync engine: `pread`/`pwrite` syscall — simplest, no async |
| `engines/libaio.c` | **hyperion** (network listener) | Linux AIO engine: `io_submit`/`io_getevents` — async with completion event polling |
| `engines/nvme.c` | **hyperion** (network listener) | NVMe engine: `/dev/ng0n1` char device — `ioctl`-based I/O, submission queues |
| `server.c` / `client.c` | **hyperion** (network listener) | fio server/client: TCP-based distributed I/O, receives jobs from client, sends results |
| `stat.c` | **iapetus** (code counting) | Statistics: latency histogram (bucketed), IOPS, bandwidth, percentiles, clat/slat data |
| `eta.c` | **iapetus** (code counting) | ETA calculation: time remaining, bandwidth estimate based on completed/duration ratio |
| `lib/axmap.c` | **mars** (persist) | Axmap: atomic bitmap for block-level tracking — used for verify state and dedup |
| `lib/lfsr.c` | **neptune** (transform) | LFSR: linear-feedback shift register for deterministic pseudo-random I/O offsets |
| `lib/flist.h` | **mars** (persist) | Inline doubly-linked list: embeddable intrusive list with overlap detection |
| `lib/prio_tree.c` | **saturn** (route) | Priority tree: interval-based overlapping I/O tracking for latency-sensitive scheduling |
| `lib/strntol.c` | **neptune** (transform) | Numeric parsing: SI unit suffixes (K, M, G, T, P), percentage, time duration parsing |
| `options.c` | **europa** (configuration assembly) | Job options: parses ~200+ CLI/job file parameters into `thread_options` struct |
| `parse.c` | **europa** (configuration assembly) | Config parser: job file parser, variable substitution, includes, group assignment |
| `verify.c` | **jupiter** (validate) | Data verification: writes known patterns (md5, sha*, crc*) then reads back to validate |
| `os/` | **neptune** (transform) | OS abstraction: POSIX, Solaris, Windows — platform-specific file locking, time, threading |
| `arch/` | **neptune** (transform) | CPU architecture: x86, ARM, RISC-V — memory barriers, NUMA, CPU-specific optimisation |
| `blktrace.c` | **callisto** (filesystem walk) | Blktrace replay: reads `blktrace` binary logs, replays I/O operations |
| `diskutil.c` | **mercury** (display) | Disk utilisation: reads `/sys/block/*/stat`, reports utilization % during run |
| `cgroup.c` | **jupiter** (validate) | Cgroup support: places fio jobs into cgroup v1/v2 for CPU/mem/blkio isolation |
| `gfio.c` | **mercury** (display) | GTK3 UI: graphical frontend for fio job configuration and live result display |

## Embedded / RTOS Projects

### FreeRTOS-Kernel (FreeRTOS/FreeRTOS-Kernel)

The classic real-time operating system kernel for microcontrollers: tasks, queues, timers, and software timers.

| FreeRTOS module | Problem Pattern | Rationale |
|-----------------|----------------|-----------|
| `tasks.c` | **saturn** (route) | Task scheduler: priority-based preemption, round-robin within priority, idle task, tickless idle |
| `queue.c` | **mars** (persist) | Queue: inter-task message passing, mutex, semaphore — blocking get with timeout |
| `timers.c` | **mars** (persist) | Software timers: daemon task, timer callback queue, auto-reload/one-shot |
| `event_groups.c` | **venus** (filter) | Event groups: bitmask-based event waiting, AND/OR wake conditions |
| `stream_buffer.c` | **mars** (persist) | Stream buffer: byte-oriented data transfer with send/receive semantics |
| `list.c` | **mars** (persist) | Linked list: intrusive doubly-linked list for ready/blocked/suspended task tracking |
| `croutine.c` | **saturn** (route) | Co-routines: simplified cooperative scheduling for extremely memory-constrained systems |
| `portable/` | **neptune** (transform) | Port layer: per-architecture context switch, interrupt entry/exit, stack frame — 40+ ports |
| `portable/MemMang/` | **mars** (persist) | Memory allocation: 5 heap implementations (static, simple, coalescing, multi-pool, custom) |
| `include/FreeRTOS.h` | **europa** (configuration assembly) | Configuration: `FreeRTOSConfig.h` — scheduler, heap size, tick rate, stack overflow detection |
| `include/portable.h` | **neptune** (transform) | Portable API: `portENTER_CRITICAL`, `portYIELD`, `portNOP` — architecture-independent macros |
| `include/mpu_wrappers.h` | **uranus** (authenticate) | MPU wrappers: privileged/unprivileged mode separation, syscall dispatch |

### RIOT (RIOT-OS/RIOT)

An open-source IoT operating system with modular architecture, supporting 200+ boards and multiple network stacks.

| RIOT module | Problem Pattern | Rationale |
|-------------|----------------|-----------|
| `core/` | **saturn** (route) | Kernel: preemptive tickless scheduler, message passing, priority queues, thread management |
| `sys/` | **neptune** (transform) | System services: shell, auto-init, logging, flash layout, benchmark |
| `sys/net/` | **saturn** (route) + **hyperion** (network listener) | Network stack: GNRC IPv6/6LoWPAN, UDP/TCP, RPL, CoAP, LoRaWAN, BLE |
| `sys/net/gnrc/` | **saturn** (route) | GNRC: modular network stack — netif, transport, application layers with pthread-like API |
| `sys/auto_init/` | **europa** (configuration assembly) | Auto-initialisation: board-specific peripheral init based on module whitelist |
| `sys/phydat/` | **neptune** (transform) | Physical data representation: sensor unit conversion (temperature, humidity, pressure) |
| `boards/` | **europa** (configuration assembly) | Board definitions: pin mux, clock config, peripheral allocation per physical board |
| `boards/common/` | **europa** (configuration assembly) | Common board logic: shared peripheral mappings across board families |
| `cpu/` | **neptune** (transform) | CPU support: Cortex-M, ESP32, RISC-V, AVR, MSP430 — register maps, interrupt vectors |
| `drivers/` | **neptune** (transform) | Device drivers: sensors (BME280, MPU9X50), actuators, displays, radios — unified API |
| `sys/shell/` | **mercury** (display) | Shell: interactive command interpreter for debugging and configuration |
| `sys/test_utils/` | **jupiter** (validate) | Test utilities: network integration tests, hardware test suites |
| `pkg/` | **europa** (configuration assembly) | Package manager: external library integration (OpenThread, lwIP, MicroPython) |
| `dist/` | **europa** (configuration assembly) | Build tools: `riotdocker`, test runner, flasher scripts, CI configuration templates |
| `kconfig/` | **europa** (configuration assembly) | Kconfig: dependency resolution, feature toggles, module configuration |

### littlefs (littlefs-project/littlefs)

A fail-safe filesystem for microcontrollers, designed for power-loss resilience with bounded RAM/ROM.

| littlefs module | Problem Pattern | Rationale |
|-----------------|----------------|-----------|
| `lfs.c` | **mars** (persist) | Core filesystem: mount, format, file ops, dir ops — copy-on-write with metadata logging |
| `lfs.h` | **neptune** (transform) | API and config: `lfs_config` struct — block device callbacks, cache sizes, read/prog/erase sizes |
| `lfs_util.c` | **neptune** (transform) | Utilities: CRC computation, byte swapping, arithmetic for block addressing |
| `bd/lfs_emubd.c` | **jupiter** (validate) | Emulated block device: file-backed block storage for testing on PC |
| `bd/lfs_rambd.c` | **jupiter** (validate) | RAM block device: in-memory block storage for fast testing |
| `bd/lfs_filebd.c` | **jupiter** (validate) | File block device: real file-backed storage for testing |
| `lfs.c` — metadata pairs | **mars** (persist) | Metadata pair: atomic directory/file update via log-structured append + compact |
| `lfs.c` — CTZ skip-lists | **saturn** (route) | CTZ skip-list: COW data structure for file blocks — reverse-traversal offset chain |
| `lfs.c` — block allocator | **saturn** (route) | Block allocator: lookahead bitmap, dynamic wear levelling (max erase cycles per block) |
| `lfs.c` — directory ops | **mars** (persist) | Directory: file listing, move, remove — requires parent/child pair update |
| `DESIGN.md` | **mercury** (display) | Design documentation: explains metadata pairs, CTZ skip-lists, wear levelling |
| `SPEC.md` | **jupiter** (validate) | On-disk specification: byte-level format, magic numbers, versioning |
| `tests/` | **jupiter** (validate) | Test suite: power-loss simulation, bad block injection, boundary conditions |

### mbedtls (Mbed-TLS/mbedtls)

A C library implementing TLS/DTLS, X.509 certificate manipulation, and PSA Cryptography for embedded systems.

| mbedtls module | Problem Pattern | Rationale |
|----------------|----------------|-----------|
| `library/ssl_tls.c` | **hyperion** (network listener) | TLS/DTLS protocol: handshake state machine, record protocol, alert messages |
| `library/ssl_tls13_client.c` | **hyperion** (network listener) | TLS 1.3 client: early data, PSK, HRR, certificate verification |
| `library/ssl_tls13_server.c` | **hyperion** (network listener) | TLS 1.3 server: session tickets, key sharing, certificate request |
| `library/ssl_ciphersuites.c` | **saturn** (route) | Ciphersuite negotiation: routes negotiated ciphersuite to correct MAC/encrypt/decript |
| `library/ssl_msg.c` | **neptune** (transform) | Record layer: fragment/reassemble, encrypt/decrypt, handshake message marshalling |
| `library/x509_crt.c` | **jupiter** (validate) | X.509 certificate: DER/PEM parsing, chain validation, CRL checking |
| `library/x509_csr.c` | **neptune** (transform) | Certificate signing request: CSR generation, subject/extension construction |
| `include/mbedtls/ssl.h` | **neptune** (transform) | SSL API: connection setup, handshake callbacks, session resumption, SNI |
| `tf-psa-crypto/` | **mars** (persist) | PSA Crypto: cryptographic key storage, algorithm implementations (AES, ECC, RSA, SHA) |
| `include/mbedtls/mbedtls_config.h` | **europa** (configuration assembly) | Configuration: compile-time feature toggles (~300 defines), algorithm selection |
| `library/debug.c` | **mercury** (display) | Debug logging: protocol dump, error string, threshold levels |
| `library/net_sockets.c` | **hyperion** (network listener) | Network abstraction: TCP socket connect/accept/recv/send with timeout + non-blocking |
| `library/timing.c` | **mars** (persist) | Timing: delay, timer, alert delay for DTLS handshake protection |
| `library/ssl_cache.c` | **mars** (persist) | Session cache: server-side session ID lookup, ticket decryption, TTL management |
| `library/ssl_cookie.c` | **uranus** (authenticate) | DTLS cookie: stateless hello verify mechanism for DoS protection |
| `library/ssl_ticket.c` | **uranus** (authenticate) | Session ticket: encryption key rotation, ticket generation/validation |
| `programs/` | **jupiter** (validate) | Example programs: SSL client/server, cert generation, key management |

### Contiki-NG (contiki-ng/contiki-ng)

An operating system for next-generation IoT devices, focused on dependable low-power communication and standard protocols.

| Contiki-NG module | Problem Pattern | Rationale |
|-------------------|----------------|-----------|
| `os/` | **saturn** (route) | Core OS: protothreads, event-driven kernel, clock/timer, process management |
| `os/net/` | **saturn** (route) + **hyperion** (network listener) | Networking: IPv6/6LoWPAN, RPL routing, TSCH MAC, CoAP, MQTT |
| `os/net/mac/` | **saturn** (route) | MAC layer: CSMA, TSCH (6TiSCH), ContikiMAC — duty cycling for energy efficiency |
| `os/net/routing/` | **saturn** (route) | Routing: RPL (storing/non-storing), mesh-under, route discovery |
| `os/services/` | **mercury** (display) | Services: shell, web server, CoAP resource directory, OTA updates |
| `arch/cpu/` | **neptune** (transform) | CPU support: ARM Cortex-M, CC2538, nRF52, ESP32 — register maps, clock config |
| `arch/platform/` | **europa** (configuration assembly) | Platform: board-specific pin mux, sensor wiring, radio configuration |
| `os/net/app-layer/` | **neptune** (transform) | Application protocols: CoAP, MQTT-SN, HTTP, SNMP — protocol implementation |
| `os/storage/` | **mars** (persist) | Storage: CFS (Coffee File System), EEPROM, ANTOF — flash-backed persistent storage |
| `os/sys/` | **mercury** (display) | System: logging (`log-conf`), debug UART, button-sensor framework |
| `regression-tests/` | **jupiter** (validate) | Regression tests: network stack tests, radio duty cycling verification |

## Graphics / Rendering Projects

### SDL (libsdl-org/SDL)

Simple DirectMedia Layer — cross-platform library for video, audio, input, and GPU rendering.

| SDL module | Problem Pattern | Rationale |
|------------|----------------|-----------|
| `src/video/` | **mercury** (display) | Video subsystem: window management, display modes, fullscreen, vsync — platform abstraction |
| `src/render/` | **ganymede** (output formatting) | 2D renderer: software, OpenGL, Vulkan, Direct3D, Metal — texture blitting with batching |
| `src/gpu/` | **ganymede** (output formatting) | GPU API: cross-platform direct GPU access, shader management, command buffer |
| `src/audio/` | **mercury** (display) | Audio playback: device enumeration, sample conversion, streaming, capture |
| `src/events/` | **hyperion** (network listener) | Event system: keyboard, mouse, joystick, touch, window events — unified polling, filter, push |
| `src/joystick/` | **hyperion** (network listener) | Joystick/gamepad: device detection, axis mapping, haptic feedback, SDL GameControllerDB |
| `src/sensor/` | **hyperion** (network listener) | Sensors: accelerometer, gyroscope — mobile/embedded sensor fusion |
| `src/timer/` | **mars** (persist) | Timer: high-resolution delay, periodic and one-shot timers |
| `src/filesystem/` | **callisto** (filesystem walk) | Filesystem: base path, preferences path, directory enumeration |
| `src/thread/` | **io** (command execution) | Threading: platform abstraction for threads, mutex, semaphore, condition variable |
| `src/atomic/` | **mars** (persist) | Atomic ops: platform-specific atomic increments, CAS, memory barriers |
| `src/power/` | **jupiter** (validate) | Power: battery state, charge level, AC status |
| `src/camera/` | **hyperion** (network listener) | Camera: device enumeration, frame capture, format conversion |
| `src/dialog/` | **mercury** (display) | File dialogs: native open/save file dialog across platforms |
| `src/haptic/` | **hyperion** (network listener) | Haptic: force feedback, rumble, periodic effects |
| `src/hidapi/` | **hyperion** (network listener) | HID: raw Human Interface Device access, custom device communication |
| `src/hints/` | **europa** (configuration assembly) | Hints: runtime configuration key-value store (`SDL_HINT_*`) |
| `src/io/` | **neptune** (transform) | I/O abstraction: stream, RWops — unified read/write over files, memory, network |
| `src/loadso/` | **io** (command execution) | Shared library loading: `dlopen`/`LoadLibrary` abstraction |
| `src/libm/` | **neptune** (transform) | Math library: portable sin/cos/sqrt for cross-platform consistency |
| `src/main/` | **saturn** (route) | Main entry: platform-specific `main` trampolines (WinMain, SDL_main) |

### HarfBuzz (harfbuzz/harfbuzz)

A text shaping engine and font platform supporting OpenType and Apple Advanced Typography.

| HarfBuzz module | Problem Pattern | Rationale |
|-----------------|----------------|-----------|
| `src/hb-shape.cc` | **neptune** (transform) | Shaping: Unicode text → positioned glyph sequence — the core transformation |
| `src/hb-ot-layout.cc` | **saturn** (route) | OpenType Layout: GSUB (glyph substitution) and GPOS (glyph positioning) lookup routing |
| `src/hb-ot-shape.cc` | **saturn** (route) | Shaper selection: routing text per script (Arabic, Indic, Thai, etc.) to appropriate shaper |
| `src/hb-ot-shape-complex-*.cc` | **saturn** (route) | Complex script shapers: Indic, Khmer, Myanmar, Tibetan — language-specific shaping logic |
| `src/hb-aat-layout.cc` | **saturn** (route) | AAT layout: Apple Advanced Typography — morx (glyph metamorphosis), kerx (kerning) |
| `src/hb-font.cc` | **mars** (persist) | Font objects: face, font, glyph metrics — cached from underlying font engine |
| `src/hb-ft.cc` | **neptune** (transform) | FreeType integration: bridges HarfBuzz API to FreeType font backend |
| `src/hb-subset.cc` | **mars** (persist) | Font subsetting: removes unwanted glyphs/tables from OpenType fonts |
| `src/hb-draw.cc` | **mercury** (display) | Draw API: glyph outline extraction via callback mechanism |
| `src/hb-paint.cc` | **mercury** (display) | Paint API: COLRv1 colour font rendering with gradient, transform, composite |
| `src/hb-raster.cc` | **ganymede** (output formatting) | Rasterisation: glyph → bitmap with hinting, anti-aliasing |
| `src/hb-vector.cc` | **ganymede** (output formatting) | Vector output: glyph → SVG path data |
| `src/hb-gpu.cc` | **ganymede** (output formatting) | GPU text: Slug algorithm → GLSL/WGSL/HLSL shaders for direct GPU glyph rendering |
| `src/OT/Layout/` | **neptune** (transform) | OpenType table parsing: binary SVC → typed C++ objects for each layout table |
| `src/OT/Color/` | **mercury** (display) | COLR/CPAL/CBDT/sbix: colour font table parsing and palette extraction |
| `src/graph/` | **saturn** (route) | Subset repacker: optimises OpenType table layout for subsetting constraints |

### stb (nothings/stb)

A collection of single-file public-domain libraries for C/C++, covering image I/O, font rasterisation, compression, and more.

| stb library | Problem Pattern | Rationale |
|-------------|----------------|-----------|
| `stb_image.h` | **neptune** (transform) | Image decoder: PNG, JPEG, BMP, GIF, PSD, TGA, HDR — format detection + decoding |
| `stb_image_write.h` | **ganymede** (output formatting) | Image encoder: PNG, TGA, BMP, JPEG — pixel buffer → encoded file |
| `stb_image_resize2.h` | **neptune** (transform) | Image resize: nearest-neighbour, bilinear, bicubic, box, lanczos, mitchell |
| `stb_truetype.h` | **mercury** (display) | TrueType rasteriser: glyph → bitmap with subpixel, hinting, kerning |
| `stb_vorbis.c` | **neptune** (transform) | Ogg Vorbis: stream → PCM — emulated fixed-point for integer platforms |
| `stb_rect_pack.h` | **saturn** (route) | Rectangle packing: bin-packing for texture atlas / sprite sheet |
| `stb_voxel_render.h` | **mercury** (display) | Voxel rendering: chunked meshing, ambient occlusion, mesh generation |
| `stb_dxt.h` | **neptune** (transform) | DXT/S3TC compression: BC1–BC3 real-time encoding |
| `stb_textedit.h` | **mercury** (display) | Text edit: core input handling, cursor movement, selection, clipboard |
| `stb_perlin.h` | **neptune** (transform) | Simplex noise: Perlin noise with configurable seeds and octaves |
| `stb_ds.h` | **mars** (persist) | Data structures: dynamic array, hash map, ordered set — typesafe macros |
| `stb_sprintf.h` | **ganymede** (output formatting) | Fast sprintf: Chebychev-based float formatting, buffer overflow safety |
| `stb_hexwave.h` | **mercury** (display) | Audio synthesis: waveform generation, envelope, effects |
| `stb_easy_font.h` | **mercury** (display) | Bitmap font: simple fixed-width font for frame rate / debug overlay |
| `stb_tilemap_editor.h` | **mercury** (display) | Tilemap editor: full embedded editor with undo, layers, animated tiles |

### Freetype (freetype/freetype)

A font rendering engine that supports TrueType, OpenType, Type1, and many other font formats with hinting and anti-aliasing.

| Freetype module | Problem Pattern | Rationale |
|-----------------|----------------|-----------|
| `src/base/` | **mercury** (display) | Base layer: glyph slot management, rasterisation dispatch, transformation, FT API core |
| `src/truetype/` | **neptune** (transform) | TrueType: glyph outline decoding, hinting bytecode interpreter, instruction sets |
| `src/sfnt/` | **neptune** (transform) | SFNT: table parsing (cmap, head, hhea, hmtx, maxp, name, os2) |
| `src/cff/` | **neptune** (transform) | CFF: PostScript Type 2 charstring interpreter, hint replacement |
| `src/type1/`, `src/type42/` | **neptune** (transform) | Type1/42: Adobe font format parsing and rendering |
| `src/raster/` | **ganymede** (output formatting) | Anti-aliased rasteriser: `ftraster.c` — scanline conversion with coverage |
| `src/smooth/` | **ganymede** (output formatting) | Smooth renderer: 256-level anti-aliasing, subpixel rendering (ClearType-compatible) |
| `src/autofit/` | **saturn** (route) | Autofitter: script-specific hinting, stem width detection, glyph-to-bluezone assignment |
| `src/cache/` | **mars** (persist) | Glyph cache: LRU cache for rendered glyphs, image caching, manager |
| `src/gxvalid/` | **jupiter** (validate) | TrueType GX validation: AAT tables sanity check |
| `src/otvalid/` | **jupiter** (validate) | OpenType validation: GDEF/GPOS/GSUB table conformance checking |
| `src/pfr/` | **neptune** (transform) | PFR: Printer Font Binary format — embedded font support |
| `src/winfonts/` | **neptune** (transform) | Windows FON/FNT: legacy bitmap font format handling |
| `include/freetype/config/` | **europa** (configuration assembly) | Build configuration: `FT_CONFIG_OPTION_*` — feature enabling, module inclusion |
| `src/bzip2/`, `src/gzip/`, `src/lzw/` | **neptune** (transform) | Stream compression: transparent decompression of font data streams |

### Helix (helix-editor/helix)

A modal code editor in Rust with tree-sitter, LSP, multiple selections, and a transaction-based undo system.

| Helix crate / module | Problem Pattern | Rationale |
|----------------------|----------------|-----------|
| `helix-core/src/transaction.rs` | **mars** (persist) | Transaction: groups primitive edits (insert, delete, replace) into undoable units with cursor preservation and coalescing |
| `helix-core/src/syntax.rs` | **titan** (syntax highlighting) + **neptune** (transform) | Tree-sitter integration: incremental parse, query captures → highlighted ranges, injection languages |
| `helix-core/src/selection.rs` | **saturn** (route) | Multiple selections: range set with primary cursor, operations dispatched across all cursors (associative edit) |
| `helix-core/src/movement.rs` | **saturn** (route) | Cursor movement: word, line, paragraph, bracket, matching pair — routes position deltas through the document model |
| `helix-core/src/search.rs` | **venus** (filter) | Search/replace: regex and literal search with selection filtering, incremental search state |
| `helix-core/src/history.rs` | **mars** (persist) | Undo/redo stack: branching undo tree, merge policy for adjacent edits, cursor memory per revision |
| `helix-core/src/indent.rs` | **neptune** (transform) | Indentation: tree-sitter query-driven indentation, mode detection (tabs vs spaces) |
| `helix-core/src/diff.rs` | **venus** (filter) + **mercury** (display) | Diff hunk: computes line-granular diff between buffer and disk, gutter markers |
| `helix-core/src/snippets.rs` | **mars** (persist) | Snippet engine: template placeholders, tab stops, mirrors, variable interpolation — edit-time state machine |
| `helix-core/src/doc_formatter/` | **neptune** (transform) | Document formatting: external formatter (rustfmt, prettier) integration, range format |
| `helix-core/src/rope_reader.rs` | **neptune** (transform) | Rope reader: gap buffer / rope abstraction over the document for efficient slice operations |
| `helix-core/src/text_annotations.rs` | **mercury** (display) | Text annotations: inline diagnostics, virtual text from LSP, gutter decorations |
| `helix-core/src/auto_pairs.rs` | **neptune** (transform) | Auto-pair insertion: bracket/quote completion with selection-wrapping, smart deletion |
| `helix-view/src/editor.rs` | **saturn** (route) | Editor state: document set, views, focus management, mode dispatch (normal/insert/select) |
| `helix-view/src/view.rs` | **mercury** (display) | View: scroll offset, anchor, viewport calculation, visual line mapping |
| `helix-view/src/document.rs` | **mars** (persist) + **neptune** (transform) | Document: path, language, encoding, history, LSP snapshot — the central workpiece |
| `helix-view/src/theme.rs` | **europa** (configuration assembly) | Theme: colour palette merging, scope-to-colour mapping, base16 inheritance |
| `helix-view/src/annotations.rs` | **mercury** (display) | Annotation rendering: diagnostic icons, gutter markers, inlay hints |
| `helix-lsp/src/transport.rs` | **hyperion** (network listener) | LSP transport: JSON-RPC over stdin/stdout, message framing, request IDs |
| `helix-lsp/src/client.rs` | **hyperion** (network listener) | LSP client: capability negotiation, server lifecycle, request queue, notification dispatch |
| `helix-lsp/src/jsonrpc.rs` | **neptune** (transform) | JSON-RPC protocol: request/notification/response serialisation, method dispatch |
| `helix-lsp/src/file_operations.rs` | **callisto** (filesystem walk) | LSP file operations: file rename/create/delete via workspace/didChangeWatchedFiles |
| `helix-term/src/` | **mercury** (display) + **hyperion** (network listener) | Terminal UI: compositor, prompt, picker, status line, key bindings, input dispatch |
| `helix-tui/src/` | **mercury** (display) + **ganymede** (output formatting) | TUI framework: compositor, widget rendering, surface management |
| `helix-loader/src/` | **callisto** (filesystem walk) + **europa** (configuration assembly) | Runtime loading: language config, theme files, tree-sitter queries, text objects |
| `helix-vcs/src/` | **venus** (filter) | VCS integration: git status, blame info, diff gutter — file-level status filtering |
| `helix-dap/src/` | **hyperion** (network listener) + **mars** (persist) | Debug Adapter Protocol: breakpoint management, thread state, variable inspection |

### Lite XL (lite-xl/lite-xl)

A lightweight text editor written in Lua with a C core, featuring a line-based document model, undo stack, and view/compositor architecture.

| Lite XL module | Problem Pattern | Rationale |
|----------------|----------------|-----------|
| `data/core/doc/init.lua` | **mars** (persist) | Document model: lines stored as string table, undo/redo stack with change IDs, dirty flag tracking |
| `data/core/doc/highlighter.lua` | **titan** (syntax highlighting) | Incremental syntax highlighter: re-highlights only changed lines via token cache invalidation |
| `data/core/docview.lua` | **mercury** (display) | Document view: maps document lines to screen position, handles scroll, selection rendering, line wrapping |
| `data/core/command.lua` | **saturn** (route) | Command system: publish-subscribe command registry, predicate-based enable/disable, command chaining |
| `data/core/commandview.lua` | **mercury** (display) | Command palette: fuzzy filter over registered commands, active parameter completion |
| `data/core/rootview.lua` | **saturn** (route) | Root view compositor: split management, tab ordering, view focus routing (doc view, log view, command view) |
| `data/core/view.lua` | **mercury** (display) | View base class: position, size, scroll offset, surface allocation |
| `data/core/keymap.lua` | **europa** (configuration assembly) | Key binding: chord matching, mode-specific maps, macOS modifier translation |
| `data/core/config.lua` | **europa** (configuration assembly) | Configuration: user config merge, plugin config namespacing, editor defaults |
| `data/core/init.lua` | (orchestration) | Bootstrap: core initialisation, plugin loading, project detection |
| `data/core/plugins/` | **europa** (configuration assembly) | Plugin system: automatic `require` from plugin directory, plugin-level configuration files |
| `data/core/project.lua` | **callisto** (filesystem walk) | Project: directory tree scanning, file listing with gitignore filtering |
| `data/core/storage.lua` | **mars** (persist) | Persistent storage: project-local config, session state across restarts |
| `data/core/syntax.lua` | **rhea** (syntax detection) | Syntax detection: filename/extension to syntax definition matching |
| `data/core/tokenizer.lua` | **titan** (syntax highlighting) | Tokeniser: pattern-based token matching for syntax highlighting |
| `data/core/process.lua` | **io** (command execution) | Process: async subprocess spawning with stdout capture via Lua coroutine |
| `data/core/dirwatch.lua` | **callisto** (filesystem walk) | Directory watching: inotify/FSEvents/kqueue detection for file-changed events |
| `src/` | **hyperion** (network listener) + **mercury** (display) | C core: SDL2 window, renderer, font rasterisation (Freetype), Lua VM, platform file I/O, clipboard |
| `data/core/style.lua` | **mercury** (display) | Style system: colour definitions, font loading, DPI-aware scaling |

## Unused Planet Names

`adrastea`, `amalthea`, `ananke`, `ariel`, `belinda`, `bianca`, `caliban`, `calypso`, `carme`, `charon`, `cordelia`, `cressida`, `deimos`, `desdemona`, `despina`, `elara`, `enceladus`, `epimetheus`, `helene`, `himalia`, `hyperion`, `janus`, `juliet`, `larissa`, `metis`, `mimas`, `miranda`, `naiad`, `nereid`, `oberon`, `ophelia`, `pandora`, `pasiphae`, `phobos`, `phoebe`, `portia`, `prometheus`, `prospero`, `proteus`, `puck`, `rosalind`, `setebos`, `sinope`, `stephano`, `sycorax`, `telesto`, `thalassa`, `thebe`, `tethys`, `titania`, `trinculo`, `triton`, `umbriel`

## Control / Robotics Projects

### grbl (grbl/grbl)

A high-performance CNC motion controller for AVR microcontrollers, implementing G-code execution with look-ahead trajectory planning.

| grbl module | Problem Pattern | Rationale |
|-------------|----------------|-----------|
| `planner.c` / `planner.h` | **pan** (resource scheduling) | Look-ahead planner: buffers up to 18 blocks, computes velocity profiles with trapezoidal S-curve acceleration, junction speed optimisation |
| `stepper.c` | **pan** (resource scheduling) + **neptune** (transform) | Stepper ISR: Bresenham line algorithm for multi-axis coordination, AMASS adaptive multi-axis step-smoothing, acceleration/deceleration ramp execution |
| `motion_control.c` | **neptune** (transform) | Motion control: arc interpolation (G2/G3), helix, dwell — decomposes curved paths into linear segments for the planner |
| `spindle_control.c` | **mercury** (display) | Spindle PWM: RPM→PWM duty cycle mapping, direction control, enable/disable — open-loop speed output |
| `coolant_control.c` | **mercury** (display) | Coolant: flood/mist enable, pin toggling |
| `gcode.c` | **neptune** (transform) + **jupiter** (validate) | G-code parser: line parsing, modal state tracking, parameter extraction, error detection |
| `protocol.c` | **hyperion** (network listener) | Serial protocol: real-time command character detection, flow control, status reporting |
| `limits.c` | **jupiter** (validate) | Limit switches: hardware homing cycle, soft limits, pin state debouncing |
| `probe.c` | **jupiter** (validate) | Probing: G38.x precision touch-off, latched position capture |
| `settings.c` | **europa** (configuration assembly) | EEPROM settings: steps/mm, acceleration, max velocity, junction deviation — persistent calibration |
| `cpu_map.h` | **neptune** (transform) | CPU pin mapping: AVR register abstractions for different Arduino boards |
| `defaults.h` | **europa** (configuration assembly) | Machine defaults: per-machine profiles (Shapeoko, X-Carve, OxCNC) |
| `report.c` | **mercury** (display) | Status reports: real-time position, feed rate, buffer state, machine state |
| `system.c` | **europa** (configuration assembly) | System state machine: idle/alarm/hold/cycle/jog/check — mode transitions with safe guards |

### ArduPilot (ArduPilot/ardupilot)

An open-source autopilot system supporting multiple vehicle types (copter, plane, rover, sub) with cascaded control and sensor fusion.

| ArduPilot module | Problem Pattern | Rationale |
|------------------|----------------|-----------|
| `libraries/AP_AttitudeControl/` | **pan** (resource scheduling) | Cascaded attitude control: outer loop (angle→rate), inner loop (rate→accel), PID with feed-forward, axis prioritisation |
| `libraries/AP_InertialNav/` | **mars** (persist) | Inertial navigation: integrates accelerometer to estimate position/velocity, drift correction via GPS/baro |
| `libraries/AP_NavEKF/` | **mars** (persist) | Extended Kalman Filter: fuses IMU, GPS, compass, baro, airspeed into a consistent state estimate — 22-state covariance |
| `libraries/AC_PID/` | **pan** (resource scheduling) | PID controller library: proportional/integral/derivative terms, integrator clamping, D-term filtering, notch filters |
| `libraries/AP_Motors/` | **neptune** (transform) | Motor mixer: maps roll/pitch/yaw/thrust commands to individual motor outputs (quad, hex, octo, coaxial, tilt-rotor) |
| `libraries/AP_Navigation/` | **saturn** (route) | Navigation: waypoint sequencing, path following, loiter orbit, RTL path generation |
| `libraries/AP_Camera/` | **mercury** (display) | Camera trigger: servo or relay-based shutter control, time-lapse, geo-tagging |
| `libraries/AP_RangeFinder/` | **jupiter** (validate) | Range finder: sonar, lidar, infrared — distance measurement with outlier rejection |
| `libraries/AP_OpticalFlow/` | **mars** (persist) | Optical flow: visual odometry from downward-facing camera, velocity estimation |
| `libraries/AP_Avoidance/` | **saturn** (route) | Obstacle avoidance: re-plans path around detected obstacles, bendy-ruler algorithm |
| `libraries/AP_Follow/` | **saturn** (route) | Follow mode: tracks another vehicle or GCS position via MAVLink |
| `libraries/AP_Terrain/` | **mars** (persist) | Terrain database: cached elevation data, look-ahead terrain following |
| `ArduCopter/mode_*.cpp` | **saturn** (route) | Flight modes: stabilize, altitude hold, loiter, auto, RTL, acro, guided — each mode is a distinct control law |
| `ArduCopter/Attitude.cpp` | **pan** (resource scheduling) | Top-level attitude controller: calls AP_AttitudeControl in a 400Hz loop, feeds AP_Motors |
| `ArduCopter/system.cpp` | **europa** (configuration assembly) | System initialisation: parameter loading, sensor calibration, failsafe arm checks |
| `libraries/AP_Common/` | **jupiter** (validate) | Common utilities: LOCATION, parameter system, bitfield operations |
| `libraries/AP_HAL/` | **neptune** (transform) | HAL (Hardware Abstraction Layer): board-specific implementations (Pixhawk, Linux, SITL) for GPIO, SPI, I2C, UART |
| `libraries/GCS_MAVLink/` | **hyperion** (network listener) + **saturn** (route) | MAVLink protocol: telemetry streams, command dispatch, parameter upload/download |

---

## Real-System Mappings

### fd (sharkdp/fd)

The canonical `fd` file search tool, decomposed by its module structure (`src/` directory).

| fd module | Problem Pattern | Rationale |
|-----------|----------------|-----------|
| `src/cli.rs` | **europa** (configuration assembly) | CLI argument parsing → `Opts` struct → merged into `Config` with env vars and platform defaults |
| `src/config.rs` | **europa** (configuration assembly) | `construct_config()` merges opts, env vars, file types, exclusion rules into a single validated `Config` |
| `src/walk.rs` | **callisto** (filesystem walk) | Parallel directory traversal via `ignore::WalkParallel`, respecting depth, follow-links, one-file-system |
| `src/filesystem.rs` | **callisto** (filesystem walk) | Helper: default path separator, directory existence checks |
| `src/filter/mod.rs` | **venus** (filter) | Pattern matching on file names via regex/glob |
| `src/filter/owner.rs` | **venus** (filter) | Owner-based filtering (unix uid/gid) |
| `src/filter/size.rs` | **venus** (filter) | Size-based filtering |
| `src/filter/time.rs` | **venus** (filter) | Time-based filtering (changed before/after) |
| `src/regex_helper.rs` | **venus** (filter) | Regex construction, smart-case detection, leading-dot detection |
| `src/filetypes.rs` | **venus** (filter) | File type classification (file, dir, symlink, executable, etc.) |
| `src/output.rs` | **ganymede** (output formatting) | Result output with optional colour, paging via `less` if output is long |
| `src/fmt/mod.rs` | **ganymede** / **mercury** (output formatting / display) | Format template parsing: `{}`, `{.}`, `{/}`, `{//}`, `{/.}` path placeholders |
| `src/fmt/input.rs` | **ganymede** (output formatting) | Per-entry formatted string construction from template |
| `src/hyperlink.rs` | **ganymede** (output formatting) | Hyperlink (OSC 8) generation for terminal output |
| `src/exec/mod.rs` | **io** (command execution) | `CommandSet`: defining the command template with placeholder tokens |
| `src/exec/command.rs` | **io** (command execution) | Per-item command instantiation: substituting `{}`, `{.}`, etc. |
| `src/exec/job.rs` | **io** (command execution) | Parallel job execution: thread pool, batch mode, output collection |
| `src/dir_entry.rs` | **neptune** (transform) | `DirEntry` wrapper type: filesystem entry → displayable value |
| `src/error.rs` | **jupiter** (validate) | Error type definitions and formatted error output |
| `src/exit_codes.rs` | **jupiter** (validate) | Exit code enumeration and aggregation |
| `src/main.rs` | (orchestration) | Top-level pipeline: parse → walk → filter → format → execute |

### bat (sharkdp/bat)

A `cat(1)` clone with syntax highlighting and Git integration.

| bat module | Problem Pattern | Rationale |
|------------|----------------|-----------|
| `src/bin/bat/main.rs` | **europa** (configuration assembly) | CLI argument parsing via `clap`, environment variable reading |
| `src/config.rs` | **europa** (configuration assembly) | `Config` struct assembled from CLI args, env vars, config file (`bat.conf`), and platform defaults |
| `src/input.rs` | **callisto** / **mercury** (filesystem walk / display) | File opening, stdin reading, encoding detection — input source abstraction |
| `src/controller.rs` | (orchestration) | Top-level pipeline: inputs → syntax detection → highlighting → formatting → paging → output |
| `src/syntax_mapping.rs` | **rhea** (syntax detection) | Maps file extension, filename, shebang, or content to Sublime Text syntax language |
| `src/assets.rs` | **rhea** / **titan** (syntax detection / highlighting) | Compiled syntax definitions and theme assets (serialized `.sublime-syntax` and `.tmTheme`) |
| `src/pretty_printer.rs` | **titan** (syntax highlighting) | Public API: syntax-highlighted output via syntect engine |
| `src/printer.rs` | **titan** / **ganymede** (highlighting / output formatting) | `InteractivePrinter` (highlighted + styled) vs `SimplePrinter` (plain — acts like `cat`) |
| `src/decorations.rs` | **ganymede** (output formatting) | Line numbers, grid lines, file header decorations |
| `src/diff.rs` | **venus** (filter) + **mercury** (display) | Git diff integration: showing modification markers (+, -, ~) in the margin |
| `src/theme.rs` | **europa** (configuration assembly) | Color theme selection and application to token scopes |
| `src/terminal.rs` | **ganymede** (output formatting) | Terminal width detection, colour capability detection |
| `src/output.rs` | **saturn** (route) | `OutputHandle` abstraction: routes output to terminal, pipe, file, or pager |
| `src/paging.rs` | **saturn** (route) | Pager selection and configuration (less), decides when to engage |
| `src/pager.rs` | **io** (command execution) | Pager process lifecycle management (spawning less with args) |
| `src/less.rs` | **io** (command execution) | `less` environment variable configuration for bat integration |
| `src/wrapping.rs` | **ganymede** (output formatting) | Line wrapping mode: char wrap, word wrap, no wrap |
| `src/nonprintable_notation.rs` | **neptune** (transform) | Non-printable character representation — replace visible chars with notation |
| `src/line_range.rs` | **venus** (filter) | Line range filtering (show only lines N–M) |
| `src/style.rs` | **europa** (configuration assembly) | Style component selection (numbers, grid, header, snip) as bitmask from CLI |
| `src/error.rs` | **jupiter** (validate) | Error type definitions and formatted error messages |

### tokei (XAMPPRocky/tokei)

A code line counter supporting 150+ languages with per-file and per-language statistics.

| tokei module | Problem Pattern | Rationale |
|--------------|----------------|-----------|
| `src/cli.rs` | **europa** (configuration assembly) | CLI argument parsing: columns, sort order, output format, exclusion patterns |
| `src/config.rs` | **europa** (configuration assembly) | `Config` merged from CLI args, config file (`tokei.toml`/`.tokeirc`), and platform defaults |
| `src/input.rs` | **callisto** (filesystem walk) | Directory traversal respecting `.gitignore`, `.ignore`, and `.tokeignore` files |
| `src/language/languages.rs` | **rhea** (syntax detection) | Extension → language mapping from `languages.json` definition file |
| `src/language/syntax.rs` | **iapetus** (code counting) | Per-language state machine: classifies lines as code, comments, blanks, doc comments |
| `src/language/embedding.rs` | **iapetus** (code counting) | Embedded language support (e.g., Markdown containing Rust code blocks) |
| `src/language/language_type.rs` | **iapetus** / **neptune** (code counting / transform) | Language enum from JSON definitions, compiled via build script |
| `src/stats.rs` | **iapetus** (code counting) | Per-file `Stats` struct: line counts by category, aggregated into language totals |
| `src/sort.rs` | **earth** (sort) | Output sorting by files, lines, code, comments, or blanks column |
| `src/consts.rs` | **neptune** (transform) | Language name normalisation constants |
| `src/utils/fs.rs` | **callisto** (filesystem walk) | Filesystem helpers (canonicalise path, check if directory) |
| `src/utils/ext.rs` | **rhea** (syntax detection) | Extension extraction with edge case handling |
| `src/lib.rs` | (orchestration) | Public API: `Languages::get_statistics()` — walk → detect → count → aggregate → sort → format |

### pet (knqyf263/pet)

A CLI snippet manager with save, tag, search, sync, and execute features.

| pet module | Problem Pattern | Rationale |
|------------|----------------|-----------|
| `main.go`, `cmd/root.go` | **europa** (configuration assembly) | CLI argument parsing, config file loading (`~/.config/pet/config.toml`) |
| `config/config.go` | **europa** (configuration assembly) | Config struct: editor, selector, sync settings, tag filtering |
| `cmd/new.go` | **mars** (persist) | Create snippet: validates input, appends to snippet TOML file |
| `cmd/edit.go` | **mars** (persist) | Read snippet file into editor, write back on save |
| `cmd/list.go` | **mars** (persist) + **mercury** (display) | Read all snippets from file and display them |
| `cmd/search.go` | **venus** (filter) + **mercury** (display) | Interactive fuzzy search via fzf/peco, filter by tag, display results |
| `cmd/exec.go` | **io** (command execution) | Select snippet, substitute parameters, execute shell command |
| `cmd/clip.go` | **mercury** (display) | Copy snippet to clipboard (OS-specific) |
| `cmd/sync.go` | **mars** (persist) | Sync snippets to remote (Gist, GitLab) — push/pull with conflict resolution |
| `sync/gist.go`, `sync/gitlab.go`, `sync/ghe.go` | **mars** (persist) + **uranus** (authenticate) | Remote storage backends with token-based authentication |
| `dialog/view.go` | **mercury** (display) | Interactive TUI dialog for snippet selection and parameter entry |
| `snippet/snippet.go` | **neptune** (transform) | Snippet data model with variable/parameter expansion (`<param>`, `<param=default>`) |
| `path/path.go` | **neptune** (transform) | Platform-specific path resolution and expansion |

### dust (bootandy/dust)

A more intuitive `du` replacement showing disk usage as a tree with proportional bars.

| dust module | Problem Pattern | Rationale |
|-------------|----------------|-----------|
| `src/cli.rs` | **europa** (configuration assembly) | CLI argument parsing: display depth, sort order, size format, filter options |
| `src/config.rs` | **europa** (configuration assembly) | Config from CLI args + optional config file (`~/.config/dust/config.toml` or `~/.dust.toml`) |
| `src/dir_walker.rs` | **callisto** (filesystem walk) | Parallel filesystem traversal with symlink handling, device boundary control |
| `src/node.rs` | **neptune** (transform) | Tree node data model: path, size, children, aggregated totals |
| `src/filter.rs`, `src/filter_type.rs` | **venus** (filter) | Size-based (`-z`) and regex-based (`-e`, `-v`) filtering, file vs. directory filter (`-D`, `-F`) |
| `src/display.rs`, `src/display_node.rs` | **ganymede** / **mercury** (output / display) | Tree display with proportional ASCII bars, coloured by depth, percentage labels |
| `src/progress.rs` | **mercury** (display) | Spinner / progress indicator during filesystem walk |
| `src/platform.rs` | **neptune** (transform) | Platform-specific path and metadata handling (Unix vs. Windows) |
| `src/utils.rs` | (helpers) | Sorting, terminal width detection |

### tide (http-rs/tide)

An async web framework for Rust, built around routing, middleware, and type-safe request extraction.

| tide module | Problem Pattern | Rationale |
|-------------|----------------|-----------|
| `src/server.rs` | **hyperion** (network listener) + **saturn** (route) | `Server` struct: binds to address(es), holds router, manages graceful shutdown |
| `src/router.rs` | **saturn** (route) | `Router` with per-HTTP-method state machines via `routefinder`, fallback to `all_method_router`, HEAD→GET fallback, 405 detection |
| `src/route.rs` | **saturn** (route) | `Route` struct wrapping endpoint with middleware chain, method filtering |
| `src/endpoint.rs` | **saturn** (route) | `Endpoint` trait + `DynEndpoint` type erasure — the unit of request handling |
| `src/middleware.rs` | **saturn** (route) | `Middleware` trait, `Next` chain runner — wraps endpoint with pre/post processing |
| `src/request.rs` | **neptune** (transform) + **jupiter** (validate) | `Request<State>` with typed extraction via `serde::Deserialize`, query/body/param parsing |
| `src/response.rs` | **mercury** (display) | `Response` with status, headers, body — serialisable to HTTP wire format |
| `src/listener/` | **hyperion** (network listener) | `Listener` trait, `TcpListener`, `TlsListener` — address binding, accept loop, TLS handshake |
| `src/cookies/` | **neptune** (transform) | Cookie parsing and serialisation (Set-Cookie, Cookie headers) |
| `src/sessions/` | **mars** (persist) | Session state management across requests |
| `src/security/cors.rs` | **uranus** (authenticate) | CORS header enforcement as middleware |
| `src/fs/` | **callisto** (filesystem walk) + **mercury** (display) | Static file serving: path resolution, MIME type detection, file streaming |
| `src/sse/` | **hyperion** (network listener) + **mercury** (display) | Server-sent events: keep-alive connection, event serialisation |
| `src/log/` | **mercury** (display) | Request logging middleware

## GUI Projects (non-web)

### ratatui (ratatui-org/ratatui)

A Rust library for building terminal user interfaces (TUIs) — the dominant Rust TUI framework.

| ratatui module | Problem Pattern | Rationale |
|----------------|----------------|-----------|
| `ratatui/src/lib.rs` | (framework core) | Public API: `Terminal`, `Frame`, backend initialisation |
| `ratatui/src/init.rs` | **europa** (configuration assembly) | Terminal backend setup, panic hook installation, raw mode entry |
| `ratatui/src/widgets.rs` | **mercury** (display) | Widget trait definitions and built-in widget implementations |
| `src/widgets/` (examples) | **mercury** (display) | Widget examples — block, chart, gauge, list, paragraph, table, tabs, calendar |
| `src/widgets/stateful_widget_ref.rs` | **mars** (persist) | Stateful widget references preserving scroll position, selection across render cycles |
| `src/widgets/widget_ref.rs` | **mercury** (display) | Widget reference abstraction for composable rendering |
| `src/layout.rs` | **saturn** (route) | Layout constraints and splitting: `Constraint`, `Layout::split()` — routes screen area to child widgets |
| `src/text.rs` | **titan** (syntax highlighting) | `Text`, `Span`, `Line` types with style application and grapheme-aware rendering |
| `src/style.rs` | **mercury** (display) | `Style` — colour, modifier, underline definitions for terminal output |
| `src/symbols.rs` | **ganymede** (output formatting) | Unicode/ASCII symbol sets for drawing borders, bars, dots |
| `src/backend.rs` | **hyperion** (network listener) | `Backend` trait abstraction over crossterm/termion/termwiz — raw terminal I/O |
| `src/terminal.rs` | **hyperion** / **mercury** (network / display) | `Terminal` — viewport management, full-screen mode, auto-resize |
| `src/buffer.rs` | **mercury** (display) | Double-buffer cell grid — the framebuffer that drives terminal output |
| `src/frame.rs` | **mercury** (display) | `Frame` — per-frame rendering context with area and buffer |

### bottom (ClementTsang/bottom)

A cross-platform graphical process/system monitor for the terminal (TUI).

| bottom module | Problem Pattern | Rationale |
|---------------|----------------|-----------|
| `src/bin/main.rs` | (orchestration) | Entry point: initialise backend, run event loop |
| `src/app.rs` | **europa** (configuration assembly) | `App` struct: central state, manages layout, tabs, widget visibility |
| `src/app/states.rs` | **mars** (persist) | `AppState`: persisted state machine for tab selection, expanded widget, filter text |
| `src/app/layout_manager.rs` | **saturn** (route) | Layout configuration: splits screen into widget regions per user config |
| `src/app/filter.rs` | **venus** (filter) | Process filtering: search by name/PID, regex, logical OR |
| `src/app/data/store.rs` | **mars** (persist) | Data store: time-series buffers for CPU/mem/disk/network metrics |
| `src/app/data/time_series.rs` | **mars** (persist) | Rolling time-series buffer for graph widgets |
| `src/app/data/process.rs` | **neptune** (transform) | Process data model: PID, CPU%, mem%, state — from OS raw data to structured display |
| `src/components/` | **mercury** (display) | Reusable UI components: tabs, layout containers |
| `src/widgets/` | **mercury** / **ganymede** (display / output) | Widget implementations: CPU graph, memory, disk, network, process table, temperature, battery |
| `src/widgets/process_table/` | **venus** (filter) + **mercury** (display) | Process table with sort, filter, tree mode, kill signal support |
| `src/components/time_series/` | **mars** (persist) + **mercury** (display) | Time-series data collection and graph rendering |
| `src/options.rs` | **europa** (configuration assembly) | Config file parsing (`~/.config/bottom/bottom.toml`) with schema validation |
| `src/constants.rs` | **europa** (configuration assembly) | Default constants: refresh rate, default layout, colour palette |
| `src/event.rs` | **hyperion** (network listener) | Input event handling: keyboard, mouse, resize |
| `src/canvas/` | **ganymede** (output formatting) | Canvas rendering: braille dots, bar characters for graph drawing |
| `src/collection.rs` | **mars** (persist) | Data collection subsystem: OS-specific metric polling |

### egui (emilk/egui)

An immediate-mode GUI library for Rust, running on web, native, and in-game.

| egui crate / module | Problem Pattern | Rationale |
|---------------------|----------------|-----------|
| `crates/egui/src/lib.rs` | (framework core) | Immediate mode GUI loop: `Context`, `Ui`, widget trait |
| `crates/egui/src/context.rs` | **saturn** (route) | `Context`: frames input, output, and widget tree — the central routing hub |
| `crates/egui/src/ui.rs` | **mercury** (display) | `Ui`: layout panels, allocate space, place widgets |
| `crates/egui/src/layout.rs` | **saturn** (route) | Layout algorithms: horizontal, vertical, columns, wrapping — screen area routing |
| `crates/egui/src/containers/` | **mercury** (display) | Container widgets: window, panel, scroll area, collapsing header, resize, menu, popup, tooltip |
| `crates/egui/src/widgets/` | **mercury** (display) | Built-in widgets: button, slider, text edit, combo box, checkbox, radio, spinner, colour picker |
| `crates/egui/src/painter.rs` | **ganymede** (output formatting) | Painter: tessellates shapes into GPU-friendly vertex buffers |
| `crates/egui/src/style.rs` | **mercury** (display) | `Style`: colours, margins, spacing, widget sizes — visual appearance definition |
| `crates/egui/src/data/input/` | **hyperion** (network listener) | Input state: keyboard, mouse, touch, IME, raw input from platform |
| `crates/egui/src/memory/` | **mars** (persist) | Widget memory: persisted state across frames (collapsed state, scroll pos, text) |
| `crates/egui/src/animation_manager.rs` | **mercury** (display) | Animation: smooth transitions for colour, position, size changes |
| `crates/egui/src/placer.rs` | **saturn** (route) | Widget placement with position/size hints |
| `crates/eframe/` | **hyperion** (network listener) + **europa** (config) | Framework: native (winit + glow/wgpu) and web (WASM) backends, window creation, event loop |
| `crates/ecolor/` | **mercury** (display) | Colour types: Color32, Rgba, Hsva — colour space conversions |
| `crates/epaint/` | **ganymede** (output formatting) | 2D graphics rendering: shapes, text, tessellation, anti-aliasing |
| `crates/egui-winit/` | **hyperion** (network listener) | Winit integration: window events → egui input translation |
| `crates/egui-wgpu/`, `crates/egui_glow/` | **ganymede** (output formatting) | GPU rendering backends: wgpu and OpenGL |
| `crates/egui_extras/` | **mercury** (display) | Extra widgets: table, date picker, image loading, syntax highlighting |
| `crates/egui_plot/` | **mercury** (display) | Plotting widgets: line chart, scatter plot, bar chart, histogram |

### rnote (flxzt/rnote)

A vector-based drawing app for handwritten notes and document annotation, built with Rust and GTK4.

| rnote crate / module | Problem Pattern | Rationale |
|----------------------|----------------|-----------|
| `crates/rnote-ui/` | **mercury** (display) | GTK4 UI layer: main window, toolbars, settings dialogs, canvas widget |
| `crates/rnote-engine/` | **neptune** (transform) + **mars** (persist) | Document engine: stroke rendering, selection, page layout, file format handling |
| `crates/rnote-engine/src/engine/export.rs` | **neptune** (transform) | Document export: SVG, PDF, PNG, JPEG — format conversion |
| `crates/rnote-engine/src/engine/import.rs` | **neptune** (transform) | Document import: PDF, bitmap, SVG — external format → internal representation |
| `crates/rnote-engine/src/document/` | **mars** (persist) | Document model: pages, strokes, images, backgrounds, serialisation |
| `crates/rnote-engine/src/document/format.rs` | **mars** (persist) | `.rnote` file format: gzipped JSON schema with versioning |
| `crates/rnote-engine/src/engine/config.rs` | **europa** (configuration assembly) | Engine configuration: pen styles, audio feedback, tool mappings |
| `crates/rnote-engine/src/camera.rs` | **mercury** (display) | Camera: viewport positioning, zoom, pan across infinite canvas |
| `crates/rnote-engine/src/audioplayer.rs` | **mercury** (display) | Pen audio feedback: play brush/marker/typewriter sounds on stroke |
| `crates/rnote-compose/` | **neptune** (transform) | Vector composition: shapes (rectangle, ellipse, line, arrow, polygon, bezier), pen paths, constraints |
| `crates/rnote-compose/src/shapes/` | **neptune** (transform) | Shape definitions: geometric primitives with transform, style, hit-test |
| `crates/rnote-compose/src/builders/` | **neptune** (transform) | Shape builders: drag-to-create interaction for each shape type |
| `crates/rnote-compose/src/style/` | **mercury** (display) | Stroke styles: smooth, rough, textured — visual rendering of ink strokes |
| `crates/rnote-cli/` | **io** (command execution) | CLI tool: export, thumbnail, validators — batch operations on .rnote files |

### MonitorControl (MonitorControl/MonitorControl)

A macOS app for controlling external display brightness, volume, and contrast via DDC, native protocols, and software dimming.

| MonitorControl module | Problem Pattern | Rationale |
|----------------------|----------------|-----------|
| `MonitorControl/main.swift` | (orchestration) | App entry point: initialise menu, display manager, keyboard tap |
| `MonitorControl/Support/AppDelegate.swift` | **europa** (configuration assembly) | App lifecycle, preferences loading, menu bar setup |
| `MonitorControl/Support/DisplayManager.swift` | **neptune** (transform) | Display detection, enumeration, protocol selection (DDC/Apple/software) |
| `MonitorControl/Support/Arm64DDC.swift` | **neptune** (transform) | ARM-specific DDC communication via IOKit |
| `MonitorControl/Support/IntelDDC.swift` | **neptune** (transform) | Intel-specific DDC communication via IOKit |
| `MonitorControl/Support/OSDUtils.swift` | **mercury** (display) | Native macOS OSD overlay for brightness/volume changes |
| `MonitorControl/Support/MenuHandler.swift` | **mercury** (display) | Menu bar: per-display sliders, combined slider, quick actions |
| `MonitorControl/Support/SliderHandler.swift` | **mercury** (display) | Slider UI: brightness, volume, contrast controls with smooth transitions |
| `MonitorControl/Support/MediaKeyTapManager.swift` | **hyperion** (network listener) | Keyboard media key interception: brightness keys, volume keys |
| `MonitorControl/Support/KeyboardShortcutsManager.swift` | **europa** (configuration assembly) | Custom keyboard shortcut registration and handling |
| `MonitorControl/Support/UpdaterDelegate.swift` | **mars** (persist) | Auto-update via Sparkle: version checking, update download |
| `MonitorControl/Model/Display.swift` | **mars** (persist) | Display model: brightness/volume/contrast state, min/max ranges, protocol |
| `MonitorControl/Model/AppleDisplay.swift` | **mars** (persist) + **neptune** (transform) | Apple/Built-in display: native protocol control |
| `MonitorControl/Model/OtherDisplay.swift` | **mars** (persist) + **neptune** (transform) | Non-Apple display: DDC or software-only control |
| `MonitorControl/View Controllers/Preferences/` | **europa** (configuration assembly) | Preferences UI: display config, keyboard shortcuts, advanced settings |
| `MonitorControl/View Controllers/Onboarding/` | **europa** (configuration assembly) | First-launch onboarding: accessibility permission prompts |
| `MonitorControl/Enums/` | **jupiter** (validate) | Command and pref key enums: validated set of allowed values |
| `MonitorControlHelper/` | **io** (command execution) | Helper process: runs with accessibility permission for media key capture |

## Web Projects

### React (facebook/react)

A declarative JavaScript library for building user interfaces.

| React package | Problem Pattern | Rationale |
|---------------|----------------|-----------|
| `packages/react/` | (framework core) | Public API: `createElement`, `Component`, hooks (`useState`, `useEffect`, etc.) |
| `packages/react-reconciler/` | **neptune** (transform) | Fiber reconciler: transforms JSX tree → fiber tree → DOM updates |
| `packages/react-dom/` | **mercury** (display) | DOM renderer: mounts components, handles DOM events, portal support |
| `packages/react-dom-bindings/` | **neptune** (transform) | DOM property/event bindings: HTML attributes → React props, event delegation |
| `packages/react-server/` | **neptune** (transform) | Server rendering: streams HTML, handles Suspense on server |
| `packages/react-server-dom-webpack/` | **neptune** (transform) | Server Components: RSC payload serialisation, code splitting |
| `packages/scheduler/` | **saturn** (route) | Task scheduler: priority-based work queue, time slicing, idle callbacks |
| `packages/react-cache/` | **mars** (persist) | Data caching: memoised resource fetching with invalidation |
| `packages/react-refresh/` | **mercury** (display) | Fast Refresh: hot module replacement preserving component state |
| `packages/react-devtools/` | **mercury** (display) | DevTools: component tree inspector, profiler, state editor |
| `packages/shared/` | **jupiter** (validate) | Shared utilities: prop types validation, object pooling, bit manipulation |
| `packages/jest-react/` | **jupiter** (validate) | Testing utilities: `act()`, test renderer for assertions |

### Vue (vuejs/core)

A progressive JavaScript framework for building UIs, with compiled templates and fine-grained reactivity.

| Vue package | Problem Pattern | Rationale |
|-------------|----------------|-----------|
| `packages/vue/` | (orchestration) | Public API: exported from all sub-packages |
| `packages/compiler-core/` | **neptune** (transform) | Template compiler: tokeniser → AST → transform → codegen (render function) |
| `packages/compiler-dom/` | **neptune** (transform) | DOM-specific compiler: HTML-specific transforms, v-model, v-show, v-on |
| `packages/compiler-sfc/` | **neptune** (transform) | Single File Component compiler: parses `.vue` files, extracts script/template/style |
| `packages/compiler-ssr/` | **neptune** (transform) | SSR compiler: generates server-rendered HTML strings |
| `packages/runtime-core/` | **saturn** (route) | Runtime: component lifecycle, VNode diffing, slot resolution, provide/inject |
| `packages/runtime-dom/` | **mercury** (display) | DOM runtime: mounts VNodes, patches DOM, handles events |
| `packages/reactivity/` | **mars** (persist) | Reactivity system: `ref`, `reactive`, `computed`, `watch` — dependency tracking |
| `packages/server-renderer/` | **neptune** (transform) | Server-side rendering: renders Vue apps to HTML streams |
| `packages/shared/` | **jupiter** (validate) | Shared utilities: type helpers, DOM attribute maps, escape handling |
| `packages/runtime-test/` | **jupiter** (validate) | Test runtime: lightweight DOM mock for testing |
| `packages/vue-compat/` | **neptune** (transform) | Vue 2 compatibility layer: maps old API to new internals |

### Svelte (sveltejs/svelte)

A compiler-based web framework that shifts work from runtime to compile time.

| Svelte package / module | Problem Pattern | Rationale |
|------------------------|----------------|-----------|
| `packages/svelte/src/compiler/` | **neptune** (transform) | Svelte compiler: parses `.svelte` files, generates vanilla JS DOM update code |
| `packages/svelte/src/runtime/` | **mercury** (display) | Runtime helpers: component mounting, lifecycle, event handling |
| `packages/svelte/src/compiler/parse/` | **neptune** (transform) | Parser: tokenises `.svelte` HTML → AST with script/style sections |
| `packages/svelte/src/compiler/transform/` | **neptune** (transform) | Transforms: optimises AST, inlines constants, hoists statics |
| `packages/svelte/src/compiler/codegen/` | **neptune** (transform) | Codegen: generates JavaScript from AST with minimal runtime overhead |
| `packages/svelte/src/reactivity/` | **mars** (persist) | Runes reactivity: `$state`, `$derived`, `$effect` — compile-time reactive primitives |
| `packages/svelte/src/store/` | **mars** (persist) | Store API: `writable`, `readable`, `derived` — runtime reactive state containers |
| `packages/svelte/src/ssr/` | **neptune** (transform) | SSR codegen: generates HTML string rendering for server-side output |
| `packages/svelte/src/events/` | **hyperion** (network listener) | Event handling: delegated event system with cleanup |
| `packages/svelte/src/animate/`, `src/transition/`, `src/easing/` | **mercury** (display) | Animation, transition, easing utilities for declarative UI motion |
| `packages/svelte/src/motion/` | **mercury** (display) | Spring/tween motion stores for smooth value interpolation |
| `documentation/` | **mercury** (display) | Documentation site rendered from markdown |

### Express (expressjs/express)

A minimal web framework for Node.js providing routing, middleware, and HTTP utilities.

| Express module | Problem Pattern | Rationale |
|----------------|----------------|-----------|
| `lib/express.js` | (orchestration) | Factory: `createApplication()` — creates the app, mixes in proto methods |
| `lib/application.js` | **saturn** (route) | `app` object: route registration (`.get()`, `.post()`), middleware pipeline (`use`), settings |
| `lib/request.js` | **neptune** (transform) | `req` object: extends Node's `IncomingMessage` with param parsing, content negotiation, protocol detection |
| `lib/response.js` | **mercury** (display) | `res` object: extends `ServerResponse` with `.send()`, `.json()`, `.render()`, `.redirect()`, `.format()` |
| `lib/router/` | **saturn** (route) | `Router`: layer stack, route matching with params, error handling middleware |
| `lib/view.js` | **neptune** (transform) | View engine: template rendering abstraction, caching compiled templates |
| `lib/utils.js` | **jupiter** (validate) | Utilities: ETag generation, content-type parsing, proxy address detection |
| `middleware/` (ecosystem) | **venus** (filter) + **neptune** (transform) | Middleware: body parsing, cookie parsing, compression, sessions (external packages) |

### htmx (bigskysoftware/htmx)

A library that extends HTML with AJAX, CSS transitions, WebSockets, and Server Sent Events via attributes.

| htmx module | Problem Pattern | Rationale |
|-------------|----------------|-----------|
| `src/htmx.js` | (framework core) | Core: bootstrapping, attribute scanning, event handling, AJAX dispatch |
| `src/htmx.js` — attribute processing | **saturn** (route) | Routes `hx-*` attributes to AJAX actions: `hx-get`, `hx-post`, `hx-put`, `hx-delete`, `hx-patch` |
| `src/htmx.js` — swap strategies | **mercury** (display) | Swap modes: `innerHTML`, `outerHTML`, `beforebegin`, `afterbegin`, `beforeend`, `afterend`, `none` |
| `src/htmx.js` — triggers | **hyperion** (network listener) | Trigger system: `hx-trigger` — click, submit, load, reveal, intersect, throttle/debounce, polling |
| `src/htmx.js` — history | **mars** (persist) | History management: push/replace URL, cache, restore, back/forward navigation |
| `src/htmx.js` — extensions | **neptune** (transform) | Extension system: hooks for custom behaviour, lifecycle callbacks |
| `src/htmx.js` — indicators | **mercury** (display) | Request indicators: `hx-indicator`, CSS class toggling during AJAX |
| `src/htmx.js` — validation | **jupiter** (validate) | Form validation: `hx-validate` — HTML5 constraint API integration |
| `src/htmx.js` — WebSocket & SSE | **hyperion** (network listener) | WebSocket and Server-Sent Events: persistent connection management, content swapping |
| `src/htmx.js` — HTTP methods | **neptune** (transform) | AJAX request construction: method, headers, params, body from HTML attributes |
| `dist/` | **ganymede** (output formatting) | Distribution: minified, ESM, TypeScript definitions |

## Game Projects

### Mindustry (Anuken/Mindustry)

An open-source automation tower defense RTS game written in Java, with a custom ECS architecture.

| Mindustry module | Problem Pattern | Rationale |
|------------------|----------------|-----------|
| `core/src/mindustry/` | (orchestration) | Root: `ClientLauncher`, `Vars`, `Mindustry` — app entry and global state |
| `core/src/mindustry/entities/` | **neptune** (transform) | ECS: entity components (`Unit`, `Building`, `Bullet`, `Player`), systems, component IO |
| `core/src/mindustry/entities/comp/` | **neptune** (transform) | Component definitions: health, position, team, shield — annotated with `@EntityDef` |
| `core/src/mindustry/world/` | **mars** (persist) | World state: tiles, blocks, chunks, world generation, save/load |
| `core/src/mindustry/world/blocks/` | **neptune** (transform) | Block definitions: conveyors, turrets, drills, factories — over 300 block types |
| `core/src/mindustry/type/` | **jupiter** (validate) | Type definitions: item types, liquid types, unit types, weather types, status effects |
| `core/src/mindustry/content/` | **europa** (configuration assembly) | Content registry: loads and registers all game content (items, blocks, units, etc.) |
| `core/src/mindustry/ai/` | **saturn** (route) | AI: pathfinding, unit AI, building AI, state machine behaviour trees |
| `core/src/mindustry/logic/` | **neptune** (transform) | Logic system: processor blocks, in-game scripting language execution |
| `core/src/mindustry/net/` | **hyperion** (network listener) + **uranus** (authenticate) | Networking: server/client, packet serialisation, remote method calls (`@Remote`), matchmaking |
| `core/src/mindustry/ui/` | **mercury** (display) | UI: menus, HUD, dialogs, minimap, fragment-based UI system |
| `core/src/mindustry/graphics/` | **ganymede** (output formatting) | Rendering: sprite batching, shaders, lighting, minimap, weather effects |
| `core/src/mindustry/audio/` | **mercury** (display) | Audio: sound effects, music tracks, ambient sounds |
| `core/src/mindustry/input/` | **hyperion** (network listener) | Input: touch, keyboard, mouse, controller — platform-abstracted input handling |
| `core/src/mindustry/maps/` | **mars** (persist) | Maps: map generation, loading, conversion, custom map files |
| `core/src/mindustry/game/` | **saturn** (route) | Game modes: survival, sandbox, PvP, campaign — mode-specific rules and win conditions |
| `core/src/mindustry/editor/` | **neptune** (transform) | Map editor: tile painting, undo/redo, import/export |
| `core/src/mindustry/io/` | **mars** (persist) | IO: save/load, settings persistence, export formats |
| `core/src/mindustry/mod/` | **europa** (configuration assembly) | Modding: mod loading, content overrides, script injection |
| `annotations/` | **neptune** (transform) | Annotation processor: generates entity classes, remote call packets, asset management code |
| `desktop/`, `server/`, `android/` | **europa** (configuration assembly) | Platform launchers: desktop (LWJGL), server (headless), Android |

### doukutsu-rs (doukutsu-rs/doukutsu-rs)

A reimplementation of the Cave Story (Doukutsu Monogatari) engine in Rust with modern enhancements.

| doukutsu-rs module | Problem Pattern | Rationale |
|--------------------|----------------|-----------|
| `src/main.rs` | (orchestration) | Entry point: game loop, scene management, platform init |
| `src/game/` | **saturn** (route) | Game state machine: map → gameplay, pause menu, inventory, dialogue — state transitions |
| `src/scene/` | **saturn** (route) | Scene management: title screen, credits, stage select, cutscene — composable scene stack |
| `src/menu/` | **mercury** (display) | Menus: title, settings, save/load, inventory — UI rendering in immediate mode |
| `src/components/` | **neptune** (transform) | ECS components: map system, tilemap, HUD, boss bar, fade, flash, text boxes, inventory |
| `src/components/map_system.rs` | **saturn** (route) + **callisto** (filesystem walk) | Map system: tile collision, entity placement, background layers — routes game objects by position |
| `src/components/tilemap.rs` | **mercury** (display) | Tilemap rendering: parallax layers, animated tiles, water shader |
| `src/components/hud.rs` | **mercury** (display) | HUD: health, weapon, timer, inventory display — overlay rendering |
| `src/components/text_boxes.rs` | **mercury** (display) | Dialogue system: text boxes, portrait display, typewriter effect, choices |
| `src/entity.rs` | **neptune** (transform) | Entity: NPCs, enemies, player, projectiles — entity data model with type hierarchy |
| `src/graphics/` | **ganymede** (output formatting) | Rendering: surface blitting, spritesheet, palette effects, screen shake, fade transitions |
| `src/sound/` | **mercury** (display) | Audio: music playback, sound effects, organya tracker, position-based panning |
| `src/input/` | **hyperion** (network listener) | Input: keyboard, gamepad, key binding — abstracted input state |
| `src/framework/` | **hyperion** (network listener) | Platform abstraction: window creation, OpenGL context, frame timing, vsync |
| `src/data/` | **mars** (persist) + **callisto** (filesystem walk) | Data loading: extracts files from game archives, resource caching |
| `src/editor/` | **neptune** (transform) | Map editor: tile editing, entity placement, script trigger editing |
| `src/discord/` | **mercury** (display) | Discord Rich Presence: game state → presence update |
| `drsandroid/` | **europa** (configuration assembly) | Android port: Java/Kotlin glue code, APK packaging |
| `drshorizon/` | **europa** (configuration assembly) | Nintendo Switch port: homebrew build, ROMFS mount |

### OpenTTD (OpenTTD/OpenTTD)

A transport simulation game based on Transport Tycoon Deluxe, written in C++.

| OpenTTD module | Problem Pattern | Rationale |
|----------------|----------------|-----------|
| `src/openttd.cpp` | (orchestration) | Entry point: game loop, main state machine, driver initialisation |
| `src/economy/` | **mars** (persist) | Economy simulation: cargo payment, industry production, inflation, loan interest |
| `src/economy/func.h` | **mars** (persist) | Economic calculation: cargo profit, running costs, vehicle purchase price |
| `src/road_cmd.cpp`, `src/rail_cmd.cpp`, `src/water_cmd.cpp` | **neptune** (transform) | Infrastructure commands: build/remove roads, rails, docks, airports — command pattern |
| `src/vehicle/` | **neptune** (transform) | Vehicle system: train, road vehicle, ship, aircraft — shared base with per-type specialisation |
| `src/vehicle_base.h` | **neptune** (transform) | Vehicle data model: position, cargo, speed, reliability, orders |
| `src/station/` | **neptune** (transform) | Station system: coverage, rating, cargo waiting, catchment area |
| `src/industry/` | **mars** (persist) | Industry system: production chains, raw material → processed goods, closure probability |
| `src/town/` | **mars** (persist) | Town simulation: population growth, building construction, road layout |
| `src/pathfinder/` | **saturn** (route) | Pathfinding: YAPF, NPF, Dijkstra — route vehicles through the network |
| `src/network/` | **hyperion** (network listener) + **uranus** (authenticate) | Multiplayer: game server, admin port, UDP discovery, packet compression |
| `src/genworld.h`, `src/genworld.cpp` | **neptune** (transform) | World generation: terrain, towns, industries, rivers — algorithmic landscape creation |
| `src/gfx/`, `src/blitter/` | **ganymede** (output formatting) | Graphics: sprite management, blitting, palette animation, zoom, anti-aliasing |
| `src/video/` | **hyperion** (network listener) | Video backends: SDL, OpenGL, Win32 GDI — window creation, input, frame loop |
| `src/widget/` | **mercury** (display) | UI widgets: dropdown, edit box, list, matrix, scrollbar — in-game GUI toolkit |
| `src/window/` | **mercury** (display) | Window management: z-order, resize, docking — window and dialog system |
| `src/saveload/` | **mars** (persist) | Save/load: savegame format, backwards compatibility across 20+ years of versions |
| `src/newgrf/` | **europa** (configuration assembly) | NewGRF: mod loading, sprite overrides, custom callbacks — content extension system |
| `src/ai/`, `src/game/` | **saturn** (route) | AI/Game scripting: Squirrel VM integration, script API, goal-based AI |
| `src/music/` | **mercury** (display) | Music backends: FluidSynth, DirectMusic, MIDI, custom stream music |
| `src/sound/` | **mercury** (display) | Sound backends: FMOD, SDL_mixer, OpenAL — effect playback |
| `src/terraform_cmd.cpp`, `src/landscape.cpp` | **neptune** (transform) | Terrain modification: raise/lower land, build canals, demolish — map editing commands |

### Endless Sky (endless-sky/endless-sky)

A sandbox-style space exploration game similar to Escape Velocity, with data-driven content and a C++ engine.

| Endless Sky module | Problem Pattern | Rationale |
|--------------------|----------------|-----------|
| `source/` | (orchestration) | C++ game engine: main loop, game state, rendering pipeline |
| `source/GameData.h` | **europa** (configuration assembly) | Data loading: parses all `data/` text files into in-memory objects |
| `source/PlayerInfo.h` | **mars** (persist) | Player state: fleet, credits, reputation, mission progress, visited systems |
| `source/Ship.h` | **neptune** (transform) | Ship: hardpoints, outfits, cargo, health, fighter/drone bays |
| `source/Outfit.h` | **neptune** (transform) | Outfit: weapons, engines, shields, cargo expansions — equipment data model |
| `source/System.h` | **saturn** (route) | Star system: planets, links to other systems, trade goods, fleets, hazards |
| `source/Planet.h` | **mars** (persist) | Planet: services (shipyard, outfitter, spaceport), government, population |
| `source/Mission.h` | **saturn** (route) | Mission: conditions, triggers, waypoints, cargo, dialogue, branches — story state machine |
| `source/AI.h` | **saturn** (route) | AI: ship combat, movement, escort behaviour, formation flying, targeting |
| `source/Fleet.h` | **mars** (persist) | Fleet: ship composition, spawn triggers, reinforcement patterns |
| `source/Engine.h` | **neptune** (transform) | Combat engine: projectile motion, collision detection, damage calculation |
| `source/DrawList.h` | **mercury** (display) | Rendering: sprite layering, animation, object culling |
| `source/MapPanel.h` | **mercury** (display) | Map: star system display, route plotting, faction colouring |
| `source/ShipInfoPanel.h` | **mercury** (display) | Ship info: cargo/manoeuvring display, outfit viewer, crew status |
| `source/HailPanel.h` | **mercury** (display) | Hail dialogue: communication with other ships, hails, boarding |
| `source/ConversationPanel.h` | **mercury** (display) | Dialogue: branching conversations with condition-based choices |
| `source/Files.h` | **callisto** (filesystem walk) | File system: save game paths, data directory detection, plugin loading |
| `source/Audio.h` | **mercury** (display) | Audio: music, sound effects, positional audio, OpenAL |
| `data/` | **europa** (configuration assembly) | Game data: ships, outfits, planets, systems, missions, conversations, fleets — all data-driven in text |

### Veloren (veloren/veloren)

A multiplayer voxel RPG written in Rust with an ECS architecture and client/server split.

| Veloren crate / module | Problem Pattern | Rationale |
|------------------------|----------------|-----------|
| `server/` | **hyperion** (network listener) + **mars** (persist) | Game server: world simulation, player persistence, NPC AI, physics |
| `client/` (voxygen/) | **mercury** (display) + **hyperion** (network listener) | Client: 3D rendering (vulkan), input handling, UI, audio |
| `common/` | **neptune** (transform) | Shared types: components, protocol messages, math, world model |
| `common/src/comp/` | **neptune** (transform) | ECS components: position, velocity, health, inventory, stats |
| `common/src/sys/` | **saturn** (route) | ECS systems: physics, combat, AI, animation, interactions |
| `common/src/terrain/` | **mars** (persist) | Voxel terrain: generation, modification, lighting, LOD |
| `common/src/trade/` | **neptune** (transform) | Trading: item economy, buy/sell, market simulation |
| `common/src/agent/` | **saturn** (route) | Agent AI: behaviour trees, pathfinding, combat tactics |
| `assets/` | **europa** (configuration assembly) | Game data: ability definitions, item stats, NPC behaviour, loot tables — in `.ron` files |
