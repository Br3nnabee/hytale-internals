# Startup Sequence & Initialization

## 1. OS Handover & Security (OEP)

The process begins at the true Original Entry Point (OEP), handling OS-level security before static initialization.

* **Function:** `entry` (The OEP)
    * **Action:** Calls `__security_init_cookie` (`FUN_1414ad9c0`).
        * **Logic:** XORs System Time, Thread ID, Process ID, and Performance Counter to generate the `/GS` canary.
    * **Action:** Calls `__tmainCRTStartup` (`FUN_1414acef4`).
        * **Logic:** Initializes the NativeAOT Runtime and the C Runtime (CRT).
        * **Signatures:** Contains calls to `_initterm` and `__scrt_acquire_startup_lock` which prepare the environment for managed code execution.

## 2. Application Entry (`WinMain`)

!!! warning "Speculative"
Control is passed to the user's main application logic.

* **Function:** `WinMain` (`FUN_1411f6b00`)
    * **Synchronization:** Calls `SpinLock_Acquire` (`FUN_1413ef880`) to lock the main thread using a `Sleep(1)` loop.
    * **Context Allocation:** Calls `ThreadArena_Alloc` (`FUN_1413f0590`) with `DAT_141abaf30` to create the root application context.
    * **MethodTable Initialization:**
        * The code `*(undefined ***)(lVar2 + 8) = &PTR_DAT_14267aee8` is initializing the **MethodTable** (Type Handle) for the application object.
    * **Subsystems:**
        * Calls `Init_Global_Managers` (`FUN_140b1b850`) for memory/logging.
        * Calls `Register_Core_Singletons` (`FUN_140b75a30`).
    * **Arguments:** Calls `Parse_Command_Line_Args` (`FUN_140b49700`) to process `ArgC`/`ArgV`.
    * **Sanity Check:** Calls `Validate_Thread_State` (`FUN_140ad46c0`) to ensure the app instance is in state `1` (Running).
    * **Handover:** Calls `OS_EntryPoint` (`FUN_1411f6ae0`) -> `Engine_Main_Loop_Orchestrator` (`FUN_14064a590`).

## 3. Platform & Mode Selection (`Engine_Main_Loop_Orchestrator`)

!!! warning "Speculative"
The Operating System Abstraction Layer.

* **Function:** `Engine_Main_Loop_Orchestrator` (`FUN_14064a590`)
    * **Platform Check:**
        * Decodes string `0x64006e00690077` -> `"dniw"` (wind).
        * Decodes string `0x730077006f0064` -> `"swod"` (dows).
    * **Mode Branching:** Checks global flag `DAT_1427af6d3`.
        * **If 0:** Calls `Run_Client_Session` (`FUN_14064a940`).
        * **If 1:** Calls `FUN_14064a9f0` (Unidentified; possibly for editors?).

## 4. Client Initialization (`Run_Client_Session`)

!!! warning "Speculative"
The specific path for the game client.

* **Function:** `Run_Client_Session` (`FUN_14064a940`)
    * **Allocation:** Allocates the Client Session Arena (`DAT_141a4a200`).
    * **Setup:** Calls `ClientSession_Initialize` (`FUN_140633630`).
        * **Identity:** References string `L"Hytale/Screenshots"`.
        * **Windowing:** Calculates "Initial window size" based on monitor info and `VideoSettings`.
        * **Injection:** Registers `RenderContext` (`0x3f800000` float flag), `AssetManager`, `InputManager`.
    * **Game Loop:** Handover to `Client_Loop_Standard` (`FUN_140649d40`).
