# 🦀 The Rust Developer Fast Track

> You already write Rust. This is your custom path through the book: what to read fully, what to skim, and how to handle the C++ requirement **without studying C++ from scratch** — by mapping it onto what you already know.

## First, the honest part: can you fully skip C++?

**No — and yes.** The JD lists C++ as a must-have, so C++ questions *will* come. What you can skip is learning C++ the long way. Almost every C++ interview question is about a concept Rust forced you to master already — ownership, lifetimes, moves, RAII, dispatch. You don't need to *learn* these; you need to learn **what C++ calls them and where C++ differs**. That's a 1–2 day job, not a 3-week one, and Section 3 below is exactly that.

Your interview positioning (say this if asked directly): *"I'm strongest in Rust; I'm fluent in C++ concepts and can read and write modern C++, and the concepts transfer directly — I can show you."* Then prove it with the mappings below. Honesty + demonstrated transfer beats faking depth you don't have.

## 1. Your reading plan (what to read / skim / skip)

| Chapter | Rust dev action | Why |
|---|---|---|
| 1 — C++ | **Replace with Section 3 below**, then skim Ch 1 once to see the C++ syntax | You need vocabulary + differences, not fundamentals |
| 2 — Rust | **Skim hard** — read only the Interview Q&A and any section you can't explain out loud | Verify, don't relearn |
| 3 — Memory | **Read fully** | Asked language-neutrally; cache/false-sharing/arenas apply to Rust too |
| 4 — Concurrency | **Read fully** | Mutex/atomics/orderings questions are language-neutral; know the C++ names (`std::mutex`, `condition_variable`) |
| 5–7 — OS, Linux, Systems | **Read fully** | Nothing Rust-specific to skip; JD must-have |
| 8–11 — APIs, DBs, Messaging, Docker | **Read fully** | JD must-haves, language-agnostic |
| 12 — Testing | Read fully (Rust examples already) | Skim the GoogleTest section — recognize the macros |
| 13 — SDLC/Git | Read fully | Language-agnostic |
| 14 — Debugging | Read fully; don't skip GDB/ASan | **Trap:** Rust devs skip sanitizers "because borrow checker" — but the JD has C++, and GDB works on Rust binaries too |
| 15 — Questions | Drill sections **B–J fully**; drill section A (C++) *using Section 3 of this guide first* | |
| 16 — Exercises | Do all Rust ones; **read** the C++ ones (B1, B2, C1) until you can narrate them | Writing perfect C++ live is unlikely to be required at Rust-first positioning; explaining it will be |
| 17 — Behavioral | Read fully | |

## 2. Adjusted 3-week plan (replaces the README's 4-week gantt)

| Week | Focus |
|---|---|
| 1 | Section 3 of this guide (C++ via Rust) + Ch 3–4 (memory, concurrency) + skim Ch 1–2 Q&As |
| 2 | Ch 5–11 (OS → Docker) — the breadth week |
| 3 | Ch 12–14 + all of Part 5: drill Ch 15, timed Ch 16 exercises, Ch 17 stories |

## 3. The C++ Survival Kit — C++ explained through Rust

This is the section that replaces "learning C++". Work through it with Ch 1 open beside it for syntax.

### 3.1 The big mental switch

One sentence to internalize: **C++ has the same ownership model as Rust — but as a *convention* the programmer must uphold, not a rule the compiler enforces.** Every C++ "best practice" below is the manual version of something rustc does for you. When you frame answers this way, you sound like someone who understands *both* languages deeply.

### 3.2 Concept map — you already know 90% of C++

