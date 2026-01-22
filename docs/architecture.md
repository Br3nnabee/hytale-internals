# Engine Architecture

## Object Instantiation Pattern
The decompilation confirms NativeAOT object creation patterns:

1.  **Allocation:** `RhpNewFast(MethodTable_Pointer)`
    * This allocates memory on the GC heap.
    * The `MethodTable_Pointer` (e.g., `&Class_141a4a200::vftable`) acts as the Runtime Type Handle.
2.  **Constructor:** `Function_Call(Object_Pointer, Args...)`
    * Constructors are standard function calls immediately following allocation.

## The "Host" Pattern
!!! warning "Speculative"
The startup sequence (`FUN_14064a950`) reveals a **Service Locator / Host** architecture.

* The game client does not own the platform directly.
* It looks up a "Host" object at `DAT_1427c23e0` (likely initialized during `Init_Global_Managers` phase).
* If this Host is missing, the client throws `ArgumentNullException` and crashes.

## Event-Driven Loop
!!! warning "Speculative"
Instead of a traditional `while(!CloseRequested)` loop visible in the main function, the engine uses an **Event Subscription** model:

1.  A "Closure" object is created to hold the game state.
2.  A Delegate is created pointing to a method on that closure.
3.  This delegate is passed to the underlying Windowing system (`FUN_140649e40`).
4.  The Windowing system calls this delegate every frame (Tick).
