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
---

# C++ Function Wrappers

## Jan Wilczek (thinkcell)

Berlin, Sep 28, 2026

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

---

# Motivating Example: Logger

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
```cpp
using Logger = void (*)(std::string_view);

void doStuff(Logger log) {
    // do stuff
    log("Stuff done.");
}

doStuff([](std::string_view str) {
    std::println("Message: {}", str);
});
```
```cpp {all|8-11}
using Logger = void (*)(std::string_view);

void doStuff(Logger log) {
    // do stuff
    log("Stuff done.");
}

doStuff([i = 0](std::string_view str) mutable {
    std::println("Message {}: {}", i, str);
    ++i;
}); // ❌
```
````

---

# `std::function`

```cpp
void doStuff(std::function<void(std::string_view)> log) {
    // do stuff
    log("Stuff done.");
}

doStuff([i = 0](std::string_view str) mutable {
    std::println("Message {}: {}", i, str);
    ++i;
});
```

---

# `std::function`

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
doStuff(logger);
```
```cpp
void doStuff(const std::function<void(std::string_view)>& log) {
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
````

---

# `std::function`: Problem 1

https://godbolt.org/z/vKsd8cPe3

````md magic-move
```cpp
std::function<void(void)> f = [i = 0] {
    std::println("i={}", i);
};
const auto& fref = f;
fref(); // will this compile?
```
```cpp
std::function<void(void)> f = [i = 0] {
    std::println("i={}", i);
};
const auto& fref = f;
fref(); // ✅
```
```cpp
std::function<void(void)> f = [i = 0] mutable {
    ++i;
    std::println("i={}", i);
};
const auto& fref = f;
fref(); // will this compile?
```
```cpp
std::function<void(void)> f = [i = 0] mutable {
    ++i;
    std::println("i={}", i);
};
const auto& fref = f;
fref(); // i=1 ⚠️
```
```cpp
std::move_only_function<void(void)> f = [i = 0] mutable {
    ++i;
    std::println("i={}", i);
};
const auto& fref = f;
fref(); // ❌
```
```cpp
std::move_only_function<void(void)> f = [i = 0] mutable {
    ++i;
    std::println("i={}", i);
};
const auto& fref = f;
fref(); // ❌
const auto g = f; // ❌
```
```cpp
std::copyable_function<void(void)> f = [i = 0] mutable {
    ++i;
    std::println("i={}", i);
};
const auto& fref = f;
fref(); // ❌
const auto g = f; // ✅
```
````

<!-- copyable_function is still not available in MSVC -->

---

# "Awesome" logger

````md magic-move
```cpp {all|7-10|1-5|12|12-13}
namespace awe {
struct AwesomeLogger {
    void log(std::string_view) & { /* ... */ }
};
}

void doStuff(const std::function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

auto logger = std::make_unique<awe::AwesomeLogger>();
doStuff(?);
```
```cpp {12-15}
namespace awe {
struct AwesomeLogger {
    void log(std::string_view) & { /* ... */ }
};
}

void doStuff(const std::function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

auto logger = [logger = std::make_unique<awe::AwesomeLogger>()] (std::string_view str) {
    logger->log(str);
};
doStuff(logger);
```
```cpp {12-15|7-15}
namespace awe {
struct AwesomeLogger {
    void log(std::string_view) & { /* ... */ }
};
}

void doStuff(const std::function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

auto logger = [logger = std::make_unique<awe::AwesomeLogger>()] (std::string_view str) {
    logger->log(str);
};
doStuff(logger); // ❌
```
```cpp {7-15}
namespace awe {
struct AwesomeLogger {
    void log(std::string_view) & { /* ... */ }
};
}

void doStuff(const std::move_only_function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

auto logger = [logger = std::make_unique<awe::AwesomeLogger>()] (std::string_view str) {
    logger->log(str);
};
doStuff(logger); // ✅
```
````

---

# Multithreaded logger

````
```cpp {all|6-9}
void doStuff(const std::move_only_function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

auto logger = [logger = std::make_unique<awe::AwesomeLogger>()] (std::string_view str) {
    logger->log(str);
};
doStuff(logger);
```
```cpp {6-10}
void doStuff(const std::move_only_function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

LoggerQueue queue{std::make_unique<awe::AwesomeLogger>()};
auto logger = [&queue] (std::string_view str) {
    queue.push(str);
};
doStuff(logger);
```
```cpp {6-10}
void doStuff(const std::move_only_function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

LoggerQueue queue{std::make_unique<awe::AwesomeLogger>()};
auto logger = [&queue, mutex = std::mutex{}] (std::string_view str) {
    std::lock_guard lock{mutex};
    queue.push(str);
};
doStuff(logger);
```
```cpp {6-10|all}
void doStuff(const std::move_only_function<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

LoggerQueue queue{std::make_unique<awe::AwesomeLogger>()};
auto logger = [&queue, mutex = std::mutex{}] (std::string_view str) {
    std::lock_guard lock{mutex};
    queue.push(str);
};
doStuff(logger); // ❌
```
```cpp {6-10}
void doStuff(const std::function_ref<void(std::string_view)>& log) {
    // do stuff
    log("Stuff done.");
}

LoggerQueue queue{std::make_unique<awe::AwesomeLogger>()};
auto logger = [&queue, mutex = std::mutex{}] (std::string_view str) {
    std::lock_guard lock{mutex};
    queue.push(str);
};
doStuff(logger); // ✅
```
```cpp {6-10}
void doStuff(std::function_ref<void(std::string_view)> log) {
    // do stuff
    log("Stuff done.");
}

LoggerQueue queue{std::make_unique<awe::AwesomeLogger>()};
auto logger = [&queue, mutex = std::mutex{}] (std::string_view str) {
    std::lock_guard lock{mutex};
    queue.push(str);
};
doStuff(logger); // ✅
```
````

---

# `std::function_ref`

- Non-owning callable wrapper
- Does for functions the same job as `std::string_view` for `std::string`

---

# Composite logger

````md magic-move
```cpp
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
        : m_logger(std::move(logger)) {}

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
        : m_logger(std::move(logger)) {}
//...
private:
    std::function_ref<void(std::string_view)> m_logger;
    int m_i;
};

CountingLogger logger{[](std::string_view str) {
    std::println("{}", str);
}};
doStuff(logger);
```
```cpp {all|10-12}
struct CountingLogger {
    CountingLogger(std::function_ref<void(std::string_view)> logger)
        : m_logger(std::move(logger)) {}
//...
private:
    std::function_ref<void(std::string_view)> m_logger;
    int m_i;
};

CountingLogger logger{[](std::string_view str) {
    std::println("{}", str);
}};
doStuff(logger); // disaster 💀
```
````

---

# `tc::function_ref`



---

# `std::function`: Problem 2

https://godbolt.org/z/arhbo6x3c

````md magic-move
```cpp
std::function<void(void)> f = [i = 0] {
    std::println("i={}", i);
};
const auto& fref = f;
fref(); // ✅
```
```cpp
std::move_only_function<void(void)> f = [i = 0] {
    std::println("i={}", i);
};
const auto& fref = f;
fref(); // ❌
```
```cpp
std::copyable_function<void(void)> f = [i = 0] {
    std::println("i={}", i);
};
const auto& fref = f;
fref(); // ❌
```
```cpp
std::copyable_function<void(void) const> f = [i = 0] {
    std::println("i={}", i);
};
const auto& fref = f;
fref(); // ✅
```
````

---

# `std::function`: Problem 3

````md magic-move
```cpp
std::function<void(void)> f = [i = 0] noexcept { // ✅
    std::println("i={}", i);
};
```
```cpp
std::move_only_function<void(void)> f = [i = 0] noexcept { // ❌
    std::println("i={}", i);
};
```
```cpp
std::copyable_function<void(void)> f = [i = 0] noexcept { // ❌
    std::println("i={}", i);
};
```
```cpp
std::copyable_function<void(void) noexcept> f = [i = 0] noexcept { // ✅
    std::println("i={}", i);
};
```
````

---

# References & and &&
---
layout: center
---

# Further differences from `std::function`

---

# It does not have the target_type and target accessors (direction requested by users and implementors).

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
f(); // throws despite noexcept
f = [] noexcept { /* ... */ };
```

</v-click>

---
layout: center
---

> I guess Java/C#/Go wasn't getting enough backend projects and Rust wasn't getting enough low level work. Gotta take the opportunity to make C++ just a little more unsafe and make sure even the most up-to-date C++ still has a mountain of gotchas baked in. Don't worry; I'm sure there will be a safety profile for that later. (Yeah right.)

> The language is too big to die, but not for lack of trying. The C++ language development strategy at this is to pretty much fiddle while Rome burns basically.

---

# Conversions?


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
- `std::copyable_function` = `std::move_only_function` + 
    - copy constructor
    - copy assignment operator
    - callables must be copy-constructible
- avoid `std::function`
- never call an empty function wrapper (UB)

</v-clicks>

<!-- function_ref is for a callable as string_view for string. std::copyable_function = std::function v2 -->

