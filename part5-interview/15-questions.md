# Chapter 15 — 200+ Interview Questions & Answers

> Rapid-fire drill deck covering every JD area. Answers are deliberately short — the *skeleton* you expand in conversation. Drill: cover the answer, say yours out loud, compare. Chapters cited for deep dives.

## How to answer in an interview
- Lead with the one-sentence answer, then one concrete detail or example, then stop. Let them pull more.
- If you don't know: "I haven't used X, but based on Y I'd expect…" — reasoning beats bluffing.

---

## A. C++ Core (Ch 1) — Q1–Q30

**Q1. Stack vs heap allocation?**
Stack: automatic lifetime, LIFO, fast (pointer bump), size-limited. Heap: manual/RAII-managed lifetime, dynamic size, slower (allocator), fragmentation possible.

**Q2. What is RAII?**
Resource Acquisition Is Initialization: acquire resources in constructors, release in destructors, so scope exit (including exceptions) guarantees cleanup. The foundation of C++ resource safety.

**Q3. `unique_ptr` vs `shared_ptr` vs `weak_ptr`?**
`unique_ptr`: sole owner, zero overhead, movable not copyable. `shared_ptr`: refcounted shared ownership, atomic counter cost. `weak_ptr`: non-owning observer of a `shared_ptr` — breaks reference cycles.

**Q4. Rule of Zero / Three / Five?**
If you manage a raw resource, define destructor + copy ctor + copy assignment (Three), plus move ctor + move assignment (Five). Best: manage resources with members that already do (string, vector, unique_ptr) and write none (Zero).

**Q5. What are move semantics?**
Transferring resources from an rvalue instead of copying: move ctor/assignment steal pointers and leave the source valid-but-empty. Enables cheap returns and container growth.

**Q6. What is `std::move` actually?**
Just a cast to rvalue reference — it moves nothing itself. It marks an object as "movable-from"; a move ctor/assignment does the stealing.

**Q7. lvalue vs rvalue?**
lvalue: has identity/address, persists (variables). rvalue: temporary, about to die (function returns, literals). Rvalue references (`&&`) bind to rvalues enabling moves.

**Q8. `const` correctness — why?**
Documents intent, lets the compiler catch mutation bugs, enables optimization, and allows const objects/refs to call the method. Mark every method that doesn't mutate as `const`.

**Q9. Virtual functions — how do they work?**
Each polymorphic class has a vtable of function pointers; each object holds a vptr. A virtual call indirects through the vptr — dynamic dispatch at runtime, ~1 indirection cost, inhibits inlining.

**Q10. Why must a base class destructor be virtual?**
Deleting a derived object via a base pointer with a non-virtual destructor is undefined behavior — the derived destructor never runs, leaking resources.

**Q11. Pure virtual functions and abstract classes?**
`virtual void f() = 0;` — no implementation, class can't be instantiated; derived classes must override. C++'s interface mechanism.

**Q12. Overloading vs overriding?**
Overloading: same name, different parameters, resolved at compile time. Overriding: derived class replaces a base virtual function, same signature, resolved at runtime. Use `override` keyword to catch mistakes.

**Q13. What is undefined behavior? Give examples.**
Behavior the standard doesn't constrain — anything may happen, including "seems to work". Examples: out-of-bounds access, use-after-free, signed overflow, data races, null deref.

**Q14. References vs pointers?**
Reference: alias, must be bound at init, can't be null or reseated. Pointer: object holding an address, nullable, reseatable, supports arithmetic. Prefer references for parameters that must exist.

**Q15. What does `constexpr` do?**
Allows evaluation at compile time — values baked into the binary, zero runtime cost. `constexpr` functions run at compile time when arguments are constant.

**Q16. Templates vs virtual functions for polymorphism?**
Templates: compile-time (static) polymorphism — zero overhead, code bloat, errors at instantiation. Virtual: runtime polymorphism — one vtable hop, single code copy, works across ABI boundaries.

**Q17. What is template specialization?**
Providing a custom implementation for specific type arguments (`template<> struct Hash<string>`). Partial specialization for a family of types (pointers).

**Q18. What is SFINAE / what are concepts?**
Substitution Failure Is Not An Error — invalid template substitutions remove overloads instead of erroring. C++20 **concepts** replace it with readable constraints: `template<std::integral T>`.

**Q19. `std::vector` vs `std::list` — when each?**
Vector: contiguous, cache-friendly, O(1) amortized push_back, O(n) middle insert — default choice, nearly always wins in practice. List: stable iterators, O(1) splice — rare niche.

**Q20. How does `std::vector` grow?**
When capacity is exhausted it allocates a bigger block (~1.5–2×), moves/copies elements, frees the old — invalidating all iterators/pointers. `reserve()` avoids repeated growth.

**Q21. `map` vs `unordered_map`?**
`map`: red-black tree, ordered, O(log n). `unordered_map`: hash table, O(1) average, no order, needs hash function. Default to unordered unless you need ordering/range queries.

**Q22. Iterator invalidation — give a classic bug.**
Erasing from a vector while iterating: `for (it...) if (bad) v.erase(it)` — `erase` invalidates `it`. Fix: `it = v.erase(it)` or the erase-remove idiom.

**Q23. What are exceptions' costs and alternatives?**
Zero-cost on the happy path (table-based), expensive to throw (unwinding). Alternatives: error codes, `std::optional`, `std::expected` (C++23) — Rust-style value-based errors.

**Q24. What is the ODR?**
One Definition Rule: every entity has exactly one definition across the program (inline entities: identical definitions per TU). Violations = linker errors or silent UB.

**Q25. `static` — its meanings?**
Namespace scope: internal linkage. In a function: one instance across calls, initialized once (thread-safe since C++11). In a class: per-class, not per-object member.

**Q26. What happens during compilation of a C++ program?**
Preprocess (headers/macros) → compile each TU to object file → link objects + libraries resolving symbols → executable. Headers are textual inclusion; templates instantiate at compile time.

**Q27. Lambda captures — by value vs reference danger?**
`[=]` copies, `[&]` references. Returning/storing a `[&]` lambda beyond the scope of captured locals = dangling reference. For async work capture by value or `shared_ptr`.

**Q28. What is `std::string_view`?**
Non-owning view (pointer + length) over character data — cheap to pass, no allocation. Danger: dangling if the underlying string dies; never return a view of a local.

