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

# The Problem with `std::function`

https://godbolt.org/z/nTvcexTf5

```cpp
std::function<void(void)> f = [i = 0] mutable {
    ++i;
    std::println("i={}", i);
};
const auto& fref = f;
fref(); // will this compile?
```

---

