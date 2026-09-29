---
theme: default
colorSchema: light
title: C++ Function Wrappers
titleTemplate: '%s - Jan Wilczek Berlin Meetup Sep 29, 2026'
info: |
  ## C++ Function Wrappers

  With C++ 26, we received two new standard function wrappers: `std::copyable_function` and `std::function_ref`. They complement C++ 23's `std::move_only_function`. But what about the good old `std::function`? When should we use each one? In this talk, we'll compare and contrast these four function wrappers and consider when to use each one in practice.
author: Jan Wilczek
export:
  format: pdf
  timeout: 30000
  dark: false
  withClicks: false
  withToc: false
class: text-center
drawings:
  persist: false
transition: none
comark: true
lineNumbers: true
fonts:
  sans: Montserrat, Open Sans
  mono: CaskaydiaCove Nerd Font
  local: CaskaydiaCove Nerd Font
duration: 45min
exportFilename: cpp-function-wrappers-talk-jan-wilczek
---

# C++ Function Wrappers

## Jan Wilczek (thinkcell)

Berlin, Sep 28, 2026

---

# Who am I?

<v-clicks>

- Jan Wilczek \[Yan Vil-check\]
- Founder of TheWolfSound.com & online course creator
- C++ Speaker at think-cell

</v-clicks>

---
layout: image
---

<img src="./assets/tc_homepage_hero.webp" class="w-full h-full"/>

---

<img src="./assets/Fast&Furious.jpeg"/>

---

# Fast & Furious

<div class="flex items-start gap-8">
  <div class="flex flex-col gap-6">
    <img v-click src="./assets/DomToretto.png" class="h-30 rounded-full shadow-xl"/>
    <img v-click src="./assets/LukeHobbs.png" class="h-30 rounded-full shadow-xl"/>
    <img v-click src="./assets/deckard-shaw.png" class="h-30 rounded-full shadow-xl"/>
  </div>
  <img v-click src="./assets/JakobToretto.png" class="h-30 rounded-full shadow-xl"/>
</div>

---

# Fast & `std::function`

<v-clicks>

- `std::function`
- `std::move_only_function` (C++23)
- `std::function_ref` (C++26)
- `std::copyable_function` (C++26)

</v-clicks>

<v-drag-arrow v-click pos="206,203,-35,-73"/>

---

# Contents

<v-clicks>

- Need for functions as first-class objects
    - const-correctness
    - copyability
    - movabilty
    - `noexcept`
- `std::function` & its problems
- `std::move_only_function` (C++23) & differences w.r.t. `std::function`
- `std::copyable_function` (C++26) & differences w.r.t. `std::move_only_function`
- `std::function_ref` (C++26) & differences
- Will `std::function` be deprecated?
- Pictures of muscled men

</v-clicks>

---

# Motivating Example: Logger

https://godbolt.org/z/MGW3bv4rr

````md magic-move
```cpp
void doStuff() {
    // do stuff
}
```
```cpp
void doStuff() {
    // do stuff
    std::println("Stuff done.");
}
```
```cpp {all|1|3-6|8-10|11}
using Logger = void (*)(std::string_view);

void doStuff(Logger log) {
    // do stuff
    log("Stuff done.");
}

auto logger = [](std::string_view str) {
    std::println("Message: {}", str);
};
doStuff(logger);
```
```cpp {8-11|12}
using Logger = void (*)(std::string_view);

void doStuff(Logger log) {
    // do stuff
    log("Stuff done.");
}

auto logger = [i = 0](std::string_view str) mutable {
    std::println("Message {}: {}", i, str);
    ++i;
};
doStuff(logger);
```
```cpp {12|all}
using Logger = void (*)(std::string_view);

void doStuff(Logger log) {
    // do stuff
    log("Stuff done.");
}

auto logger = [i = 0](std::string_view str) mutable {
    std::println("Message {}: {}", i, str);
    ++i;
};
doStuff(logger); // ❌
```
```cpp {all|1|10}
void doStuff(std::function<void(std::string_view)> log) {
    // do stuff
    log("Stuff done.");
}

auto logger = [i = 0](std::string_view str) mutable {
    std::println("Message {}: {}", i, str);
    ++i;
};
doStuff(logger); // ✅
```
````