**Q29. Struct vs class in C++?**
Only default access differs: struct public, class private. Convention: struct for passive data, class for invariant-holding types.

**Q30. What's new/important in modern C++ (17/20)?**
17: structured bindings, `optional`/`variant`, `string_view`, parallel algorithms. 20: concepts, ranges, coroutines, modules, `span`, three-way comparison. Modern C++ ≈ value semantics + RAII + algorithms, not raw new/delete.

---

## B. Rust Core (Ch 2) — Q31–Q60

**Q31. Explain ownership in one minute.**
Every value has exactly one owner; when the owner goes out of scope the value is dropped. Assignment/passing *moves* ownership unless the type is `Copy`. The compiler enforces this — memory safety without GC.

**Q32. The borrowing rules?**
Any number of shared refs (`&T`) XOR exactly one mutable ref (`&mut T`), and references must never outlive the data. Enforced at compile time; eliminates data races and iterator invalidation.

**Q33. What are lifetimes?**
Names for the scopes references are valid for. Usually inferred (elision); explicit annotations (`fn f<'a>(x: &'a str) -> &'a str`) tell the compiler how output lifetimes relate to inputs. They change nothing at runtime.

**Q34. `String` vs `&str`?**
`String`: owned, growable, heap-allocated. `&str`: borrowed slice view into string data. APIs take `&str` for flexibility; return `String` when producing new data.

**Q35. `Box`, `Rc`, `Arc`, `RefCell` — when?**
`Box<T>`: single-owner heap allocation (recursive types, trait objects). `Rc<T>`: shared ownership, single-thread. `Arc<T>`: atomic refcount, multi-thread. `RefCell<T>`: interior mutability, borrow rules checked at *runtime* (panics on violation).

**Q36. What is `Copy` vs `Clone`?**
`Copy`: implicit bitwise copy, only for trivial types (ints, bools, &T). `Clone`: explicit `.clone()`, may be expensive (String, Vec). Moves are the default for non-Copy types.

**Q37. `Result` vs `Option`?**
`Option<T>`: presence/absence (Some/None). `Result<T, E>`: success/failure with an error value. Both force handling at compile time — no forgotten error checks or null derefs.

**Q38. What does the `?` operator do?**
On `Err`/`None` it returns early from the function, converting the error via `From`; on `Ok`/`Some` it unwraps. Ergonomic error propagation.

**Q39. When is `panic!` appropriate vs `Result`?**
Panic for unrecoverable programmer errors (violated invariants, impossible states) and tests. Result for anything expected to fail: I/O, parsing, network. Libraries should return Result and let callers decide.

**Q40. Traits vs interfaces/inheritance?**
Traits define shared behavior; types implement them. No data inheritance — composition instead. Traits allow default methods, generics bounds (`T: Display`), and can be implemented for existing types (coherence rules permitting).

**Q41. Static vs dynamic dispatch in Rust?**
`fn f(x: impl Trait)` / generics: monomorphized per type, zero cost, bigger binary. `dyn Trait` (trait object): vtable pointer, one code copy, needed for heterogeneous collections.

**Q42. What is `Send` and `Sync`?**
Marker traits, auto-derived: `Send` = safe to move to another thread; `Sync` = safe to share `&T` across threads. The compiler uses them to make data races *impossible* to compile. `Rc` is neither; `Arc<Mutex<T>>` is both.

**Q43. How does `match` differ from switch?**
Pattern matching: destructures enums/structs/tuples, binds variables, guards — and is **exhaustive**: the compiler errors if a variant is unhandled. This makes adding enum variants safe.

**Q44. What are enums in Rust really?**
Full algebraic data types — each variant can carry data (`enum Shape { Circle(f64), Rect{w: f64, h: f64} }`). `Option` and `Result` are just enums. With match exhaustiveness, they model states so illegal states are unrepresentable.

**Q45. Shadowing vs mutability?**
`let x = 5; let x = x + 1;` creates a *new* binding (can change type); `let mut x` allows mutation of one binding. Shadowing is idiomatic for transformation pipelines (parse then validate).

**Q46. What are closures and their three traits?**
Anonymous functions capturing the environment. `FnOnce` (consumes captures), `FnMut` (mutates), `Fn` (reads). The compiler infers the minimal one; `move` forces ownership capture — needed when spawning threads.

**Q47. Iterators in Rust — cost?**
Lazy adapters (`map/filter/take`) fused into a single loop by the optimizer — "zero-cost abstraction": usually compiles to the same asm as a hand loop, sometimes better (no bounds checks).

**Q48. What is `unsafe` Rust and what does it allow?**
A block where you may: deref raw pointers, call unsafe/extern fns, mutate statics, implement unsafe traits, access union fields. It does NOT turn off the borrow checker for references — it's a promise the *programmer* upholds the invariants. Keep unsafe blocks tiny and documented.

**Q49. How does Rust prevent data races?**
Race = aliasing + mutation + no synchronization. Borrow rules ban aliasing+mutation; crossing threads requires Send/Sync; shared mutation must go through sync types (`Mutex`, atomics). Non-compliant code doesn't compile.

**Q50. What is Cargo and crates.io?**
Cargo: build system + package manager — `Cargo.toml` declares deps with semver, `Cargo.lock` pins them; `cargo build/test/doc/publish`. crates.io is the registry. Workspaces share one lockfile across crates.

**Q51. What does `#[derive(...)]` do?**
Auto-implements traits via macros: `Debug`, `Clone`, `PartialEq`, `Serialize`… Cuts boilerplate; custom derives are procedural macros.

**Q52. Error handling in real apps — thiserror vs anyhow?**
Libraries: `thiserror` — typed error enums callers can match on. Applications: `anyhow` — one dynamic error type with context (`.context("reading config")`). Common pattern: thiserror at boundaries, anyhow in main.

**Q53. What is async/await in Rust?**
`async fn` returns a `Future` — an inert state machine; an executor (tokio) polls it. `.await` yields until ready — thousands of tasks on few threads. Key difference from threads: cooperative, so don't block the executor (use `spawn_blocking`).

**Q54. `tokio::spawn` vs `std::thread::spawn`?**
tokio: green task on the async runtime, cheap (KBs), must be `Send + 'static`, for I/O-bound work. Thread: OS thread (MBs stack), for CPU-bound or blocking work.

