# Startup Sequence & Initialization

## 1. Native Entry (CRT & OEP)
The process begins at the standard Windows C Runtime (CRT) entry point.

* **Function:** `wmainCRTStartup`
    * **Action:** Calls `__security_init_cookie` (Stack protection).
    * **Action:** Calls `__scrt_common_main_seh` (CRT Initialization).
        * **Logic:** Initializes the C Runtime (Heap, Stdio).
        * **Handover:** Calls the NativeAOT Bootstrapper `FUN_1413df360`.

## 2. NativeAOT Runtime Bootstrapper
This section was previously misidentified as "Context Allocation". It is actually the **Redhawk (NativeAOT) Runtime** initializing itself.

* **Function:** `FUN_1413df360` (The Bootstrapper)
    * **Runtime Init:** Calls `RhInitialize(0)`.
        * *Purpose:* Initializes the Garbage Collector (GC) and suspended thread lists.
    * **Module Registration:** Calls `RhRegisterOSModule`.
        * *Purpose:* Registers the executable with the runtime so the GC can scan its stack and exceptions can be caught.
    * **Managed Entry:** Calls `__managed__Main`.

## 3. Managed Environment Setup (`__managed__Main`)
This is the bridge between C++ and C#.

* **Function:** `__managed__Main`
    * **Transition:** Calls `RhpReversePInvoke` to mark the thread as entering managed code.
    * **Pre-allocation:** Calls `RhpNewFast` to allocate the `OutOfMemoryException` singleton (address `DAT_141abaf30`) so it is ready if memory fails later.
    * **CoreLib Init:** Calls `S_P_CoreLib_..._ClassConstructorRunner__Initialize`.
    * **Args:** Calls `StartupCodeHelpers__InitializeCommandLineArgsW` to convert C++ `argv` to C# `string[]`.
    * **User Entry:** Calls `Noesis_Reference__Module___MainMethodWrapper`.

## 4. Application Main Logic (`FUN_14064a590`)
*Previously identified as "Engine_Main_Loop_Orchestrator".* This contains the high-level branching for the Hytale client.
!!! warning "Speculative: Implementation Detail"
    While the purpose is clearly a "Single Instance Check" (based on the error message), the underlying mechanism is assumed to be a `Mutex` or `Event` based on standard Windows patterns. The exact handle type hasn't been checked.

* **Function:** `FUN_14064a590`
    * **Mode Selection:** Checks global flag `DAT_1427af6d3`.
        * **If 0:** Sets string to "Game" (`dn_u_Game...`).
        * **If 1:** Sets string to "Editor" (`dn_u_Editor...`).
    * **Single Instance Check (New Finding):**
        * **Action:** Calls `FUN_14058f3c0` (likely `Mutex.Create`).
        * **Failure:** If the mutex exists, displays MessageBox: *"Another instance of Hytale is already running"*.
        * **Exit:** Calls `ExitProcess` if the check fails.
    * **Loop Dispatch:**
        * **Game Mode:** Calls `FUN_14064a950` (Client Init).
        * **Editor Mode:** Calls `FUN_14064aa00`.

## 5. Game Client Initialization (`FUN_14064a950`)
!!! warning "Speculative: Class Identity"
    I have labeled `Class_141a4a200` as the `HytaleClient` based on usage context. No string metadata at this time confirms this specific class name.

!!! warning "Speculative: Architecture"
    The object at `DAT_1427c23e0` is being treated as a "Host" or "Engine Platform" driver. This is inferred because the client crashes with `ArgumentNullException` if it is missing.

* **Function:** `FUN_14064a950`
    * **Instantiation:** Calls `RhpNewFast` with MethodTable `Class_141a4a200` (The `HytaleClient` object).
    * **Constructor:** Calls `FUN_140633630` (`.ctor`).
    * **Host Lookup:** Retrieves the Engine Host (Platform Driver) from global context `DAT_1427c23e0`.
    * **Run:** Calls `FUN_140649d80` to begin the loop.

## 6. Loop Wiring & Event Subscription
!!! warning "Speculative: Event Type"
    I assume this event is the "Tick" or "Render" event because it drives the loop. However, it could technically be an "Idle" event or a custom "Frame" callback since I have yet to analyze it.

!!! warning "Speculative: Message Pump"
    This function is assumed to be the Message Pump (e.g., `Application.Run`) because it blocks execution until the game exits.

The game does not run a simple `while(true)` loop immediately. It wires itself into the windowing system.

* **Function:** `FUN_140649d80` (Delegate Factory)
    * **Closure:** Creates a closure object (`Class_141a622d8`) holding the Game and Host instances.
    * **Delegate:** Creates a delegate (`Class_141ab8490`) pointing to the loop logic.
* **Function:** `FUN_140649e40` (Event Subscription)
    * **Window Init:** Calls `FUN_140634680` to initialize the OS Window.
    * **Subscribe:** Calls `FUN_1401e6f20` to attach the Game Loop delegate to the Window's "Tick" or "Render" event.
    * **Message Pump:** Calls `FUN_140add280` (Blocking) to start processing OS messages.