| You know (Rust) | C++ calls it | The difference to mention |
|---|---|---|
| Ownership + `Drop` | **RAII** + destructors | Same idea; C++ destructors run deterministically at scope exit too. RAII is THE word to use — it predates Rust |
| `Box<T>` | `std::unique_ptr<T>` | Same: sole owner, zero overhead, move-only. `make_unique` ≈ `Box::new` |
| `Arc<T>` | `std::shared_ptr<T>` | Same atomic refcount. C++ has no `Rc` split — `shared_ptr` is always atomic |
| `Weak<T>` | `std::weak_ptr<T>` | Identical purpose: break refcount cycles; `w.lock()` ≈ `upgrade()` |
| Move semantics (default!) | Move semantics via `std::move` + `T&&` | **Key difference:** Rust moves are the default and invalidate the source (compile error to reuse). C++ moves are opt-in casts, and the moved-from object is "valid but unspecified" — still usable, source of bugs |
| `&T` / `&mut T` with borrow checker | `const T&` / `T&` — **no checker** | C++ references can dangle; nothing stops aliasing+mutation. This is *the* difference — say "C++ trusts the programmer; Rust proves it at compile time" |
| Lifetimes (`'a`) | No equivalent — programmer discipline | Dangling refs are UB found by ASan/code review, not the compiler |
| `dyn Trait` + vtable | `virtual` functions + vtable | Same mechanism. C++: vptr lives *inside* the object; Rust: fat pointer carries the vtable pointer |
| Traits | Abstract classes (pure virtual = required method) | C++ interfaces are classes with `virtual ... = 0`. No trait coherence; also has *implementation* inheritance which Rust deliberately dropped |
| Generics + monomorphization | **Templates** | Same zero-cost codegen. Rust checks bounds at definition (traits); C++ checks at instantiation (ugly errors) — C++20 concepts ≈ trait bounds |
| `match` + enums with data | `std::variant` + `std::visit` (clunky) | C++ enums are bare integers; sum types are bolted on. A real gap — mention it as why you like Rust |
| `Option<T>` | `std::optional<T>` | No `?`, no exhaustive match — `*opt` on empty is UB, not a panic |
| `Result<T, E>` | Exceptions (traditional) / `std::expected` (C++23) | Errors are invisible in C++ signatures; RAII is what makes exceptions safe. Know: catch by `const&`, destructors never throw |
| `Mutex<T>` (owns the data) | `std::mutex` + `std::lock_guard` (beside the data) | C++ mutex doesn't wrap data — nothing stops you touching data without locking. `lock_guard`/`scoped_lock` ≈ the RAII guard you get from `.lock()` |
| `Send`/`Sync` markers | No equivalent | C++ data races are found by TSan at runtime, not the compiler |
| `String` / `&str` | `std::string` / `std::string_view` | Same owned/borrowed split; `string_view` can dangle silently |
| `Vec<T>` / `HashMap` | `std::vector` / `std::unordered_map` | Same. Same iterator-invalidation-on-realloc trap — but in C++ it compiles and corrupts memory |
| `cargo` | CMake + a package manager (vcpkg/Conan) | Fragmented; genuinely miss cargo. Safe thing to say out loud |
| `unsafe` blocks | The entire language 🙂 | Glib but true — refine it: "modern C++ with smart pointers + sanitizers narrows the gap by convention" |

### 3.3 The C++-only topics with no Rust twin (learn these cold — ~2 hours)

These have no Rust equivalent, and they're classic questions:

