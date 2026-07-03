# Chapter 14 — Debugging & Tooling

> JD: "strong debugging skills", "development tools", performance analysis. Interviewers use debugging questions to separate people who *write* code from people who can *own* code in production. Have war stories ready.

## 14.1 The debugging method (say this before any tool)

1. **Reproduce** reliably — a bug you can't reproduce, you can't verify as fixed.
2. **Minimize** — smallest input / shortest path that still fails.
3. **Hypothesize → test** — binary-search the space (which layer? which commit? `git bisect`).
4. **Fix the root cause**, not the symptom.
5. **Add a regression test** (Ch 12) and ask "where else does this pattern exist?"

**Interview line:** "I debug with hypotheses, not by randomly changing code. Every experiment should cut the search space in half."

## 14.2 GDB / LLDB — the core session

```bash
# Build with debug info first
g++ -g -O0 main.cpp -o app        # C++
cargo build                        # Rust debug profile has symbols by default

gdb ./app
(gdb) break main.cpp:42            # or: break parse_rpm
(gdb) run --config test.toml
(gdb) next        # step over        (gdb) step   # step into
(gdb) print rpm   # inspect          (gdb) print *machine
(gdb) backtrace   # where am I / call stack
(gdb) watch counter                # break when a VARIABLE changes — gold for corruption bugs
(gdb) frame 2 && info locals       # move up the stack, dump locals
(gdb) continue
```

### Post-mortem: core dumps (the production skill)

```bash
ulimit -c unlimited                # allow core dumps
./app                              # ...crashes...
gdb ./app core                     # open the corpse
(gdb) bt full                      # full backtrace with locals — usually enough
```

On modern Linux, `coredumpctl gdb <pid>` retrieves dumps captured by systemd. Mention `rust-gdb`/`rust-lldb` wrappers that pretty-print Rust types.

Attach to a *running* hung process: `gdb -p <pid>`, then `thread apply all bt` — instantly shows a deadlock (two threads each waiting in `lock()`).

## 14.3 Sanitizers & Valgrind (memory bugs — links to Ch 3)

| Tool | Catches | Cost |
|---|---|---|
| **ASan** (`-fsanitize=address`) | use-after-free, buffer overflow, leaks | ~2× slowdown — use in CI tests |
| **TSan** (`-fsanitize=thread`) | data races | ~10× — run on concurrency tests |
| **UBSan** (`-fsanitize=undefined`) | signed overflow, null deref, bad casts | tiny — always on in debug |
| **Valgrind memcheck** | same class as ASan, no recompile needed | ~20–30× — occasional deep runs |

```bash
g++ -g -fsanitize=address,undefined main.cpp && ./a.out
valgrind --leak-check=full ./app
```

**Rust angle:** safe Rust makes ASan/TSan mostly unnecessary — the borrow checker rejects those bugs at compile time. You still use them for `unsafe` blocks and FFI (`cargo +nightly ... -Zsanitizer=address`), and **Miri** interprets Rust to catch undefined behavior in unsafe code.

## 14.4 Logging & tracing — debugging without a debugger

Production bugs are debugged from logs, not breakpoints. Principles:

- **Structured logging** (JSON key=value) so logs are queryable, not grep-only prose.
- **Levels** with intent: ERROR (someone should act), WARN (self-healed anomaly), INFO (state changes: started, connected, config loaded), DEBUG/TRACE (dev only).
- **Correlation/request IDs**: generate at the edge, pass through every layer and message header — one ID reconstructs a request's journey across services (crucial with Kafka pipelines, Ch 10).

```rust
// Rust: the tracing crate — spans give context to everything inside
#[tracing::instrument(fields(machine_id = %id))]
async fn poll_machine(id: MachineId) -> Result<Reading, PollError> {
    tracing::debug!("connecting");
    let r = read_sensor(id).await?;
    tracing::info!(rpm = r.rpm, "reading ok");
    Ok(r)
}
```

C++ equivalents to name: **spdlog** (fast, structured-ish), `std::format`-based wrappers; **OpenTelemetry** for distributed tracing in both languages.

## 14.5 Performance analysis — find the slow part, don't guess

```bash
# CPU: where is time spent?
perf record -g ./app && perf report          # sampled call graph
perf top                                      # live view
# Flamegraphs — the picture worth 1000 profiles
cargo flamegraph                              # Rust one-liner (uses perf)

# Syscalls: is it even MY code?
strace -c ./app        # syscall summary — spot excessive read()/futex()
ltrace ./app           # library calls

# System-wide triage (know the USE method: Utilization, Saturation, Errors)
htop                   # CPU per core, memory
iostat -x 1            # disk latency/util
vmstat 1               # run queue, swapping
```

