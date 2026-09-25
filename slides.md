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
transition: slide-left
comark: true
lineNumbers: true
fonts:
  sans: Montserrat, Open Sans
  mono: CaskaydiaCove Nerd Font
  local: CaskaydiaCove Nerd Font
duration: 45min
---

# C++ Function Wrappers

Jan Wilczek (thinkcell)

---

# Motivating Example: Callbacks

```cpp
struct AppWindow {
    AppWindow() {
        m_button.onClick(/* code to trigger when clicked */);
    }

    Button m_button;
};

struct Button {
    void onClick(??? handler) {
        m_handler = std::move(handler);
    }

private:
    void handleClick() {
        m_handler();
    }

    ??? m_handler;
};
```

---



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
# It does not have the target_type and target accessors (direction requested by users and implementors).

---
# Invocation has strong preconditions.

---

# Conversions?



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