---

# `std::function`

<img src="./assets/DomToretto.png" class="h-30 rounded-full shadow-xl"/>

<v-clicks>

- C++11
- pass callables without explicit types or templates
- functions as first-class objects
- dependency injection (Strategy)
- callbacks (Observer, GUI, threads, async operations)
- deferred execution
- algorithms
- ABI stability

</v-clicks>

---

# `std::function`

https://godbolt.org/z/nG7ME146d

````md magic-move
```cpp
void doStuff(std::function<void(std::string_view)> log) {
    // do stuff
    log("Stuff done.");
}

auto logger = [i = 0](std::string_view str) mutable {
    std::println("Message {}: {}", i, str);
    ++i;
};
doStuff(logger);
```
```cpp
void doStuff(std::function<void(std::string_view)> log) {
    // do stuff
    log("Stuff done.");
}

auto logger = [i = 0](std::string_view str) mutable {
    std::println("Message {}: {}", i, str);
    ++i;
};
doStuff(logger);
doStuff(logger);
```
```cpp {all|1|all}
void doStuff(std::function<void(std::string_view)> log) {
    // do stuff
    log("Stuff done.");
}

auto logger = [i = 0](std::string_view str) mutable {
    std::println("Message {}: {}", i, str);
    ++i;
};
doStuff(logger); // Message 0: Stuff done.
doStuff(logger); // Message 0: Stuff done.
```
```cpp {all|1|6-9|10-11}
void doStuff(const std::function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

auto logger = std::function{[i = 0](std::string_view str) mutable {
    std::println("Message {}: {}", i, str);
    ++i;
}};
doStuff(logger);
doStuff(logger);
```
```cpp {10-11|all|1,6,8}
void doStuff(const std::function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

auto logger = std::function{[i = 0](std::string_view str) mutable {
    std::println("Message {}: {}", i, str);
    ++i;
}};
doStuff(logger); // Message 0: Stuff done.
doStuff(logger); // Message 1: Stuff done.
```
````

---

# `std::function`: Problem 1 ("constness bug")

https://godbolt.org/z/vKsd8cPe3

````md magic-move
```cpp {all|1-4|5-6|all}
auto f = std::function{[i = 0] mutable {
    ++i;
    std::println("i={}", i);
}};
const auto& fref = f;
fref(); // i=1 ⚠️
```
```cpp
auto f = std::move_only_function{[i = 0] mutable {
    ++i;
    std::println("i={}", i);
}};
const auto& fref = f;
fref(); // ❌
```
````

