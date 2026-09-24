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

