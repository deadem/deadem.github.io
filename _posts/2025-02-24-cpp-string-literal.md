---
layout: post
title:  "C++, кастомные литералы"
date:   2025-02-22 00:00:00 -0400
categories: [cpp, code]
---

Периодически они случаются нужны, но надоело каждый раз гуглить и вспоминать, как писать правильно. Помечу-ка тут, чтоб не искать потом.

На примере свитча по строке:

```cpp
#include <cstdint>
#include <string>

constexpr uint64_t hash(std::string_view str) {
  uint64_t hash = 0;
  for (const char chr : str) {
    hash = (hash * 131) + chr;
  }
  return hash;
}

constexpr uint64_t operator"" _hash(const char* str, size_t len) {
  return hash(std::string_view(str, len));
}

int main() {
  const std::string test = "test";

  switch (hash(test)) {
    case "one"_hash: {
      return 1;
    }
    case "two"_hash: {
      return 2;
    }
    default:
      break;
  }

  return 3;
}
```