**Q55. What is interior mutability?**
Mutating through a shared reference via controlled types: `Cell` (copy in/out), `RefCell` (runtime borrow check), `Mutex`/`RwLock` (thread-safe), atomics. Moves borrow-check from compile time to runtime where needed.

**Q56. Why no null in Rust?**
`Option<T>` replaces null: absence is explicit in the type, the compiler forces handling before use. Billion-dollar mistake fixed by the type system; `Option<&T>` even compiles to a plain nullable pointer (niche optimization).

**Q57. What is monomorphization?**
Generic functions are compiled once per concrete type used — like C++ templates. Zero runtime cost; the price is compile time and binary size.

**Q58. Explain `Deref` and deref coercion.**
`Deref` lets smart pointers act like `&T`: `&String → &str`, `&Box<T> → &T` automatically at call sites. Why `fn f(s: &str)` accepts `&String`.

**Q59. What are the borrow checker's most common beginner fights and fixes?**
(1) Use-after-move → borrow or clone. (2) Mutable + immutable borrow overlap → shrink borrow scopes (NLL helps). (3) Returning refs to locals → return owned. (4) Self-referencing structs → indices or `Rc`.

**Q60. Why Rust over C++ for a new backend service? (JD favorite)**
Memory/thread safety at compile time (whole bug classes gone), modern tooling (cargo, rustfmt, clippy out of the box), fearless concurrency, C-level performance. C++ still wins for existing ecosystems/ABI and some domains — interop via FFI is solid, so incremental adoption works.

---

## C. Memory & Performance (Ch 3) — Q61–Q80

**Q61. What is a memory leak and how do you find it?**
Allocation never freed while unreachable/unused — RSS grows over time. Find: ASan `detect_leaks`, Valgrind memcheck, heaptrack; in Rust, leaks are rare (drop-based) but possible via `Rc` cycles or `mem::forget`.

**Q62. Use-after-free — what and why dangerous?**
Accessing freed memory. The allocator may have reused it → silent corruption, security exploits. C++: ASan catches it; Rust: borrow checker rejects it at compile time.

**Q63. What is a dangling pointer/reference?**
Points to destroyed data — freed heap block or dead stack frame. Classic: returning address of a local. UB when dereferenced.

**Q64. Stack overflow — causes?**
Unbounded/deep recursion, huge stack arrays. Stacks are ~1–8 MB per thread. Fix: iteration, heap allocation, explicit stack data structure.

**Q65. What is memory fragmentation?**
Free memory split into unusably small non-contiguous chunks (external) or wasted padding within blocks (internal). Mitigate: pools/arenas for same-size objects, fewer long-lived mixed-size allocations.

**Q66. How does `malloc` work at a high level?**
Manages free lists/bins of freed chunks; small allocations from arenas (per-thread caches in tcmalloc/jemalloc), large ones via `mmap`. Syscalls (`brk`/`mmap`) only when the heap must grow.

**Q67. What is cache locality and why care?**
CPU loads 64-byte cache lines; sequential access is prefetched. RAM is ~100× slower than L1. Arrays beat pointer-chasing structures — why vector beats list in practice regardless of big-O.

**Q68. False sharing?**
Two threads write different variables on the *same* cache line — the line ping-pongs between cores, serializing them. Fix: pad/align per-thread data to 64 bytes (`alignas(64)`, `#[repr(align(64))]`).

**Q69. What is alignment and padding?**
Types must sit at addresses divisible by their alignment; compilers insert padding between struct fields. Order fields large→small to shrink structs; Rust reorders automatically unless `#[repr(C)]`.

**Q70. Copy vs move — cost difference?**
Copy duplicates the buffer: O(n) allocation + memcpy. Move steals the pointer: O(1), three word writes. Why `std::move`/Rust moves matter for containers of strings/vectors.

**Q71. What is an arena/pool allocator?**
Preallocate a big block, hand out chunks by pointer bump, free everything at once. No fragmentation, no per-free cost, great cache locality — pattern for per-request or per-frame data.

**Q72. Zero-copy — what does it mean?**
Avoiding buffer duplication: `string_view`/`&[u8]` slices instead of substring copies, `sendfile`/`mmap` to skip kernel↔user copies, serde borrowing deserialization. Big win in parsers and network paths.

**Q73. RAII vs garbage collection trade-offs?**
RAII: deterministic, immediate, works for *all* resources (files, locks), no pauses; requires ownership discipline. GC: handles cycles, easier sharing; nondeterministic finalization, pauses, memory-only.

**Q74. `shared_ptr` overhead — why prefer `unique_ptr`?**
Control block allocation, atomic refcount on every copy (contended cacheline in MT), possible cycles. Ownership is usually a tree, not a graph — unique_ptr is free.

**Q75. What is memory-mapped I/O (`mmap`)?**
Map a file into the address space — page faults load pages on demand; the page cache is shared across processes. Great for large read-mostly files; no read() copies.

**Q76. Big-O vs real performance — when does big-O mislead?**
Constants and caches: linear scan of a 100-element vector beats a hash map; O(log n) tree with pointer chasing loses to O(n) contiguous scan for small n. Measure (Ch 14.5).

**Q77. What is NUMA (mention-level)?**
Multi-socket machines: memory attached to one CPU is slower from the other. Pin threads near their data for latency-critical services.

**Q78. How do you reduce allocations in a hot loop?**
Reuse buffers (`clear()` not new), `reserve()` up front, small-string/inline optimizations, arenas, avoid temporary String/vector in formatting — profile allocations with heaptrack / `#[global_allocator]` counters.

**Q79. Rust's ownership vs C++ smart pointers — the key difference?**
Same ideas (unique/shared ownership, RAII) but Rust *enforces* them: use-after-move is a compile error, aliasing rules are checked. C++ trusts the programmer — one raw pointer escape and guarantees are gone.

**Q80. What is `mem::swap` / placement — why no self-referential structs in Rust?**
Rust values must be safely movable via memcpy; a struct holding a pointer into itself breaks when moved. Async futures do this internally — hence `Pin`, which guarantees an object won't move again.

---

## D. Concurrency (Ch 4) — Q81–Q105

**Q81. Process vs thread?**
Process: own address space, isolated, IPC to communicate. Thread: shares the process's memory, cheap to create/switch, communicates via shared memory — hence synchronization.

