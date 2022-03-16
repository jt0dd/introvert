## Underlying Concept


#### **(Defense) Hooking:**

Hooking is a key form of Dynamic Analysis. Security engineers know that emulation is not a reliable medium for analysis. This is where hooking comes into play: Detect the malicious behavior at run-time. So, how does it work? Most programs designed for user-mode execution call out to standard libraries and APIs which are dynamically linked into the program's memory space from shared memory at run-time by the linker. Security products commonly patch hooks into this shared memory space. Hooks work by replacing the entrypoint to an API subroutine with a jump out to an intermediary analytic subroutine before jumping back into the intended API subroutine. Hooking can facilitate analysis of OS API and shared library call sequences and system call sequences.

## State of the Art (Unhooking)

The approaches to perform unhooking are fairly straight-forward: Disable the hook, or side-step it. [Cylance's approach to unhooking](https://blogs.blackberry.com/en/2017/02/universal-unhooking-blinding-security-software), leveraging a disabling approach, was cutting edge in the problem-space at the time of publication in 2017:

> "Basically, in the user-land space, it goes through all the modules loaded into a process, and then for each module it opens the file, processes the data, [gets a] clean view of what the DLL should look like. And then for each section in the DLL that isn't writeable, we compare that clean version to the current version and if they don't match replace the current version with the clean." - Cylance CEO  Stuart McClure, RA Conference 2017
But there's a weakness in this approach. It requires that the attacker trust the DLL on disk. By applying hooks to the DLLs on disk, a defender would theoretically win. While it is true that DLLs in the system folder are protected from modification, it is possible through drivers to redirect any filesystem loads of the protected system DLLs to the ones modified by the security product." - [Stuart McClure, CEO, Cylance, RSA Conference 2017](https://www.youtube.com/watch?v=QZ6mEq6DQOo)

This approach likely still represents state of the art, however this writing attempts to outline a potential weakness in the method and explore an alternative.

## Disabling Approach

A Disabling approach is any unhooking methodology which involves modifying hooked code in order to bypass or remove the hook.

### Downside

By overwriting the hooked DLLs at run-time with unhooked ones, the Disabling approach takes an invasive measure. While provenly effective against a wide array of security products, a defender would only need inspect the DLLs at run-time to notice the hook removal and alert on the anomaly. In other words, the down-side is that this approach will only work until defenders patch their tools to address it. Additionally, this approach requires the attacker to trust the defender's DLLs on disk. While it is true that Windows protects files in the system folder from modification, a driver can be leveraged to [observe](https://docs.microsoft.com/en-us/windows-hardware/drivers/ifs/filter-manager-concepts) and redirect any file reads of the unhooked DLLs to retrieve hooked copies in an alternate location instead.

### Upside

This approach allows the attacker's payload to be portable. That's a big upside.

## Side-stepping Approach

A Side-Stepping approach is any unhooking methodology which involves using a trusted, clean copy of the DLL directly without modifying the defender's hooks. it also does not involve intrusive modification of an asset that an endpoint security product could easily notice.

### Downside

This approach is not portable and results in a larger payload size. Because each version of Windows has the potential to change its kernel bindings, a DLL linked on one Windows version may not work on another version. This approach requires the attacker to send a payload corresponding to the specific target host's Windows version. That adds an outbound network request which must avoid alerting the defender's sensors and analysts. Depending on how similar the DLLs are across different versions of Windows, it might be feasible to use [Delta Encoding](https://en.wikipedia.org/wiki/Delta_encoding) to send a single binary representative of every version of the DLL and decode the one that corresponds to the target OS. This would eliminate the extra network request down-side, but aggravate the size increase. The concept is heavily dependent on how much added payload size this would result in. Static linking already multiplies payload size as it is. For example a C++ program which only imports `iostream` compiled via `g++.exe -static` increases the file size from 44KB to 2MB.

### Upside

This approach does not require the attacker to trust the defender's DLL copies on disk.


## Problem

Doing this with source code is a simple matter of compiler flags. However, the requirement of access to source code can be overly limiting. Many offensive tools are not open source. It is preferable to be able to emulate any adversary tool, regardless of access to source code.

## Objective

The outcome this library seeks to achieve is cutting-edge unhooking via plug-and-play static re-linking. To address the portability limitation created by static linking, Windows kernel and API DLLs should be stored into a database table corresponding to each release of the OS. A loader is embedded which, when executed on a target machine, calls back to the C2 server signaling the target OS version, which in turn sends the corresponding statically linked payload as a response. This approach, while sophisticated, removes any opportunity for the defender's endpoint sensors to defeat an unhooking attempt by hooking the DLLs on disk.