Rules to state: **measure before optimizing** (Amdahl's law — optimize the 80% hot path); benchmark with **criterion** (Rust) / **Google Benchmark** (C++) so improvements are provable; watch for the classics — accidental O(n²), allocation in hot loops, false sharing (Ch 4), N+1 queries (Ch 9).

## 14.6 The everyday toolbox

| Category | Tools to name-drop |
|---|---|
| Build | CMake + Ninja (C++), Cargo (Rust), ccache/sccache |
| Lint/format | clang-tidy, clang-format, cppcheck / clippy, rustfmt |
| Editor | VS Code / CLion with LSP (clangd, rust-analyzer) |
| Network debugging | `curl -v`, `tcpdump`, Wireshark, `ss -tlnp`, Postman |
| Containers | `docker logs -f`, `docker exec -it <c> sh`, `docker stats` |
| Docs | Doxygen (C++), `cargo doc` (Rust) |

Debugging *inside* a container: `docker exec` in, or run the debug image with `--cap-add=SYS_PTRACE` so gdb can attach; check `docker inspect` for env/config drift between "works locally" and prod.

## 14.7 Three rehearsed war-story templates (fill with your experience)

1. **The crash**: "Intermittent segfault in production → enabled core dumps → `bt full` showed use-after-free on a callback whose owner had been destroyed → fixed ownership with `shared_ptr`/lifetime, added ASan to CI so the class of bug is caught pre-merge."
2. **The hang**: "Service froze under load → `gdb -p` + `thread apply all bt` → two mutexes taken in opposite order in two code paths → enforced global lock ordering, added a lock-hierarchy assert in debug builds."
3. **The slowdown**: "P99 latency crept up 10× → flamegraph showed 60% of time in JSON re-serialization inside a loop → hoisted it, added a criterion benchmark to CI to catch regressions."

---

## 🎯 Chapter 14 Interview Q&A

**Q1. A service crashes in production but you can't reproduce it locally. Steps?**
Grab evidence first: logs around the timestamp, core dump (`coredumpctl`), metrics. `gdb app core` → backtrace shows the crash site; correlate with the request via correlation ID. If evidence is thin, add targeted logging/asserts, deploy, wait for recurrence. Diff prod vs local environment (config, data volume, concurrency).

**Q2. How do you debug a deadlock?**
Attach: `gdb -p <pid>`, `thread apply all bt` — deadlocked threads sit in lock acquisition; the stacks show which locks and in which order. Prevention: single global lock order, `std::scoped_lock` taking both at once, or redesign to message passing.

**Q3. Program is slow. First three things?**
(1) Define slow — which metric, since when (a regression? bisect it). (2) Profile, don't guess: perf/flamegraph for CPU-bound; strace/iostat if it's I/O; check saturation with USE. (3) Fix the top of the flamegraph, benchmark before/after.

**Q4. What is a watchpoint?**
A breakpoint on *data*: `watch var` stops when the value changes, hardware-assisted. The tool for "who is corrupting this field?" when the write site is unknown.

**Q5. ASan vs Valgrind?**
Both find memory errors. ASan is compile-time instrumentation, ~2× overhead, great in CI; Valgrind needs no rebuild but is 20–30× slower, so it's for occasional deep analysis or third-party binaries.

**Q6. How does the borrow checker change debugging in Rust?**
Whole classes disappear at compile time: use-after-free, data races, iterator invalidation. Remaining bugs are logic errors, panics (debug via `RUST_BACKTRACE=1`), and async issues (missed wakeups, blocking the executor) — debugged with tracing spans and tokio-console.

**Q7. Debug vs release builds — why do bugs appear only in release?**
Optimizations change timing and memory layout: UB that "worked" at -O0 breaks at -O2 (uninitialized reads, races). That's why UBSan/TSan and testing release builds in CI matter. Keep symbols in release (`-g` doesn't affect codegen; Rust: `debug = true` in the release profile) so crashes are diagnosable.

**Q8. What is a flamegraph and how do you read it?**
Visualization of sampled stacks: x-axis is proportion of samples (not time order), y-axis is stack depth. Wide boxes = hot code. Look for wide plateaus you didn't expect — that's your optimization target.

**Q9. Your fix works locally but the bug persists in the container. Why might that be?**
Different image/env: config drift, older binary cached (rebuild without cache), missing capability or ulimit, different libc/locale/timezone, resource limits triggering OOM-kill (`docker inspect`, `dmesg`). Reproduce inside the same image to close the gap.

**Q10. How do you debug across multiple services (Kafka pipeline)?**
Correlation IDs propagated in message headers + centralized structured logs — query the ID and reconstruct the flow; distributed tracing (OpenTelemetry) shows spans per hop with latency; consumer lag metrics tell you *where* the pipeline stalls, DLQs capture the poison messages.