**Q82. What is a race condition vs a data race?**
Data race: unsynchronized concurrent access, ≥1 write — UB in C++/Rust. Race condition: broader — correctness depends on timing/ordering (check-then-act gaps), possible even with data-race-free code.

**Q83. What is a mutex and how does it work?**
Mutual exclusion lock: one holder at a time; others block (futex: spin briefly, then sleep in kernel). Protects invariants across a critical section. Always pair with the *data* it guards (Rust: `Mutex<T>` enforces this).

**Q84. Deadlock — conditions and prevention?**
Four Coffman conditions: mutual exclusion, hold-and-wait, no preemption, circular wait. Prevent: global lock ordering, acquire-all-at-once (`std::scoped_lock`), timeouts, or design away shared state.

**Q85. Mutex vs spinlock?**
Mutex sleeps the thread when contended (syscall cost, no CPU burn). Spinlock busy-waits (no syscall, burns CPU) — only for very short critical sections, mostly kernel/embedded.

**Q86. `RwLock` — when and its risk?**
Many readers or one writer. Wins when reads vastly outnumber writes and read sections are long; risks writer starvation and is slower than Mutex under write-heavy load. Measure.

**Q87. What is a condition variable?**
Wait for a predicate while releasing the mutex: `cv.wait(lock, []{ return !q.empty(); })`. Always re-check the predicate (spurious wakeups). Pair: producer pushes then `notify_one`.

**Q88. What are atomics?**
Lock-free indivisible operations on single variables (load/store/fetch_add/CAS) with memory-ordering guarantees. For counters and flags; anything with multi-variable invariants needs a lock.

**Q89. Memory ordering — relaxed vs acquire/release vs seq_cst?**
Relaxed: atomicity only, no ordering — counters. Acquire/release: release-store publishes prior writes to the acquire-loader — the handoff pattern. seq_cst: single total order, default, safest, slightly slower. When unsure, seq_cst.

**Q90. What is compare-and-swap (CAS)?**
Atomically: if value == expected, write desired; else load actual. The primitive under lock-free structures and spinlocks; used in retry loops (beware ABA).

**Q91. Livelock and starvation?**
Livelock: threads keep acting/retrying but no one progresses (mutual backoff loops). Starvation: some thread never gets the resource (unfair locks, writer vs readers). Fix: fairness, backoff with jitter.

**Q92. Producer-consumer — how do you build it?**
Bounded queue + mutex + two condvars (not-empty, not-full), or a channel (Rust `mpsc`/crossbeam; C++ TBB). Bounded = backpressure — unbounded queues just move the failure to OOM.

**Q93. What is a thread pool and why?**
Fixed workers pulling from a task queue: amortizes thread creation, caps concurrency (≈ cores for CPU-bound), provides backpressure. Rust: rayon (data parallel), tokio (async I/O).

**Q94. Async vs threads — when each?**
Async: massive I/O-bound concurrency (10k connections) — tasks are cheap, cooperative. Threads: CPU-bound parallelism, blocking libraries, simplicity. Mixing: async runtime + `spawn_blocking`/worker pool.

**Q95. What happens if you block inside an async task?**
You stall the executor thread — every task scheduled on it stops progressing (executor starvation). Use async I/O, or offload with `tokio::task::spawn_blocking`.

**Q96. Rust: why does `thread::spawn` require `move` and `'static`?**
The thread may outlive the caller's stack frame, so it can't borrow locals — it must own its captures (or share via `Arc`). `'static` bound enforces this at compile time.

**Q97. `Arc<Mutex<T>>` — explain the layering.**
`Mutex<T>` gives exclusive access; `Arc` shares ownership of that mutex across threads. Arc handles *lifetime*, Mutex handles *access*. Clone the Arc per thread; lock to touch T.

**Q98. What are channels and why prefer them?**
Typed message queues transferring *ownership* of data between threads ("share memory by communicating"). No locks in user code, natural pipelines, backpressure with bounded variants.

**Q99. C++ `std::async` vs `std::thread`?**
`async` returns a `future` for a result and may pool (launch policy); `thread` is a raw OS thread you must join and returns nothing (use promise/future or `std::jthread` C++20 with stop tokens).

**Q100. What is thread-local storage?**
Per-thread variable instances (`thread_local`) — no sharing, no locks. Used for caches, RNGs, error states (errno).

**Q101. How many threads should a CPU-bound pool have?**
≈ hardware cores (`hardware_concurrency`). More adds context-switch overhead without throughput. I/O-bound: more, or better, async.

**Q102. What is a semaphore vs a mutex?**
Semaphore: counter permitting N concurrent holders, no ownership (any thread may release) — resource limiting. Mutex: binary, owner-must-release — invariant protection.

**Q103. What is a memory barrier/fence?**
An instruction preventing CPU/compiler reordering across it — what acquire/release orderings compile down to. Rarely written by hand; use atomics' ordering parameters.

**Q104. Lock-free vs wait-free (mention-level)?**
Lock-free: some thread always progresses (CAS retry loops). Wait-free: every thread progresses in bounded steps. Hard to get right — use proven libraries (crossbeam) and justify why a Mutex isn't enough first.

**Q105. How would you find a data race in production code?**
C++: ThreadSanitizer on the test suite (races are detected even without crashes). Rust: safe code can't have them — audit `unsafe` and FFI. Symptoms: heisenbugs varying with load/timing.

---

## E. OS & Linux (Ch 5–6) — Q106–Q135

**Q106. What does the OS scheduler do?**
Multiplexes runnable threads over cores using priorities/timeslices (Linux CFS/EEVDF: fair share of CPU). Preemption via timer interrupts; context switches save/restore registers + switch address space.

**Q107. What is a context switch and its cost?**
Saving one thread's CPU state, loading another's. Direct: µs-level register/kernel work; indirect (dominant): cold caches and TLB. Why thread-per-request doesn't scale to 100k.

**Q108. User mode vs kernel mode?**
CPU privilege levels: user code can't touch hardware/other memory; syscalls (via trap) switch to kernel mode for privileged work. Protects the system from buggy/hostile processes.

**Q109. What is a system call — trace `read()`.**
`read(fd,...)` → libc wrapper → `syscall` instruction traps into the kernel → VFS → filesystem/page cache (maybe block I/O, process sleeps) → data copied to user buffer → return. `strace` shows the sequence.

