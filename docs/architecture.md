# Engine Architecture

## NativeAOT Specifics
!!! warning "Important"
    This application is compiled with **C# NativeAOT**. While the disassembly thus far resembles C++, the underlying structures should (theoretically) represent .NET objects compiled to native code.

* **MethodTables:** What appears to be C++ VTable assignment (`*ptr = &DAT_...`) is likely(?) MethodTable Initialization. The pointer is the Type Handle for the C# class.
* **Interop Strings:** The use of `wchar_t` (`L"string"`) indicates the engine uses Pinned Memory or Interop strings for performance, rather than managed `System.String` objects.

## Threading Model
The engine relies on **Thread Local Storage (TLS)** to avoid passing context pointers explicitly.

* A per-thread context object is lazily created and retrieved via TLS.
* Major systems access this context at function entry.

## Synchronization (SpinLocks)
Instead of standard OS Mutexes, the engine uses custom **hybrid locks** (spin-then-yield) for critical initialization sections.  
**Pattern** `GlobalInitGuard_Enter` (`FUN_1413ef880`):
```c
if (Locked) {
    while (Locked) {
        Sleep(1); // Yields CPU
    }
}
```

## Memory Management (Arenas)
!!! warning "Speculative"  
!!! note "Design Pattern"
    The engine appears to bypass the Managed Heap (GC), likely using `Unsafe.AsPointer` or `NativeMemory`.  
**Pattern** `ThreadArena_Alloc` (`FUN_1413f0590`):

* Attempts fast bump from TLS buffer.
* Falls back to heap if capacity exceeded.
* Allocated objects are registered via `RegisterTrackedPointer` (`FUN_1413f0bc0`).

## Subsystem Registration
!!! warning "Speculative"
Subsystems are manually registered into global tables during initialization (originated from CRT startup hooks).  
**Pattern:**

* Allows ordered construction and phased startup/shutdown.
* Systems are not true singletons but shared or per-session instances. 
    * They are registered via `RegisterTrackedPointer` (`FUN_1413f0bc0`) to specific offsets in the Session object.
* Registration order in client mode is fixed.
