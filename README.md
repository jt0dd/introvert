## Introvert

Introvert seeks to isolate imported subroutine calls in a way that deters endpoint security drivers from patching in run-time analysis hooks. This transformation is designed to demonstrate a fundamental weakness in modern endpoint threat detection and encourage the exploration of new solutions to address this critical cyber security challenge.

## Underlying Concept


#### **(Defense) Hooking:**

Hooking is a key form of Dynamic Analysis. Security engineers know that emulation is not a reliable medium for analysis. This is where hooking comes into play: Detect the malicious behavior at run-time. So, how does it work? Most programs designed for user-mode execution call out to standard libraries and APIs which are dynamically linked into the program's memory space from shared memory at run-time by the linker. Security products commonly patch hooks into this shared memory space. Hooks work by replacing the entrypoint to an API subroutine with a jump out to an intermediary analytic subroutine before jumping back into the intended API subroutine. Hooking can facilitate analysis of OS API and shared library call sequences and system call sequences.

#### **(Counter-Defense) Unhooking:** 
**Mitigates: \[** [_Hooking_](https://github.com/jt0dd/phantom-v/blob/main/README.md#defense-hooking) **\]**

In order to evade endpoint security hooks within shared libraries, it is necessary to either remove them (an invasive and unstable option) or side-step them by loading a trusted, unhooked clean version of the dependency and store it statically. The Unhooking transformation step mitigates not only Hooking but also Static Analysis. Merely including certain API resources as imports for the linker can tip off a security product to the malicious potential of the program. The process of Unhooking removes such traces from the observation scope of Static Analysis. Unhooking is complicated by one problem: Dynamically linked subroutines exposed by the Windows kernel are wrappers which have different implementations across different OS versions. These cannot be statically linked, unless the attacker wants the responsibility of generating a separate version of the malware for every individual target environment. Fortunately for the attacker, this problem can be side-stepped. Thanks to Windows' Patch Guard, security products cannot apply hooks to kernel components; at least not in modern 64-bit versions of the OS. Therefore, while all other external imports should be consolidated, Windows kernel imports can be left alone.