**Q110. What is virtual memory?**
Per-process address space mapped to physical frames via page tables (+TLB caching). Enables isolation, overcommit, mmap, swapping, copy-on-write. A page fault triggers the kernel to map/load a page.

**Q111. Page fault types?**
Minor: page in memory, mapping missing (first touch after mmap/fork) — fast. Major: must read from disk (swap/file) — slow. Segfault: invalid address — SIGSEGV.

**Q112. What is copy-on-write?**
`fork()` shares pages read-only; a write faults and copies just that page. Makes fork cheap; also used by `Cow` types conceptually.

**Q113. `fork()` vs `exec()`?**
fork: clone current process (child returns 0). exec: replace current process image with a new program, keeping PID/fds. Shells do fork → exec; `posix_spawn` combines them.

**Q114. What is a zombie process? An orphan?**
Zombie: exited child whose parent hasn't `wait()`ed — just an exit-status entry; fix the parent. Orphan: parent died — child reparented to init/systemd (harmless).

**Q115. What are signals? Handling SIGTERM vs SIGKILL?**
Async notifications (kill, Ctrl-C=SIGINT). SIGTERM is catchable → graceful shutdown (finish requests, flush, exit); SIGKILL can't be caught — the process is destroyed. Docker stop sends TERM, waits, then KILL — handle TERM!

**Q116. What is an inode?**
Filesystem metadata record: owner, permissions, size, timestamps, block pointers — everything except the *name*. Names live in directory entries; hard links = multiple names, one inode.

**Q117. Hard link vs symlink?**
Hard: another directory entry for the same inode — same file, survives original's deletion, same fs only. Symlink: a small file containing a path — can dangle, cross filesystems.

**Q118. File permissions rwxrwxrwx — explain 754.**
Owner/group/other triads: 7=rwx, 5=r-x, 4=r--. `chmod 754 f`; `chown user:group f`. Execute on a directory = permission to traverse it.

**Q119. What is a file descriptor?**
Small integer indexing the process's open-file table (0=stdin,1=stdout,2=stderr). Sockets, pipes, files all uniform — everything is an fd; redirect with `2>&1`, inspect with `lsof`.

**Q120. Pipes — what does `cmd1 | cmd2` do?**
Kernel creates a pipe buffer; cmd1's stdout fd is replaced by the write end, cmd2's stdin by the read end; both run concurrently with backpressure (writer blocks when full).

**Q121. How do you find what's eating CPU / memory / disk?**
CPU: `top/htop`, then `perf top`. Memory: `free -h`, `ps aux --sort=-rss`, smaps for detail. Disk: `df -h` (space), `iostat -x` (I/O load), `du -sh *` (usage), `iotop` (per-process).

**Q122. How do you check which process listens on port 8080?**
`ss -tlnp | grep 8080` (or `lsof -i :8080`).

**Q123. Explain `grep`, `awk`, `sed` one-liners you actually use.**
`grep -rn "ERROR" logs/` — recursive search with line numbers. `awk '{print $2}'` — extract column. `sed -i 's/old/new/g' f` — in-place replace. Combine: `grep ERROR app.log | awk '{print $5}' | sort | uniq -c | sort -rn` — top error codes.

**Q124. Write a bash loop over files with error handling.**
```bash
#!/usr/bin/env bash
set -euo pipefail                      # exit on error, unset vars, pipe failures
for f in /data/*.csv; do
    [[ -e "$f" ]] || { echo "no files" >&2; exit 1; }
    process "$f" || echo "failed: $f" >&2
done
```
Always quote variables; `set -euo pipefail` is the professional signature.

**Q125. What is `$?`, `$#`, `$@`?**
`$?` last exit code (0=success), `$#` arg count, `$@` all args (quote as `"$@"` to preserve spacing). Also `$$` PID, `$1..$n` positional args.

**Q126. cron — schedule a job at 2 AM daily.**
`0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1` (min hour dom mon dow). Modern alternative: systemd timers (logging, dependencies, catch-up runs).

**Q127. What is systemd? Write a minimal service unit.**
Init system managing services/dependencies/logging.
```ini
[Unit]
Description=RPM collector
After=network-online.target
[Service]
ExecStart=/opt/app/collector
Restart=on-failure
User=appuser
[Install]
WantedBy=multi-user.target
```
`systemctl enable --now collector`, logs via `journalctl -u collector -f`.

**Q128. What is the OOM killer?**
When memory is exhausted, the kernel kills the process with the worst badness score (usually the biggest). Check `dmesg | grep -i oom`. In containers, exceeding the cgroup limit OOM-kills *your* process — set limits realistically.

**Q129. What is swap and is it bad?**
Disk-backed overflow for RAM pages. Light swapping of idle pages is fine; sustained swapping (thrashing) destroys latency. Watch `vmstat` si/so.

**Q130. `nice`, `ionice`, cgroups?**
nice: CPU priority hint (-20..19). ionice: disk I/O priority. cgroups: hard resource *limits* (CPU/mem/IO) per process group — the mechanism under Docker.

**Q131. How does SSH public-key auth work?**
Client proves possession of the private key by signing a server-provided challenge; server verifies against `~/.ssh/authorized_keys`. No password crosses the wire; key files never leave the client.

**Q132. What is `/proc`?**
Virtual filesystem exposing kernel/process state as files: `/proc/<pid>/status` (memory), `/proc/<pid>/fd` (open fds), `/proc/cpuinfo`, `/proc/meminfo`. What `ps`/`top` read.

**Q133. Environment variables vs config files?**
Env vars: per-process, easy per-environment overrides, the 12-factor default (and Docker-native); no comments/structure. Files: structured, versioned; risk of env drift. Common: file + env overrides.

**Q134. What happens when you type `ls` in bash?**
Shell parses, expands globs/vars, finds `ls` via `$PATH` (hash cache), fork() → child exec()s `/bin/ls` with fds inherited, parent wait()s, prints prompt with `$?` set.

**Q135. What is `ulimit`?**
Per-process resource limits: open fds (`-n`, classic cause of "too many open files" under load), core size (`-c`), stack (`-s`). Raise fd limit for servers.

---

## F. Systems Programming & Networking (Ch 7) — Q136–Q150

**Q136. TCP vs UDP?**
TCP: connection, reliable ordered byte stream, flow/congestion control — APIs, DBs. UDP: connectionless datagrams, no guarantees, low latency — metrics, discovery, real-time. Know that Kafka/HTTP ride on TCP.

