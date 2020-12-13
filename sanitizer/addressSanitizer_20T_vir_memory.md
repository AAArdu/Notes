#### More details about 20T virtual memory in AddressSanitizer

When it comes to AddressSanitizer, 20T virtual memory is often memtioned, here we talk about more details about this number base on LLVM AddressSanitizer.

As the comment in compiler-rt/lib/asan/mapping.h, the memory layers is

```c
// Typical shadow mapping on Linux/x86_64 with SHADOW_OFFSET == 0x00007fff8000:
// || `[0x10007fff8000, 0x7fffffffffff]` || HighMem    ||
// || `[0x02008fff7000, 0x10007fff7fff]` || HighShadow ||
// || `[0x00008fff7000, 0x02008fff6fff]` || ShadowGap  ||
// || `[0x00007fff8000, 0x00008fff6fff]` || LowShadow  ||
// || `[0x000000000000, 0x00007fff7fff]` || LowMem     ||
```

At AddressSanitizer initialization phase (fucntion ***AsanInitInternal()*** in compiler-rt/lib/asan/asan_rtl.cpp), function call (***InitializeShadowMemory()*** (compiler-rt/lib/asan/asan_shaow_setup.cpp)) will allocate 16TB memory (HighShadow + ShadowGap + LowShadow) , using below code.

```c
 if (full_shadow_is_available) {
    // mmap the low shadow plus at least one page at the left.
    if (kLowShadowBeg)
      ReserveShadowMemoryRange(shadow_start, kLowShadowEnd, "low shadow");
    // mmap the high shadow.
    ReserveShadowMemoryRange(kHighShadowBeg, kHighShadowEnd, "high shadow");
    // protect the gap.
    ProtectGap(kShadowGapBeg, kShadowGapEnd - kShadowGapBeg + 1);
    CHECK_EQ(kShadowGapEnd, kHighShadowBeg - 1);
  }
```

Remaining 4T heap memory will be pre-allocated by sanitizer allocator (fucntion call ***InitializeAllocator()*** (compiler-rt/lib/asan/asan_allocator.h)), as the size config

```c
const uptr kAllocatorSpace = 0x600000000000ULL;
const uptr kAllocatorSize  =  0x40000000000ULL;  // 4T.
```
