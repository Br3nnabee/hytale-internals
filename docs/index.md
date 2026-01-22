# Introduction

Welcome to the **Hytale Internals** research project. 

Since the Hytale client uses .NET **NativeAOT** compilation, all code is collapsed into machine code.
This means that most of the program becomes very difficult to read, with no metadata such as function or variable names to guide our understanding. 
This project is dedicated to piecing all of the assembly code together into something that can be understood by developers.

???+ note
    Much of the "noise" has been identified by compiling fake EXEs and using their pdbs to generate signatures. This may cause some inaccuracies/poor naming.

!!! warning
    This is largely an internal document for myself until I get a grasp of the project, however there are others helping me at the [discord server](https://discord.gg/FNYG3uuZgn). Function IDs etc are not pinned and may vary, however a ghidra XML export exists for those who wish to help. It does not contain the binary, so ensure you have the initial HytaleClient.exe.

???+ note  
    This analysis is currently being performed on HytaleClient.exe release version 1.0.0.0

Documentation will regularly include admonitions. "Speculative" tags are those which have been read through and fit a logical label, but are not yet confirmed. 
Anything tagged as speculative will have a high probability of being changed as more information is gathered.

??? note "Ready To Run Header"
    The RTR header is not automatically found by NativeAOT. It is located at 1421a8cc0.