**Q137. The TCP three-way handshake?**
SYN → SYN-ACK → ACK: exchanges initial sequence numbers, establishes both directions. Teardown: FIN/ACK pairs, TIME_WAIT on the closer.

**Q138. What is a socket, lifecycle server-side?**
`socket() → bind(addr:port) → listen(backlog) → accept()` per client → `read/write` → `close`. Each accepted connection is a new fd.

**Q139. What does "address already in use" mean and the fix?**
Previous socket in TIME_WAIT still owns the port. `setsockopt(SO_REUSEADDR)` before bind — standard for servers.

**Q140. Blocking vs non-blocking I/O; what is epoll?**
Blocking: read() sleeps until data. Non-blocking + epoll: register many fds, kernel reports which are ready — one thread handles thousands of connections. The engine under tokio/nginx/Node.

**Q141. What is io_uring (mention-level)?**
Modern Linux async I/O: submission/completion ring buffers shared with the kernel — fewer syscalls than epoll, covers file I/O too.

**Q142. Little-endian vs big-endian — why care?**
Byte order of multi-byte ints. Network protocols use big-endian ("network order") — convert with hton/ntoh (or Rust `u32::to_be_bytes`). Bugs appear when serializing structs raw across machines.

**Q143. How do processes communicate (IPC) — options and trade-offs?**
Pipes (parent-child streams), Unix domain sockets (general, fast, fd-passing), shared memory + semaphores (fastest, needs sync discipline), message queues, signals (events only), TCP (works across hosts). Default: Unix sockets; shared memory when copying is the bottleneck.

**Q144. What is serialization; compare JSON vs Protobuf.**
Converting structures to bytes. JSON: human-readable, self-describing, slower/bigger. Protobuf: binary, schema'd, fast/compact, versioned fields — inter-service APIs. Rust: serde makes both one `#[derive]`.

**Q145. What is a memory-mapped file used for in IPC?**
Two processes map the same file/shm: shared memory region — zero-copy data exchange; synchronize with semaphores or lock-free rings. Pattern for high-rate sensor/telemetry fan-out.

**Q146. What happens on `write()` to a TCP socket — is delivery guaranteed?**
Data is copied to the kernel send buffer; success ≠ delivered — TCP retransmits in the background, but the peer may crash. Application-level acks are needed for business-level guarantees (ties into Kafka acks, Ch 10).

**Q147. What is Nagle's algorithm / TCP_NODELAY?**
Nagle batches small writes to reduce packets — adds latency to request/response protocols. Latency-sensitive services set TCP_NODELAY.

**Q148. How does DNS resolution work (short)?**
App → stub resolver → recursive resolver → root → TLD → authoritative; answers cached with TTLs. `dig +trace` shows the chain. Failures/slowness here masquerade as "network is down".

**Q149. What is TLS in one paragraph?**
Handshake: negotiate versions/ciphers, server proves identity via certificate chain to a trusted CA, key exchange (ECDHE, forward secrecy) derives symmetric session keys; then all traffic is encrypted+authenticated. HTTPS = HTTP over TLS.

**Q150. Cross-compilation — how does it work for Rust/C++?**
Build on x86 targeting e.g. ARM: `cargo build --target aarch64-unknown-linux-gnu` + matching linker/sysroot; C++ via toolchain files in CMake. Relevant for industrial/embedded targets; Docker buildx does multi-arch images.

---

## G. APIs & Web (Ch 8) — Q151–Q165

**Q151. What makes an API RESTful?**
Resources addressed by URLs, manipulated with uniform HTTP verbs, stateless requests, representations (JSON), status codes for outcomes. Not a protocol — an architectural style.

**Q152. HTTP verbs and idempotency?**
GET (read, safe), PUT (replace, idempotent), DELETE (idempotent), POST (create/act, NOT idempotent), PATCH (partial update). Idempotent = same effect if retried — decides what clients may safely retry.

**Q153. Status codes you must know?**
200 OK, 201 Created, 204 No Content, 400 Bad Request, 401 Unauthenticated, 403 Forbidden, 404 Not Found, 409 Conflict, 422 Unprocessable, 429 Too Many Requests, 500 Internal, 503 Unavailable.

**Q154. Design endpoints for machines and their readings.**
`GET /api/v1/machines`, `POST /api/v1/machines`, `GET /machines/{id}`, `PATCH /machines/{id}`, `GET /machines/{id}/readings?from=&to=&limit=` — nouns, plural, hierarchy for ownership, filtering via query params, version in the path.

**Q155. How do you version an API?**
URL path `/v1/` (visible, cache-friendly, most common) or headers. Rules: additive changes don't need a version; breaking changes (removing/renaming fields) do; support N and N-1 during migration.

**Q156. Pagination approaches?**
Offset (`?page=2&size=50`): simple, degrades on deep pages, unstable under inserts. Cursor/keyset (`?after=id123`): stable, fast (indexed WHERE >), no random page jumps. APIs at scale use cursors.

**Q157. Authentication: session vs JWT vs API keys?**
Sessions: server-side state, easy revocation. JWT: signed self-contained claims, stateless, revocation is hard (short TTL + refresh tokens). API keys: service-to-service simplicity. Always over TLS.

**Q158. What is SOAP and when would you still see it?**
XML protocol with WSDL contracts, WS-Security, strict typing — legacy enterprise/industrial integrations (relevant to machinery vendors!). Verbose but contract-first and tooling-rich. Know: envelope/header/body, operations not resources.

**Q159. REST vs WebSocket — when each?**
REST: request/response, cacheable, stateless CRUD. WebSocket: persistent bidirectional connection — server push, live dashboards (machine RPM!), chat. Pattern: REST for commands/history, WS for live updates.

**Q160. What is the WebSocket handshake?**
HTTP GET with `Upgrade: websocket` + key headers → server replies 101 Switching Protocols → same TCP connection becomes a framed message channel.

**Q161. How do you keep WebSocket connections healthy at scale?**
Ping/pong heartbeats to detect dead peers, reconnect with backoff + resume tokens client-side, sticky sessions or a pub/sub backplane (Redis/Kafka) so any node can serve any client.

**Q162. What is CORS?**
Browser policy: JS may call other origins only if the server opts in via `Access-Control-Allow-Origin` (+ preflight OPTIONS for non-simple requests). Server-side config, not a client fix.