[N4159](https://wg21.link/n4159)

<!-- copyable_function is still not available in MSVC -->

---

# "Awesome" logger

https://godbolt.org/z/xK5bK7vxo

````md magic-move
```cpp {all|7-10|1-5|12|3,7,12-13}
namespace awe {
struct AwesomeLogger {
    void log(std::string_view) & { /* ... */ }
};
}

void doStuff(std::function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

auto logger = std::make_unique<awe::AwesomeLogger>();
doStuff(/* ? */);
```
```cpp {7,12-16}
namespace awe {
struct AwesomeLogger {
    void log(std::string_view) & { /* ... */ }
};
}

void doStuff(std::function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

auto logger = std::function{
        [logger = std::make_unique<awe::AwesomeLogger>()] (std::string_view str) {
    logger->log(str);
}};
doStuff(logger);
```
```cpp {7,12-16}
namespace awe {
struct AwesomeLogger {
    void log(std::string_view) & { /* ... */ }
};
}

void doStuff(std::function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

auto logger = std::function{
        [logger = std::make_unique<awe::AwesomeLogger>()] (std::string_view str) {
    logger->log(str);
}}; // ❌
doStuff(logger);
```
```cpp {7,12-16}
namespace awe {
struct AwesomeLogger {
    void log(std::string_view) & { /* ... */ }
};
}

void doStuff(std::move_only_function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

auto logger = std::move_only_function<void(std::string_view)>>{
        [logger = std::make_unique<awe::AwesomeLogger>()] (std::string_view str) {
    logger->log(str);
}}; // ✅
doStuff(logger);
```
````

---

# `std::move_only_function`

<img src="./assets/LukeHobbs.png" class="h-30 rounded-full shadow-xl"/>

<v-clicks>

- C++23 ([P0288](https://wg21.link/P0288))
- non-copyable
- allows move-only callables
- allows copyable callables
- const-correct
- fixes a few more problems of `std::function`...
- Available in [github.com/think-cell/think-cell-library](https://github.com/think-cell/think-cell-library)

</v-clicks>

---

# `std::function`: Problem 2

https://godbolt.org/z/WzsfjGrev

````md magic-move
```cpp
auto f = std::function<void(void)>{[i = 0] {
    std::println("i={}", i);
}};
const auto& fref = f;
fref(); // ✅
```
```cpp
auto f = std::move_only_function<void(void)>{[i = 0] {
    std::println("i={}", i);
}};
const auto& fref = f;
fref(); // ❌
```
```cpp
auto f = std::move_only_function<void(void) const>{[i = 0] {
    std::println("i={}", i);
}};
const auto& fref = f;
fref(); // ✅
```
````

---

# `std::function`: Problem 3

https://godbolt.org/z/P5EhvsoMv

````md magic-move
```cpp
std::function<void(void)> get_f();
//...
auto f = get_f();
f(); // does this throw?
```
```cpp
std::move_only_function<void(void) noexcept> get_f();
//...
auto f = get_f();
f(); // if throws, program terminates
```
````

---

# `std::function`: Problem 3

https://godbolt.org/z/1zx8vKK6W

````md magic-move
```cpp
auto f = std::move_only_function<void(void) const noexcept>{[] {
    std::println("I don't throw");
}}; // ❌
```
```cpp
auto f = std::move_only_function<void(void) const noexcept>{[] noexcept {
    std::println("I don't throw");
}}; // ✅
```
````

---

# Limitation of `std::move_only_function`

````md magic-move
```cpp
auto f = std::move_only_function<void(void) const noexcept> {[i = 0] noexcept {
    std::println("i={}", i);
}};
const auto g = f; // ❌
```
```cpp
auto f = std::copyable_function<void(void) const noexcept>{[i = 0] noexcept {
    std::println("i={}", i);
}};
const auto g = f; // ✅
```
````

---

# `std::copyable_function`

<img src="./assets/JakobToretto.png" class="h-30 rounded-full shadow-xl"/>

$\iff$ `std::move_only_function` + 

<v-clicks>

- copy constructor
- copy assignment operator
- callables must be copy-constructible
- C++26 ([P2548](https://wg21.link/p2548))

</v-clicks>

---

# Thread-safe logger

https://godbolt.org/z/4YrE485sj

````md magic-move
```cpp {all|6-9}
void doStuff(std::move_only_function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

auto logger = std::move_only_function<void(std::string_view)>{
        [logger = std::make_unique<awe::AwesomeLogger>()] (std::string_view str) mutable {
    logger->log(str);
}};
doStuff(logger);
```
```cpp {6-10}
void doStuff(std::move_only_function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

auto logger = std::move_only_function<void(std::string_view)>{
        [logger = std::make_unique<awe::AwesomeLogger>(), mutex = std::mutex{}] (std::string_view str) mutable {
    std::lock_guard lock{mutex};
    logger->log(str);
}};
doStuff(logger);
```
```cpp {6-10}
void doStuff(std::move_only_function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

auto logger = std::move_only_function<void(std::string_view)>{
        [logger = std::make_unique<awe::AwesomeLogger>(), mutex = std::mutex{}] (std::string_view str) mutable {
    std::lock_guard lock{mutex};
    logger->log(str);
}}; // ❌
doStuff(logger);
```
```cpp {6-11|1}
void doStuff(std::move_only_function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

auto logger =
        [logger = std::make_unique<awe::AwesomeLogger>(), mutex = std::mutex{}] (std::string_view str) mutable {
    std::lock_guard lock{mutex};
    logger->log(str);
}; // ✅
doStuff(logger); // ❌
```
```cpp {1|all}
void doStuff(std::function_ref<void(std::string_view)> log) {
    // do stuff
    log("Stuff done.");
}

auto logger =
        [logger = std::make_unique<awe::AwesomeLogger>(), mutex = std::mutex{}] (std::string_view str) mutable {
    std::lock_guard lock{mutex};
    logger->log(str);
}; // ✅
doStuff(logger); // ✅
```
````

---

# `std::function_ref`

<img src="./assets/deckard-shaw.png" class="h-30 rounded-full shadow-xl"/>

<v-clicks>

- C++26 ([P0792](https://wg21.link/P0792))
- Non-owning callable wrapper
- Does for functions the same job as `std::string_view` for `std::string`
- Similar features to `std::move_only_function`
    - const correctness
    - `const`/`noexcept`/ref qualifiers
    - more
- Available in [github.com/think-cell/think-cell-library](https://github.com/think-cell/think-cell-library)

</v-clicks>

---

# Composite logger

https://godbolt.org/z/dxav63a7K

````md magic-move
```cpp {all|11-12|2-3|5-8|all}
struct CountingLogger {
    CountingLogger(std::function<void(std::string_view)> logger)
        : m_logger(std::move(logger)) {}

    void operator()(std::string_view str) {
        ++m_i;
        m_logger(str);
    }

private:
    std::function<void(std::string_view)> m_logger;
    int m_i;
};
```
```cpp
struct CountingLogger {
    CountingLogger(std::function_ref<void(std::string_view)> logger)
        : m_logger(logger) {}

    void operator()(std::string_view str) {
        ++m_i;
        m_logger(str);
    }

private:
    std::function_ref<void(std::string_view)> m_logger;
    int m_i;
};
```
```cpp
struct CountingLogger {
    CountingLogger(std::function_ref<void(std::string_view)> logger)
        : m_logger(logger) {}

    void operator()(std::string_view str) {
        ++m_i;
        m_logger(str);
    }

private:
    std::function_ref<void(std::string_view)> m_logger;
    int m_i;
};

auto logger = CountingLogger{[name = "foo"](std::string_view str) {
    std::println("[{}]: {}", name, str);
}};
doStuff(logger);
```
```cpp {all|11}
struct CountingLogger {
    CountingLogger(std::function_ref<void(std::string_view)> logger)
        : m_logger(logger) {}

    void operator()(std::string_view str) {
        ++m_i;
        m_logger(str);
    }

private:
    std::function_ref<void(std::string_view)> m_logger;
    int m_i;
};

auto logger = CountingLogger{[name = "foo"](std::string_view str) {
    std::println("[{}]: {}", name, str);
}};
doStuff(logger); // disaster 💀
```
````

---
layout: center
---

# Further differences from `std::function`

---

# Ref-qualifiers

https://godbolt.org/z/8KbMErccf

```cpp {all|1-2|3|4|5-6}
auto f = std::function<void(void)>{ // cannot have & or && within <>
    [] { std::println("Hello, Ref/Ref Ref/Val!"); }};
f();
std::move(f)();
std::function<void(void)>{
    [] { std::println("Hello, Ref/Ref Ref/Val!"); }}();
```

---

# Ref-qualifiers

https://godbolt.org/z/8KbMErccf

```cpp {1-3|5-7|9-12}
std::move_only_function<void(void) &>{[] {
        std::println("Hello, Ref!");
}}(); // ❌

auto f_ref = std::move_only_function<void(void) &>{
    [] { std::println("Hello, Ref!"); }};
f_ref();

auto f_ref_ref = std::move_only_function<void(void) &&>{
    [] { std::println("Hello, Ref Ref!"); }};
f_ref_ref(); // ❌
std::move(f_ref_ref)()
```

---

# No `target_type()` and `target()` accessors (requested by users and implementors)

https://godbolt.org/z/6rxGvfGEW

```cpp {1-5|6|8-10}
void foo() {
    std::println("Hello, Type!");
}

auto f = std::function<void(void)>{foo};
std::println("{}", f.target_type().name()); // prints "PFvvE"

if (void (*const* ptr)() = f.target<void(*)()>()) {
    std::println("Guessed it!");
}
```

---

# "Invocation has strong preconditions"

https://godbolt.org/z/8b5fMbaoe

```cpp {all|1|2-3|4}
std::function<void(void)>{}(); // std::bad_function_call
std::move_only_function<void(void)>{}(); // UB
std::copyable_function<void(void)>{}(); // UB
std::function_ref<void(void)>{}(); // ❌
```

---

# "Invocation has strong preconditions"

## Why? 🤔

<v-click>

https://www.reddit.com/r/cpp_questions/s/yxX2MXa4Yy

<img src="./assets/reddit1.png" class="h-100"/>

</v-click>

---

# "Invocation has strong preconditions"

## Why? 🤔

<v-click>

- Invoking an empty `std::function` is a bug

</v-click>
<v-click>

- Implementations can now assert on this

</v-click>

<v-click>

- "Don't pay for what you don't use"

</v-click>
<v-click>

- Exceptionless environments

</v-click>
<v-click>

- `noexcept` propagation
```cpp
std::throwing_move_only_function<void(void) noexcept> f;
f(); // throws despite noexcept because empty
f = [] noexcept { /* ... */ }; // assign
```

</v-click>

---
layout: center
---

<blockquote>
<p>I guess Java/C#/Go wasn't getting enough backend projects and Rust wasn't getting enough low level work. Gotta take the opportunity to make C++ just a little more unsafe and make sure even the most up-to-date C++ still has a mountain of gotchas baked in. Don't worry; I'm sure there will be a safety profile for that later. (Yeah right.)</p>

<br>

<p>The language is too big to die, but not for lack of trying.</p>

<span v-mark.highlight.yellow="1">The C++ language development strategy at this is to pretty much fiddle while Rome burns basically.</span>
</blockquote>

---

# Conversions


| From \ To | `function` | `move_only_function` | `copyable_function` | `function_ref` |
|---|:---:|:---:|:---:|:---:|
| `function` | | ✅ | ✅ | ✅ |
| `move_only_function` | ❌ | | ❌ | ✅ |
| `copyable_function` | ✅ | ✅ | | ✅ |
| `function_ref` | ✅ | ✅ | ✅ | |

---

# Summary: `std::function` problems

<v-clicks>

- can call a non-const callable via a const resf
- disallows move-only callables
- disallows non-movable, non-copyable callables
- does not propagate `const`, `noexcept`, `&`, or `&&` to `operator()`
- throws `std::bad_function_call`

</v-clicks>

---
layout: center
---

# Will `std::function` be deprecated?

---
layout: center
---

# Probably not 🙃

<v-click>

## But there is a proposal for it: [P2721](https://wg21.link/P2721)

</v-click>

---

# Summary

<v-clicks>

- function pointers and STL function wrappers are great solutions for dependency inversion and callbacks
- use `std::move_only_function` (C++23) if your function wrapper doesn't have to be copied or the callable cannot be copied
- use `std::function_ref` (C++26) if you don't need to store the function wrapper or the callable cannot be moved
- use `std::copyable_function` for a copyable function wrapper
- `std::copyable_function` $\iff$ `std::move_only_function` + 
    - copy constructor
    - copy assignment operator
    - callables must be copy-constructible
- avoid `std::function`
- never call an empty function wrapper (UB)
- check out think-cell-library: [github.com/think-cell/think-cell-library](https://github.com/think-cell/think-cell-library)

</v-clicks>

<img v-click v-drag="[644,251,271,271]" src="./assets/qr_code.png"/>

<!--
function_ref is for a callable as string_view for string. std::copyable_function = std::function v2
-->