1. **Rule of Three/Five/Zero** — if you write a destructor, you must handle copy ctor + copy assign (+ move pair). Rust needs no rule: `Drop` + moves compose automatically. *(Ch 1.7 + Q4 in Ch 15.)*
2. **Virtual destructors** — deleting a derived object via base pointer with non-virtual dtor is UB. No Rust analog (no inheritance). Memorize: *"base classes need virtual destructors."*
3. **Object slicing** — assigning `Derived` to `Base` *by value* silently chops off the derived part. Impossible in Rust (no subtype assignment).
4. **The diamond problem** — two bases sharing an ancestor duplicate it; fixed with `virtual` inheritance. Rust avoided it by not having data inheritance.
5. **`const` correctness** — `const` methods, `const T&` params, read pointer types right-to-left. It's `&self` vs `&mut self`, but opt-in and less powerful.
6. **Headers, translation units, ODR** — C++ compiles each `.cpp` separately and textually includes headers; the linker joins them. No modules-by-default like Rust crates. Explains: include guards, "why is my build slow", linker errors.
7. **Copy elision / RVO** — the compiler builds return values in-place, skipping copies. (Rust does this too but nobody quizzes it; in C++ it's a named interview topic.)
8. **`new[]`/`delete[]` pairing**, `malloc` vs `new` — mismatches are UB. Rust: `Box`/`Vec` make this unrepresentable.

### 3.4 Answering C++ questions live — the formula

**"Correct C++ answer first → Rust contrast second."** The contrast is a bonus that shows depth — but only after you've answered the actual question.

> **Q: "What's the difference between `unique_ptr` and `shared_ptr`?"**
> "`unique_ptr` is sole ownership — zero overhead, move-only, the default choice. `shared_ptr` adds a control block with an atomic refcount, so copies share ownership and the object dies when the count hits zero; cycles leak, which `weak_ptr` breaks. *They map exactly to `Box` and `Arc` in Rust — same decision tree, and it's the model I use daily.*"

> **Q: "What happens after `std::move(x)`?"**
> "`std::move` is just a cast to rvalue reference — it moves nothing itself. If a move constructor consumes it, `x` is left valid-but-unspecified: you may assign or destroy it, but not read it. *This is the one place Rust is stricter — using a moved-from value is a compile error there, and C++'s 'valid but unspecified' is a real bug source.*"

> **Q: "How do you prevent data races in C++?"**
> "Discipline plus tooling: guard shared data with `std::mutex` via `lock_guard`/`scoped_lock`, use `std::atomic` for simple flags/counters, establish lock ordering to avoid deadlock — and run ThreadSanitizer in CI, because the compiler won't catch violations. *Rust encodes those conventions in the type system with `Send`/`Sync` and `Mutex<T>` owning its data — same rules, machine-checked.*"

Rehearse all Ch 15 section A questions (Q1–Q30) this way: cover the answer, respond with the formula.

### 3.5 Minimum C++ you should be able to *write* on a whiteboard

Realistically you may be asked to sketch C++. Be able to write from memory:

```cpp
// 1. A RAII class with the Rule of Five shape (Ch 16 B2 has the full version)
class Buffer {
    char* data_ = nullptr;
    size_t size_ = 0;
public:
    explicit Buffer(size_t n) : data_(new char[n]), size_(n) {}
    ~Buffer() { delete[] data_; }
    Buffer(const Buffer&) = delete;              // or implement a deep copy
    Buffer& operator=(const Buffer&) = delete;
    Buffer(Buffer&& o) noexcept : data_(o.data_), size_(o.size_) {
        o.data_ = nullptr; o.size_ = 0;          // manual version of a Rust move
    }
};

// 2. Runtime polymorphism (a "trait object")
class Sensor {
public:
    virtual ~Sensor() = default;                 // ALWAYS virtual in a base
    virtual double read() const = 0;             // = 0 → like a required trait method
};
class RpmSensor : public Sensor {
public:
    double read() const override { return 15000.0; }
};

// 3. STL basics — the map/filter of C++
std::vector<int> v{5, 1, 4};
std::sort(v.begin(), v.end());
auto n = std::count_if(v.begin(), v.end(), [](int x) { return x > 3; });
std::unordered_map<std::string, int> counts;
for (const auto& w : words) ++counts[w];
```

And be able to **spot** the five bugs in Ch 16 B1 (dangling return, leak on early return, `delete` vs `delete[]`, non-virtual dtor, realloc invalidation) — spot-the-bug is the most common C++ screen for Rust-leaning candidates, and every one of those bugs is something the borrow checker rejects, which you can say.

### 3.6 Practice (half a day, do it once)

Type, compile, and break this on your machine — reading is not enough:

```bash
sudo apt install g++ # or: xcode-select --install on macOS
g++ -std=c++17 -Wall -Wextra -g -fsanitize=address main.cpp && ./a.out
```

1. Type the three snippets from 3.5 and make them compile.
2. Deliberately write a use-after-free (return a ref to a local, use it) — watch ASan catch at runtime what rustc catches at compile time. Now you have a *personal* story for "how does Rust improve on C++".
3. Do Ch 16 B2 (`UniquePtr` from scratch) once — it exercises every move-semantics rule in one exercise.

## 4. Where to *spend* the time you saved

The C++ shortcut buys you roughly a week. Invest it where Rust devs actually fail this interview:

1. **Breadth chapters (5–11).** The most common failure mode for language-strong candidates is fumbling "explain isolation levels" or "what's a consumer group". These are must-haves in this JD, and no amount of Rust fluency compensates.
2. **Ch 16 D1** (telemetry pipeline design) — do it twice, out loud, with a timer. It's practically this team's day job.
3. **Your Rust showcase.** Since Rust is your strength, prepare to go *deep* when invited: a 5-minute walkthrough of the most interesting Rust thing you've built — the design, an ownership problem you hit, how you tested it. When they hear real depth in Rust, the C++ bar drops accordingly.
4. **Async Rust specifically.** As the resident Rust expert you'll get the hard ones: pinning and why futures need it, `Send` bounds on spawned tasks, blocking-the-executor, cancellation safety. Ch 2 + Ch 15 Q53–Q55, plus C4 in Ch 16.

## 5. Fast-track final checklist

- [ ] Can answer all 30 C++ questions (Ch 15 section A) with the "C++ first, Rust contrast second" formula
- [ ] Can write the three 3.5 snippets from memory and spot all five B1 bugs
- [ ] Compiled and broke real C++ with ASan at least once (Section 3.6)
- [ ] Ch 15 sections B–J drilled to fluency
- [ ] Ch 16 D1 whiteboarded twice + all Rust exercises done under a timer
- [ ] Six behavioral stories written (Ch 17.2), one of them Rust-flavored
- [ ] One sentence rehearsed for "how's your C++?" — honest, confident, transfer-focused

> Bottom line: a Rust developer following this track needs ~3 weeks, spends 1–2 days on C++ instead of a week, and walks in with a *stronger* story than a generic C++ candidate — because "I know why these rules exist, my daily language enforces them" is a genuinely senior answer.