**Q163. Rate limiting — algorithms?**
Token bucket (allows bursts, steady refill — most common), leaky bucket (smooth output), fixed/sliding window counters. Return 429 + Retry-After. Implement at gateway; per-key/per-IP.

**Q164. What is OpenAPI/Swagger?**
Machine-readable API contract (YAML/JSON): endpoints, schemas, auth. Generates docs, clients, servers, validation — contract-first development; the thing Postman/codegen consume.

**Q165. How should a good error response body look?**
Consistent envelope: machine code, human message, correlation/request id, optionally field-level details. Never leak stack traces; log the details server-side keyed by the id.

---

## H. Databases (Ch 9) — Q166–Q185

**Q166. What is ACID?**
Atomicity (all-or-nothing), Consistency (constraints hold), Isolation (concurrent txns don't see partial states), Durability (committed survives crash — WAL/fsync).

**Q167. SQL vs NoSQL — when each?**
Relational: structured data, relationships, transactions, ad-hoc queries — the default. Document (MongoDB): flexible/nested schemas, horizontal scale-out. Choose by data shape, consistency needs, and query patterns — not fashion.

**Q168. How does a B-tree index work and what does it cost?**
Sorted balanced tree: O(log n) lookups and range scans. Costs: extra writes per INSERT/UPDATE, space. Index columns in WHERE/JOIN/ORDER BY; composite index order matters (leftmost prefix).

**Q169. Why is `SELECT *` discouraged?**
Fetches unneeded columns (I/O, network), breaks covering-index-only scans, and couples code to schema changes. Select what you use.

**Q170. What is an execution plan / EXPLAIN?**
The optimizer's strategy: scan types (seq vs index), join algorithms, row estimates. `EXPLAIN ANALYZE` runs it with real numbers — first tool for any slow query.

**Q171. The N+1 query problem?**
One query for a list, then one per item for details — 1+N round trips. Fix: JOIN, `IN` batch, or ORM eager loading. Detect via query logs.

**Q172. Isolation levels and their anomalies?**
Read uncommitted (dirty reads) → read committed (no dirty; non-repeatable reads possible — Postgres default) → repeatable read (stable rows; phantoms in some DBs) → serializable (as-if-sequential; retry on conflict).

**Q173. Optimistic vs pessimistic locking?**
Pessimistic: `SELECT ... FOR UPDATE` — hold row locks, contention. Optimistic: version column, `UPDATE ... WHERE version = n` — retry on conflict; wins when conflicts are rare.

**Q174. What is a transaction deadlock in the DB and its handling?**
Two txns lock rows in opposite order; the DB detects the cycle and aborts one (error 40P01 in PG). App must catch and retry; prevent by consistent access order and short transactions.

**Q175. Normalization vs denormalization?**
Normalize (3NF) to remove duplication/update anomalies — default for OLTP. Denormalize deliberately for read performance (precomputed aggregates, embedded documents) accepting update complexity.

**Q176. What is a foreign key and why use (or skip) it?**
Referential integrity constraint enforced by the DB — bad data can't get in. Skipped sometimes at extreme write scale/sharding; then integrity moves to app code (risky).

**Q177. Prepared statements — why?**
Plan cached, parameters sent separately: performance + **SQL injection immunity**. Never build SQL by string concatenation with user input.

**Q178. What is connection pooling?**
Connections are expensive (handshake, auth, server memory); a pool reuses N of them across requests. Size ≈ small multiple of DB cores — not thousands. Rust: sqlx/deadpool; extern: PgBouncer.

**Q179. MongoDB documents — model machines + readings.**
Embed what you read together (machine config as one doc); reference high-volume series (readings in their own collection keyed by machine_id + timestamp, or a time-series collection). Rule: embed 1-to-few, reference 1-to-millions.

**Q180. What is MVCC?**
Multi-Version Concurrency Control (Postgres): writers create new row versions instead of overwriting; readers see a snapshot — readers never block writers. Old versions cleaned by VACUUM.

**Q181. WAL — what and why?**
Write-Ahead Log: changes are appended to a sequential log and fsynced before data pages update — crash recovery replays it. Also the basis of replication. (Same design idea as Kafka's log!)

**Q182. Replication: sync vs async; what is a read replica?**
Async: primary confirms before replicas apply — fast, possible loss/lag. Sync: waits for replica ack — durable, slower. Read replicas offload SELECTs; beware replication lag (read-your-writes issues).

**Q183. What is sharding?**
Horizontal partitioning across nodes by a shard key (hash or range). Scales writes/storage; costs: cross-shard queries/transactions get hard, resharding is painful. Choose keys with even distribution (machine_id, not date).

**Q184. Postgres vs MySQL — one-liner each?**
Postgres: richest SQL/features (JSONB, window functions, extensions like TimescaleDB), strict correctness. MySQL/InnoDB: ubiquitous, great read-heavy performance, simpler. For sensor/time-series + complex queries, Postgres is my default.

**Q185. How would you store high-rate sensor readings?**
Append-only narrow table `(machine_id, ts, metric, value)` partitioned by time; batch inserts (COPY / multi-row); index (machine_id, ts); aggressive retention + downsampled rollup tables; TimescaleDB/hypertables if allowed.

---

## I. Messaging: Kafka & RabbitMQ (Ch 10) — Q186–Q200

**Q186. Why use a message broker at all?**
Decouples producers/consumers in time and availability, absorbs bursts (buffering/backpressure), fan-out to many consumers, retries/DLQs — turns tight synchronous coupling into resilient async flow.

**Q187. Kafka in three sentences.**
Distributed append-only log: topics split into partitions, each an ordered immutable sequence of records with offsets. Producers append; consumers read at their own pace tracking offsets. Retention is time/size-based, not consumption-based — many consumers replay the same data independently.

**Q188. What do partitions give you and what do they cost?**
Parallelism (one consumer per partition per group) and scalability. Costs: ordering only *within* a partition; partition count changes reshuffle key mappings.

**Q189. How does Kafka decide which partition a message goes to?**
By key hash (same key → same partition → ordered per key, e.g. per machine_id); round-robin/sticky if no key.

**Q190. What is a consumer group?**
Consumers sharing a group id split partitions among themselves; each partition is consumed by exactly one member. Scale by adding consumers up to the partition count. Different groups each get the full stream.

**Q191. At-most-once vs at-least-once vs exactly-once?**
Commit offset before processing = at-most-once (may lose). Process then commit = at-least-once (may duplicate — the practical default; make handlers **idempotent**). Exactly-once: Kafka transactions/EOS within Kafka-to-Kafka pipelines; end-to-end still needs idempotent sinks.

**Q192. What are producer acks?**
acks=0 fire-and-forget; acks=1 leader only; acks=all leader + in-sync replicas — durable, use with `enable.idempotence=true` to avoid retry duplicates.

**Q193. What is consumer lag and what do you do about it?**
Latest offset minus committed offset — how far behind consumers are. Monitor it; fix by scaling consumers (≤ partitions), speeding handlers (batching), or increasing partitions.

**Q194. RabbitMQ model in three sentences.**
Producers publish to **exchanges**; exchanges route to **queues** by bindings/routing keys (direct, topic, fanout, headers). Consumers ack individual messages; unacked messages are redelivered. Messages are deleted once consumed — it's a smart post office, not a log.

**Q195. Kafka vs RabbitMQ — when each?**
Kafka: high-throughput event streams, replay, multiple independent readers, ordering per key — telemetry/event sourcing. RabbitMQ: task queues, complex routing, per-message acks/priorities, lower setup weight — background jobs, RPC-ish work distribution.

**Q196. What is a dead-letter queue?**
Destination for messages that repeatedly fail processing (or expire) — prevents poison messages from blocking the queue; inspect, fix, replay. Both brokers support the pattern.

**Q197. How do you handle a poison message in Kafka?**
Can't skip in-place per message like Rabbit; pattern: try N times → produce to a DLQ topic → commit offset and move on. Alert on DLQ rate.

**Q198. Ordering guarantees — how do you get strict per-entity ordering?**
Key by entity id (machine_id) so all its events hit one partition; single consumer per partition; idempotent handler because at-least-once still redelivers.

**Q199. What is backpressure in a messaging system?**
Consumers control pull rate (Kafka: poll loop; Rabbit: prefetch/QoS count) so they're not overwhelmed; producers see buffering/limits. Design queues bounded end-to-end or memory becomes the failure mode.

**Q200. Design: 10k machines each sending 10 readings/sec — sketch the pipeline.**
Producers → Kafka topic `readings` keyed by machine_id, ~50–100 partitions, acks=all; consumer group of stateless workers batching inserts into partitioned Postgres/Timescale; second consumer group feeds live WebSocket dashboards; DLQ topic + lag monitoring + idempotent writes (upsert on machine_id+ts).

---

## J. Docker, Testing, SDLC (Ch 11–13) — Q201–Q215

**Q201. Container vs VM?**
Container: shares host kernel, isolated via namespaces + cgroups — MBs, ms startup. VM: full guest OS on a hypervisor — GBs, stronger isolation. Containers package apps; VMs partition hardware.

**Q202. Image vs container?**
Image: immutable layered filesystem + metadata (the class). Container: a running (or stopped) instance with a writable layer (the object).

**Q203. What is a multi-stage Dockerfile and why?**
Build in a heavy toolchain image, COPY the binary into a slim runtime image (`debian-slim`/`distroless`). Result: small attack surface, fast pulls — essential for Rust/C++ (build deps are huge).

**Q204. Why does layer order matter in a Dockerfile?**
Each instruction is a cached layer; a change invalidates everything after. Order stable → volatile: deps manifest + install first, source code last — rebuilds reuse the dependency layer.

**Q205. How do containers talk to each other?**
User-defined bridge network: containers resolve each other by name (built-in DNS). Compose does this automatically — `db:5432` from the app container. Publish ports (`-p`) only for the outside world.

**Q206. Volumes vs bind mounts?**
Volume: Docker-managed named storage — data outliving containers (DB data). Bind mount: host path mapped in — dev-time source syncing. Container's own writable layer is throwaway.

**Q207. How do you debug a container that won't start?**
`docker logs <c>` first; `docker inspect` for config/exit code (137 = OOM/SIGKILL); override entrypoint `docker run -it --entrypoint sh image` to poke around; check env vars and file permissions.

**Q208. What belongs in `.dockerignore`?**
`target/`, `node_modules/`, `.git`, secrets, local configs — smaller build context, faster builds, no leaked credentials.

**Q209. How do you handle secrets with Docker?**
Never bake into images or Dockerfiles; inject at runtime: env vars from a secret store, Docker/Compose secrets, or mounted files. Rotate; scan images.

**Q210. Health checks — why and how?**
`HEALTHCHECK` (or orchestrator probes) runs a liveness command/endpoint; orchestrators restart unhealthy containers and gate traffic on readiness. Expose `/healthz` that checks critical deps shallowly.

**Q211. Test pyramid in 30 seconds?**
Many fast unit tests, fewer integration tests (real DB/broker in containers), few E2E. Push logic down the pyramid; the higher, the slower and flakier. (Details Ch 12.)

**Q212. How do you test a Kafka consumer?**
Extract handler logic → unit test it pure. Integration: testcontainers Kafka, produce test records, assert side effects (DB rows), including duplicate delivery (idempotency test).

**Q213. What is CI and what belongs in the pipeline?**
Every push: build, format check (rustfmt/clang-format), lint (clippy/clang-tidy), unit + integration tests, sanitizers for C++, image build. Merge is blocked on green — main is always releasable.

**Q214. Sprint ceremonies in 30 seconds?**
Planning (commit to sprint backlog), daily standup (sync/blockers), review (demo to stakeholders), retro (improve process). PO owns priorities; the team owns estimates and the *how*. (Details Ch 13.)

**Q215. Merge vs rebase — 20-second version?**
Merge preserves history with a merge commit; rebase rewrites yours on top for linear history. Rebase local/private branches only; never rewrite shared history; PR merge into main.

---

## Drill plan

| Pass | What | Goal |
|---|---|---|
| 1 | Read all, mark unknowns | Inventory gaps |
| 2 | Deep-dive marked chapters | Close gaps |
| 3 | Verbal answers, cover text | Fluency |
| 4 (final week) | Random 30/day + Ch 16 exercises | Speed under pressure |

> Next: [Chapter 16 — Coding Exercises & System Design](16-exercises.md) to practice *applying* these under interview conditions.
