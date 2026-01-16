# Introduction

Welcome to the **Hytale Internals** research project. 

Since the Hytale client uses .NET **NativeAOT** compilation, all code is collapsed into machine code.
This means that most of the program becomes very difficult to read, with no metadata such as function or variable names to guide our understanding. 
This project is dedicated to piecing all of the assembly code together into something that can be understood by developers.

???+ note  
    This analysis is currently being performed on HytaleClient.exe release version 1.0.0.0

Documentation will regularly include admonitions. "Speculative" tags are those which have been read through and fit a logical label, but are not yet confirmed. 
Anything tagged as speculative will have a high probability of being changed as more information is gathered.
